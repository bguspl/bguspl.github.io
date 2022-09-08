Atomic Instructions
Most CPU operations (like add, mov etc.) are atomic - i.e., the instruction result can be seen as if it "happened at once". We only need to use synchronization if we want several such instructions to be atomic.
Most CPUs today offers a set of atomic instructions that designed for multi-threading - an important one is compare-and-set or CAS. CAS is an instruction that behaves as follows:

download
boolean cas(int[] target, int oldValue, int newValue) {
  if (target[0] == oldValue) {
    target[0] = newValue;
    return true;
  }
 
  return false;
}
The cas instruction is atomic which means that threads sees its result as if it happens at once - in other words - the scheduler cannot stop a thread in the middle of this operation - only before or after it.

Although it does not look like one, the cas instruction is extremely useful and powerful. Java has several classes (all beginning with Atomic* that allow us to use cas). To understand its power, lets rewrite the Even counter class using AtomicInteger:

downloadtoggle
11 lines ...
class Even {
  private AtomicInteger counter = new AtomicInteger(0);
 
  public int add() {
    int val;
    do {
      val = counter.get();
    }while (!counter.compareAndSet(val, val + 2));
  }
 
  public int get() { return c.get(); }  
}
Lets revise the code above:

AtomicInteger is a class that holds integer that we can call cas on (i.e., it will be the target of the cas)
There is no usage of synchronized anywhere in the class.
if there are n threads t_1,...,t_n that attempt to invoke add one time all at once then, without the loss of generality: t_1 will enter the while loop once, t_2 at most twice, and t_n at most n times.
This code run significantly faster than the synchronized one as we do not send a thread to sleep (i.e., invokes a system call) whenever a concurrent access takes place.

This kind of implementation is called lock-free.
Lock-free implementation
An implementation is called lock-free if threads can not block each other while running that code.
Can we create a lock-free implementation for data structures that require operations that are more complex than adding two numbers? YES, in many cases the basic idea stays the same, copy the cas target to a local variable, change this variable locally, try to commit your change using cas, if failed, retry - this is it!. Lets see another example and implement our linked list lock-free:

downloadtoggle
13 lines ...
public class LinkedList<T> {
 
    private AtomicReference<Link<T>> head = new AtomicReference(null);
 
    public  void add(T data) {
      Link<T> oldVal;
      Link<T> newVal = new Link<>(null, data);
 
      do {
        oldVal = head.get();
        newVal.next = oldVal;        
      } while (!head.compareAndSet(oldVal, newVal));
    }
}
Again this implementation uses no locks - it is faster and more efficient.

An important note! while lock free data structures seems better than lock based structures they also has their limitations:

Since there are no locks, lock-free data structures cannot block threads which may be a required property of the data structure - like for example blocking queues.
Lock-Free data structures are notoriously known of being much harder to write correctly.
There are some cases where the state of the data-structure may be complex and may require a lock-free implementation that need to copy (to local variables) large amount of data - which may not always be possible or may lead to poor performance.

In other words, in programming - there is no silver bullet - you need to master many different techniques and choose the one that is most suitable to your task. In addition, like most other techniques we learn about in this lecture java has some pre-made implementation for us to use - some useful examples of such are ConcurrentHashMap and ConcurrentLinkedQueue which you can read more about via their APIs javadoc.
Thread Safe Singleton
Recall the implementation of the Singleton we saw in the previous practical session:
downloadtoggle
14 lines ...
public class ClassicSingleton {
 
       private static ClassicSingleton instance = null;
 
       private ClassicSingleton() {
          // initialization code..
       }
 
       public static ClassicSingleton getInstance() {
          if(instance == null) {
             instance = new ClassicSingleton();
          }
          return instance;
       }
}
We mentioned that this implementation is not thread safe because two instances of the Singleton can be created if a thread is preempted after the "if" statement in the getInstance() method. The following will work:

downloadtoggle
11 lines ...
public class ClassicSingleton {
 
    private static ClassicSingleton instance = new ClassicSingleton();
 
    private ClassicSingleton() {
        // initialization code..
    }
 
    public static ClassicSingleton getInstance() {
        return instance;
    }
}
Because only one thread is responsible for loading classes in java, this implementation will be thread safe and promise only one instance.

Problem: whenever the ClassicSingleton class will be imported, the instance will be created. This is called eager initialization - creating the instance before we actually need it. This is a problem because most classes that are designed as a Singleton will be large and their initialization won't be trivial. We want lazy initialization - creating the instance only when it is needed.

We need another solution to make the Singleton thread safe. The natural solution will be to synchronize the getInstance() method:
download
public static synchronized ClassicSingleton getInstance() {
    if(instance == null) {
        instance = new ClassicSingleton();
    }
    return instance;
}

Problem: this creates a bottleneck if a lot of threads call the method. Using synchronized means blocking every thread that wants to get the singleton if another thread is currently getting it. We want a solution that allows every thread to call this method without blocking it.

How about the following solution?
downloadtoggle
9 lines ...
public static ClassicSingleton getInstance() {
    if(instance == null) {
        synchronized (ClassicSingleton.class) {
            if (instance == null) {
                instance = new ClassicSingleton();
            }
        }
    }
    return instance;
}

At first sight, it looks good. We use double checking because when a thread waits for the lock, another thread that already has the lock can create the instance.

Problem: new is not an atomic action. It breaks down into two actions: 1. Allocating memory for the object. 2. Calling its constructor. Let's say that a thread was preempted after the first action. This means that the memory needed for the object was already allocated (the object isn't null) but the constructor wasn't called yet. If another thread calls getInstance(), the condition won't hold and a reference to an uninitialized object will be returned.

How about this?
downloadtoggle
10 lines ...
public static ClassicSingleton getInstance() {
    if(instance == null) {
        synchronized (ClassicSingleton.class) {
            if (instance == null) {
                ClassicSingleton x = new ClassicSingleton();
                instance = x;
            }
        }
    }
    return instance;
}

Problem: the java compiler uses optimizations and one of them is to get rid of unnecessary variables. So this code, when compiled, will be the same code in the previous solution.

The easiest (working) solution:
downloadtoggle
10 lines ...
public class ClassicSingleton {
    private static class SingletonHolder {
        private static ClassicSingleton instance = new ClassicSingleton();
    }
    private ClassicSingleton() {
        // initialization code..
    }
    public static ClassicSingleton getInstance() {
        return SingletonHolder.instance;
    }
}

The SingletonHolder class won't be loaded until we reference it. This means that when we import the ClassicSingleton class, the instance won't be created until the moment the getInstance() method is called (the SingletonHolder is referenced). Because only a single thread loads classes, only one instance will be created and we still achieve lazy initialization.
Callables
download
public interface Callable<V> {
    V call() throws Exception;
}
In case you expect your threads to return a computed result you can use java.util.concurrent.Callable. Callables allow to return values after completion. Callable uses generics to define the type of object which is returned. Since you wish to receive the returned result, you need to keep a connection to the Callable thread. In order to do so, you may use one of these tools: Futures, and ExecutorCompletionService.

Futures
If you submit a callable to an executor the framework returns a java.util.concurrent.Future. This Future objects can be used to check the status of a callable and to retrieve the result from the callable. On the executor you can use the method submit to submit a Callable and to get a Future object. To retrieve the result of the Future object - use the get() method.
downloadtoggle
46 lines ...
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class MyCallable implements Callable<String> {
 
    @Override
    public String call() throws Exception {
        Thread.sleep(1000);
        return Thread.currentThread().getName();
    }
    
    public static void main(String args[]){
        
        //Get ExecutorService from Executors utility class, thread pool size is 10
        ExecutorService executor = Executors.newFixedThreadPool(10);
        
        //create a list to hold the Future object associated with Callable
        List<Future<String>> list = new ArrayList<Future<String>>();
        
        //Create MyCallable instance
        Callable<String> callable = new MyCallable();
        
        for(int i=0; i< 10; i++){
            //submit Callable tasks to be executed by thread pool
            Future<String> future = executor.submit(callable);
            //add Future to the list, we can get return value using Future
            list.add(future);
        }
 
        for(Future<String> fut : list){
            try {
                //print the return value of Future, notice the output delay in console
                // because Future.get() waits for task to get completed
                System.out.println(fut.get());
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
            }
        }
        //shut down the executor service now
        executor.shutdown();
    }
}
Note that we waste a lot of time waiting for one Future to finish while there are other Futures already finished. That the wait is performed by the blocking function future.get()

ExecutorCompletionService
A CompletionService that uses a supplied Executor to execute tasks. This class arranges that submitted tasks are, upon completion, placed on a queue accessible using take. The class is lightweight enough to be suitable for transient use when processing groups of tasks. Upon Callable completion, the returned results are added to the ExecutorCompletionService's queue, where you can fetch the results in a convenient manner.
downloadtoggle
46 lines ...
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.CompletionService;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorCompletionService;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class CallableWithCompletionService implements Callable<String> {
 
    @Override
    public String call() throws Exception {
        Thread.sleep(1000);
        return Thread.currentThread().getName();
    }
    
    public static void main(String args[]) throws InterruptedException, ExecutionException{
        
        //Get ExecutorService from Executors utility class, thread pool size is 10
        ExecutorService executor = Executors.newFixedThreadPool(10);
        CompletionService<String> ecs = new ExecutorCompletionService<String>(executor);
        
        //create a list to hold the Future object associated with Callable
        List<Future<String>> list = new ArrayList<Future<String>>();
        
        //Create MyCallable instance
        Callable<String> callable = new CallableWithCompletionService();
        
        for(int i=0; i< 10; i++){
            //submit Callable tasks to be executed by thread pool
            Future<String> future = ecs.submit(callable);
            //add Future to the list, we can get return value using Future
            list.add(future);
        }
        
        for(int i = 0; i < numberOfFutures; i++){
            // ExecutorCompletionService's take() method
            // returns the first finished Future available
            String finished = ecs.take().get();    
            System.out.println(finished);
        }
        //shut down the executor service now
        executor.shutdown();
    }
}
Thread Cancellation
Each thread has a predefined task it should be working on. When the task ends, the thread finishes working and exits. However, sometimes we need to tell a thread to stop working (for example, in assignment 3, threads should stop when the user enters a certain command in the observer).
How do we tell a thread to stop working?

One way is calling the thread's stop() command. This is always unsafe, and you should never do that.

Another way is telling the thread it should stop, by changing a variable the thread sees. For example:

downloadtoggle
20 lines ...
class Worker implements Runnable {
 
        private boolean shouldStop_ = false;
 
        public Worker() { }
 
        public synchronized void stop() { 
            this.shouldStop_ = true; 
        }
 
        public synchronized boolean shouldStop() {
            return this.shouldStop_;
        }
 
        public void run() {
                while (!this.shouldStop()){
                        doSomeWork();
                }
                System.out.println("stopping ;)");
        }
}
Here, another thread can call the stop() method of a task, and inform the thread that currently runs the task that it should stop. The thread does not stop immediately, and there is no gurantee it will stop at all. In our case, it will stop while it gets to the while(!this.shouldStop()) line and see that shouldStop() returns true.

This approach may fail. Consider the case in which the method doSomeWork() has a wait() inside it. It is possible for the thread to never come out of the notify, and thus never getting to the this.shouldStop() check, and thus never stop.

The interrupt() mechanism
The above problem is handled with the interrupt mechanism.
Each thread has a flag specifying if it is "interrupted" or not. Being in an interrupted state means that another thread signals that it should stop (very similar to the shouldStop() method we implemented above). However, if a thread is inside wait() or sleep() when it is being interrupted, an InterruptedException will be thrown.

In summary:

t.isInterrupted() is used in code to check if t is in interrupted state.
t.interrupt() will:
if t is blocked (e.g. in wait() or sleep()), then InterruptedException is thrown.
if t is not blocked, isInterrupted() will return true.

Note that if InterruptedException is thrown, then isInterrupted() will return false unless you call interrupt() again.
Code example:

downloadtoggle
51 lines ...
class interruptedWorker implements Runnable {
 
    public interruptedWorker() {}
 
    public void doSomeWork() {
        WorkerHelper workerHelper = new WorkerHelper();
        while (!Thread.currentThread().isInterrupted()) {
            try {
                workerHelper.doSomeWork();
            } catch (InterruptedException e) 
            {
                // raise the interrupt. This is very important!
                Thread.currentThread().interrupt();             
            }
        }
    }
 
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            doSomeWork();
        }
        System.out.println("stopping ;)");
    }
}
 
class WorkerHelper {
    public synchronized void doSomeWork() throws InterruptedException {
        try {
            // doing some work...
            Thread.sleep(10);
            // waiting for some condition...
            this.wait();
        } catch (InterruptedException e) {
            // re-throwing the exception to let the thread itself handle the exception.
            throw e;
        }
    }
}
 
class Main {
       public static void main(String[] args) {
     
          Thread t = new Thread(new interruptedWorker());
          t.start();
     
          try {
             Thread.sleep(100);
          } catch (InterruptedException e) { }
     
          t.interrupt();
       }
    }
isInterrupted() is implemented as the following:

download
public boolean isInterrupted() {
   return isInterrupted(false);
}
isInterrupted(false) call means that we dont want to change the current interrupt's value (the false parameter says that).

If we wish to change, you can call interrupted():

download
public static boolean interrupted() {
   return currentThread().isInterrupted(true);
}
Interrupt() when using an Executor
If using an Executor e to run your threads, calling e.shutdownNow() will call interrupt() on all of the threads in the Executor.
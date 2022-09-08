Practical Session #06 - Threads and Concurrency / Safety
General
All the computer programs you've seen so far were sequential – only one thing was performed at any given time. Sometimes it is desirable to do several things simultaneously.
Modern operating systems (Windows / Linux / OSX) allow the execution of several programs (several processes) at the same time. If the tasks to be performed are not related in any way, using a different process for each task is a good solution. If the tasks are somewhat related, and need to share data between them, it is beneficial to put them inside the same process, so they can share resources (e.g., memory, code and files). This is achieved using Threads.

Threads enable us to perform several tasks at a given time. Several Threads can run simultaneously on a single process, sharing its resources. Special care should be taken when using those shared resources.

In this practical session we will learn how to use Threads in Java, and discuss some of the problems that arise from Threads usage. In future practical sessions, we'll see how to solve these problems.

Code Files
All the code examples presented in class are available here.
Please make sure to compile and run (and understand) them at home.
Threads
Multitasking vs. Multithreading
Shared Resources.
Context Switching.
How to use Threads
In order to run several tasks in parallel, two steps should be taken:
Define the tasks.
Create threads and assign each with a task.

Defining a task is done by implementing the Runnable interface. This is a very simple interface with a single method: void run(), which defines the work unit (task) to be performed.
downloadtoggle
13 lines ...
class SimpleRunnable implements Runnable {
 
    private int num;
         
    public SimpleRunnable(int i) {
        this.num = i;
    }
         
    public void run() {
        for (int i = 0; i < 5; i++) {
        System.out.print(num); 
    }
    }
}

We will now see two ways of assigning tasks to a thread.
The first way to run threads is by using Java's Thread class. Thread is a class which its constructor takes a single Runnable object. Upon calling the start() method, the thread will invoke the supplied Runnable run() method (without blocking the calling thread). Calling start() - first sets up the thread internals (allocates resources), then when it is ready, calls the Runnable run() method.

Implementing Runnable, and creating a Thread that will run it. (See Threads01.java and its output Threads01.txt)
Here is an example of using the Thread mechanism:

downloadtoggle
10 lines ...
public class Threads01e {
 
    public static void main(String[] args) {
 
        for (int i = 0; i < 2; i++) {
            Runnable r = new SimpleRunnable(i);
            Thread t = new Thread(r);
            t.start();    
        }
    }
}
Another way of running threads is by using Java's Executor framework. An Executor is a service that internally manages one or more threads and allows you to assign tasks for them. There are many types of Executors (each of them with a different policy regarding how to manage the internal threads – how many tasks should be performed at the same time, etc), but all of them implement the ExecutorService interface.

In the following example, we create several Runnable tasks, and them pass them to the ExecutorService for execution.

Implementing Runnable and passing it to an Executor. (See Threads01e.java and its output Threads01e.txt).
downloadtoggle
13 lines ...
public class Threads01e {
 
    public static void main(String[] args) {
 
        ExecutorService e = Executors.newFixedThreadPool(3);
 
        for (int i = 0; i < 5; i++) {
            Runnable r = new SimpleRunnable(i);
            e.execute(r);
        }
        e.shutdown(); // the executor won't accept any more tasks, and will 
                      // kill all of its threads when the submitted tasks are done.
    }
}

Note the decoupling of "task submission" (creating a Runnable) from task execution (creating a thread for the task and calling "start()")
Concurrency / Safety
Shared Resources
Safety problems arise when multiple threads access the same resource. But what exactly is a shared resource?
A shared resource is any object that is visible to several threads. This includes global (static) objects, non-primitive method parameters and class members, but not local function variables.

Consider the following example illustrating a problem with sharing a resource that is mutable (value can be changed) between different threads in the system.
The example is available here: unprotectedCounter
downloadtoggle
51 lines ...
public class Counter{
    private int count;
 
    public Counter(){
        count = 0;
    }
    
    public void increment(){
        count++;
    }
    
    public int getValue(){
        return count;
    }
}
 
public class Incrementor implements Runnable {
 
    private Counter counter;
 
    public Incrementor(Counter ctr) {
        counter = ctr;
    }
 
    public void run() {
        for (int i = 0; i<500; i++) {
                counter.increment();
        }
    }
}
 
public class Threads03 {
    
    public static void main(String[] a) throws InterruptedException {
 
        for (int i=0;i<10;i++){
            
            Counter ctr = new Counter();
            
            Thread t1 = new Thread(new Incrementor(ctr));
            Thread t2 = new Thread(new Incrementor(ctr));
            
            t1.start();
            t2.start();
            
            t1.join(); 
            t2.join();   
            
            System.out.println("Attempt " + i + ", total value: " + ctr.getValue());
        }
    }
}
The following is a possible output for this code:

Attempt 0, total value: 1000
Attempt 1, total value: 1000
Attempt 2, total value: 1000
Attempt 3, total value: 972
Attempt 4, total value: 972
Attempt 5, total value: 979
Attempt 6, total value: 907
Attempt 7, total value: 685
Attempt 8, total value: 1000
Attempt 9, total value: 1000
Why does this happen? Let's consider the following lines:

download
public void increment(){
    count++;
}
The second line in this code involves three basic operations:
Read the value of count
Increment that value by 1
Assign the new value to count

Let's say a thread is preempted after he performed operations #1 (read value X) and #2 (will save X+1 but not assign it to count yet). If another thread tries to perform those actions, the value of count he reads at #1 is not yet updated (value X). This means that both of these threads will assign the same value (X+1) to count. Because of this, a lot of increment operations can be lost.
This example demonstrates the problematic nature of sharing resources between threads. In order to take care of this problem, we need to handle synchronization in a way that doesn't allow the threads to access the problematic parts of our code (the increment part) at the same time.
Threads can be Dangerous
As we have just seen, having things run "at the same time, in the same place", can be dangerous. The two tasks can interfere with each other, and unexpected results might occur.
Below, we present another problem that may occur in multi-threaded environment - the invariant of an object may break!

Invariant violation
In this example we use the object Even, which holds an even numeric counter. We'll define the pre and post conditions and see if they are being kept.
The conditions are kept when using a single thread; yet in a multi-threaded environment this class is not safe: See Threads03.java and its output Threads03.txt.

downloadtoggle
52 lines ...
public class Threads03 { 
 
    public static void main(String[] args) {
 
        ExecutorService e = Executors.newFixedThreadPool(10);
        Even ev = new Even();
        for (int i = 0; i < 5; i++) {
            e.execute(new EvenTask(ev));
        }
        e.shutdown();
    }
}
 
class EvenTask implements Runnable {
    
    Even m_even;
 
    EvenTask(Even even) {
        m_even = even;
    }
 
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(m_even.next());
        }
    }
}
 
/**
 * (unsafe) Even object, producing a greater even number on each call to next()
 */
class Even {
    
    private int n = 0;
 
    /**
     * return an even number.
     * @return an even number not returned before by this object.
     */
    //@ pre-condition: n is even
    public int next() {
        
        n++;
        // sleep(30) will cause synchronization problem to occur more frequently in this example.
        // in a real application (without the sleep) the problem will be rare.
        // thats precisely what makes it dangerous!
        try {Thread.sleep(30);} catch (InterruptedException e) {}
        n++;
        
        return n;
    }
    //@ post-condition: n is greater by two
}
The following is a possible output for this code:

..
9
17
20
19
18
16
26
30
29
27
..
A simple solution: check if the pre / post conditions are broken, and throw an exception if so: Threads04.java and its output Threads04.txt.

downloadtoggle
45 lines ...
public class EvenTask implements Runnable {
 
    private Even m_even;
    
    public EvenTask(Even even) {
        m_even = even;
    }
 
    public void run() {
        for (int i = 0; i < 5; i++) {
            try {
                System.out.println(m_even.next());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
 
class Even {
    
    private long n = 0;
 
    /**
     * return an even number.
     * @return an even number not returned before by this object.
     * @throws NotEvenException in case a pre- or post- condition is broken.
     */
    //@ pre-condition: n is even
    public long next() throws Exception {
        
        if (n%2 != 0) {
            throw new Exception("PRE: n is not even!");
        }
        n++;
        
        try {Thread.sleep(30);} catch (InterruptedException e) {}
        
        n++;
        if (n%2 != 0) {
            throw new Exception("POST: n is not even!");
        }
        return n;
    }
    //@ post-condition : n is greater in two
}
The following is a possible output for this code:

/**
 * Exception in Thread-2
 * NotEvenException: PRE: n is not even!
 *         at Even.next(Threads08.java:43)
 *         at EvenThread.run(Threads08.java:28)
 * Exception in Thread-6
 * NotEvenException: PRE: n is not even!
 *         at Even.next(Threads08.java:43)
 *         at EvenThread.run(Threads08.java:28)
 * Exception in Thread-9
 * NotEvenException: PRE: n is not even!
 *         at Even.next(Threads08.java:43)
 *         at EvenThread.run(Threads08.java:28)
 * Exception in Thread-10
 * NotEvenException: PRE: n is not even!
 *         at Even.next(Threads08.java:43)
 *         at EvenThread.run(Threads08.java:28)
 * 2
 * 4
 * 6
 * 8
 * 10
 */
It doesn't really solve the problem – we may still get a bad output and no exception.
Notice that the last number of the output can be less then 50, this means that the command n++ in the next() method hadn't been executed 25 times as you would expect.

This can happen for several reasons:
In first example it happens since n++ is not an atomic command and is composed of 3 atomic actions.
In the second example it might happen due to the exception catching in the main loop of the run function (which terminates the incrementing of the Even object), but also due to the fact that n++ is not an atomic command.

To understand more about what commands are composed of, you can print the java byte code generated after its compilation using the console command javap -c [name of the tested class file]
Further details on javap can be found here.

Few Design Approaches towards Avoiding Safety Problems
1. Thread Confinement
If a variable/resource is accessed only by one thread, no synchronization is required. You may design your program such that certain variables are confined to one thread.
In the following example, everything in the createCar() method is confined, and so the method is safe to use in a multi-threaded situation. However, the createCar(Engine e) method gets an Engine as a parameter, and as a result might not be thread safe – the e is not thread confined – you don't know anything about the engine, and who else can access it beside your thread.

downloadtoggle
31 lines ...
public class ThreadConfinedExample 
 {
      //ThreadConfined
      public Car createCar() {
          Engine e = new FuelEngine();
          List<Door> doors = new LinkedList<Door>();
          doors.add(new FrontDoor()); 
          doors.add(new FrontDoor());
          doors.add(new BackDoor());
          doors.add(new BackDoor());
          Radio r = new AMFMRadio();
 
          Car c = new Car(e, doors, r);
 
          return c;
      }
 
      //NotThreadConfined
      // e is not confined to this method - it is visible from the outside
      public Car createCar(Engine e) {
          List<Door> doors = new LinkedList<Door>();
          doors.add(new FrontDoor()); 
          doors.add(new FrontDoor());
          doors.add(new BackDoor());
          doors.add(new BackDoor());
          Radio r = new AMFMRadio();
 
          Car c = new Car(e, doors, r);
 
          return c;
      }
 }
2. Immutability
An immutable object's state cannot be changed after construction. Thus, it can be safely used as a read-only object by multiple threads. As an example, the String and Integer classes in Java are immutable.

For an object to be immutable, it is recommended that all of its fields will be final, the types of the object field are recommended to be either primitive types or references to immutable objects. The object is immutable only after it has been safely published (reference to this hasn't escaped during construction and no thread have accessed the object before the construction completed).

Definition: The final keyword (similar to const in C/C++) - final fields can't be modified, but objects they refer to (internal objects) can be modified if they are mutable. It is good practice to make all fields final unless they need to be mutable.

It is not always possible to make all your fields immutable (for example, you want to hold a list of things, but your collection class isn't immutable). In such cases, you can still get the immutability effect if you follow the following rules:

A class will be immutable if all of the following are true:

All of its fields are final
The class is declared final
The this reference is not allowed to escape during construction (the object instance is not accessible until its construction is complete)
Any fields that contain references to mutable objects, such as arrays, collections, or mutable classes like Date:
Are private
Are never returned or otherwise exposed to callers
Are the only reference to the objects that they reference
Do not change the state of the referenced objects after construction

Example:
downloadtoggle
20 lines ...
/**
  * The constructed class is immutable even-though it consists of mutable objects.
  * The class is immutable due to its design, not just by using the final keyword.
  * class ThreeStooges is declared to be final, which means that it cannot have subclasses.
  */
 public final class ThreeStooges {
     private final Set<String> stooges = new HashSet<String>();
    /*it won't change after construction, meaning 
      it won't be assigned with another reference, 
      but still the object stooges is mutable: its internal values may change */
  
     public ThreeStooges() {
         stooges.add("Moe");
         stooges.add("Larry");
         stooges.add("Curly");
     }
  
     public boolean isStooge(String name) {
         return stooges.contains(name);
     }
 }
Java tip: unmodifiable
java.util.Collections.unmodifiableMap(Map)
Returns an unmodifiable view of the specified map. This is important to understand that the returned object is only a view of the original map - changing the original map will change the view and therefore the resulting object can only be considered immutable if no other class has reference to the original map.

Callback Functions
A callback function is a function that is passed to another function as a parameter, and can be activated there. For example, a function F2 can receive a function F1 as a parameter. F2 can activate F1 from inside its scope.
In Java, we can simulate this behavior by using interfaces.

Consider the following example: (ignore the comments for now)

downloadtoggle
66 lines ...
public interface TimePassedEvent
{
    public void callBack();
}
 
public class TimeNotifier implements Runnable
{
    private List<TimePassedEvent> subscribers;
    
    public TimeNotifier ()
    {
        subscribers = new LinkedList<TimePassedEvent>();
    } 
    
    public void subscribeToNotifier(TimePassedEvent ie) {
        subscribers.add(ie);
    }
 
    @Override
    public void run() {
            try {
               Thread.sleep(3000);
            } catch (InterruptedException e1) {}
 
        for (TimePassedEvent e : subscribers) {
                e.callBack();
        }
    } 
}
 
public class Worker {
 
    private int id;
    
    public Worker(int id) {
        this.id = id;
    }
    
    public void subscribeToNotifier(TimeNotifier notifier) {
 
    long threadID = Thread.currentThread().getId();
        
        notifier.subscribeToNotifier(() -> 
            {
                System.out.println("{Worker "+ id +" was notified that three seconds passed. " +
                        "This callback function was defined in Thread: "+threadID +
                        " and activated on Thread: "+Thread.currentThread().getId() +"}");
            });
    }
 
}
 
public class Simulator {
    
    public static void main(String[] args) {
        
        TimeNotifier notifier = new TimeNotifier();
        
        for (int i=0;i<3;i++) {
            Worker w = new Worker(i);
            w.subscribeToNotifier(notifier);
        }
        
        Thread t = new Thread(notifier);
        t.start();
    }
}
The output will be:

{Worker 0 was notified that three seconds passed.
This callback function was defined in Thread: 1
and activated on Thread: 15}
{Worker 1 was notified that three seconds passed.
This callback function was defined in Thread: 1
and activated on Thread: 15}
{Worker 2 was notified that three seconds passed.
This callback function was defined in Thread: 1
and activated on Thread: 15}
Anonymous classes (and lambdas) can capture variables with some restrictions:

An anonymous class has access to the members of its enclosing class.
An anonymous class cannot access local variables in its enclosing scope that are not declared as final, or effectively final.
A declaration of a variable in an anonymous class shadows any other declarations in the enclosing scope that have the same name. In lambdas, this is only true for the enclosing class.

An effectively final variable is a variable whose value is never changed after it is initialized. The reason why anonymous classes and lambdas can only access final and effectively final variables in the enclosing scope is because when you create an instance of an anonymous class, any variables which are used within that class have their values copied in via the autogenerated constructor. This way the compiler doesn't have to create extra types to manage these local variables.
The Singleton
In some cases, you'll want to have exactly one instance of a class in your program. For example: window managers, user-made loggers, and filesystems. Typically, those types of objects (which are known as singletons) are accessed throughout a software system, and therefore require a global point of access.
The Singleton design pattern addresses these concerns. With the Singleton design pattern you can ensure that only one instance of a class is created and that a global point of access to the object is provided.

The classic Singleton (naive implementation):

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
In a single threaded application, this Singleton implementation works. Let's assume that we are running a multi-threaded application. Let's have a look at the following lines:
download
if(instance == null) {
     instance = new ClassicSingleton();
}
If a thread is preempted at Line 2 before the assignment is made, the instance member variable will still be null, and another thread can subsequently enter the if block. In that case, two distinct singleton instances will be created.
In the next practical session, we'll explain how to solve this problem.

Summary:
The most useful policies for using and sharing objects in a concurrent program are:
Thread-confined A thread-confined object is owned exclusively by and confined to one thread, and can be modified only by the thread that owns it.
Shared read-only A shared read-only object can be accessed concurrently by multiple threads without additional synchronization, but cannot be modified by any thread. Shared read-only objects are immutable.
Shared thread-safe. A thread-safe object performs synchronization internally, so multiple threads can freely access it through its public interface without further synchronization.

References:
B. Goetz “ Java Concurrency in Practice”, Section 3 and 6, 7, 8.
Book Excerpt On-line: Executing tasks in threads
new : Publish/Escape
new : Immutability
Anonymous Classes
Singleton
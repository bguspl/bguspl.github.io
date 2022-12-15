Few Design Approaches towards Avoiding Safety Problems
Locking / Synchronization
Sometimes we want to allow several threads to access a resource which is not read-only.
Locking is a means of achieving consistent data, i.e., all threads see the most up-to-date values of shared variables if they synchronize on a common lock. Locking is achieved using the synchronized mechanism in Java.

Synchronization
Java's synchronization mechanism can be compared to a "room with one key only"; entering a synchronized section stands for "taking the key", and leaving the section "returns the key". The "key" is associated with an object instance. If no implicit pointer is specified, "this" is assumed.

download
/* notice the synchronized keyword */ 
 public synchronized void increment(){
      count++;
 }
See class Counter from practical session #06

Synchronization syntax
You can synchronize a method, in which case the object on which it is invoked acts as the synchronization "key". The key is acquired when entering the method, and is released on leaving it.

Another option is to synchronize a specific code block. In this case, you have to explicitly specify the synchronization object.

downloadtoggle
18 lines ...
public class Counter{
    private int count;
    private Object someLock;
 
    public Counter(){
        count = 0;
        someLock = new Object();
    }
    
    public void increment(){
        synchronized(someLock) {
               count++;
        }
    }
    
    public int getValue(){
        return count;
    }
}
Notice that the following is functionally equivalent to the first example:

download
public void increment(){
      synchronized(this) {
          count++;
      }
 }
Java concurrency
Computer users take it for granted that their systems can do more than one thing at a time. They assume that they can continue to work in a word processor, while other applications download files, manage the print queue, and stream audio. Even a single application is often expected to do more than one thing at a time. For example, that streaming audio application must simultaneously read the digital audio off the network, decompress it, manage playback, and update its display. Even the word processor should always be ready to respond to keyboard and mouse events, no matter how busy it is reformatting text or updating the display. Software that can do such things is known as concurrent software.
The Java platform is designed from the ground up to support concurrent programming, with basic concurrency support in the Java programming language and the Java class libraries. Since version 5.0, the Java platform has also included high-level concurrency APIs.

A better fix for the Counter from the previous practical session is to use an increment method that is thread safe. This means that reading the value, incrementing it and assigning the new value will happen within a single clock tick (no other thread can interfere in between). This can be achieved using the Javas' Atomic library. In the following example, we will use AtomicInteger:

downloadtoggle
14 lines ...
public class Counter{
    private AtomicInteger count;
 
    public Counter(){
        count = new AtomicInteger();
    }
    
    public void increment(){
        count.incrementAndGet();
    }
    
    public int getValue(){
        return count.get();
    }
}
This solution doesn't require blocking other threads while a thread is in the problematic scope. The following is the implementation of incrementAndGet in Java 7 (it's different in Java 8):

download
public final int incrementAndGet() {
    for (;;) {
        int current = get();
        int next = current + 1;
        if (compareAndSet(current, next))
            return next;
    }
}
The compareAndSet(expected, update) action is atomic in Java (happens within a single clock tick). This action checks if the current value is equal to the expected value. If it is, it updates the value to the given update.
Concurrency - States
Liveness
A concurrent application's ability to execute in a timely manner is known as its liveness. This section describes the most common kind of liveness problems, deadlock, and goes on to briefly describe two other liveness problems, starvation and livelock.
Deadlock
Deadlock describes a situation where two or more threads are blocked forever, waiting for each other. Here's an example.
downloadtoggle
61 lines ...
package spl.examples.deadlock;
/**
 * This is a demonstration of how NOT to write multi-threaded programs.
 * It is a program that purposely causes deadlock between two threads that
 * are both trying to acquire locks for the same two resources.
 * To avoid this sort of deadlock when locking multiple resources, all threads
 * should always acquire their locks in the same order.
 **/
public class Deadlock {
  public static void main(String[] args) {
    // These are the two resource objects we'll try to get locks for
    final StringBuffer resource1 = new StringBuffer("resource1");
    final StringBuffer resource2 = new StringBuffer("resource2");
    // Here's the first thread.  It tries to lock resource1 then resource2
    Thread t1 = new Thread(new Runnable() {
      public void run() {
        // Lock resource 1
        synchronized(resource1) {
          System.out.println("Thread 1: locked " + resource1);
 
          // Pause for a bit, simulating some file I/O or something.  
          // Basically, we just want to give the other thread a chance to
          // run.  Threads and deadlock are asynchronous things, but we're
          // trying to force deadlock to happen here...
          try { Thread.sleep(50); } catch (InterruptedException e) {}
          
          // Now wait 'till we can get a lock on resource 2
          synchronized(resource2) {
            System.out.println("Thread 1: locked " + resource2);
          }
        }
      }
    });
    
    // Here's the second thread.  It tries to lock resource2 then resource1
    Thread t2 = new Thread(new Runnable() {
      public void run() {
        // This thread locks resource 2 right away
        synchronized(resource2) {
          System.out.println("Thread 2: locked " + resource2);
 
          // Then it pauses, for the same reason as the first thread does
          try { Thread.sleep(50); } catch (InterruptedException e) {}
 
          // Then it tries to lock resource1.  But wait!  Thread 1 locked
          // resource1, and won't release it 'till it gets a lock on
          // resource2.  This thread holds the lock on resource2, and won't
          // release it 'till it gets resource1.  We're at an impasse. Neither
          // thread can run, and the program freezes up.
          synchronized(resource1) {
            System.out.println("Thread 2: locked locked " + resource1);
          }
        }
      }
    });
    
    // Start the two threads. If all goes as planned, deadlock will occur, 
    // and the program will never exit.
    t1.start(); 
    t2.start();
  }
}
One way of dealing with such problems, is using resource ordering. All threads should always acquire their locks in the same order.

Another known example related to problems in concurrency is the dining philosophers problem. For more info see here
Starvation and Livelock
Starvation and livelock are much less common a problem than deadlock, but are still problems that every designer of concurrent software is likely to encounter.
Starvation
Starvation describes a situation where a thread is unable to gain regular access to shared resources and is unable to make progress. This happens when shared resources are made unavailable for long periods by "greedy" threads. For example, suppose an object provides a synchronized method that often takes a long time to return. If one thread invokes this method frequently, other threads that also need frequent synchronized access to the same object will often be blocked. Another example, think that you have a pool of threads, and you are in charge of the scheduling. Imagine that every thread has a certain priority, so you group threads with similar priority and run them in a round robin manner (i.e. every thread from the same priority group receives a similar amount of time to run). Furthermore, A process from a lower priority group runs only if there is no higher priority process waiting. Now, it is clear that threads with lower priority will not run (unless there are no threads with higher priorities).
A way to attack this problem, although not in our control, is the usage of semaphores, which guarantees some kind of fairness in the threads scheduling, in contrast to the synchronize mechanism and the wait/notify mechanism.

Livelock
A thread often acts in response to the action of another thread. If the other thread's action is also a response to the action of another thread, then livelock may result. As with deadlock, livelocked threads are unable to make further progress. However, the threads are not blocked — they are simply too busy responding to each other to resume work. This is comparable to two people attempting to pass each other in a corridor: Alphonse moves to his left to let Gaston pass, while Gaston moves to his right to let Alphonse pass. Seeing that they are still blocking each other, Alphone moves to his right, while Gaston moves to his left. They're still blocking each other, so…
Another Example:
A husband and wife eating at a restaurant, and they are sharing a fork. Since each one of them is so in love with the other, they won't eat unless they are sure the other one has eaten first. This results in a state of livelock: The wife takes the fork, checks if the husband has eaten, returns the fork. The husband takes the fork, checks if the wife has eaten, returns the fork. This situation may go on indefinitely causing a state of livelock.
downloadtoggle
55 lines ...
public class Livelock {
    static class Fork{
        private Diner owner;
        public Fork(Diner d) { owner = d; }
        public Diner getOwner() { return owner; }
        public synchronized void setOwner(Diner d) { owner = d; }
        public synchronized void use() { System.out.printf("%s has eaten!", owner.name); }
    }
 
    static class Diner{
        private String name;
        private boolean isHungry;
 
        public Diner(String n) { name = n; isHungry = true; }       
        public String getName() { return name; }
        public boolean isHungry() { return isHungry; }
 
        public void eatWith(Fork fork, Diner spouse){
            while (isHungry)
            {
                // Don't have the fork, so wait patiently for spouse.
                if (fork.owner != this){
                    try { Thread.sleep(1); } catch(InterruptedException e) { continue; }
                    continue;
                }                       
 
                // If spouse is hungry, insist upon passing the fork.
                if (spouse.isHungry()) {                   
                    System.out.printf("%s: You eat first my darling %s!%n", name, spouse.getName());
                    fork.setOwner(spouse);
                    continue;
                }
 
                // Spouse wasn't hungry, so finally eat
                fork.use();
                isHungry = false;               
                System.out.printf("%s: I am stuffed, my darling %s!%n", name, spouse.getName());                
                fork.setOwner(spouse);
            }
        }
    }
 
    public static void main(String[] args){
       final Diner husband = new Diner("Bob");
       final Diner wife = new Diner("Alice");
 
       final Fork fork = new Fork(husband);
 
       //anonymous class
       Thread t1 = new Thread(new Runnable() { public void run() { husband.eatWith(fork, wife); } });
       Thread t2 = new Thread(new Runnable() { public void run() { wife.eatWith(fork, husband); } });
 
       t1.start();
       t2.start();
    }
}
Guarded methods - Introduction
All the problems presented above originated from our desire to keep our programs running in a timely manner. Adding too much safety elements is complicated and hard to predict. We would like to be able to synchronize lock and block as less code as possible, and the Guarded methods model gives us this ability.
It is not enough to have thread safe object, we also need to ensure that every thread will not eventually lock up. This can happen due to locking, waiting, input, CPU starvation, failures. Sometimes threads depend on each other to continue with execution, and wait for some condition to be met. The guarded method model delays the execution of a thread until a condition is satisfied. The simplest way to understand the dependency between 2 threads is this case:

When T1 sends a message m to a shared object O and the method pre-condition for m is not met, T1 cannot proceed. In the sequential case, the only thing we can do that makes sense is to throw an exception. In the concurrent case, T1 can wait until another active object T2 changes the state of O so that the precondition holds.

This means that a new way to deal with precondition failure is for a thread to simply wait - with the understanding that another thread will cooperate with us and make the condition we are waiting for become true in the future. In this practical session we present various methods for the guarded method model. We begin with basic methods and then we introduce Java classes for advanced timing.

Basic Threads Timing
In this section we talk about basic methods of the guarded method model. The user is responsible for the timing and setting the appropriate variables. Notice that the two first example are usually a wrong solution and the last example is the right solution.
Before you is the model of the Objects in the following example. Notice that only main is calling the change method, and only the created threads are calling the check method.

model.PNG

Busy Wait (a.k.a Spinning)
Each thread constantly checks whether the condition is met. This results in heavy CPU usage, and is considered to be a very bad solution (Threads01.java). Imagine a situation where a customer wants to buy a TV, which is not in stock, persistently asks the salesman whether the TV has arrived. It would waste a lot of that salesman time to answer this repeating question. Spinning is appropriate in very specific cases: when we know that the condition we are waiting for will occur very shortly, and it is critical to react to it with no delay. These cases are extremely rare.
downloadtoggle
133 lines ...
package spl.examples.busywait;
 
/**
 * SPL121 PS #07 Busy Wait, Threads01.java
 *
 *
 * Each thread uses the checker object to check whether it should stop running
 * @author splTeam
 */
 
public class Threads01
{
/**
 * main will create and run the thread objects
 * see output in the end.
 */
    public static void main(String[] args) {
 
        Checker checkerObject = new Checker(0); // the syncronized - release object
 
        // Each thread gets 3 parameters:
        // awake num, output messege, checker object
 
        Thread t1 = new Thread(new SleepThread(1, "NO. 1 done, was waiting for 1", checkerObject));
        Thread t2 = new Thread(new SleepThread(1, "NO. 2 done, was waiting for 1", checkerObject));
        Thread t3 = new Thread(new SleepThread(1, "NO. 3 done, was waiting for 1", checkerObject));
        Thread t4 = new Thread(new SleepThread(2, "NO. 4 done, was waiting for 2", checkerObject));
        Thread t5 = new Thread(new SleepThread(2, "NO. 5 done, was waiting for 2", checkerObject));
        Thread t6 = new Thread(new SleepThread(2, "NO. 6 done, was waiting for 2", checkerObject));
        Thread t7 = new Thread(new SleepThread(3, "NO. 7 done, was waiting for 3", checkerObject));
        Thread t8 = new Thread(new SleepThread(3, "NO. 8 done, was waiting for 3", checkerObject));
        Thread t9 = new Thread(new SleepThread(3, "NO. 9 done, was waiting for 3", checkerObject));
 
        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();
        t7.start();
        t8.start();
        t9.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(1); // release waiting for 1
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(3); // release waiting for 3
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(2); // release waiting for 2
    }
}
/**
 * class SleepThread
 * Each thread gets 3 parameters:
 * awake num, output messege, checker object
 *
 */
class SleepThread implements Runnable {
    /**
    * awake number
    */
    private int m_num;
    /**
    * the string the thread object will print when done
    */
    private String m_strToPrintWhenDone;
    /**
    * checker object- in this example is mutual
    */
    private Checker m_checkerObject;
 
    /**
    * constructor
    */
    SleepThread(int num, String strToPrintWhenDone, Checker checkerObject)
    {
        this.m_num = num;
        this.m_strToPrintWhenDone = strToPrintWhenDone;
        this.m_checkerObject = checkerObject;
    }
 
    /**
    * the run method of the thread
    * this is where the busy-wait occurs
    */
    public void run()
    {
        // This line is Busy Wait (Spinning)
        while (!this.m_checkerObject.check(this.m_num));
 
        System.out.println(this.m_strToPrintWhenDone);
    }
}
/**
 * class Checker
 */
class Checker {
    /**
    * the checker number
    */
    private  int m_num;
    /**
     * constructor
     */
    public Checker(int num)
    {
        this.m_num = num;
    }
    /**
     * the change method changes m_num to num
     * @ return void
     */
    public synchronized void  change(int num)
    {
        this.m_num = num;
    }
    /**
     * the heck method checkes if m_num equals num
     * @return boolean
     */
    public synchronized boolean check(int num)
    {
        return (num == this.m_num);
    }
}
Sleep & Check
This method is very similar to busy waiting in 2.3.1 but this time the thread is ‘nice’ enough to go to sleep each time the condition fails. By doing this we do not waste much of the CPU time (Threads02.java). This time the annoying customer comes every day, asks the question about the TV and then leaves the store. This time not so much of the salesman's' time is wasted. The disadvantage is that we introduce a delay in how fast the waiting thread will react after the condition is met. In our example, the customer may miss his TV by 23:55 hours - if he asks the salesman at 10:00 on Monday whether the TV is ready, and the TV arrives at 10:05 on Monday, the customer will only get his TV on Tuesday at 10:00.
downloadtoggle
128 lines ...
package spl.examples.sleepandcheck;
 
 
/**
 * SPL121 PS #07 Sleep & Check, Threads02.java
 *
 * Each thread uses the checker object to check whether it should stop running
 * @author splTeam
*/
 
public class Threads02
{
/**
 * main will create and run the thread objects
 * see output in the end.
 */
    public static void main(String[] args) {
 
        Checker checkerObject = new Checker(0); // the syncronized - release object
 
        // Each thread gets 3 parameters:
        // awake num, output messege, checker object
 
        Thread t1 = new Thread(new SleepThread(1, "NO. 1 done, was waiting for 1", checkerObject));
        Thread t2 = new Thread(new SleepThread(1, "NO. 2 done, was waiting for 1", checkerObject));
        Thread t3 = new Thread(new SleepThread(1, "NO. 3 done, was waiting for 1", checkerObject));
        Thread t4 = new Thread(new SleepThread(2, "NO. 4 done, was waiting for 2", checkerObject));
        Thread t5 = new Thread(new SleepThread(2, "NO. 5 done, was waiting for 2", checkerObject));
        Thread t6 = new Thread(new SleepThread(2, "NO. 6 done, was waiting for 2", checkerObject));
        Thread t7 = new Thread(new SleepThread(3, "NO. 7 done, was waiting for 3", checkerObject));
        Thread t8 = new Thread(new SleepThread(3, "NO. 8 done, was waiting for 3", checkerObject));
        Thread t9 = new Thread(new SleepThread(3, "NO. 9 done, was waiting for 3", checkerObject));
 
        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();
        t7.start();
        t8.start();
        t9.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(1); // release waiting for 1
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(3); // release waiting for 3
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(2); // release waiting for 2
    }
}
/**
 * class SleepThread
 * Each thread gets 3 parameters:
 * awake num, output messege, checker object
 *
 */
class SleepThread implements Runnable {
    /**
    * awake number
    */
    private int m_num;
    /**
    * the string the thread object will print when done
    */
    private String m_strToPrintWhenDone;
    /**
    * checker object- in this example is mutual
    */
    private Checker m_checkerObject;
 
    /**
    * constructor
    */
    SleepThread(int num, String strToPrintWhenDone, Checker checkerObject)
    {
        this.m_num = num;
        this.m_strToPrintWhenDone = strToPrintWhenDone;
        this.m_checkerObject = checkerObject;
    }
 
    /**
    * the run method of the thread
    * this is where the thread goes to sleep while he is waiting for his turn to run
    */
    public void run()
    {
        // Here is the Check part
        while (!this.m_checkerObject.check(this.m_num))
        {
            try {
                // Here is the Wait part
                Thread.sleep(100);
            } catch (InterruptedException e) {}
        }
 
        System.out.println(this.m_strToPrintWhenDone);
    }
}
/**
 * class Checker
 */
class Checker {
    private  int m_num;
 
    public Checker(int num)
    {
        this.m_num = num;
    }
 
    public synchronized void change(int num)
    {
        this.m_num = num;
    }
 
    public synchronized boolean check(int num)
    {
        return (num == this.m_num);
    }
}
Wait & Notify
The wait & notify mechanism supports communication between threads, as follows: a thread voluntarily waits until some condition (a prerequisite for continued execution) occurs. Some other thread can then notify the waiting thread, to continue its execution (Threads03.java). The most advanced (and most practiced method in reality) is that once the customer has learn that there is no TV in stock, he would leave a phone number. Once the TV is delivered, the salesman will call the customer to come and take it. Once the thread is notified, it must validate the condition again. This is because multiple threads might be waiting for notification. In our example, once the customers are notified about the TV, only the first one to reach the store would get it. The others would have to go home and wait again.
downloadtoggle
130 lines ...
package spl.examples.waitandnotify;
 
 
/**
 * SPL121 PS #07 Wait & Notify, Threads03.java
 *
 * Each thread uses the checker object to check whether it should stop running
 * @author splTeam
 */
 
public class Threads03
{
/**
 * main will create and run the thread objects
 * see output in the end.
 */
    public static void main(String[] args)
    {
 
        Checker checkerObject = new Checker(0); // the syncronized - release object
 
        // Each thread gets 3 parameters:
        // awake num, output messege, checker object
        Thread t1 = new Thread(new SleepThread(1, "NO. 1 done, was waiting for 1", checkerObject));
        Thread t2 = new Thread(new SleepThread(1, "NO. 2 done, was waiting for 1", checkerObject));
        Thread t3 = new Thread(new SleepThread(1, "NO. 3 done, was waiting for 1", checkerObject));
        Thread t4 = new Thread(new SleepThread(2, "NO. 4 done, was waiting for 2", checkerObject));
        Thread t5 = new Thread(new SleepThread(2, "NO. 5 done, was waiting for 2", checkerObject));
        Thread t6 = new Thread(new SleepThread(2, "NO. 6 done, was waiting for 2", checkerObject));
        Thread t7 = new Thread(new SleepThread(3, "NO. 7 done, was waiting for 3", checkerObject));
        Thread t8 = new Thread(new SleepThread(3, "NO. 8 done, was waiting for 3", checkerObject));
        Thread t9 = new Thread(new SleepThread(3, "NO. 9 done, was waiting for 3", checkerObject));
 
        t1.start();
        t2.start();
        t3.start();
        t4.start();
        t5.start();
        t6.start();
        t7.start();
        t8.start();
        t9.start();        
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
 
        checkerObject.change(1); // release waiting for 1
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(3); // release waiting for 3 no one
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {}
 
        checkerObject.change(2); // release waiting for 2
    }
}
/**
 * class SleepThread
 * Each thread gets 3 parameters:
 * awake num, output message, checker object
 *
 */
class SleepThread implements Runnable {
    /**
    * awake number
    */
    private int m_num;
    /**
    * the string the thread object will print when done
    */
    private String m_strToPrintWhenDone;
 
    /**
    * checker object- in this example is mutual
    */
    private Checker m_checkerObject;
 
    /**
    * constructor
    */
    SleepThread(int num, String strToPrintWhenDone, Checker checkerObject)
    {
        this.m_num = num;
        this.m_strToPrintWhenDone = strToPrintWhenDone;
        this.m_checkerObject = checkerObject;
    }
    /**
    * the run method of the thread
    * the thread is sent to the checkerObject method
    */
    public void run()
    {
        // This is the waiting until notification line
        this.m_checkerObject.returnWhenCheckIsTrue(m_num);
 
        System.out.println(m_strToPrintWhenDone);
    }
}
/**
 * class Checker
 */
class Checker {
    private int m_num;
 
    public Checker(int num)
    {
        this.m_num = num;
    }
 
    public synchronized void change(int num)
    {
        this.m_num = num;
        this.notifyAll();
    }
 
    public synchronized void returnWhenCheckIsTrue(int num)
    {
        //notice the while statement and not an if statement
        while (num != m_num)
        {
            try {
                this.wait();
            } catch (InterruptedException e) {}
        }
 
    }
}
Producers-Consumers pattern and Blocking Queues
Blocking Queue
A Blocking Queue is a queue that blocks a caller if it tries to put an element into a full queue, or take an element from an empty queue, until the operation can be performed. (When the queue becomes non-full or non-empty, respectively.)
The Java API contains an interface for Blocking Queue in java.util.concurrent, called BlockingQueue. This interface contains the following methods (among others):

void put(E o)
Adds the specified element to this queue, waiting if necessary for space to become available.
E take()
Retrieves and removes the head of this queue, waiting if no elements are present on this queue.
Implementing Blocking Queue with wait and notifyAll
We can implement the blocking queue functionality using the wait() and notifyAll primitives.
downloadtoggle
42 lines ...
import java.util.Vector; 
  
class SimpleQueue <E> implements BlockingQueue <E>{ 
  
        // a vector, used to implement the queue 
        private Vector<E> vec_;
        private final int MAX; 
  
        public SimpleQueue(int max) { MAX = max; } 
  
        public synchronized int size(){ 
                return vec_.size(); 
        } 
  
        public synchronized void put(E e){ 
                while(size()>=MAX){ 
                        try{ 
                                this.wait(); 
                        } catch (InterruptedException ignored){} 
                } 
  
                vec_.add(e); 
                // wakeup everybody. If someone is waiting in the get() 
                // method, it can now perform the get. 
                this.notifyAll(); 
        } 
  
        public synchronized E take(){ 
                while(size()==0){ 
                        try{ 
                                this.wait(); 
                        } catch (InterruptedException ignored){} 
                } 
  
                E e = vec_.get(0);
                vec_.remove(0); 
                // wakeup everybody. If someone is waiting in the add()  
                // method, it can now perform the add. 
                this.notifyAll(); 
                return e; 
        }
// Implementations of additional methods... 
}
Tips when using the wait/notify mechanism
Use a while loop (and not an if condition) to check the precondition. This is important because another thread could have changed something so that the condition won't hold before the thread that woke up continued. In addition, threads can spontaneously wake up without being notified (doesn't happen very often).
notify() Vs. notifyAll() - which of them should you use?
When calling wait(), notifyAll() or notify() on an object, make sure the calling thread holds the object's lock (since it changes the object's state by adding/removing to/from its queue).Notice that if you do not hold the object's lock you will receive an illegal monitor runtime exception.
After performing a wait call on the object, the thread releases the object's lock and that lock alone, so if the tread acquired more than one lock it releases only the lock of the waiting object. Furthermore, before exiting the wait set of the object, the thread must re-lock the object.
A thread that releases an object's lock, will NOT release other locks it has.

ArrayBlockingQueue
An implementation of Blocking Queue is present in java.util.concurrent. It is called ArrayBlockingQueue.
Producer-Consumer with Blocking Queue
The producer-consumer problem deals with the situation in which several producers generate instances of some product. Once a producer produces a product, it puts it into a shared bounded capacity queue. In addition there are several consumers, each may try to consume a product at any time, that is take a product from the shared queue. A producer should block if the queue is full, and a consumer should block if the queue is empty. A solution to this problem is illustrated using ArrayBlockingQueue.
downloadtoggle
78 lines ...
package spl.util;
import java.util.concurrent.*;
 
 
class Producer implements Runnable {
       private final ArrayBlockingQueue<Integer> queue;
       private final int id;
       private final int amount; //Number of products to produce
       private final int PERIOD = 700; //Time period required for producing one product
       Producer(int id, int amount, ArrayBlockingQueue<Integer> q) {
           this.id = id;
           this.amount = amount;
           queue =q;   
       }
       public void run() {
         try {
             for (int i =0; i < amount; i++){
                 Thread.sleep(PERIOD);
                 queue.put(produce(id,i)); 
             }
             
         } catch (InterruptedException ignored) { }
       }
 
           /**
           * produce an Integer product composed of producer's id and current iteration
           * @param id - producer id
           * @param iteration - current iteration
           * @return Integer product
           */
       private Integer produce(int id, int iteration) {
           int product = id * amount + iteration;
           System.out.println("Producing: " + product);
           return new Integer(product);
           
       }
     }
 
class Consumer implements Runnable {
       private final ArrayBlockingQueue<Integer> queue;
       private final int id;
       private final int amount; //Number of products to consume
       private final int PERIOD = 2000;  //Time period required for consuming one product
       Consumer(int id, int amount, ArrayBlockingQueue<Integer> q) { 
           this.id = id;
           this.amount = amount;
           queue =q; 
        }
       public void run() {
             try {
                 for (int i =0; i < amount; i++){
                     consume(queue.take());
                     Thread.sleep(PERIOD);
                 }
                 
             } catch (InterruptedException ignored) { }
           }
           /**
           * consume an Integer product
           * @param prod - the product to be consumed
           */
       private void consume(Integer prod){
           System.out.println("Consumed:  " + prod);
       }
     }
 
public class ProdCons {
       public static void main(String args[]) {
         final int BOUND = 5;
         final int AMOUNT = 10;
         ArrayBlockingQueue<Integer> q = new ArrayBlockingQueue<Integer>(BOUND);
         Producer p1 = new Producer(1,AMOUNT,q);
         Producer p2 = new Producer(2,AMOUNT,q);
         Consumer c = new Consumer(3,2*AMOUNT,q);
         new Thread(p1).start();
         new Thread(p2).start();
         new Thread(c).start();
       }
     }
# **Thread Safety: Definition and Immutable Objects**

This lecture introduces several issues which must be considered when designing systems for concurrent execution environments. By system, we mean a collection of objects and their interactions. The general system properties to which we aspire are:
* Safety - the property that nothing bad ever happens
* Liveness - the property that anything ever happens at all
* Correctness - the property that the system does what it was meant to
* Reusability of objects - the property that objects can be reused in several systems without requiring changes to their code
* Performance of the activity - the property that the intended activity eventually completes

In the remainder of this lecture, we will focus on Safety. Other properties will be addressed in the rest of the course.

## **Safety** 

The main principle which we must follow is that nothing bad should ever happen to an object. At all times, we should strive to preserve the consistency of our objects. In order to reach a more formal definition of safety, we must define what we mean when we say that an object remains consistent, and that something bad happens during our computation.
There are several kinds of safety that we must consider. In this lecture, we will concentrate on multi-threaded safety.

### **Multi-Threaded Safety**

One major problem of acheiving Multi-Threaded Safety is that the compiler is unable to help us with this goal. To achieve multi-threaded safety, one must design classes carefully. In particular, we must understand what are the pre & post conditions of the class methods and most important what is the invariant of the class.

**Example: Consistency of a class state** Let us consider the following code example. We have a class called Even, which should contain a counter that is kept even at all times. We want to support incrementing the counter, but wish to keep it even. We start out with the following code:

```java
/* A simple counter class, which keeps an even counter */
class Even {
 
   /* the internal state counter */
   private int counter = 0;
 
   /* default constructor */
   public Even() { } 
 
   int getCounter() {
      return counter;
   }
 
   int setCounter(int count) {
      counter = count;
   }
 
   /* increment the counter.
    * return the current counter value 
    */
   public int add() {
      setCounter(getCounter() + 1);
      setCounter(getCounter() + 1);
      return getCounter();
   }
}
```

### **Pre-conditions, Post-conditions and Invariants**

Before discussing the properties of this class, let us introduce the notion of *Pre-conditions*, *Post-conditions* and *Invariants*.

An Invariant is a property of an object, usually a statement regarding its internal state, which should hold throughout the lifetime of the object. In our example of class Even, the invariant is that the internal counter must always remain even. But, considering that the counter might be tinkered with during actions performed on (or by) the object, we also employ the notion of Pre and Post conditions (in short, PnPC). PnPC are statements, much like an invariant, about the internal state of the object. However, PnPC may not hold throughout the lifetime of the object, but only just before and right after a method invocation (an action performed on/by the object).

For example, in class ```Even```, the precondition of the ```add()``` method is that the counter is even and the postcondition is that (yes, again… ) the counter is even. Moreover, the postcondition must also state that the counter has been indeed incremented.

Let us write now the ```Even``` class using those new definitions. please notice the new notations used in the source:

```java
/* A simple counter class, which keeps an even counter */
class Even {
 
   /* the internal state counter 
    * @INV counter % 2 = 0
    */
   private int counter = 0;
 
   /* default constructor */
   public Even() { } 
 
   int getCounter() {
      return counter;
   }
 
   int setCounter(int count) {
      counter = count;
   }
 
 
   /* increment the counter.
    * return the updated counter value 
    * @PRE getCounter() % 2 = 0 (this is redundant, as it is promised by the @INV)
    * @POST @POST(getCounter()) = @PRE(getCounter())+2
    */
   public int add() {
      setCounter(getCounter() + 1);
      setCounter(getCounter() + 1);
      return getCounter();
   }
}
```

You might ask yourself why do we need all of this "unnecessary" baggage, and you may be right in sequential execution. Indeed, in a sequential execution environment nothing wrong may happen to an Even object. However, in concurrent execution environments, things may fall apart. **With dire consequences!**.

#### $$Consistency and Computation$$

The true reason why we introduce these constructs though, is that @inv, @pre and @post statements express exactly what we mean when we say that an object is in a consistent state, and that a computation is correct. The object is consistent if its @inv condition holds. @inv, @pre and @post express formally what the programmer intends: what does it mean that an object is in a consistent state, and what each method is supposed to achieve.

Getting back to the abstract model of objects presented in the previous lecture, a computation involving an object can be viewed as an ordered sequence of messages received by an object. What the object does is:

* Receive a message
* Dispatch the message to a method
* Execute the body of the method as a reaction
* As part of the execution, the object can send messages to other objects

Remember that the internal state of an object in the abstract model is fully encapsulated by the object: only the object can update its own internal state. We can then view a computation as a sequence of transitions:

* The object is first constructed; it is the responsibility of the class constructor that constructed objects are in a consistent state (that is, after the constructor exits, @inv holds).
* Each time the object receives a message (a method is invoked), the method's @pre must be checked. If it does not hold, the computation is invalid.
* After the method completes, the method's @post must be checked. If it does not hold, the computation is invalid.
The execution of the method has moved the object from one internal state Si to the next internal state Si+1.

For a given computation, the objects involved move from states S1, S2, … Sn.
* At each transition, the @inv condition must hold. That is, for all i, @inv(Si) holds.
* For each transition Si–m–>Si+1, where the object processes a message m, we must have that @pre(m)(Si) holds and @post(m)(Si+1) holds.

Naturally, a computation involves more than one object. The overall correctness condition for a system of objects is that all the object computations are correct - as viewed from the perspective of each object.

NOTE 1: while a method is executing, there is no constraint that the @inv remains enforced. For example, consider the Even counter class: in the middle of the execution of the add() method, the internal state of the counter_ variable moves from i to i+1, then to i+2. When it is at value i+1, it holds an odd value (that is, the @inv constraint does not hold). This is ok. Our correctness constraint only requires that the object remains consistent between invocation of methods - not during invocations.

NOTE 2: we haven't yet provided a complete definition of what we mean by correct computation for a system of objects. The constraint that each object sees a correct computation sequence is only part of our complete definition. We'll get back to this later.

### **The Dangers of Concurrent Execution**

How can we reason about code correctness? We want to write code in such a way that all computations involving this code will result in correct executions. If the code we write is executed in a sequential RTE, then we can analyze the code in a pretty simple way: analyze each method, check all potential execution paths, and make sure the @inv, @pre and @post hold. (Easier said than done, but at least we can think of ways to do this.)

When we run in the hybrid execution model described in the previous lecture, we have additional risks that can make code that looks correct run in an incorrect manner. Here is an example.

```java
/* a simple class which has a reference to an Even object. */
class Adder implements Runnable {
 
   /* the reference to an Even Object */
   private Even e;
 
   /* simple constructor
    * @param e a reference to an Even object
    */
   public Adder(Even e) {
      this.e = e;
   }
 
   /* run the counter a little */
   public void run() {
      for (int i=0; i<10; i++) {
         System.out.println(e.add());
      }
      return;
   }
 
   public static void main(String[] args) {
      // Note: a single Even instance is shared by 2 Adder instances.
      Even e = new Even();
      Thread t1 = new Thread(new Adder(e));
      Thread t2 = new Thread(new Adder(e));
  
      t1.start();
      t2.start();
   }
}
```

Try to run this. This is what I got: ```2 5 7 9 11 13 15 17 19 21 23 25 27 29 31 33 35 37 38 40```. What? ```2 5 7...``` ? Why didn't the counter stay even? What is going on here? Once you understand and master the answer, you will never look at your code with the same naiveté ;-)
The Even class is not thread safe

O.K, so what did happen?

To understand this, we need to remember that since my computer has only one CPU, only one thread may be executing at a time. So, at some point in each thread's execution, the thread is preempted by the environment in favor of another thread (either in the same process or a different process). The preemption may take place any time during the execution of the thread. Now, consider the function add() of class Even. The function performs many more actions than you can see in the source code. Let us break the execution to pseudo JVM code:

```
c = c+1;
write c to the location of counter;
c = c+1;
write c to the location of counter;
return c;
```

Since the execution of any thread may be interrupted at any line, consider the following execution flow (time goes from top to bottom):
```
Thread A             Thread B

line 1 to 6	
line 1 to 4	
                     lines 1 to 4
lines 5 to 6	
                     lines 5 to 6
```

After the first thread finishes the first ```add()``` invocation, the value of ```counter_``` is ```2```. After the second thread finishes the first invocation, the value returned to the second thread is ```5```!.
you might just say that we are to blame for this catastrophic event, since we incremented ```counter_``` twice. O.K, continuing this thought, let us change the relevant code:

```java
Class Even {
 ...
  public int add() {
    setCounter(getCounter()+2);
    return getCounter();
  }
}
```

Again, converting to pseudo Byte Code:
```
register c = read counter_ from memory;
c = c+2;
write c to the location of counter_;
```

What happens when one thread is interrupted just before line 3? All of our @pre, @post and @inv hold, but another constraint is broken. This is explained in the next section.

### **An Additional Global Criterion on Computation Correctness**

In fact, the situation can get even more complex. Assume 2 threads T1 and T2 are accessing the same counter, with the update in a single step ```(counter_+=2)``` statement. Assume that the interleaving of execution between T1 and T2 is such that:
```
Assume counter_ = 2 at time t0
T1 executes line 1 and is interrupted. (c == 2 / @pre holds when T1 starts execution of add)
T2 executes line 1 and is interrupted. (c == 2 / @pre holds when T2 starts execution of add)
T1 executes lines 2 and 3 (c == 4 / @post holds / @inv holds)
T2 executes lines 2 and 3 (c == 4 / @post holds / @inv holds)
```

What happened in this specific execution scenario is that the same object (instance of ```Even```) is shared between 2 threads (T1 and T2). The object executes the method ```add()``` twice – but at the end, the state of the object is incremented only once. What is troubling here, is that our local constraints (@inv / @pre / @post) never failed. T1 thinks all is fine. T2 thinks all is fine. But if you think that this counter is reporting how much money is in your bank account each time you deposit money into it, you will be legitimately upset if one of your deposits is ignored in such a way. We have the intuition that "something wrong happened" but our formal tools cannot tell us what.

To address this problem, we must introduce a stronger global constraint to define execution correctness at the object system level (and not only for each object separately):

The (finite) concurrent execution of a program on a system of objects is correct iff:

1. Every step of the computation satisfies the object correctness criteria (@inv/@pre/@post)
2. At the end of the computation, the system is in one of the states that could have been reached by a sequential execution of the same program and the sequence of states reached by each object could have been reached by a sequential execution.

Let us see one simple example: the system includes one object with a single integer variable initialized at value 0. The object has 2 methods inc3 adds the value 3 to the state and inc8 adds 8 to the state of the object. The program includes 2 invocations of inc3 and inc8. If there are no ordering constraints on these 2 steps of the program, the possible sequential executions paths are: (0, 3, 11) or (0, 8, 11). In this case, all sequential executions lead us to the same final state (11). If a concurrent execution leaves the object in state 3 or 8 – even though each of the steps of the computation is correct, the overall computation would not be correct because it missed one of the steps of the computation.

Note that this new constraint on correctness is much stronger (it looks at all possible sequential executions of the program and at all the objects in the sytem). It is still not fool-proof (think of what happens if the method inc8() is defined as if (i > 0) i+=8).

In general, there exist several definitions of what it means for a global distributed system to execute a program correctly. The formal study of consistency is introduced in this Wikipedia article on [Consistency models](http://en.wikipedia.org/wiki/Consistency_model). The informal definition we discussed above is most closely related to the model called [linearizability](http://en.wikipedia.org/wiki/Linearizability).

#### **Understanding What Went Wrong**

What we have demonstrated is a class which is **correct** in a sequential RTE, but **incorrect** in a concurrent RTE. In other words, because we live in a concurrent RTE, we (the developers) must invest extra-effort to ensure correctness.

Let us reflect on what went wrong: in the hybrid model of execution of object computation, we distinguish between active objects and passive objects. The passive objects are shared between several active objects. At any given point of the execution of a method of the passive object, an active object is "simulating" the abstract object whose state is maintained in the passive object.

You will review in the course PPL (second semester) a formal view of the problem using the Scheme language. For those interested, the following [Chapter](http://mitpress.mit.edu/sicp/full-text/book/book-Z-H-23.html#%_sec_3.4) of the excellent book Structure and Interpretation of Computer Programs by Abelson and Sussman presents the material with great clarity.

In the meantime, we must understand that in the hybrid model, the smallest steps of the computation are not a single method execution, but each instruction executed by the JVM, at the instruction set level.

Now consider this example: an abstract computation system involves 2 objects O1 and O2. From the perspective of each object, the computation involves these sequences of transitions:
```
O1: S11 --m1--> S12 --m2--> S13
O2: S21 --n1--> S22
```

When this abstract system is implemented in a sequential model RTE, the 3 transitions must be ordered relative to each other into a single execution. How is this achieved? We need to look at the details – which object sends which message to whom? If there is no dependency among O1 and O2, then all possible interleaving executions are possible:
```
S11 S12 S13 S21 S22
S11 S12 S21 S13 S22
S11 S12 S21 S22 S13
S11 S21 S12 S22 S13
S11 S21 S22 S12 S13
S21 S11 S22 S12 S13
S21 S22 S11 S12 S13
```
If there is a dependency - for example, the message n1 is sent by O1 during the execution of m2, then there are less possible total orderings in the sequential execution that correspond to the abstract computation. The fact that n1 is sent by O1 during the execution of m2 is translated to an additional ordering constraint S22 > S12. The orderings which become possible in this case are only:
```
S11 S12 S13 S21 S22
S11 S12 S21 S13 S22
S11 S12 S21 S22 S13
S11 S21 S12 S22 S13
```
In general, any of the mechanisms we will introduce to ensure safety in concurrent RTEs rely on this principle: reduce the scheduling of primitive state transitions among passive objects by imposing serialization constraints among independent transitions.

### **Safe Concurrent Programming Ingredients**

To avoid the disastrous effects we observed in the example above, we must make sure our program is thread-safe.
To achieve that we must rely on one of the safe concurrent programing ingredients:

* **Immutability** - Avoiding state changes
* **Synchronization** - Dynamically ensuring exclusive access
* **Containment** - Structurally (using design patterns for) ensuring exclusive access

The rest of this lecture discusses Immutability.

### **Immutable Objects**

The most simple and elegant solution of ensuring thread safety for our objects is ensuring that no thread may change the internal state of the object. This must be done at the **design** stage. That means that you may not be able to make a class immutable after it was already used as non immutable without re-factoring large parts of your code. You should decide in advance which of your objects would be immutable.
As an example, consider the following implementation of the class Even:

```java
/* A simple counter class, which keeps an even counter */
class Even {
 
   /* the internal state counter - we made this final.
    * @INV counter % 2 = 0
    */
   private final int counter;
 
   /* private default constructor */
   private Even() { 
      counter = 0;
   }
 
   /* factory method - constructs instances of Even*/
   public static Even create() {
      return new Even();
   }
 
   /* a private constructor. */
   /* @PRE c % 2 = 0 */
   /* NOTE: It is OK to assign a value to a final field in the constructor - */
   /* as long as the field declaration does not include an initial value     */
   private Even(int c) {
      counter = c;
   } 
 
   /* increment the counter.
    * return a new Object  
    * @PRE counter % 2 = 0 (this is redundant, as it is promised by the @INV)
    * @POST @POST(counter) = @PRE(counter_)
    * @POST @POST(return.counter) = @PRE(this.counter_)+2
    */
   public Even add() {
      return new Even(counter+2);
   }
}
```

This implementation of Even is *immutable*, meaning that once a new object of this class has been instantiated, its internal state may not change. Ever. This ensures that the object is always safe, even in concurrent execution environments.

**How to create immutable objects**

* Don't provide "setter" methods.
* Make all fields final and private.
* Don't allow subclasses to override methods. The simplest way to do this is to declare the class as final. A more sophisticated approach is to make the constructor private and construct instances in factory methods.
* If the instance variables (members) include references to mutable objects, don't allow those objects to be changed:
  * Don't provide methods that modify the mutable objects.
  * Don't share references to the mutable objects. Never store references to external, mutable objects passed to the constructor; if necessary, create copies, and store references to the copies. Similarly, create copies of your internal mutable objects when necessary to avoid returning the originals in your methods.
* Immutable instance variables are always initialized during construction.

**Immutable objects are possibly applicable when**

* Object serves as instances of a simple abstract data type representing values. For example: colors, numbers, strings.
* When different classes supporting different usage can be designed, one immutable and the another updatable. For example, in Java ````java.lang.String```` is immutable while ````java.lang.StringBuffer```` is updatable.
* When the benefit of never needing to protect the object outweighs the cost of copying the object each time it needs to be changed. (See our Even example.) This copying technique is popular and is valid as long as we keep in mind the trade-off (both readability and execution time). Java (unlike C++) does not support pass-by-copy for non-scalar types so the author must enforce the copy by another assignment. Furthermore, in some RTE (not Java though), the actual copying may be delayed to the moment a change occur to the variable, thus saving execution time in some scenarios.
* When you would like to have multiple objects representing the same values (for a reason not related to safety). In this case just keep in mind to make the objects immutable.

Another kind of objects that may be immutable are helper classes, for example:

```java
/* a helper for some Server class */
class Relay{
 
   private final Server server;  // this variable will not change
   
   /* constructor 
    * @param s - a server object to be relayed
   */
   Relay(Server s) {
     server = s;
   }
 
   /* do some action */   
   void doIt() {
     server.doIt();
   }
}
```
More examples and details about immutable objects can be found here

#### **Stateless methods**

Another aspect of immutability are stateless methods. A stateless method is a method that does not change the object state. Sometime a class can provide services hence all its methods are stateless.

### **Publish and Escape**

When considering thread safety, it is convenient to introduce new terminology regarding the behavior and implementation of classes.

**Publish**
We say that the internal state of an object is published if it is accessible from outside the object. For example, consider a public member. By definition, this member is published as soon as the object is created. Another example is a private member to which a reference is returned by a public method call, e.g., a private Vector that is returned in a public getter. Each published member must be thoroughly protected (either be immutable or protected by using locks, which we will discuss in the next lecture).

**Escape**
We say that some of the internal state of an object has escaped if a reference to the internal state of the object is available outside of the object inadvertently. For example, consider returning a reference to an internal member of an immutable object (which is not immutable in itself). Special care must be taken to avoid 'this' to escape during construction of an object (otherwise, other objects could access the object before it has reached a valid state).

**Example 1**
In this example we give a degenerate implementation of the Observer design pattern[^DesignPatterns]. The idea is that an object can register itself to some event source and be notified each time something happens on that source. For example, consider a vector who can notify each time a modification happens (element added/removed). In our example the EventListenerImpl is an object who would like to be notified on events of the EventSource.

[^DesignPatterns]: Design patterns are general reusable solution to a commonly occurring problem in software design. A design pattern is not a finished design that can be transformed directly into code. It is a description or template for how to solve a problem that can be used in many different situations. During the course we will encounter some design patterns and you are encouraged to learn more about this topic.

```java
public Interface EventListener 
{
   public onEvent();
}
 
public class EventListenerImpl implements EventListener { 
  public EventListenerImpl(EventSource eventSource) {
    // do some initialization
    ...
    // register ourselves with the event source
    eventSource.registerListener(this);
  }
  public onEvent() { 
    // handle the event
  }
}
```

In example 1, 'this' of EventListener escaped during construction of the class. This code is in danger of exposing an incompletely constructed EventListener object to other threads. Once registered, eventSource can call the onEvent method although the class has not yet been constructed.

**Example 2**
```java
public class EventListener2  {
 
//inner class
class InnerListener implements EventListener {
        public void onEvent() { 
          eventReceived();
        }
}
// end of inner class
 
  public EventListenerImpl2(EventSource eventSource) {
    eventSource.registerListener(new InnerListener());
  }
 
  public void eventReceived() {
     // handle the event
  }
}
```

In example 2 'this' escaped implicitly. A reference to the object under construction is being published – in this case indirectly – where another thread can see it. When creating the new InnerListener it received 'this' of EventListenerImpl2 (or how would it be able to call the eventReceived method).

The interested reader is referred to [Safe construction techniques](http://www.ibm.com/developerworks/java/library/j-jtp0618.html) for more details about publish and escape.
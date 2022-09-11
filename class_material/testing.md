# **Test-Driven Development**

In this lecture, we introduce testing terminology and a code-writing style known as Test-Driven Development (TDD).

## **Context: Requirements, Modeling and Testing**

When a developer writes code, he is addressing requirements: that is, code comes to solve a problem that someone (either the developer himself or someone he is working for) has specified. When code is written, we must verify that it answers the requirements – that is, the program that we have developed indeed performs according to specification.
The key words in this simple story are: requirement, specification and verification. <!-- We will discuss in another lecture (System Modeling) how requirements and specifications can be derived from an informal problem description.--> The only point we will assume here is that a complex problem specification can be broken down into many smaller problems – until each small problem is small enough to be solved by writing short pieces of code. In this lecture, we discuss how these small pieces of code should be specified and verified.

Code specification and verification means: how do we know what code should do, and how do we know that code indeed performs what it should do.

Another aspect of the context we will take for granted is that the requirements of program will change over time. This evolution is caused by many factors – the most significant one being that as we develop more of a program, we understand better what the program should do. The developer asks better questions, and gets better answers. This often means our understanding of the problem we solve evolves as we solve it. The consequence of this unavoidable process of change in the specifications, is that existing code must be changed. And when code is changed, other parts of the code are affected in ways that are often difficult to predict.

## **What is Testing**

In general, testing is a process. Its goal is to certify that code meets certain properties. The most important property we wish to certify is correctness – that is, the code performs according to specification.
One can distinguish 3 types of code tests:

* **Unit tests**: used to verify that a particular module of source code is working properly. Unit tests are implemented by the programmer. They take the form of simple test cases for all functions and methods. A test case is itself a program - whose goal is to test another program.

* **Integration tests**: verify that a combination of modules work well together and exchange messages according to their published protocols (callers satisfy pre-conditions).

* **Acceptance tests**: put a whole system under test to check that the expected functionality meets the requirements - including functional and performance criteria.

In the rest of this lecture, we will focus on unit tests. Unit tests are the responsibility of the developer.

### **Unit Test Terminology**

**Test case**: tests a single scenario of usage of the object under test. A Test case should only test one aspect of the protocol of the object: its invariant, a single pre-condition or a single post-condition.

**Test suite**: a collection of test cases that fully verify the public protocol published by an object.

**Object under test (OUT)**: the object being tested by a test suite. A test suite should focus on testing a single object - assuming all other objects perform correctly.

**Test coverage**: the part of the code of the object under test that is exercised/executed by running a test suite. Optimally, every line of code of the OUT should be executed when running the test suite.

**Test fixture**: as part of a test case, one needs to define other objects that will interact with the OUT. These objects are called test fixtures. A test case must be self-contained, it must create all the required test fixtures, initialize them, then create all the conditions necessary before the test scenario can be executed. Test fixtures must be cleaned up after the test case is run.

**Mockup**: a basic, test-specific, implementation for classes on which OUT depends[^Interface]. Although a real implementation may already exists in your program you would avoid using it so the test of the OUT will be independent of other objects. We are not interested in the behavior of the mockup and we will implement just enough for the test to run. Usually, you create a mockup for data-retrieval, but not for logical behavior. Examples: user input, database access.

[^Interface]: A mockup and the "real" object will usually satisfy the same interface - so when the OUT is being tested the mockup is given as the implementation but otherwise the "real" object is given. In that way you know you are really testing the OUT and not other classes.
Positive test case: a test case that verifies that a public operation of the OUT performs as expected.

**Negative test case**: a test case that verifies that a public operation of the OUT fails as expected - that is, a call to a method when a pre-condition does not hold properly throws an exception.

**Repeatable and deterministic tests**: test cases must be repeatable - that is, if one runs the same test twice in a row, we expect to obtain the same result. The test case cannot depend on external data or on the timing of the runtime environment scheduler. The test case must produce the same results in a deterministic manner (that is, do not define tests that sometimes succeed and sometimes fail).

## **Test-Driven Development (TDD)**

Test first design is primarily a programming technique with a side effect of ensuring that your source code is thoroughly unit tested. It means repeatedly first writing a test case and then implementing only the code necessary to pass the test. See the [Wikipedia TDD](http://en.wikipedia.org/wiki/Test-driven_development) article for background information.

It is the responsibility of the programmer to define the unit tests as part of the code delivery. It is not the responsibility of "someone else" (QA department, another programmer). Unit tests define the correctness criteria of the object under test. The same holds for Integration test.

Read again the previous paragraph: at first glance, we would assume that a test comes to verify that existing code stands by existing requirements. The problem is that no one gives you requirements in a clear enough manner before you write code. There are several reasons for this: non-programmers use a different language than programmers, and, therefore, cannot express requirements in a formal enough manner; programmers need requirements at the level of granularity of the module they have defined; since they are the ones who decided how to break-down the program into modules, we cannot expect someone else to know what each internal module is responsible of doing; the requirements on one module depend on the behavior of other modules the programmer has decided to use.

There are two side effects of using TDD:

1. The programmer first expresses requirements on the code he will write by writing a test. In other words, the test case IS the expression of the requirement. The programmer expresses in code what the requirement on the OUT is.

2. The programmer then implements code to stand by the requirements he has specified.

#### **Test First Design cycle:**

1. Decide what is your Object Under Test (OUT). This decision corresponds to an understanding of the context, the problem and the fact that other objects will exist in the program, with which the OUT will interact. The OUT is responsible for only specific functionality.

2. Write the tests for each Object Under Test (OUT):
   * Define the **interface** of the OUT
   * Define the **contract** for each method of the OUT interface
   * Specify **pre-conditions** and **post-conditions** for each method and **invariant** for the OUT. (see DBC below)
   * Write test cases for each invariant, pre and post-condition of each method in the interface.
3. Write the code so it will pass the test
4. Run tests
5. Refactor!
   * Improve **the design** of the code by removing code that "looks bad". These bad aspects of code are usually called "code smells".
6. Repeat the process (This is because you have changed the code – every code change is suspicious as it can introduce new errors, or change the rules – which means the tests must be changed).

When needed - Break the contract, Redefine tests, Refactor code. (This happens when your understanding of the responsibility of the OUT evolves – it is expected that this happens as you code more of the code of the OUT.)

#### **Design By Contract (DBC)**

Design by Contract is a method that helps programmers express the requirements on an object. It is a very simple language that is constructed of these basic elements:
* preconditions : things that must be true before we invoke a method.
* postconditions : things that must be true after a method is invoked.
* Invariants: things that must be true both before and after a method is invoked.

See the Wikipedia [DBC](http://en.wikipedia.org/wiki/Design_by_contract) article for background information on DBC.

#### **Benefits of TDD**

As a development method, TDD has several benefits:

* It gives a way to the programmer to **express correctness criteria** on the code he writes (using the DBC terminology of invariant, pre and post-conditions).

* It creates a set of test cases that ensure the **validation of correctness**: errors caused by code/design modifications are caught quickly by the programmer himself, immediately after the code is changed.

* It gives the programmer **courage** to make code modifications and refactoring, because the programmer does not fear anymore that a change in the code will "break" the program.

* **Integration tests** - test cases can help design intelligent integration tests.

* **Design improvement** - writing tests (that uses the OUT interface) before the OUT is implemented ensures that the OUT is indeed usable, that it is easy to prepare the context in which the OUT can be invoked. If the OUT depends on too many other objects or global objects, writing tests becomes very difficult. When the programmer writes the test cases, he feels this pain, and has a strong incentive to improve the design of the OUT.

* **Documentation** - test classes provide examples of code usage.

## **Using JUnit for TDD**

JUnit is a testing framework written by Erich Gamma and Kent Beck. It is used by developers who implement unit tests in Java. JUnit is Open Source Software. It has become extraordinarily popular in the past 10 years. In particular, it is supported in most Java IDEs.

### **How to write a test class**

We will use a short example. We will go into greater details in later examples. A test class should correspond to one OUT. It should have:

1. A public default constructor

2. A method to prepare the conditions that must hold before the test case can run. This method is marked by the @Before annotation, and is often called setUp().

3. A method to "clean up" the test object after a test has run. This method is marked by the @After annotation, and is often called tearDown(). This method often "undoes" what the @Before method did.

4. A test method for each test case. The test methods are marked by the @Test annotation.
When running a test method the following steps are executed by the JUnit test runner:

   a. An instance of the test class is constructed.
   b. Run setUp()
   c. Run the test case
   d. Display test results (pass/fail)
   d. Run tearDown()

JUnit uses the @Before, @After and @Test annotations in the code to mark the setUp, test and tearDown methods. The JUnit runner knows how to identify these annotations and execute the calls in the right order.

### **Test First Design Example**

We will iterate through an example to illustrate the TDD method.

#### **Analyze the requirement**

We are given this request: Write a simple Stack data object.

Our first reaction is: What is a "Stack"?

We start with an informal description:
```
A Stack is a container of Objects. 

The objects inside the stack are ordered in Last-In-First-Out order.

You can add an Object to the Stack and take an Object out. 
```

#### **Turn the Description into an interface**

In our attempt to formalize our understanding, we turn the verbal description into an interface. Here is how we describe a Stack using an interface:

[^link]: This is a Javadoc annotation that refers to Java source code. For example: {@link spl.utils.Stack}.

```
package spl.utils;
 
 /**
 * Stack is an {@link[^link] Object}s container. Contents are ordered in 
 * Last-In-First-Out order.
 * You can add {@link Object} to the Stack 
 * and take an {@link Object} out. 
 * 
 * @author Student
 *
 */
 public interface Stack {
 }
 ```

**Generics** - It is a "code smell", to let a container receive any object. The reason is that Object is way too general… If we use Object, it means we haven't thought enough about what specific objects we are interested in. We should consider using [Generics](https://github.com/bguspl/bguspl.github.io/blob/main/class_material/generics.md):

```
package spl.util;
 
 /**
 * Stack is an objects container. All objects are of type T. Contents are ordered in Last-In-First-Out order.
 * You can add an object T to the Stack and take a contained Object T out. 
 * 
 * @author Student
 *
 */
 public interface Stack<T> {
 }
 ```

We now fill the interface with more data. We use the DBC terminology to force us to think about "what it means to use a Stack": Methods, Preconditions, Postconditions. To indicate clearly what we mean, we use the @pre, @post and @inv annotations inside our comments to refer to pre-condition, post-condition and class invariant. This is just a convention, nothing else depends on it.

```
package spl.util;
 
/**
 * Stack is an objects container. All objects are of type T. Contents is
 * ordered in Last-In-First-Out order. You can add an object T to the Stack and
 * take a contained Object T out.
 * 
 * @author Student
 * 
 */
public interface Stack<T> {
 
    /**
     * Will add the object at the top of the stack.
     * 
     * @param obj
     *            any non null T object.
     * @pre: none.
     * @post: this.isEmpty() == false
     * @post: this.pop(); this == @pre(this)
     * @post: this.pop() == @param obj
     */
    void push(T obj);
 
    /**
     * Will remove the top object in the stack and return it.
     * 
     * @return the top most object, if there is one. Null, otherwise.
     * @throws Exception
     * @pre: this.isEmpty() == false
     * @post: none.
     */
    T pop() throws Exception;
 
    /**
     * @return True if the Stack is empty, or False if the Stack contains at
     *         least one {@link Object}.
     * @pre: none.
     * @post: none.
     */
    boolean isEmpty();
 
}
```

**push** - The method push does not return a value (void). This was a design decision. If we choose to limit the size of the stack (another design decision) we might have chosen to let push return true/false.
We will see below that we will change our design decisions to make this class testable.

#### **Create a test-class for the interface**

We will write a test class manually. When you work in Eclipse, you can use the Eclipse JUnit support to assist you in the creation of test cases. We first create a test-class skeleton for the interface.

```
package spl.util;
 
/**
 * This is a Unit Test for the {@link Stack} interface. 
 * 
 * @author Student
 * 
 */
public abstract class StackTest<T> {
 
    /** Object under test. */
    private Stack<T> stack;
 
    /**
     * Test method for {@link spl.util.Stack#isEmpty()}.
     */
    @Test public void testIsEmpty() {
        fail("Not yet implemented");
    }
 
    /**
     * Test method for {@link spl.util.Stack#pop()}.
     */
    @Test public void testPop() {
        fail("Not yet implemented");
    }
 
    /**
     * Test method for {@link spl.util.Stack#push(java.lang.Object)}.
     */
    @Test public void testPush() {
        fail("Not yet implemented");
    }
}
```

We don't yet need to think of how to implement these tests. Let us think of more complicated scenarios. Think how we would use a stack object:

* parameters
* exceptions
* return values

Think of even more complicated scenarios. What happens when we do sequences of calls on our stack? For example:
* push-push-pop.
* push-pop-pop
* pop and get Exception
* isEmpty returns true or false
* push a null
  * What should we do?
  * If you were allowed to change the Stack API, then you could add Exception to push().
  * If we do not change the interface, we must specify in the Javadoc what is the expected behavior (do nothing) and test it.
  * This is an example how writing the test is a different frame-of-mind than writing the code. While thinking of tests you may have ideas for constraints over the tested class.
* push objects of different types (not T, a subclass of T): what should happen?

We then fill in the tests behavior. Note that we think of all these tests BEFORE we have started implementing the stack code. This is the whole point of TDD.

When we write our tests, we must define the type T. We could define ```Stack<Object>``` and do ```testPushIntegerPushString```. But, we will keep it simple and first define ```Stack<Integer>```. In all the test methods, we will assume "this.stack" is already instantiated. To reflect this, we add a @Before method that actually creates the stack to be tested.

```
public class StackTest {
    /** Object under test. Note that we refer to the OUT through its interface */
    private Stack<Integer> stack;
 
    /**
     * Set up for a test.  Note the @Before annotation.  It indicate this method is executed before the tests of
         * this test case are executed.
     */
    @Before protected void setUp() throws Exception {
        this.stack = createStack();
    }
 
    /**
     * This creates the object under test.  Note that we must create a specific implementation (StackImpl) 
         * of the interface under test. The rest of the test class only refers to the interface under test.
     * 
     * @return a {@link Stack} instance.
     */
    protected Stack<Integer> createStack() {
                return new StackImpl<Integer>();
        }
 
    /**
     * Test method for {@link spl.util.Stack#isEmpty()}: 
         * after a stack is created, it is initially empty.
         * After we push an object, the stack is not empty.
     */
    @Test public void testIsEmpty() {
        Integer i1 = 1;
        assertTrue(stack.isEmpty());
        stack.push(i1);
        assertFalse(stack.isEmpty());
    }
 
    /**
     * Test method for {@link spl.util.Stack#pop()}. Positive test.
         * Note: This is a weak test.  We will need to revise it.
     */
    @Test public void testPop() {
        Integer i1 = 1;
        stack.push(i1);
        try {
            stack.pop();
        } catch (Exception e) {
            fail("Unexpected exception: " + e.getMessage());
        }
    }
 
    /**
     * Test method for {@link spl.util.Stack#pop()}. 
         * This is a negative test - cause an exception to be thrown.
         * We verify that the OUT fails when we invoke it without keeping the pre-condition.
         * When we invoke a method without keeping the pre-condition, we expect the method to throw an exception.
     */
    @Test public void testPopException() {
        try {
            stack.pop();
            fail("Exception expected!");
        } catch (Exception e) {
            // test pass
        }
    }
}
```
We add a bit more complex test scenarios.

```
package spl.util;
 
import static org.junit.Assert.*;
import org.junit.Before;
import org.junit.Test;
 
/**
 * This is a Abstract Unit Test for the {@link Stack} interface.
 * 
 * @author Student
 * 
 */
public class StackTest {
 
    private Stack<Integer> stack;
 
    /**
 
     * Set up for a test.
     */
    @Before protected void setUp() throws Exception {
        stack = createStack();
    }
 
    /**
     * This creates the object under test.
     * 
     * @return a {@link Stack} instance.
     */
    protected Stack<Integer> createStack() {
                return new StackImpl<Integer>();
        }
 
    /**
     * Test method for {@link spl.util.Stack#isEmpty()}.
     */
    @Test public void testIsEmpty() {
        Integer i1 = 1;
        assertEquals(true, stack.isEmpty());
        stack.push(i1);
        assertEquals(false, stack.isEmpty());
    }
 
    /**
     * Test method for {@link spl.util.Stack#pop()}. Positive test.
     */
    @Test public void testPop() {
        Integer i1 = 1;
        stack.push(i1);
        try {
            stack.pop();
        } catch (Exception e) {
            fail("Unexpected exception: " + e.getMessage());
        }
    }
 
    /**
     * Test method for {@link spl.util.Stack#pop()}. Negative test - throw an
     * exception.
     */
    @Test public void testPopException() {
        try {
            stack.pop();
            fail("Exception expected!");
        } catch (Exception e) {
            // test pass
        }
    }
 
    /**
     * Test method for {@link spl.util.Stack#push(java.lang.Object)}. Verify
     * post-conditions
     * 
     * @throws Exception
     */
    @Test public void testPush() throws Exception {
        Integer i1 = 1;
        Integer i2;
        stack.push(i1);
        assertFalse(stack.isEmpty());
        i2 = stack.pop();
        assertEquals(i1, i2);
    }
 
    /**
     * Test method for {@link spl.util.Stack#toString()}.
     */
    @Test public void testToString() {
        String expected = "{}";
        String actual = this.stack.toString();
        assertEquals("The string representation of an empty Stack is incorrect.", expected, actual);
    }
 
    /**
     * Test the construction/set-up of the Stack
     */
    @Test public void testStackConstructor() {
        assertNotNull("new Stack is not null", this.stack);
        assertTrue("new Stack is empty", this.stack.isEmpty());
    }
 
    /**
     * Test {@link spl.util.Stack} for Last-In-First-Out. Will push three
     * objects and then pop them and will make sure they are ordered from last
     * to first.
     */
    @Test public void testStackLIFO() {
        Integer i1 = 1;
        Integer i2 = 2;
        Integer i3 = 3;
        Integer p;
 
        Assert.assertTrue("Stack s is not empty", this.stack.isEmpty());
        stack.push(i1);
        stack.push(i2);
        stack.push(i3);
        try {
            p = stack.pop();
            assertEquals("wrong object is popped out of the stack", i3, p);
            p = stack.pop();
            assertEquals("wrong object is popped out of the stack", i2, p);
            p = stack.pop();
            assertEquals("wrong object is popped out of the stack", i1, p);
            assertTrue("stack is not empty after last pop()", stack.isEmpty());
        } catch (Exception e) {
            Assert.fail("Exception was thrown.");
        }
    }
 
    /**
     * Push a null
     */
    @Test public void testPushNull() {
        this.stack.push(null);
        assertTrue("Should still be empty, after push(null).", stack.isEmpty());
    }
}
```

The ```setup()``` method will run before every test method when the test case is executed. Its function is to initialize the stack (the OUT).

We now create a skeleton class, ```StackImpl```, implementing the ```Stack``` interface:

```
package spl.util;
 
/**
 * @see Stack
 * @author Student
 *
 */
public class StackImpl<T> implements Stack<T> {
 
}
```

We only now start to care about passing the tests. According to TDD, we write the minimum (!) code in StackImpl that will pass the tests. That is, if no test fails, we don't need to write any code.

While writing Stack we may see we need another class. Such a class will require test cases of its own.

Any code that passes the tests is a valid implementation of Stack. (Remember, the code still has to keep with the coding standards as discussed in the Exercise session.)

Tip: for debugging purposes, it is a good idea to override (and test) the toString method, for all your classes.

### **Refactoring our Tests**

We now look at our tests with a critical eye. Some of our tests are not very specific. We will illustrate 6 principles on how to write testable interfaces and the corresponding DBC contracts. This discussion is derived from "Design by Contract by Example", Mitchell & McKim, 2002, Addison Wesley, Chapter 2.

Consider first the positive test for the ```push``` method:

```
/**
     * Test method for {@link spl.util.Stack#push(java.lang.Object)}. 
     * 
     * @throws Exception
     */
    @Test public void testPush() throws Exception {
        Integer i1 = 1;
        Integer i2;
        stack.push(i1);
        assertFalse(stack.isEmpty());
        i2 = stack.pop();
        assertEquals(i1, i2);
    }
```

We are using the method ```pop()``` to test the behavior of the method ```push()```. This sounds circular, but it is a valid way of specifying the behavior of methods.

A more serious concern might be that the only way we have to test the state of the stack after we have pushed an item on it, is to use a method that actually changes the state of the stack: ```pop()``` not only tells us what is on the top of the stack, it also removes it.

Another concern appears when we look at the test of ```pop()```:

```
/**
     * Test method for {@link spl.util.Stack#pop()}. Positive test.
     */
    @Test public void testPop() {
        Integer i1 = 1;
        stack.push(i1);
        try {
            stack.pop();
        } catch (Exception e) {
            fail("Unexpected exception: " + e.getMessage());
        }
    }
```
This test is really weak. It just verifies that if we push an item on the stack, we can pop the stack without getting an exception. We should in fact test that: (1) ```pop()``` returns the last element pushed on the stack; but also (2) that the stack has now one element less than what it had before. If we were to write that test, we would end up with a copy… of ```testPush()```. In other words, we have only one test to verify 2 functions (```push``` and ```pop```). This is not right.

When we see that our tests must verify complex post-conditions - it is time to think: is the contract of the method ```pop()``` well understood? In this case, we realize that ```pop()``` does 2 things: it removes an item from the stack and it returns the value of this item. It is, in general, better to design methods to do one thing. This makes them more reusable and more testable.

#### **Separate Commands and Queries**

We will use the following general rule to help us sort out methods that have a side-effect on the object and functions that only get information out of the object without modifying it. For example, push() only has a side-effect and isEmpty() only gets information, without changing the object.

**Principle 1**
**Separate commands and queries** - *Queries* return a value and do not change the visible state of the object. *Commands* change the internal state of the object and do not return values.

If we apply this principle, we realize that ```pop()``` does not stand by this rule: it is neither a command nor a query.

We will now refactor the ```Stack``` interface. Our main motivation is to make the interface more testable. We replace ```pop()``` by two methods: one query ```top()``` returns the value of the top object, and one command ```remove()``` removes the top object from the stack.

**Changing an interface**
It seems quite radical to remove the ```pop()``` method from a stack interface. People expect ```pop()``` there. What we have written above does not mean we must remove ```pop()``` completely - it means we should introduce more primitive methods that stand by Principle 1 and can be validated. One can then introduce a derived method that uses the tested primitive methods. For example ```T pop() { T top = top(); remove(); return top; }```

Our Stack interface should now look as follows:

```
package spl.util;
 
 /**
 * Stack is an objects container. All objects are of type T. Contents are ordered in Last-In-First-Out order.
 * You can add an object T to the Stack and take a contained Object T out. 
 * 
 * @author Student
 *
 */
 public interface Stack<T> {
 }
```

We now fill the interface with more data. We use the DBC terminology to force us to think about "what it means to use a Stack": Methods, Preconditions, Postconditions. To indicate clearly what we mean, we use the @pre, @post and @inv annotations inside our comments to refer to pre-condition, post-condition and class invariant. This is just a convention, nothing else depends on it.

```
package spl.util;
 
/**
 * Stack is an objects container. All objects are of type T. Contents is
 * ordered in Last-In-First-Out order. You can add an object T to the Stack and
 * take a contained Object T out.
 * 
 * @author Student
 * 
 */
public interface Stack<T> {
 
    /**
     * Will add the object at the top of the stack. (This is a command.)
     * 
     * @param obj
     *            any non null T object.
     * @pre: none.
     * @post: this.isEmpty() == false
     * @post: this.pop(); this == @pre(this)
     * @post: this.pop() == @param obj
     */
    void push(T obj);
 
    /**
     * Will remove the top object from the stack. (This is a command.)
     * 
     * @throws Exception
     * @pre: this.isEmpty() == false
     * @post: ???
     */
    void remove() throws Exception;
 
    /**
     * Returns the value of the top object on the stack. (This is a query.)
     * 
     * @throws Exception
     * @pre: this.isEmpty() == false
     */
    T top() throws Exception;
 
    /**
     * @return True if the Stack is empty, or False if the Stack contains at
     *         least one {@link Object}. (This is a query.)
     * @pre: none.
     * @post: none.
     */
    boolean isEmpty();
 
}
```

Note that commands always have post-conditions and queries have no post-conditions (since they do not change anything).

#### **Separate Basic Queries from Derived Queries**

Let us now consider the test for push(): using the new top() query, we can write a test that has no side-effect.

```
/**
     * Test method for {@link spl.util.Stack#push(java.lang.Object)}. 
     * 
     * @throws Exception
     */
    @Test public void testPush() throws Exception {
        Integer i1 = 1;
        Integer i2;
        stack.push(i1);
        assertFalse(stack.isEmpty());
        i2 = stack.top();
        assertEquals(i1, i2);
    }
```

Is this test strong enough? Let us think in terms of "what could go wrong in the implementation" - or in other words, assume the person writing the code we are testing has bad intentions, and tries to pass the test with minimal effort. We could pass this test if the implementation does not really store a full stack, but just a single item. Each time push is used, the "bad" implementation would replace the single item with the new value, and return it as the value of ```top()```.

If we look at further tests, we'll see that a test like ```testStackLIFO()``` introduced above will catch such an impostor. But if we play this game, we'll see that ```testStackLIFO()``` can be fooled by an implementation that remembers only 3 items. To make such a bad implementation fail our tests, we must test for 4 items. By induction, we see that this strategy will not prevail against our opponents (the bad programmer).

One way out of this predicament, is to add a new query to our stack that indicates how many elements are currently stored in the Stack. Let us call this query count().

```
package spl.util;
 
/**
 * Stack is an objects container. All objects are of type T. Contents is
 * ordered in Last-In-First-Out order. You can add an object T to the Stack and
 * take a contained Object T out.
 * 
 * @author Student
 * 
 */
public interface Stack<T> {
 
    /**
     * add the object at the top of the stack. (This is a command.)
     * 
     * @param obj
     *            any non null T object.
     * @pre: none.
     * @post: isEmpty() is false.
     * @post: count() == @pre(count()) + 1
     * @post: top() == @param obj
     */
    void push(T obj);
 
    /**
     * remove the top object from the stack. (This is a command.)
     * 
     * @throws Exception
     * @pre: isEmpty() == false
     * @post: count() == @pre(count()) - 1
     */
    void remove() throws Exception;
 
    /**
     * @return the value of the top object on the stack. (This is a query.)
     * 
     * @throws Exception
     * @pre: this.isEmpty() == false
     */
    T top() throws Exception;
 
    /**
     * @return True if the Stack is empty, or False if the Stack contains at
     *         least one {@link Object}. (This is a query.)
     * @post: @return (count()==0)
     */
    boolean isEmpty();
 
        /**
         * @return The number of items contained in the Stack. (This is a query.)
         */
        int count();
}
```

Note how the introduction of ```count()``` makes it possible to write the post-condition of the commands ```push()``` and ```remove()``` in a more natural manner. Technically, we use the ```@pre(count())``` notation to refer to the value of ```count()``` before the command was executed.

By introducing ```count()```, we have introduced a redundancy: ```isEmpty()``` is equivalent to ```count()==0```.

When we find such a case - we will say that ```isEmpty()`` is a *derived query* and ```count()``` is a *primitive query*. To document derived queries, we annotate their post-condition as being computed on the basis of a primitive query (see the ```isEmpty()``` post-condition).

**Principle 2**
**Separate basic queries and derived queries** - Derived queries can be specified in terms of basic queries.

**Principle 3**
**Define derived queries in terms of basic queries** - Define the post-conditions of derived queries in terms of basic queries only.

#### **Specify how Commands affect Basic Queries**

We are now equipped with the basic query count(). Let us consider again the test for ```push()```:

```
/**
     * Will add the object at the top of the stack. (This is a command.)
     * 
     * @param obj
     *            any non null T object.
     * @pre: none.
     * @post: isEmpty() is false.
     * @post: count() == @pre(count()) + 1
     * @post: top() == @param obj
     */
    void push(T obj);
```
    
The post-conditions of ```push()``` include both a test on ```isEmpty()``` and a test on ```count()```. We know ```isEmpty()``` is a derived query – so all post-conditions that involve ```isEmpty()``` can be removed. But we must make sure there is a post-condition that involves ```count()```.

In fact, it is a general principle: a command by definition modifies the state of the object under test. If we want to test this modification, there must be a basic query whose value is affected by the change. So when we review the post-conditions of commands, the safe way to make sure we do not "forget" an effect, is to review all the basic queries of the object interface and decide whether they are changed.

If none of the basic queries is affected by the command, we have a problem: our public interface is not detailed enough to allow us to test the object. In this case, the object interface must be extended or revised.

**Principle 4**
For each basic command, write post-conditions that specify the value of every basic queries.

Let us consider again the test for the remove() command.

```
/**
     * remove the top object from the stack. (This is a command.)
     * 
     * @throws Exception
     * @pre: isEmpty() == false
     * @post: count() == @pre(count()) - 1
     */
    void remove() throws Exception;
```

The contract as encoded in the pre and post-conditions specifies that remove() removes one element from the container. It also tells us that we cannot call this when the stack is empty. But, it does not tell us WHICH element is removed (specifically the top element).

How can we make this contract more specific? There is no other way than to introduce another basic query that will let us observe the contents of the container.

```
/**
     * Item at logical position i in the stack (This is a query.)
     * The oldest item is at position 1.
         * The newest item is at position count().
         *
     * @throws Exception
     * @pre: 1 <= i <= count()
     */
    T itemAt(int i) throws Exception;
```

Let us now redefine the whole Stack interface using this new basic query:

```
package spl.util;
 
/**
 * Stack is an objects container. All objects are of type T. Contents is
 * ordered in Last-In-First-Out order. You can add an object T to the Stack and
 * take a contained Object T out.
 * 
 * @author Student
 * 
 */
public interface Stack<T> {
 
    /**
     * add the object at the top of the stack. (This is a command.)
     * 
     * @param obj
     *            any non null T object.
     * @pre: none.
     * @post: count() == @pre(count()) + 1
     * @post: itemAt(count()) == @param obj
     */
    void push(T obj);
 
    /**
     * remove the top object from the stack. (This is a command.)
     * 
     * @throws Exception
     * @pre: isEmpty() == false
     * @post: count() == @pre(count()) - 1
     */
    void remove() throws Exception;
 
    /**
     * @return the value of the top object on the stack. (This is a derived query.)
     * 
     * @throws Exception
     * @pre: this.isEmpty() == false
         * @post: return itemAt(count())
     */
    T top() throws Exception;
 
    /**
     * @return True if the Stack is empty, or False if the Stack contains at
     *         least one {@link Object}. (This is a query.)
     * @post: @return (count()==0)
     */
    boolean isEmpty();
 
        /**
         * @return The number of items contained in the Stack. (This is a query.)
         */
        int count();
 
    /**
     * Item at logical position i in the stack (This is a query.)
     * The oldest item is at position 1.
         * The newest item is at position count().
         *
     * @throws Exception
     * @pre: 1 <= i <= count()
     */
    T itemAt(int i) throws Exception;
}
```

Did this make our post-condition for ```remove()``` stronger? We now specify that ```count()``` has been decreased by one. We do not need to specify that the last element is the one that is removed: by specifying that ```count()``` changed, we now know that the rest of itemAt for all values of i from 1 to the new ```count()``` are not affected by the command. We can, therefore, safely assume that only the last element is removed, and all other elements remain unchanged.

**Did we just break encapsulation?**
To make our Stack interface testable, we introduced a basic query ```itemAt()``` that opens up the content of the stack at any position. This seems like a drastic change. A stack is known to limit access to only the top element. Does the ```itemAt()``` query break encapsulation?

In fact, it does not. The simple proof is that, a client that has access to only ```isEmpty()```, ```push()``` and ```pop()``` can already know the value of all the elements in the stack(). It just needs to iterate:

```
while (! s.isEmpty()) {
  System.out.println(s.pop());
}
```
The introduction of ```itemAt()``` did not change the protection level given to the elements in the stack.

#### **Verify Pre-conditions in terms of Basic Queries**

Now that we have established the basic queries and verified that they are sufficient to capture the post-conditions of all the basic commands, we can revise the pre-conditions. Pre-conditions as well should be specified in terms of basic queries.

**Principle 5**
For each basic command and basic query, express the pre-conditions in terms of basic queries.

#### **Specify Class Invariants**

When we focus on each method (command or query), we think in terms of the changes introduced by the method. In addition, it is important to capture properties that remain true of the object in all legal states. We call such properties class invariants.

In our example, we can state one important class invariant:

```
// @inv count() >= 0
```

What we specified here is that the container cannot contain a "negative number of items". We should verify that the contract of the commands that affect ```count()``` cannot break this invariant. In our interface, the only candidate that could do this is ```remove()``` (since it decreases ```count()```). We verify that the pre-condition prevents ```count()``` from changing from ```0``` to ```-1```. Hence, the invariant remains enforced.

**Principle 6**
Specify class invariants that impose constraints that must remain always true on the basic queries.

### **Summary**

TDD can be summarized by the following points:
* Specify the interface of the objects you program before you implement them.
* Part of the interface specification includes the contract that each method must fulfill. This contract is expressed in terms of @pre, @post and @inv conditions.
* Write tests for the interface that verify that the contract is enforced before you * write the implementation.
* Design the interface of the objects to be testable.
* The following 6 principles ensure testability:
  1. Separate commands and queries.
  2. Separate basic queries and derived queries.
  3. Define derived queries in terms of basic queries.
  4. For each basic command, write post-conditions that specify the value of every basic queries.
  5. For each basic command and basic query, express the pre-conditions in terms of basic queries.
  6. Specify class invariants that impose constraints that must remain always true on the basic queries.

### **References**

[Design by Contract by Example](http://www.amazon.com/Design-Contract-Example-Richard-Mitchell/dp/0201634600), Richard Mitchell & Jim McKim, 2002, Addison Wesley.

[Introduction to Test Driven Design (TDD)](http://www.agiledata.org/essays/tdd.html) by Scott Ambler.

[Test Driven Development](http://en.wikipedia.org/wiki/Test-driven_development) Wikipedia.

[Design by Contract] (http://en.wikipedia.org/wiki/Design_by_contract) Wikipedia.

[Extreme Test-Driven Development with UNA (Java)](http://www.vimeo.com/1653402), a 30 min video demo of a Stack development using TDD.

[Generics](https://github.com/bguspl/bguspl.github.io/blob/main/class_material/generics.md)
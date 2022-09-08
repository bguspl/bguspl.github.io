Design by contract and Unit testing
Design By Contract (DbC)
DBC is a simple terminology that helps us express contracts between specification and implementation. It relies on three words:
preconditions : things that must be true before we invoke a method.
postconditions : things that must be true after a method is invoked.
Invariants: things that must be true both before and after any method is invoked and immediately after construction.
Rules that ensure testability
Separate commands and queries.
Separate basic queries and derived queries.
Define derived queries in terms of basic queries.
For each basic command, write post-conditions that specify the value of every basic query.
For each basic command and basic query, express the pre-conditions in terms of basic queries.
Specify class invariants that impose constraints that must remain always true on the basic queries.
JUnit
JUnit is a testing framework written by Erich Gamma and Kent Beck. It is used by developers who implement unit tests in Java. JUnit is Open Source Software.
How to get JUnit (in Maven)
If you don't have maven on your computer install it - https://maven.apache.org/download.cgi and install it - https://maven.apache.org/install.html
If you don't have a maven project - create one (you can do it from an IDE or by running in the terminal:
mvn -B archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DgroupId=com.mycompany.app \
  -DartifactId=my-app
notice that "com.mycompany.app" and "my-app" are parameters you can change as you wish), this will create your project with junit.
Add/Update the following dependency in your "pom.xml" -
download
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.6.0-M1</version>
    <scope>test</scope>
</dependency>
Testing a Stack
In this section we are going to use the new concepts we learn in order to implement the following interface
downloadtoggle
72 lines ...
package spl.util;
 
import java.util.function.Function;
import java.util.function.Predicate;
 
/**
 * Stack is an objects container. All objects are of type E. Contents is
 * ordered in Last-In-First-Out order. You can add an object of type T to the
 * Stack and
 * take a contained Object of type T out.
 * <p>
 * @author Student
 * <p>
 */
public interface Stack<E> {
 
    /**
     * add {@code element} to the top of the stack.
     * {@code element} must not be null.
     * <p>
     * @param element
     */
    void push(E element);
 
    /**
     * retrieve and remove the top of the stack.
     * will throw {@link IllegalStateException} if the stack is empty
     * <p>
     * @return the element at the top of the stack
     */
    E pop();
 
    /**
     * @return the amount of elements currently in the stack
     */
    int size();
 
    /**
     * @return true if the stack does not contains any element and false
     *         otherwise.
     */
    boolean isEmpty();
 
    /**
     * pop elements from the stack until the popped element {@code e} satisfying
     * {@code predicate.test(e) == true}
     * <p>
     * @param predicate a tester for the stack elements
     * @return the first element satisfying {@code predicate.test(e) == true} or
     *         null if no such element found
     */
    E popUntil(Predicate<E> predicate);
 
    /**
     * creates a new stack which contains the results of applying the given
     * {@code function} to each of the elements in this stack.
     * this stack is not affected by this call.
     * <p>
     * for example, for a stack {@code s} which contains the integers
     * {@code 1, 2, 3} the following code will produce a stack of doubles which
     * contains the doubles {@code 1, 1/2, 1/3}:
     * {@code s.map((Integer i) -> 1.0/i);}
     * <p>
     * note that after running this code - {@code s} will still contain
     * {@code 1, 2, 3}.
     * <p>
     * @param function the function to apply on each stack element
     * @return a new {@code Stack<T>} which contains the elements returned by
     *         {@code function}.
     */
    <T> Stack<T> map(Function<E, T> function);
 
}
Step-by-step (START)

Read the javadoc of the Function, Predicate and the supplied Stack interfaces and make sure you understand what all the methods should do.
Create a new maven project as described above (see "How to get JUnit") with artifactId - practice5 and groupId - spl.util
Create a new interface named Stack under spl.util (copy/paste the code above).
Create a new class: StackImpl that implements Stack.
Implement the stack methods, if there are some methods that the Stack can contain as default methods modify the Stack interface to contain them.
Create a main class and test every method at least once, use both lambda functions and anonymous classes in your tests.

Step-by-step (END)
A simple (but inefficient) solution:

downloadtoggle
78 lines ...
public interface Stack<E> {
    void push(E element);
    E pop();
    int size();
 
    default boolean isEmpty() {
        return size() == 0;
    }
 
    @SuppressWarnings("empty-statement")
    default E popUntil(Predicate<E> predicate) {
        E result = null;
        while (!isEmpty() && !predicate.test(result = pop()));
        return result;
    }
 
    <T> Stack<T> map(Function<E, T> function);
}
 
public class StackImpl<E> implements Stack<E> {
 
    private static final class Link<E> {
        E item;
        Link<E> next;
 
        Link(E item, Link<E> next) {
            this.item = item;
            this.next = next;
        }
        
        <T> void map(Stack<T> into, Function<E, T> mapper) {
            if (next != null) {
                next.map(into, mapper);
            }
                
            into.push(mapper.apply(item));
        }
    }
 
    private Link<E> top = null;
    private int size = 0;
 
    @Override
    public void push(E element) {
        if (element == null) {
            throw new IllegalArgumentException("null");
        }
        top = new Link<>(element, top);
        size++;
    }
 
    @Override
    public E pop() {
        if (isEmpty()) {
            throw new IllegalStateException("empty stack");
        }
 
        E result = top.item;
        top = top.next;
        size--;
        return result;
    }
 
    @Override
    public int size() {
        return size;
    }
 
    @Override
    public <T> Stack<T> map(Function<E, T> function) {
        Stack<T> result = new StackImpl<>();
        if (top != null) {
            top.map(result, function);
        }
            
        return result;
    }
 
}
We illustrate the usage of JUnit in Maven on the stack example.
We show the use of JUnit, and we show the work process you need to follow.

Step-by-step (START)

Add JUnit to your project using the instructions above (see "How to get JUnit").
Add the following section to your pom.xml so we can use java 8 syntax and junit -
downloadtoggle
16 lines ...
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.1</version>
        </plugin>
    </plugins>
  </build>
Before writing the actual tests, we will learn a bit about Junit and naming conventions.

Naming Conventions
Test classes: A test class for spl.util.Node will be named spl.util.NodeTest
Why? Same package - so all package members will be accessible, like for Node. Same "first name" - so if you write Node and click ctrl+space (auto-complete) you will get all Node related classes, such as the test.
Test methods: the method public int add(int a, int b); should have a test method: public void testAdd();
All JUnit test methods are public, un-typed (void) and without parameters.
The tag @Test indicates that the method is a test method.

Many tutorials on the web say that test methods should start with a word "test". This is a leftover from older versions of Junit, and it is not true anymore.

In older versions of Junit, the @Test tag was not available. Instead, the test methods were recognized as such by having their names start with "test", for example testIsEmpty(). This is not longer required, as the test methods are identified by the @Test tag. You can still use names starting with "test" if you want.
Assert functions
JUnit passes or fails a test depending on the result of the class Assert methods. For example, the method assertEquals(int, int) will fail the test if both integers aren't equal. You may choose to write your own methods such as assertEquals(BigObject, BigObject). For more info about asserts see https://github.com/junit-team/junit/wiki/Assertions
JUnit Test Execution Cycle (Behind the Scenes)
Create an instance of the class.
The class must have a public default constructor.
For each method that fits: @Test public void _____ () { … }
Run a method with the tag @BeforeEach (e.g., setUp()) - preparing the conditions for the test.
Run the method _____ ()
Display test results
Run a method with the tag @AfterEach (e.g., tearDown()) - returning the system to normal functionality.

An example for important use of setUp and tearDown: your OUT was badly written and uses System.in. So the setUp will replace System.in input-stream with a mockup and the tearDown will restore the standard input-stream.
Now that we know a bit about JUnit, let's start by writing our tests. We are going to keep the tests and the source code as in the maven structure convention -

practice5
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- spl
    |           `-- util
    |               `-- Node.java
    `-- test
    |   `-- java
    |       `-- spl
    |           `-- util
                    `-- NodeTest.java
. If you prefer to put the tests and the code in a different structure it is possible, but we give the instructions for two source folders.
Add a class file to src/test/java/spl/util named StackImplTest.java
Package: spl.util
Create a class with the name: StackImplTest (by convention, test classes are called [OriginalClass]Test)
StackImplTest needs to inherit from the class TestCase (
download
public class StackImplTest extends TestCase
)
create "@BeforeEach public void setUp()" and "@AfterEach public void tearDown()"
Create an empty test function "@Test public void testIsEmpty()"
Finish
Now let's implement the test.

The first thing we need is an actual object to test. In our case, an implementation of a Stack.

So, let's add to our test class a member of type Stack

download
Stack<Integer> stack;
We want to have a fresh new empty stack created before each test. This is what the @BeforeEach method is used for.

Add the following to the body of the setUp() method:

download
this.stack = new StackImpl<Integer>();
Now we can actually write some tests!

We will start with testing the isEmpty function.

We expect a newly created stack to be empty, so let's express it in a test:

Create the method:
download
@Test
public void testIsEmpty(){
    assertEquals(true, stack.isEmpty());
}
Run the test:
Simply run mvn clean test
If we see a red line it means that the test failed! in such case we need to go and fix our code.

Now you need to test the rest of the stack functions.

At some point, you will see that you have a problem testing pop(): it both changes the state and removes a value.

This means our interface is not very good – we do not separate queries from commands.

Change the interface to include public void remove() and public T top() in addition to pop(), and implement pop() using remove and top.

Similarly, add T elementAt(int i) function to the interface to help testing.

At this point you may find it useful to define pre- and post- conditions to your interface, so you would know how to write your tests better.

Go ahead and do that. Remember:

For each basic command, write post-conditions that specify the value of every basic queries.
For each basic command and basic query, express the pre-conditions in terms of basic queries.

Step-by-step (END)
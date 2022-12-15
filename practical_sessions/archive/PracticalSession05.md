Generics, Lambdas and Reflection
This practical session is divided into three main parts.
Packages (general information)
Generics, Lambdas and Default methods.
Reflection, Class object.
Part 1: Java Packages
Please follow the explanations in the Java Packages page.
Part 2: Generics and Lambdas
Java generics is a mechanism for creating objects which can be parameterized by a type. Before attempting to explain further how to use and define generics we will first learn about their most common use case - collections.
Collections are objects that holds other objects in some manner, a List, Map and Set are example of collections.

Java platform provides many ready to use collection classes. We are not going to go into them in class, you should learn on your own.

At the minimum, look at the List and Map interfaces, and at ArrayList and HashMap classes.

For more in-depth material, start from here: Oracle's Collections Trail.

Arrays vs. Collections
In Intro to CS, you worked a lot with arrays, There are other, more convenient alternatives.
Unless you need to write really optimized code, there is no reason to use arrays. Instead, use Java's collection classes. There are many useful classes you should know that come builtin with java.

Generic Classes And Interfaces
In the following section we are going to look at the decleration of one of the simplest known collections - the stack. You want your stack to hold stuff, and you want all the stuff to be of the same type. One option is to specify in advance what type you are going to use:
So here is a Stack of Integers:

download
interface StackOfIntegers {
    void push(Integer i);
    Integer pop();
    int size();
}
and here is a Stack of Cows:

download
interface StackOfCows {
    void push(Cow i);
    Cow pop();
    int size();
}
This of course leads to a lot of code duplication! The implementation of StackOfCows and StackOfIntegers is the same, aside from the type of the held object.

In order to solve this issue one thing you can do is to use Object, the most general type, which can then hold anything:

download
interface StackOfEverything {
    void push(Object i);
    Object pop();
    int size();
}
But usually, you want your stack to hold just one type of things (only Cows, only Integers, etc). You could have your StackOfEverything holding Objects and be careful, but someday you or one of your fellow programmers are going to make a mistake and have both Cows and Integers in the same Stack. When this will happen the compiler will not be able to warn you about it.

You want both type-safety and a general stack. The way to achieve it is using generics.

download
interface Stack<E> {
    void push(E something);
    E pop();
    int size();
}
You now defined a Stack holding objects of type E. E is not a real type, just a parameter - you can think of it as a place holder for objects (any type extending Object - i.e., non-primitives).

Then, in your code you can create type-specific stacks:

downloadtoggle
11 lines ...
public class Example {
 
    public static void main(String[] args) {
    
      Stack<Cow> stackOfCows = new StackImpl<Cow>();
      Stack<Float> stackOfFloat = new StackImpl<Float>();
    
     stackOfFloat.push(5f); // ok
     stackOfCows.push(new Cow()); // ok
     stackOfFloat.push(new Cow()); // compilation error!
    }
}
Note that the type Float is not a primitive type but a wrapper to the primitive float, in java, generics only applicable to non-primitive types therefore for every primitive type there is a corresponding "wrapper type" that can be used when the context requires that (you can read more about the available wrapper types here.)

Default Methods In Interfaces
A default method is a method which implemented directly in an interface, it provides default implementation for the implementations of that interface.
To understand why this is needed lets revise our Stack interface and add an isEmpty method to it.

download
interface Stack<E> {
    void push(E something);
    E pop();
    int size();
    boolean isEmpty();
}
A straight forward implementation of the isEmpty method can be

download
public class SomeStackImplementaion<E> implements Stack<E> {
  ... //implementations of pop, push and size
 
  public boolean isEmpty() {
    return size() == 0;
  }
}
This implementation should suite for most of the implementations for Stack. In addition, this implementation only requires calling methods that was already defined in the Stack interface. Most Stack implementation will just duplicate that code. in order to avoid that we can provide a default implementation directly on the interface:

download
interface Stack<E> {
    void push(E something);
    E pop();
    int size();
    default boolean isEmpty() { return size() == 0; }
}
Now, implementating classes do not have to provide their own implementation of the isEmpty method, although they can choose to do so.

Abstract classes vs. interfaces
Another solution to the previous problem is to declare an abstract AbstractStack class which implements the Stack interface and the method isEmpty. Since you can only derive from one parent class in Java, doing so will enforces any implementations of Stack that wants to benefit from the default implementations to derive from your class (or any class that derive from your class) - this in some occasions can be very constraining. Since in Java a class can implements any number of interfaces - the default method solution does not enforce such constraints.
You can read more about default methods here and here.

Generic Methods
In addition to generic classes and interfaces Java also supports generic methods.
Assume that we want to create a utility method for finding the largest item in an array. Like before we can have arrays of Integers, Cows, and more.

Here is how a cow class looks like:

downloadtoggle
11 lines ...
public class Cow {
  private int age;
  private int weight;
 
  public Cow(int age, int weight) {
    this.age = age;
    this.weight = weight;
  }
 
  public int getAge() { return age; }
  public int getWeight() { return weight; }
}
Given an array of Cow objects, what is the maximal object? Is it the heaviest cow? Maybe the oldest? It depends on how you want to compare the cows. Jave defines a generic interface called Comparator which contains a single (non-default) method called "compare" with the following signature:

download
public interface Comparator<T> {
        ...
        int compare(T o1, T o2);
        ...
    }
Given 2 objects of type T: o1,o2 the implementation of the method compare is assumed to return:

any non-zero negative number if o1 is the smallest between the two
any non-zero positive number if o2 is the smallest between the two
0 if o1 and o2 are equals

Given the above, The following is a generic method that finds the maximal element in a given array using the ordering enforced by the given comparator.
downloadtoggle
15 lines ...
public class Example2 {
        public static <T> T max(T[] array, Comparator<T> comparator) {
            if (array.length == 0) {
                throw new IllegalArgumentException("empty array");
            }
         
            int maxIndex=0;
            for (int i=1; i<array.length; i++) {
                if (comparator.compare(array[maxIndex], array[i]) < 0) {
                    maxIndex = i;
                }
            }
        
            return array[maxIndex];
        }
    }
Note the <T> in front of the method signature which parametrize the method. The following is a usage example for this method:

downloadtoggle
22 lines ...
public class Example2 {
        public static <T> T max(T[] array, Comparator<T> comparator){...}
 
        public static class CowComparatorByAge implements Comparator<Cow> {
            public int compare(Cow o1, Cow o2) {
                 return o1.getAge() - o2.getAge();
            }
        }
 
        public static class IntComparator implements Comparator<Integer> {
            public int compare(Integer o1, Integer o2) {
                 return o1 - o2;
            }
        }
 
        public static void main(String[] args) {
            Integer[] ints = {1,4,3};
            Cow[] cows = {new Cow(7,50), new Cow(9,200), new Cow(3,100)};
 
            System.out.println(max(ints, new IntComparator())); // prints 4
            System.out.println(max(cows, new CowComparatorByAge()).getAge()); // prints 9
        }
    }
In the example above we implemented the CowComparatorByAge so that we can get the oldest cow. If we wanted to, we could get the heaviest cow using the following comparator:

download
public class CowComparatorByWeight implements Comparator<Cow> {
  public int compare(Cow o1, Cow o2) {
      return o1.getWeight() - o2.getWeight();
  }
}
Anonymous classes and Lambdas
Anonymous classes enable you to make your code more concise. They enable you to declare and instantiate a class at the same time. They are like local classes except that they do not have a name. Use them if you need to use a local class only once.
In the previous example we implemented two classes IntComparator and CowComparatorByAge that we can use later in our max method. The following has the same output but uses anonymous classes instead.

downloadtoggle
20 lines ...
public class Example2 {
    public static <T> T max(T[] array, Comparator<T> comparator){...}
 
    public static void main(String[] args) {
        Integer[] ints = {1,4,3};
        Cow[] cows = {new Cow(7,50), new Cow(9,200), new Cow(3,100)};
 
        System.out.println(max(ints, new Comparator<Integer>(){
            public int compare(Integer o1, Integer o2) {
                 return o1 - o2;
            }
        })); // prints 4
 
        System.out.println(max(cows, new Comparator<Cow>(){
            public int compare(Cow o1, Cow o2) {
                 return o1.getAge() - o2.getAge();
            }
        }).getAge()); // prints 9
 
    }
}
Usually anonymous classes are very simple. E.g., in the above example our anonymous class contains only one simple method. Actually, we wanted to to pass functionality as an argument to another method. Lambda expressions enable you to do so in a more concise and aesthetic way.

The following is a version of the previous example which uses lambdas instead of anonymous classes.

downloadtoggle
15 lines ...
public class Example2 {
    public static <T> T max(T[] array, Comparator<T> comparator){...}
 
    public static void main(String[] args) {
        Integer[] ints = {1,4,3};
        Cow[] cows = {new Cow(7,50), new Cow(9,200), new Cow(3,100)};
 
        System.out.println(max(ints, (Integer o1, Integer o2) -> {
            return o1 - o2;
        })); // prints 4
 
        System.out.println(max(cows, (Cow o1, Cow o2) -> o1.getAge() - o2.getAge()).getAge()); // prints 9
 
        
    }
}
Notes about lambdas:

the lambda syntax is (args...) -> { body }
You can replace anonymous class with lambda only if the interface you are implementing has a single non default method - in that case the lambda signature must be the same as that non-default method
If all that your lambda does is returning a value (e.g., the body of your lambda looks like this: (args...) -> {return ...;}) then you can write only the return value as the body of the lambda, e.g., (args...) -> ... (see in the previous example the max over cows) otherwise you will have to write a full body (see in the previous example the max over ints)
There are more features of lambda which you can read more about here, here and here.
Lambda Thoroughly Explained
Lambda Syntax Rules
Optional type declaration
No need to declare parameter types - the compiler can inference them from the value of the parameter.
Optional parenthesis around parameter
No need to declare a single parameter in parenthesis.
For multiple parameters, parentheses are required.
Optional curly braces
No need to use curly braces in expression body if the body contains a single statement.
Optional return keyword
The compiler automatically returns the value if the body has a single expression to return the value.
Otherwise, curly braces are required to indicate that expression returns a value.
Labmda Example
downloadtoggle
36 lines ...
public class Tester{ 
    public static void main(String args[]){ 
        // with type declaration on the parameters
        MathOperation addition = (int a, int b) -> a + b; 
        // without type declaration on the parameters
        MathOperation subtraction = (a, b) -> a - b; 
        // with return statement along with curly braces 
        MathOperation multiplication = (int a, int b) -> { return a * b; }; 
        // without return statement and without curly braces
        MathOperation division = (int a, int b) -> a / b; 
 
        System.out.println("10 + 5 = " + Tester.operate(10, 5, addition)); 
        System.out.println("10 - 5 = " + Tester.operate(10, 5, subtraction)); 
        System.out.println("10 x 5 = " + Tester.operate(10, 5, multiplication)); 
        System.out.println("10 / 5 = " + Tester.operate(10, 5, division)); 
        
        // without parenthesis around the parameter
        GreetingService greetService1 = message -> System.out.println("Hello " + message); 
        // with parenthesis around the parameter
        GreetingService greetService2 = (message) -> System.out.println("Hello " + message); 
 
        greetService1.sayMessage("Hi"); 
        greetService2.sayMessage("Hi there!"); 
    } 
 
    interface MathOperation { 
        int operation(int a, int b); 
    } 
 
    interface GreetingService { 
        void sayMessage(String message); 
    } 
 
    private static int operate(int a, int b, MathOperation mathOperation){ 
        return mathOperation.operation(a, b); 
    }
 }
Class Object
In Java, there is a special class called Class. Instances of the class Class represents classes and interfaces in a running Java application. Class has no public constructor. Instead, Class objects are constructed automatically by the Java Virtual Machine. Every class in Java (and even arrays, primitives and the keyword void) has a corresponding (a.k.a., reflecting) Class object that can be received at runtime using one of:
The static "class" property. For example: for a class A, A.class returns an object of the type Class.
The method getClass(). Example: for an object a of type A, a.getClass() returns an object of the type Class.

The Class object has several interesting methods. For example, it has a getName() method which returns the corresponding class name. One important method of the class Class is the isAssignableFrom(Class) method. This method returns true whenever the calling Class represents a superclass or superinterface of the class represented by the given Class object. For example:
downloadtoggle
22 lines ...
class A {}
class B extends A {}
class C extends B {}
 
public class Main {
 
    public static void main(String[] args) {
        A a = new A();
        B b = new B();
        C c = new C();
 
        System.out.println(
                A.class.isAssignableFrom(B.class)); //true
        System.out.println(
                a.getClass().isAssignableFrom(C.class)); //true
        System.out.println(
                B.class.isAssignableFrom(c.getClass())); //true
        System.out.println(
                b.getClass().isAssignableFrom(a.getClass())); //false
        System.out.println(
                C.class.isAssignableFrom(B.class)); //false
    }
}
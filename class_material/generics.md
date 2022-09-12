# **Type Safety**

[^UntypedLanguages]:In many programming languages, notably scripting languages, variables are untyped. The type of each variable is determined dynamically, at runtime. In other words, the same variable can contain at different times values of different types. This is the case for example, for Perl, Python and Scheme.
Java is a statically typed language, meaning that the type of each variable is known at compilation time.

Type safety[^UntypedLanguages]S is about making sure the Object referenced by a variable is indeed of a type compatible with the type of the variable, i.e., it is of the same class, of a sub-class or the type is an interface and the object implements it. In general, the compiler verifies type safety for us. If we try to assign a string value to an integer variable, the compiler will flag this as an error. There are, however, language constructs which allow the programmer to bend the rules of type safety a little bit. These constructs are necessary to enable the programmer to write more generic code. You have encountered such constructs in the past, whenever you wrote code like this:

```java
SomeObject so = getAnObject();
if (so instanceof SomeClass){
   SomeClass sc = (SomeClass) so;
   sc.aMethod();
}
```

Another example you have encountered is when using standard Java containers, such as ```ArrrayList```. Consider an ```ArrayList``` of integers:

```java
List l = new ArrayList();
l.add(2); //auto boxing primitive int to Integer
l.add(false); //auto boxing primitive boolean to Boolean
Object o = l.get(0); //getting 2 (of type Integer)
```

Now, since both the ```get()``` and ```add()``` methods of the class ```List``` work with, and return, object of the class ```Object``` (which is the super class of every class in Java), we can never be sure, at **compilation time**, what is the type of ```o```. The reason why the interface of the List collection operate over ```Object```s is to make them as reusable as possible: we can make a ```List``` of any object. The cost is that we lose in type safety: we can now have Lists of apples and oranges, the compiler will not help us check what we intend: we would like a List of only apples or only oranges.

## **Generics**
Starting from java 1.5, generics were introduced. The idea of generics is similar to C++ templates but much simpler and less powerful. Traditionally java takes ideas from C++ and simplifies them - providing a less powerful but much more easy to use alternatives, one additional example is the fact that while java supports inheritance, it only support single direct ancestor which is easier to use but less powerful than the multiple inheritance mechanism of C++. Like in C++ Java's generics can be defined on methods, classes and interfaces.
Lets first examine a simple example of a generic pair class

```java
public class Pair<X,Y> {
  private X x;
  private Y y;
 
  public Pair(X x, Y y) { this.x = x; this.y = y; }
 
  public X getX() { return x; }
 
  public Y getY() { return y; }
 
  public static void main(String[] args) {
    Pair<Integer, Integer> intPair = new Pair<>(1, 2);
    Pair<String, String> stringPair = new Pair<>("three", "four");
    Pair<String, Integer> mixedPair = new Pair<>("five", 6);
    
    Integer a = intPair.getX(); //OK!
    String b = intPair.getX(); //Compile time error!
    String c = stringPair.getX(); //OK!
  }
 
}
```

Some notes about the above code:

* ```Integer``` is a *reference type* (*i.e.*, a non primitive type) that wraps/boxes the primitive integer type, there is such wrapper type for each primitive type in java.
* Unlike C++ templates, generics in java can only receive reference types (i.e., non primitive types) as arguments, therefore it is impossible to create ```Pair<int, int>``` and we must use the wrapper types
* Java automatically perform boxing and unboxing from wrapper types back and from primitive types for us. One should note though that this is a relatively costly operation
* Most of the container types in the standard java collection framework use generics e.g., ```ArrayList<T>```, ```HashMap<K,V>```, etc.

And now lets see how one can define a generic method

```java
public class Example {
    public static <T extends Comparable<T>> T argmax(T... array) { //variadic function - T... is the same as T[]
      if (array == null || array.length == 0) {
        throw new IllegalArgumentException("array must be non null and with at least one element");
      }
 
      int maxIndex = 0;
      for (int i=1; i<array.length; i++) {
        if (array[i].compareTo(array[maxIndex]) > 0) {
          maxIndex = i;
        }
      }
 
      return array[maxIndex];
    }
  }
 
  public static void main(String[] args) {
 
    //note that since argmax is variadic - the following:
    System.out.println(argmax(1,2,3)); //prints 3
    
    //is the same as:
    System.out.println(argmax(new Integer[]{1,2,3})); //prints 3
 
    System.out.println(argmax("a","c","b")); // prints c
}
```

Some notes about the above code:

* A generic type argument can have a lower bound type defined using the extend keyword (and though less used it can also define an upper bound but this is out of the scope of this lecture)
* ```Comparable``` is an interface which defines a single method ```int compareTo(T arg)``` which returns an int value ```x``` s.t. ```x=0``` if ```this == arg```, ```x < 0``` if ```this < arg``` and ```x > 0``` if ```this > arg```.
* All java's wrapper types implements this interface. In addition, ```String``` also implement it by using a lexicographic order

### **Generic limitations**

In addition to not being able to work on primitive types, Generics are "erased" at runtime which means that while at complie time the compiler knows that ```List<Integer>``` is not a ```List<String>``` at run time, both of them looks like ```List<Object>```. This limitation can be seen in the following code:

```java
public class Example<X,Y> {
    void take(X x) {...}
    void take(Y y) {...}
}
```
The above code will not compile as the compiler knows that at runtime the defined methods has the same signature. Another limitation is that arrays of generic types cannot be instantiated directly i.e., the following will not compile:

```java
Pair<Integer, Integer>[] pairs = new Pair<>[10];
```
Instead you can work around this limitation using the following code:
```java
Pair<Integer, Integer>[] pairs = (Pair<Integer, Integer>) new Pair<?,?>[10];
```
which essentially means that pairs is an array of Pair without any additional generic type information. You will learn more about wildcards (the ```?``` in ```Pair<?,?>```) at the practical sessions.

For more info on generics, and how to write your own generic class, see this [Java Technical Article on Generics](http://java.sun.com/developer/technicalArticles/J2SE/generics/) and this [Generics tutorial](http://www.cs.bgu.ac.il/~spl081/Generics).

## **Default Methods In Interfaces**

A default method is a method which implemented directly in an interface, it provides default implementation for the implementations of that interface.
To understand why this is needed lets look at the following ```Stack``` interface:

```java
interface Stack<E> {
    void push(E something);
    E pop();
    int size();
    boolean isEmpty();
}
```

A straight forward implementation of the isEmpty method can be:

```java
public class SomeStackImplementaion<E> implements Stack<E> {
  ... //implementations of pop, push and size
 
  public boolean isEmpty() {
    return size() == 0;
  }
}
```

this implementation should suite for most of the implementations for ```Stack```, in addition, this implementation only requires calling methods that was already defined in the Stack interface. Most Stack implementation will just duplicate that code. in order to avoid that we can provide a default implementation directly on the interface:

```java
interface Stack<E> {
    void push(E something);
    E pop();
    int size();
    default boolean isEmpty() { return size() == 0; }
}
```

Now, implementations does not have to provide their own version of the ```isEmpty``` method, although they may choose to do so if they like.

### **Abstract classes vs. interfaces**

Another solution to the previous problem is to declare an abstract AbstractStack class which implements the Stack interface and the method isEmpty. Since you can only derive from one parent class in Java, doing so will enforces any implementations of Stack that wants to benefit from the default implementations to derive from your class (or any class that derive from your class) - this in some occasions can be very constraining. Since in Java a class can implements any number of interfaces - the default method solution does not enforce such constraints.

You can read more about default methods [here](https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html) and [here](https://dzone.com/articles/interface-default-methods-java).

## **Anonymous classes and Lambdas**

*Anonymous classes* enable you to make your code more concise. They enable you to declare and instantiate a class at the same time. They are like local classes except that they do not have a name. Use them if you need to use a local class only once.
In the previous example we saw an implementation of argmax that receives a Comparable items. Some classes like our Pair class are not Comparable, In addition, some classes may have more than one way to be compared, e.g., we can compare String objects in reverse lexicographicaly or by their length only. For these cases Java defines the interface ```Comparator<T>``` which defines a single function ```int compare(T a, T b)``` and works similar to the ```Comparable<T>``` interface.

We can therefore, change our ```argmax``` to the following method

```java
public class Example {
    public static <T> T argmax(T[] array, Comparator<T> cmp) { 
      if (array == null || array.length == 0) {
        throw new IllegalArgumentException("array must be non null and with at least one element");
      }
 
      int maxIndex = 0;
      for (int i=1; i<array.length; i++) {
        if (cmp.compare(array[i], array[maxIndex]) > 0) {
          maxIndex = i;
        }
      }
 
      return array[maxIndex];
    }
}
```

we can now define many different comparators for String for example and use them as follows:

```java
public class Example {
    //argmax code here... 
 
    public static class StringCompareByLength implements Comparator<String> {
      public int compare(String s1, String s2) { return s1.length() - s2.length(); }
    }
 
    public static class StringComparatorByReverseLex implements Comparator<String> {
      public int compare(String s1, String s2) { return s2.compareTo(s1); }
    }
 
    public static void main(String[] args) {  
      String[] strings = new String[]{"the", "cake", "is", "a", "lie"};
      argmax(strings, new StringCompareByLength()); //return "cake"
      argmax(strings, new StringComparatorByReverseLex()); //return "a"
    }
 
}
```

The following has the same output but uses anonymous classes instead.

```java
public class Example2 {
  public static void main(String[] args) {  
      String[] strings = new String[]{"the", "cake", "is", "a", "lie"};
      argmax(strings, new Comparator<String>() {
        public int compare(String s1, String s2) { return s1.length() - s2.length(); }
      }); //return "cake"
 
      argmax(strings, new Comparator<String>() {
        public int compare(String s1, String s2) { return s2.compareTo(s1); }
      }); //return "a"
  } 
}
```

One issue with anonymous classes is that if the implementation of your anonymous class is very simple, such as an interface that contains only one method, then the syntax of anonymous classes may seem unwieldy and unclear. In these cases, you're usually trying to pass functionality as an argument to another method, such as with our Comparator example. Lambda expressions enable you to do this in a more concise and aesthetic way.

The following is a version of the previous example which uses lambdas instead of anonymous classes.

```java
public class Example2 {
  public static void main(String[] args) {  
      String[] strings = new String[]{"the", "cake", "is", "a", "lie"};
 
      argmax(strings, (s1,s2) ->  s1.length() - s2.length()); //return "cake"
 
      argmax(strings, (s1,s2) ->  s2.compareTo(s1)); //return "a"
  }
}
```
Notes about lambdas:

* the lambda syntax is ```(args...) -> { body }```
* You can replace anonymous class with lambda only if the interface you are implementing has a single non default method - in that case the lambda signature must be the same as that non-default method
* If all your lambda does is return a value (e.g., the body of your lambda looks like this: ```(args...) -> {return ...;}``` than you can write only the return value as the body of the lambda, e.g., ```(args...) -> ...``` otherwise you will have to write a full body
* If all your lambda does is to take its arguments and pass them to another function ```foo``` of object ```x``` you can write ```x::foo``` instead (which is also called method reference)

There are more features of lambda which you can read more about [here](http://viralpatel.net/blogs/lambda-expressions-java-tutorial/), [here](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html) and [here](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html).

## **Reflection**

*Reflection* is commonly used by programs which require the ability to examine or modify the runtime behavior of applications running in the Java virtual machine. This is a relatively advanced feature and should be used only by developers who have a strong grasp of the fundamentals of the language. With that caveat in mind, reflection is a powerful technique and can enable applications to perform operations which would otherwise be impossible.

### **The Class Object**

In Java, there is a special class called Class. Instances of the class ```Class``` represents classes and interfaces in a running Java application. ```Class``` has no public constructor. Instead, ```Class``` objects are constructed automatically by the Java Virtual Machine. Every class in Java (and even arrays, primitives and the keyword void) has a corresponding (a.k.a., reflecting) ```Class``` object that can be received at runtime using one of:
* The static "class" property. For example: for a class ```A```, ```A.class``` returns an object of the type Class.
* The method ```getClass()```. Example: for an object ```a``` of type ```A```, ```a.getClass()``` returs an object of the type Class.

The Class object has several interesting methods. For example, it has a ```getName()``` method which returns the corresponding class name. One important method of the class ```Class``` is the ```isAssignableFrom(Class)``` method. This method returns true whenever the calling Class represents a superclass or superinterface of the class represented by the given Class object. For example:

```java
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
```

Other ```Class``` methods includes getting all the interfaces that the class implements, all the methods and fields it defines, etc. For example, the following method prints a simple string representation of any given object

```java
class A {
  private int aInt;
  private String aString;
}
 
class B extends A {
  private double bDouble;
}
 
class Example {
  public static void print(Class c) {
    System.out.println("class: " + c.getName());
 
    if (c.getDeclaredFields().length > 0) {
      System.out.println("fields: ");
      for (Field f : c.getDeclaredFields()) {
        System.out.println("\t" + f.getName() + ": " + f.getType());
      }
    }
 
    if (c.getDeclaredMethods().length > 0) {
      System.out.println("methods: ");
      for (Method m : c.getDeclaredMethods()) {
        System.out.println("\t" + m.getName() + ": " + m.toString());
      }
    }
 
    if (c.getSuperclass() != null) {
      System.out.print("inherits from ");
      print(c.getSuperclass());
    }
  }
 
  public static void main(String[] args) {
    print(B.class);
  }
}
```
This code will result in the following output:

```
class: B
fields: 
	bDouble: double
inherits from class: A
fields: 
	aInt: int
	aString: class java.lang.String
inherits from class: java.lang.Object
methods: 
	finalize: protected void java.lang.Object.finalize() throws java.lang.Throwable
	wait: public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
	wait: public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
	wait: public final void java.lang.Object.wait() throws java.lang.InterruptedException
	equals: public boolean java.lang.Object.equals(java.lang.Object)
	toString: public java.lang.String java.lang.Object.toString()
	hashCode: public native int java.lang.Object.hashCode()
	getClass: public final native java.lang.Class java.lang.Object.getClass()
	clone: protected native java.lang.Object java.lang.Object.clone() throws java.lang.CloneNotSupportedException
	registerNatives: private static native void java.lang.Object.registerNatives()
	notify: public final native void java.lang.Object.notify()
	notifyAll: public final native void java.lang.Object.notifyAll()
```
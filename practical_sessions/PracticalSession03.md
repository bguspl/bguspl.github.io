# Practical Session #3: C++ Classes and Objects



## 1. Goal

The goal for this practical session is to explain Object Oriented programming in C++ with emphasis on classes that hold resources - specifically, classes with pointer member variables.

We will learn basic syntax definitions and explore the differences between C++ conventions and Java conventions.  



## 2. Function Parameters and Return Values



### 2.1. Function Parameters

Passing parameters (a.k.a. arguments) correctly is important to avoid bugs and generate more efficient code.

It is done in C++ in 2 ways:

1. Passing by value

2. Passing by reference

   

#### 2.1.1. Passing parameters by value<sup>**†**</sup>

When passing parameters by value changes made to the parameter inside the function are not reflected to the caller. 

Usually, when passing primitive variables this method is used since there is little to no performance benefit for passing primitives by reference.

Passing by value also works with objects, in which case a call to the class's copy constructor will happen which impacts performance. Usually, unless a new copy of the object is really needed, passing objects by value is not recommended<sup>‡</sup>.

<small><sup>**†**</sup> This method is also called *call by value*.</small>

<small><sup>‡</sup> Since C++11, when using [move semantics](https://stackoverflow.com/q/3106110/2375105), passing by value *is* in fact the recommended method.</small>



Example:

```c++
int power(int i)
{
    i = i * i;
    return i;
}

int main(int argc, char **argv)
{
    int number = 5;
    power(number); // here number (a primitive integer variable) is passed by value
}
```



> Note: C++ arrays are never passed by value unless wrapped inside a struct or class.



#### 2.1.2. Passing parameters by reference<sup>**†**</sup>

The syntax of passing parameters by reference is similar to passing by value, except of the & (ampersand) before the variable name, which indicate that the variable is actually a reference (a "link") to another variable. Changes made to the variable inside the function are reflected to the caller.

References to objects must be initialized (there is no "null reference").

Passing parameters this way to a function emphasizes that the owner of the object is the caller (so there is no confusion as to who is responsible for freeing the memory). 

Example:

```c++
void emphasize(std::string &word)
{
    word.append(3, '!');
}

int main(int argc, char **argv)
{
    std::string mySentence = "Everything is awesome";
    emphasize(mySentence);
}
```

In the example, any change in to the variable _word_ will also apply to _mySentence_.

<small><sup>**†**</sup> This method is also called *call by reference*.</small>



#### 2.1.3. Passing pointers

To avoid passing objects by value you can also pass a pointer to the object. The pointer itself can be either passed by value or by reference (although passing a pointer by reference is only useful if you want to change the value of the pointer variable of the caller, or you need the pointer variable's address for some reason).

When passing a pointer we obtain a reference to the object but can also pass `nullptr` as an "uninitialized reference". 

Example:

```c++
void emphasize(std::string *word)
{
    if (word == nullptr)
        return;
    
    word->append(3, '!'); // same as (*word).append(3, '!');
}

int main(argc, char **argv)
{
    std::string mySentence = "Everything is awesome";
    emphasize(&mySentence);
}
```

In this example, the function `emphasize` receives a string and if it is not `nullptr`, it adds 3 exclamation marks to the end of a string. 

This function accepts a pointer since it must perform the change on the caller's object. If we passed the string by value the change would not have been reflected to the caller. If we passed the string by reference we could not have passed `nullptr`.



## 3. Classes



### 3.1. General

A `class` (or `struct`<sup>†</sup>) is a user-defined data type that contains variables of primitive data types and/or other classes.

An *object* is an allocated instance (a variable) of a class data type.

We shall start with a simple class example and move on later to classes that hold a resource.

<small><sup>†</sup> *Structures* in C++ are simply classes with `public` default access modifier.</small>



Example 3.1.1:

Header file `Point.h`:

```c++
class Point
{
    double x;
    double y;

public:
    Point();
    Point(double xVal, double yVal);
    void move(double dx, double dy);
    double getX() const; // this is a const method - it cannot change the object's data
    double getY() const; // ditto
}; // note: a semicolon is required at the end of the class declaration
```

Implementation file `Point.cpp`:

```c++
#include "Point.h"

Point::Point() : x(0), y(0) {}

Point::Point(double xVal, double yVal) : x(xVal), y(yVal) {}

void Point::move(double dx, double dy)
{
    x = x + dx;
    y = y + dy;
}

double Point::getX() const
{
    return x;
}

double Point::getY() const
{
    return y;
}
```



### 3.2. Differences from Java

The syntax of classes in C++ is somewhat different than it is in Java;

*   **Declarations and implementations are separate<sup>†</sup>** - The class definition contains class variables and the declarations of the methods. The actual implementation is listed separately, where each method name is prefixed by the class name. The scope resolution operator (the :: operator) designates to which class the method belongs.
*   **Semicolon at the end of the class** - A semicolon at the end of the class declaration. Not placing it will result in a compilation error.
*   **Public and private <u>section</u>** - In C++, there are public, private and protected *sections* as opposed to Java where each individual class member must be tagged separately.
*   **`Const` methods** - `const` methods do not change the state of an object<sup>‡</sup>, can only call other `const` methods and cannot return non-const references or pointers to members of the class. They are declared with the keyword `const` after the parameter list (see example 3.1.1).
*   **`Const` objects** - Once an object is declared as `const`, one can only use `const` methods on that object.

<small><sup>†</sup> The separation of declaration and implementation is not mandatory and may be omitted for inner, local private or helper classes.</small>

<small><sup>‡</sup> With the exception of data members marked as `mutable`.</small>

Example of a `const` object:

```c++
...
const Point p(0,0);
p.getY();         // this is ok since getY is declared const
p.move(1,1);      // compilation error since move is not declared const
...
```



### 3.3. Member Initialization List

In C++ constructors, we use a _member initialization list_ to initialize class variables. It appears after a single colon character, between the function declaration and the function body of the constructor. This code initialized the class variables and is executed before the code in the body of the constructor.

Any class variable (field) that does not appear in this list it will be initialized with it's *default constructor*. If the object's class has no default constructor the compilation will fail. 

Const class variables can *only* be initialized in the _member initialization list_. Attempting to change their value anywhere else, *including in the constructor's body* will cause a compilation error.

The order of initialization is according to the **order the fields in the class** - **not the order in the *member initialization list***. It is advisable to keep the order in the initialization list the same as in the class to avoid confusion.

In example 3.1.1 the initialization list is `x(xVal), y(yVal)` as follows:

```c++
Point::Point(double xVal, double yVal) : x(xVal), y(yVal) {}
                    ^^^^^^^^^^^^^^^^^^
//                  initialization list
```

> Notice that the *body* of this constructor is empty: `{}`



Since C++11 you can also use curly braces for initialization:

```c++
Point::Point(double xVal, double yVal) : x{xVal}, y{yVal} {}
```



### 3.4. Project Handling and Division into Files

As mentioned before, the declarations and the implementation are defined separately in C++. We place the class declaration file in a header file (e.g `Point.h`) and the class implementation in a cpp file (e.g. `Point.cpp`).

It is possible to write implementation code within header files but requires special care and understanding of the implications of doing so. In this course we do not allow this.

To avoid including a Header twice, we check whether a preprocessor unique variable is defined. If not, we define it and include the header. A common convention is to use the name of the header file as the preprocessor variable name. 

For example in `Point.h`:

```c++
#ifndef POINT_H
#define POINT_H

// all the header file content goes here

#endif
```

The same can be achieve by using the `#pragma once` preprocessor directive. For example:

```c++
#pragma once

// all the header file content goes here
```

In order to create an executable file (or a library) you need to *build* your project. To build the project you need to compile each class separately to object files and then link them.

> Important: the terms *preprocess*, *compile* and *link* are extremely important. If you are not familiar with them please refer to previous practical sessions! 

[Make](http://en.wikipedia.org/wiki/Make_software) is a tool that can help with complex build processes (it is somewhat equivalent to Java's *Ant*).

[Here](https://github.com/bguspl/bguspl.github.io/blob/main/code/srcPS3.tar.gz) is an example of a project with 2 header files, 3 cpp files and a make file.



## 4. Objects



### 4.1. General

An object is an instance of a class.

In C++, object variables hold *values*, not *references*. In the variable declaration you supply the parameters to the constructor after the variable name: 

```c++
// create the variable using the constructor Point::Point(double xVal, double yVal)
Point p(1, 2); 
```

If you do not supply construction parameters, then the object will be constructed using the default (no arguments) constructor: 

```c++
// create the variable using the (default) constructor Time::Time()
Time now;
```

> Note: In Java, `Time now` would create an uninitialized reference. In C++, it allocates and constructs an actual object.



### 4.1. The dot and arrow operators

The dot (.) operator and the arrow (->) operator are used to reference class members.

The dot operator in C++ is similar to Java's dot operator.

The arrow operator is used to dereference a point first and then reference the class member.

```c++
int main( )
{
    Point *p1 = new Point(0,0);
    Point p2(0,0);
    (*p1).getX();
    p1->getX(); // exactly the same as (*p1).getX()
    p2.getX();
    (&p2)->getX(); // exactly the same as p2.getX()
}
```



### 4.2. The `this` pointer

In C++ every object can access it's own address using the keyword `this` (a.k.a. the `this` pointer). It is an implicit (hidden) parameter to all non-static class methods and is used within the class method to refer to the invoking object.

If a method variable name is the same as a class variable it is said that it *shadows* the variable. A possible usage of the `this` pointer is to differentiate between method arguments and class variables - however, it is better to *avoid these situation altogether by using different names* (see in the example below).

The `this` pointer is an `rvalue` (and therefore it cannot be modified).



For example:

```c++
class MyClass
{
    int variable;
    
public:
    void setVariable(int);
};
```

```c++
void MyClass:setVariable(int variable)
{
    this->variable = variable; // possible, but not recommended
}
```

However, a more common way to do this is by using :: (the scope resolution operator):

```c++
void MyClass:setVariable(int aVariable)
{
    MyClass::variable = variable; // also possible but not recommended
}
```

Or better yet - avoid this altogether by using different names:

```c++
void MyClass:setVariable(int _variable)
{
    variable = _variable; // recommended - avoid shadowing altogether
}
```



> Notes: 
>
> 1. In C++ `this` is an rvalue and cannot be changed.
>
> 2. C++ allows objects to destroy themselves by executing the code:
>
>    ```c++
>    delete this; // not recommended!
>    ```
>
>    However, this requires special handling and is **not** recommended.



### 4.3. Creation of objects

As in Java, in C++, objects are created by calling a class *constructor*. In addition to regular constructors, there are a few additional "special" constructors and related methods. 

In the definition of constructors there is no return value but they always return a constructed object (unless an exception is thrown). 

Constructors can be called explicitly but there are also various situations in which they are called implicitly (some examples below).

#### 4.3.1. The default constructor

A *default constructor* is a constructor that can be called with no arguments (it either accepts no arguments or has default values for all it's arguments).

Example:

```c++
class MyClass1
{
    ...
public:
    MyClass1(); // default constructor
};

MyClass1 mc11; // implicit call to the default constructor
MyClass1 mc12{}; // explicit call to the default constructor
```



If a class has no explicitly defined constructors (of any kind), the compiler will automatically generate a *default constructor* for it.

Example:

```c++
class MyClass2
{
    int x;
    // a default constructor was automatically generated by the compiler
};

MyClass2 mc21; // implicit call to the default constructor
MyClass2 mc22{}; // explicit call to the default constructor
```



> Note: Since C++11 you can force the compiler to generate or to not generate such methods by using the keywords `default` and `delete`. For example: `MyClass4() = default;` `MyClass4() = delete;`



#### 4.3.2. Copy constructor

A *copy constructor* is a constructor who's first parameter is a reference of the same class (a regular reference or a `const` reference) and either accepts no arguments or has default values for all it's other arguments.

If a class has no explicitly defined copy constructor the compiler will automatically generate a default copy constructor for it that receives a `const` reference.

Example:

```c++
class MyClass3
{
    ...
public:
    MyClass3(const MyClass3 &other);
};

MyClass3 mc31;
MyClass3 mc32{mc31}; // explicit call to the copy constructor

someFunc(MyClass3 mc3) {...}

someFunc(mc31); // implicit call to the copy constructor
```



### 4.4. Destruction of objects

When an object is going to be destroyed (when it goes out of scope or because of a explicit use of `delete`), a special member function called a `destructor` is automatically called. The purpose of the `destructor` is to release any resources that were held by the object and perform any other necessary cleanup.

The `destructor` does not receive any arguments and does not return a value.

The name of the destructor is the class name preceded by a tilde symbol (~).

If a class has no explicitly defined destructor, the compiler will automatically generate a default destructor for it.

Example:

```c++
class MyClass4
{
    int *x;
public:
    MyClass4();
    ~MyClass4();
}
```

```c++
MyClass4::MyClass4() : x(new int) {}
MyClass4::~MyClass4() { delete x; }
```



### 4.5 Assigning objects

The `copy assignment operator` is called whenever an object appears on the left side of an assignment expression (and in some other situations). It takes exactly one parameter that is an object or a reference (or const reference) of the same class and returns a reference to an object of the class.

If a class has no explicitly defined copy assignment constructor the compiler will automatically generate a default copy constructor for it that receives a const reference.

The copy assignment operator must not throw exceptions (this may leave the object in an invalid state). If the automatically-generated default copy assignment operator may throw exceptions, one must be written explicitly.

Example (copy assignment operator of MyClass4):

```c++
MyClass::MyClass4 &operator=(const MyClass4 &other)
{
    if (this != &other) // if this is the same as other. do nothing.
        *x = *other.x; 
    return *this;
}

MyClass4 x, y;
x = y; // calls the copy assignment operator
```

> Note: using assignment in the variable declaration (e.g. `MyClass3 x, y = x;`) will actually call the *copy constructor* (and **not** the copy assignment operator).



### 4.6. The Rule of 3

If a class requires a user-defined destructor, a user-defined copy constructor, or a user-defined copy assignment operator, it almost certainly requires all three.[^1]

> Note: 
>
> Since C++11 there are also "The Rule of 5" and since C++20 - "The Rule of Zero" [^1]



[1]: https://en.cppreference.com/w/cpp/language/rule_of_three   ""The rule of three/five/zero""



## 5. A Complete Example: String Queue-Stack Class

> Sources and makefile for this example are [here](https://github.com/bguspl/bguspl.github.io/raw/main/code/StringQueueStack.zip).

In this example we implement an efficient data structure that can operate both as a queue and as a stack. It is implemented using a dynamic (linked) list of strings. Strings can be added from both ends of the list but can be removed only from the head. The class supports (deep) copy and assignment. We will demonstrate declarations of the class's destructor, copy constructor, and assignment operator.

> Note: in this particular example, it is (by design) the caller's responsibility to free the memory of the strings returned from pop()

Supported operations are:

- pop - removes the string at the head of the list and return it or nullptr if the stack is empty.
- push - pushes a string to the head of the list.
- append - inserts a string to the end of the list.
- clear - clears up the list (freeing all memory).

The Link class represents a single node in the list.



### 5.1. Object Diagram

![StringQueueStack](https://github.com/bguspl/bguspl.github.io/raw/main/images/StringQueueStack.jpg)

### 5.2. The file `StringQueueStack.h`:

```c++
#pragma once

#include <string>

class StringQueueStack
{
    struct Link
    {
        std::string* data;
        Link* next;
        Link(std::string* data, Link* next);
    } * first, * last; // declares first, last as class variables of type "pointer to struct Link"

public:

    StringQueueStack() = default;

    // the rule of 3
    StringQueueStack(const StringQueueStack& other);
    ~StringQueueStack();
    StringQueueStack& operator=(const StringQueueStack& other);

    // the rule of 5
    StringQueueStack(StringQueueStack&& other) noexcept;

    // other class methods
    void push(const char* data);
    void append(const char* data);
    std::string* pop();
    void clear();
};
```



### 5.3. The file `StringQueueStack.cpp`:

```c++
#include "StringQueueStack.h"

StringQueueStack::Link::Link(std::string* data, StringQueueStack::Link* next) : data(data), next(next) {}

void StringQueueStack::push(const char* data)
{
    first = new Link{new std::string(data), first}; // insert a new link before first

    if (!last)
        last = first;
}

void StringQueueStack::append(const char* data)
{
    Link* newLink = new Link{new std::string(data), nullptr};

    if (last)
        last = last->next = newLink; // evaluates "last->next = newLink" and then "last = last->next"
    else
        first = last = newLink;
}

std::string* StringQueueStack::pop()
{
    if (!first) // stack is empty?
        return nullptr;

    if (last == first) // only 1 item in the stack?
        last = nullptr;

    std::string* data = first->data; // save the data so that we can return it later
    Link* oldHead = first; // save the head so that we can free it later
    first = first->next; // set the head to whatever comes next
    delete oldHead;
    return data;
}

void StringQueueStack::clear()
{
    while (first)
        delete pop(); // delete the data
}

StringQueueStack::StringQueueStack(const StringQueueStack& other) : first{nullptr}, last{nullptr}
{
    *this = other; // invoke copy assignment operator to do the work for us
}

StringQueueStack::~StringQueueStack()
{
    clear();
}

StringQueueStack& StringQueueStack::operator=(const StringQueueStack& other)
{
    if (this != &other) { // is this not self assignment?
        clear();

        for (Link* link = other.first; link != nullptr; link = link->next)
            append(link->data->c_str()); // append will keep the correct order
    }

    return *this;
}
```



### 5.4 Execution example:

```c++
#include "StringQueueStack.h"
#include <iostream>

void pushAndPrint(StringQueueStack* sqs, const char* s)
{
    std::cout << "Pushing " << *s << std::endl;
    sqs->push(s);
}

void appendAndPrint(StringQueueStack* sqs, const char* s)
{
    std::cout << "Appending " << *s << std::endl;
    sqs->append(s);
}

void popAndPrint(StringQueueStack* sqs)
{
    std::string* s = sqs->pop();
    std::cout << "Popped: " << *s << std::endl;
    delete s;
}

int main()
{
    StringQueueStack* sqs = new StringQueueStack{};

    pushAndPrint(sqs, "1");
    pushAndPrint(sqs, "2");

    popAndPrint(sqs);
    popAndPrint(sqs);

    appendAndPrint(sqs, "1");
    appendAndPrint(sqs, "2");

    popAndPrint(sqs);
    popAndPrint(sqs);

    return 0;
}
```

```
Pushing 1
Pushing 2
Popped: 2
Popped: 1
Appending 1
Appending 2
Popped: 1
Popped: 2
```



## 6. Additional links

*   [examples1](http://pages.cs.wisc.edu/~hasti/cs368/CppTutorial/NOTES/CLASSES-PTRS.html)
*   [examples2](http://www.cs.fiu.edu/~weiss/phc++/code/)
*   [Pointers and Functions](http://mathbits.com/MathBits/CompSci/Pointers/Functions.htm)
*   [The C++ 'const' declaration](http://duramecho.com/ComputerInformation/WhyHowCppConst.html)

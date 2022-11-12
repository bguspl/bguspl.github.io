# Practical Session #4: Advanced Topics in C++



## 1. Goal

The goal for this practical session is to explain several key terms in modern C++ (version 11 and later):

- Move semantics
- The *auto* keyword
- Smart pointers

We will also discuss:

- Public, protected and private inheritance
- Constructors of derived classes
- Virtual functions
- Namespaces



> Note: Some of the examples in this document follow up on the StringQueueStack example from the previous practical session.



## 2. Object Lifetime

Every object<sup>†</sup> and reference has a ***lifetime***.  The lifetime is a runtime property (it is the period during which the object exists in the program's memory space).

The lifetime of an object begins when storage for it is obtained and it gets initialized.

The lifetime of an object ends when the object is destroyed and the storage it used is freed.

<small><sup>†</sup> In this context the term "object" also refers to a primitive variable.</small>



## 3. Value Categories

Every C++ expression has a ***type*** and belongs to a ***value category***. These value categories are the basis for the rules that compilers follow for creating, copying and moving temporary objects when evaluating expressions.

Two such categories are<sup>†</sup>:

1. ***lvalue*** - something that can appear in the left side of an assignment expression.
2. ***rvalue*** - something that can appear *only* in the right side of an assignment expression. 

<small><sup>†</sup> These are heuristical definitions (the formal ones are more complex).</small>

Literals (numbers, characters, etc.), for example, are rvalues. You cannot change their values so naturally they cannot appear in the left side of an assignment expression.



## 4. Temporary Objects

A ***temporary object*** is an object that is created temporarily in order to evaluate some expression. It is an rvalue and it's lifetime usually ends immediately after the expression is evaluated.

##### Example

```c++
int x = 2, y = 3;
int z = x * 2 + y; // this creates a temporary object
```

When the expression `x * 2` is evaluated (lets call it `tmp` for the sake of this explanation), a temporary object must be created to hold the result. Another temporary object will be created for `tmp + y` before it is assigned to `z`.

##### What is wrong with temporary objects?

In some cases temporary objects are a mandatory part of the evaluation process of an expression, but in other cases they are not. Creation of unnecessary temporary objects during the evaluation of expressions is a frequent culprit that incur a significant performance penalty (and also memory).



## 4. Const References

A common way to avoid the creation of unnecessary temporary objects is by using const references.

##### Example

Consider the following code:

```c++
void func3(MyClass x)
{
    ...
}

void func2(MyClass y)
{
    y.someConstMethod();
    func1(y);
}

void func1()
{
    func2(MyClass{});
}
```

The function `func2` receives `y`, an object of `MyClass`, and passes it along to `func3`. It does not change it at any point (assuming `someConstMethod` is indeed a const method). 

When `func1` calls `func2`, a copy of the object is created (because it is passed by value). Making a copy here is redundant since `func2` doesn't change `y`.

This can be avoided by declaring `y` as a const reference:

```c++
void func2(const MyClass &y)
{
    y.someConstMethod();
    func1(y);
}
```



## 5. Move Semantics and rvalue References

C++11 introduced the `move semantics` mechanism to further avoid redundant copying of objects. Using this mechanism, an object (under certain conditions) may take ownership of another object's external resources (such as memory allocation). 

The two main use cases for this are:

1. Avoiding unnecessary copying of objects, saving both runtime and memory.
2. Safely transferring ownership of resources that cannot be shared (such as certain types of locks and file handles).

## 5.1. **rvalue** references

To further unleash the potential of *move semantics*, C++11 also introduced *rvalue references*. An *rvalue reference* is a reference to a *temporary object*. We use double-ampersand to indicate that a variable is an *rvalue reference*.

Example:

```c++
void func3(MyClass &&x)
{
    ...
}

MyClass y;

func3(y); // compilation error - y is not an rvalue
func3(MyClass{y}); // this is fine
```

The line `func3(MyClass{y})` is fine because `MyClass{y}` creates a *temporary object* using the *copy constructor*.

## 5.2 Converting *lvalue* to *rvalue*

It is possible to convert an *lvalue* to an *rvalue* by using `std::move`.

In fact, `std::move` doesn't actually move anything (it is technically a function but is considered a cast).

By converting it's argument to an *rvalue*, `std::move` allows the compiler to perform optimizations that are not allowed for an *lvalue* at the cost of invalidating the argument after finishing the evaluation of the expression it is in.

##### Example

```c++
void func1(MyClass x) 
{
    ...
}

void func2(MyClass y)
{
    func1(std::move(y));
    MyClass z = y; // this compiles but is a bug!
}
```

The expression `MyClass z = y` above will compile but **is a bug** since the `y` has been invalidated by `std::move`.



## 6. The Rule of 5

As mentioned above,  C++ may create unnecessary temporary objects which may incur a significant performance (and sometimes also memory) penalty.

One way of eliminating (or at least mitigating) this is to employ the "rule of 5" heuristic, which is "the rule of 3" plus 2 more special methods: the *move constructor* and the *move assignment operator*.

> Note: both the *move constructor* and the *move assignment operator* should never throw exceptions since this may leave objects in an invalid state.



### 6.1. The Move Constructor

A *move constructor* is a constructor who's first parameter is an *rvalue reference* of the same class (a regular or a const reference) and either accepts no arguments or has default values for all it's other arguments.

A *move constructor* "steals" the resources of the object it is given as an argument rather than make copies of them, but leaves the other object in some valid state that will ensure that when the other object is destroyed it will not attempt to free it's resources.

If a class has no explicitly defined copy and move constructors, no copy and move assignment operators and no destructor (all of which must not exist), the compiler will automatically generate a default *move constructor* for it that receives a normal (non-const) reference to an *rvalue*.

Example: 

```c++
StringQueueStack::StringQueueStack(StringQueueStack&& other) noexcept
    : first{other.first}, last{other.last}
{
    other.first = nullptr;
    other.last = nullptr;
}
```

Notice that we assign `nullptr` to `other.first` and `other.last` since `other` does not own these resources anymore (for example, we do not want it to free the memory when it gets destroyed).



### 6.2 Move Assignment Operator

The *move assignment operator* is called when an object appears on the left side of an assignment expression and on the right side of the assignment operation there is an *rvalue* of the same type (or an implicitly-convertible type).

Similarly to *move constructor*, the *move assignment operator* typically "steals" the resources of the other object.

If a class has no explicitly defined copy and move constructors, no copy and move assignment operators and no destructor (all of which must not exist), the compiler will automatically generate a default *move assignment operator* for it that receives a normal (non-const) reference to an *rvalue*.

Example: 

```c++
StringQueueStack& StringQueueStack::operator=(StringQueueStack&& other) noexcept
{
    clear();
    first = other.first;
    last = other.last;
    other.first = nullptr;
    other.last = nullptr;
}
```

Again, we assign `nullptr` to `other.first` and `other.last` since `other` does not own these resources anymore.



## 7. Temporary Object Lifetime Extension

References to variables can be also be declared as any other variables. When a reference to a temporary object is declared, the temporary object's lifetime is extended to the reference's lifetime (making the temporary object "not so temporary"...).

There are 2 types of references to an *rvalue* that are allowed:

1. A const reference
1. An rvalue reference

##### Examples

###### Const value reference for an object

```c++
const MyClass& tmp = MyClass{};
MyClass* ref = const_cast<MyClass*>(&tmp);
```

> Note: this is bad practice (because of the *const cast*), but will work.

###### rvalue reference for an object

```c++
MyClass&& tmp = MyClass{};
MyClass* ref = &tmp;
```

###### With primitives

```c++
int x = 1;
int &y = x; // y is a reference to x
int &&z = int(y); // z is a reference to a temporary variable created by the casting
y = 2; // changing y will change x
z = 3; // changing z will not change y
std::cout << "x = " << x << " y = " << y << " z = " << z << std::endl;
```

Output:

```
x = 2 y = 2 z = 3
```

> Note: a expression such as `int &&z = y` will not compile since y is not a temporary variable

In this example, the lifetime of the temporary object created by casting `y` to an `int` has been extended. If it was not so, the expression `z = 3` might have caused a *memory access violation* (a.k.a. *segmentation fault*).

> Note: casting `y` to an `int` creates a temporary object here even though `y` is already an `int`. In other circumstance this might be automatically removed by compiler optimization.



## 8. Placeholder Type Specifier (the *auto* keyword)

Another mechanism introduced in C++11 is the *placeholder type specifier* - the `auto` keyword.

The `auto` keyword may (sometimes) used in variable / function declarations instead of an explicit type specifier in order to automatically deduce the type from the variable initializer / function return value.

Using `auto` where applicable is generally considered a recommended practice for reason of usability and robustness (it reduces the chance of typos for example), performance (no conversions are guaranteed when using it) and other reasons.

You can force it to be reference / pointer and/or const by declaring with `auto&` / `auto*` and/or `const auto`.

Example:

```c++
auto list = new std::vector<MyClass>{};

for (const auto& item: *list)
    item.doSomething();
```

> Note: the `auto` type specifier can *almost always* be replaced by an explicit type specifier (except in some very esoteric scenarios).



## 9. Smart Pointers

In modern C++ programming, the Standard Library includes *smart pointers*. These are class templates that  contain a pointer and provide a limited garbage collecting facility, *with little to no overhead* over built-in pointers, by automatically deleting the pointer when it is no longer needed.

There are currently 3 types of smart pointers:

1. Shared pointers
2. Unique pointers
3. Weak pointers



### 9.1. Shared Pointers

A *shared pointer* is a smart pointer that  designed for scenarios in which more than one owner might need to manage the lifetime of an object.  It uses a *reference count control block* to designate when the pointer can be deleted. 

Whenever a new object is created from the shared pointer (as a result of it being copied, passed by value or assigned) the reference is incremented. When any of these objects gets destroyed the reference count is decremented. When the reference count reaches zero, the underlying pointer and the control block are deleted.

##### Example

```c++
auto sp1 = std::shared_ptr<MyClass>(new MyClass());
auto sp2 = std::shared_ptr<MyClass>(new MyClass(1, 2));

sp2 = sp1;
```

The expression `sp2 = sp1` will also have 2 consequences:

1. The current pointer `sp2` stores will be deleted.
2. The pointer `sp1` stores will be copied to `sp2`, making both `sp1` and `sp2` owners and the reference count will increase to 2.



### 9.2. Unique Pointers

A *unique pointer* allows exactly one owner of the underlying pointer. It can be moved to a new owner, but not copied or shared.

> Note: *unique pointers* replaces the older (now deprecated) *auto pointers*.

##### Example

```c++
auto up1 = std::unique_ptr<MyClass>(new MyClass());
auto up2 = std::unique_ptr<MyClass>(new MyClass(1, 2));

up2 = up1;
```

The expression `up2 = up1` has two consequences:

1. The current pointer `up2` stores will be deleted.
2. The pointer `up1` stores will be transferred to the variable `up2` (resetting `up1` to an empty state).



### 9.3. Weak Pointers

This type of pointers are used in conjunction with *shared pointers*. They are intended for situations when we need access to shared pointer without increasing the reference count and the state of the pointer might have been changed elsewhere (i.e. the reference count reached zero and it was deleted). A common scenario for this is when implementing a memory cache.

You can query the state of the shared pointer by using the weak pointer's `expired()` method. If you want to access the resource itself you must create a shared pointer from the weak pointer by using the `lock()` method.

##### Example

```c++
auto sp1 = std::make_shared<MyClass>();
std::weak_ptr<MyClass> wp = sp; // implicit cast 

if (wp.expired())
    std::cout << "object expired";

auto sp2 = wp.lock();
if (sp2 == false)
	std::cout << "object expired";
else
    // you can use sp2 to access the resource
```



### 9.4. Creation of Smart Pointers

It is better (for reasons that are beyond the scope of this lesson) to use `std:make_shared` and `std::make_unique` to create a smart pointer. These functions call the constructor that matches their argument list to create the object.

##### Example

Instead of the following code:

```c++
auto sp1 = std::shared_ptr<MyClass>(new MyClass());
auto sp2 = std::shared_ptr<MyClass>(new MyClass(1, 2));
auto up1 = std::unique_ptr<MyClass>(new MyClass());
auto up2 = std::unique_ptr<MyClass>(new MyClass(1, 2));
```

Do this:

```c++
auto sp1 = std::make_shared<MyClass>(); // calls the default constructor
auto sp2 = std::make_shared<MyClass>(1, 2); // calls a constructor with 2 arguments
auto up1 = std::make_unique<MyClass>(); // calls the default constructor
auto up2 = std::make_unique<MyClass>(1, 2); // calls a constructor with 2 arguments
```



## 10. Inheritance in C++

*Inheritance* is a mechanism for reusing and extending existing classes without modifying them by creating hierarchical relationships between them. 

When a class inherits from another class it is said to be *derived* from it (it is a *derived class* or a *subclass*). When a class is inherited from, it is said to be the *base class* (or a *superclass*). 

We say there is an "is-a" relationship between the derived and base classes. In the example below this relationship is illustrated as follows: a Student **is a** Person.

A *derived class* inherits the members of a *base class*. The access modifier of the *base class* members in the derived class depends on the *access specifier* of the inheritance. In C++ there are 3 possible *access specifiers* of inheritance:

1. ***Public inheritance*** - members of the base class keep their original access specifiers (*public* stays public, *protected* stays protected) except *private* members which become inaccessible.
2. ***Protected inheritance*** - members that have *public* access in the base class become protected. *Protected* stays protected. *Private* members are inaccessible.
3. ***Private inhertiance*** - members that have *public* and *protected* access in the base class become private. *Private* members are inaccessible.

> Note: public inheritance is the most common. Protected and private inheritance are rarely used as in most cases a better strategy is object *composition*. 



### 10.1 Constructors in Derived Classes

When a derived class is initialized (using a constructor), the constructor of the base class is called first of all - either explicitly (as in the example below) or implicitly, in which case the default constructor is called. 

Compilation will fail if no constructor is explicitly called and the base class has no default constructor or the default constructor is declared *private*.

##### Example

```c++
class Person
{
public:
    Person(std::string _name) : name{_name} {}
	std::string getName();
    
protected:
    std::string name;
};

class Student : public Person
{
public:
    Student(std::string _name) : Person{name} {}
    void doHomework();
};

void Student::doHomework()
{
    std::cout << name << " is doing homework" << std::endl;
}
```



### 10.2 Virtual Functions

As mentioned earlier, there is an "**is-a**" relationship between a derived class and it's base class, it therefore makes perfect sense that a reference or pointer to the base class can be used with an object of the derived class.

##### Example

```c++
void DriveToSchool(Person &person) 
{ 
    person.OpenGate();
}

Student student{};

Walk(student);
```

If a (non-virtual) method of the base class is overridden in a derived class and we call this method using a reference or pointer to the base class, the method of the base class will be called - not the derived class's method, which is usually not the desired behavior. This is called *compile time (early) binding*.

The solution for this is a *virtual function*. A virtual function is a method in the base class that we expect to redefine in derived classes. When you call a virtual function the "correct" method (i.e. the method of the actual class of the object) will automatically be called according to the type of the reference or pointer at runtime and not according to the type in the declaration. This is called *runtime (late) binding*.

When *overriding* a virtual function we use the `override` keyword in the method declaration (there is no need to repeat the `virtual` keyword though since it is implied by the `override` keyword).

##### Example

```c++
class Person
{
public:
    Person(std::string _name) : name{std::move(_name)} {}
	virtual std::string doSomething();
};

class Student : public Person
{
public:
    Student(std::string _name) : Person{std::move(_name)} {}
    std::string doSomething() override;
};
```

##### Rules of virtual functions

1. Virtual functions cannot be static.
2. The prototype of a virtual function must be the same in the base and derived classes.
3. If a virtual function is not overridden in the derived class, the method of the base class will be automatically called instead.
4. Constructors cannot be defined as virtual.



#### 10.2.1 The Virtual Destructor

Virtual destructors are particularly important  because if an object of a derived class is deleted using a pointer of a base class the result is undefined and almost always a bug. To address this, destructors should always be declared as virtual if a class is inherited from (as a guideline, any time you have a virtual function in a class, you should immediately add a virtual destructor - even if it does nothing).

##### Example

```c++
class MyClass
{
public:
    virtual void doSomething();
    virtual ~MyClass() = default; // a default destructor defined as virtual
}
```



#### 10.2.2 Pure Virtual Functions and Abstract Classes

A *virtual function* can be declared as *pure virtual* by appending `= 0` in the function declaration. In which case it is not defined (implemented) at the class where it is declared and it is not possible to create an instance (a variable) of this class. 

An instance of a class can only be created if all virtual functions are implemented either by the derived class or by an ancestor (i.e. the base class or the base class's base class, etc.). A class that cannot be instantiated due to this reason is called an *abstract class* (it is similar to a Java *interface* or *abstract class*) .

##### Example

```c++
class MyClass
{
public:
    virtual void doSomething() = 0; // pure virtual function => the class is abstract
}
```



## 11. C++ Namespaces

A *namespace* is a declarative region that provides an artificial scope to identifiers (type names, variables, functions, classes, etc.) declared within it. It is somewhat similar to Java packages although it is not derived from the directory structure but is declared explicitly.

*Namespaces* are used to organize code into logical groups and to prevent name collisions that may occur (mostly) when a program includes different libraries written by different parties.

A *namespace* is declared using the `namespace` keyword and must be declared in the *global scope* or within other namespaces. To access identifiers that is within a namespace from outside the namespace, you must use the *scope resolution operator* (::) or with the `using` keyword.

##### Example

```c++
namespace mine
{
struct MyStruct
{
    int x;
};
} // end of namespace "mine"

namespace your
{
struct MyStruct
{
    int y;
};
} // end of namespace "your"

int main()
{
    mine::MyStruct myMyStruct{};
    your::MyStruct yourMyStruct{};
    
    myMyStruct.x = yourMyStruct.y;
    
    using namespace mine;
    MyStruct myMyStruct2;
    
    using namespace your;
    MyStruct bad; // error "ambiguous reference"!

    return 0;
}
```


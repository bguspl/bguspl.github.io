The Rule of 5
We saw in class that the rule of 3 suffers from inherent efficiency problems. Therefore, starting from C++11 two addition functions were added to the rule of 3 thumb rule, which aim to solve these problems: the move constructor and the move assignment operator. In total, the thumb rule states that the following methods need to be implemented:
destructor
copy constructor
move constructor
copy assignment operator
move assignment operator

Note that sometimes we don't want to hold a copy of a certain object in the field of a class, but still hold the object. This can be done by declaring the field as a reference. However, in this case we must remember to initialize it in the constructor using initialization list.
You can find more examples in the code below:

The Rule of 5
Its important to note that the rule of 5 and the rule of 3 implies only for classes that manage dynamic resources, such as pointers. In general, we would usually try to avoid such classes and use classes that use only primitive or static fields. Such classes are not considered as classes that manage resources and therefore does not require an implementation based on the rule of 5 or the rule of 3.
The rule of 0 states that classes that does not manage dynamic resources (pointers) should not implement any of the methods mentioned in the rule of 5.

The Rule of 5 Example
We now see an example for the rule of 5:
downloadtoggle
34 lines ...
#ifndef RULEOFFIVE_H
#define RULEOFFIVE_H
 
class charArray
{
private:
    char *cstring; // raw pointer used as a handle to a dynamically-allocated memory block
    int cstringlen;
 
    void copy(const char *other_cstring, const int &other_cstringlen);
    void clear();
public:
 
    const char* toString() const;
 
    // Constructor
    charArray(const char *arg);
 
    // Destructor
    virtual ~charArray();
 
    // Copy Constructor
    charArray(const charArray &other);
 
    // Move Constructor
    charArray(charArray &&other);
 
    // Copy Assignment
    charArray& operator=(const charArray &other);
 
    // Move Assignment
    charArray& operator=(charArray &&other);
};
 
#endif
downloadtoggle
30 lines ...
#include "RuleOfFive.h"
#include <string>
#include <cstring>
 
 
const char* charArray::toString() const { return cstring; }
 
void charArray::copy(const char *other_cstring, const int &other_cstringlen)
{
        cstringlen = other_cstringlen;
        cstring = new char[cstringlen + 1]; // allocate
        strcpy(cstring, other_cstring); // populate
}
 
// Constructor
charArray::charArray(const char *arg) {    copy(arg, std::strlen(arg)); }
 
// Copy Constructor
charArray::charArray(const charArray &other) { copy(other.cstring, other.cstringlen); }
 
// Copy Assignment
charArray& charArray::operator=(const charArray &other)
{
    if (this != &other)
    {
        clear();
        copy(other.cstring, other.cstringlen);
    }
 
    return *this;
}
The Destructor
Similar to the destructor we built in PS3, we use a private function clear() to release the memory. This is done as a preparation for the move and copy assignment operators that also need to perform a clearing operation.
Note that the last line cstringlen = 0; is not actually required in the destructor, however it is done to maintain the correctness of our object: if its cstring is null, not changing its length to 0 may lead to logical mistake later on.

downloadtoggle
11 lines ...
// Destructor
charArray::~charArray() { clear(); }
 
void charArray::clear()
{
    if (cstring)
    {
        delete[] cstring; // deallocate
        cstring = nullptr;
        cstringlen = 0;
    }
}
The Move Constructor
The move constructor is typically called when an object is initialized from rvalue. This is done using rvalue references that we've seen in class. This usually happens in the following cases:
initialization, std::string a = "str"
function argument passing: v.push_back("str")
function return by value

download
// Move Constructor
charArray::charArray(charArray&& other) 
    : cstring(other.cstring), cstringlen(other.cstringlen)
{
    other.cstring = nullptr;
    other.cstringlen = 0;
}
Move constructors typically "steal" the resources held by the argument (e.g. pointers to dynamically-allocated objects, etc), rather than make copies of them, and leave the argument in some valid but otherwise indeterminate state. For example, moving from a std::string or from a std::vector may result in the argument being left empty. However, this behavior should not be relied upon. For some types, such as std::unique_ptr, the moved-from state is fully specified.

For example, in the following line

download
v.push_back(charArray("I will be moved, then destroyed"));
The following actions will happen:
A new object will be created with the string "I will be moved, then destroyed"
The move constructor will create a new object inside the vector and will move the created object to it
The pointer that was created in the first object will become nullptr and the object inside the vecter will now hold the first pointer
The object that was created outside of the vector will be destroyed. Since its cstring pointer was null, the destruction will not effect the object inside the vector

rvalue references are defined using &&, i.e., int&& i is an rvalue reference to int. In this way, we can use temporary variables without making an actual copy of them.
downloadtoggle
26 lines ...
#include <vector>
#include "RuleOfFive.h"
#include <iostream>
 
int main(){
    charArray t1("I a will be copied");
 
    charArray t2(t1);
 
    std::vector<charArray> v;
    v.push_back(charArray("I will be moved, then destroyed"));
 
    std::cout << v.front().toString() << std::endl;
 
    charArray t3("I will be copied by assignment");
    std::cout << t2.toString() << std::endl;
    t2 = t3;
    std::cout << t3.toString() << std::endl;
 
    charArray t4("I will be moved by assignment");
 
    std::cout << t2.toString() << std::endl;
    t2 = std::move(t4); //explicit 
    //std::cout << t4.toString() << std::endl; //This line will not work as t4 is empty
    t2 = charArray("I will be moved by assignment");
    return 0;
}
The Move Assignment
The move assignment operator is built using the same steps we've seen in the copy assignment operator. The only difference is that the move assignment operator doesn't copy the values, and instead only moves the ownership of its pointers from one object to another.
downloadtoggle
13 lines ...
// Move Assignment
charArray& charArray::operator=(charArray &&other)
{
    if (this != &other)
    {
        clear();
        cstringlen = other.cstringlen;
        cstring = other.cstring;
        other.cstring = nullptr;
        other.cstringlen = 0;
    }    
 
    return *this;
}
Smart Pointers
In modern C++ programming, the Standard Library includes smart pointers, which are used to help ensure that programs are free of memory and resource leaks and are exception-safe.
std::unique_ptr
This smart pointers provides a limited garbage-collection facility, with little to no overhead over built-in pointers. Objects of this kind "steals" the ownership of a pointer. Once ownership is obtained, they manage the pointed object by becoming responsible for its deletion at some point.
unique_ptr objects automatically delete the object they manage, as soon as they themselves are destroyed, or as soon as their value changes either by an assignment operation or by an explicit call to unique_ptr::reset.

unique_ptr objects own their pointer uniquely: no other facility shall take care of deleting the object, and thus no other managed pointer should point to its managed object, since as soon as they have to, unique_ptr objects delete their managed object without taking into account whether other pointers still point to the same object or not, and thus leaving any other pointers that point there as pointing to an invalid location.

downloadtoggle
29 lines ...
#include <memory>
 
class Cow {
private:
    int id;
public:
    Cow(int _id): id(_id) {}
    void printCowId() { std::cout << "Cow id=" << id << std::endl; }
};
 
int main()
{
    std::unique_ptr<Cow> p1(new Cow(100));  // p1 owns Cow(100)
    p1->printCowId();    //100
 
    {  
        std::unique_ptr<Cow> p2(std::move(p1));  
        if (p2)
            p2->printCowId();   //100
        if (p1)                 //if (p1) is false
            p1->printCowId();
        p1 = std::move(p2);     // ownership returns to p1
        std::cout << "destroying p2...\n";
    }  
 
        if (p1){   //if (p1) is true
        p1->printCowId();   //100
    }
    return 0;
} // Cow instance is destroyed when p1 goes out of scope
unique_prt is implemented using two components:

A stored pointer: the pointer to the object it manages
A stored deleter: a callable object that takes an argument of the same type as the stored pointer and is called to delete the managed object
Inheritance
Inheritance is a relationship among classes, in which one class shares the structure or behavior defined in other classe(s). In object-oriented programming (OOP), inheritance is a way to reuse code by creating collections of attributes and behaviors called objects which can be based on previously created objects.
In C++ class can be defined by means of an older, pre-existing class, which leads to a situation in which a new class has all the functionality of the older class, and additionally introduces its own specific functionality.

The new class inherits the functionality of an existing class, while the existing class does not appear as a data member in the definition of the new class. When speaking of inheritance the existing class is called the base class, while the new class is called the derived class.

For example, lets create a class intArray:

downloadtoggle
16 lines ...
class intArray
{
public:
    intArray();
    intArray(const int &size);
    int insert(const int &num, const unsigned int &index);
    bool isEmpty() const;
 
private:
    int *arr;
    unsigned int arrSize;
 
    void expandArray(const unsigned int &newArrSize);
 
//protected:
    //unsigned int arrSize;
};
downloadtoggle
28 lines ...
intArray::intArray() : arr(nullptr), arrSize(0) { }
 
intArray::intArray(const int &size) : arr(new int[size]), arrSize(size) { }
 
int intArray::insert(const int &num, const unsigned int &index)
{
    expandArray(index + 1);
    return arr[index] = num;
}
 
bool intArray::isEmpty() const { return arr == nullptr; }
 
void intArray::expandArray(const unsigned int &newArrSize)
{
    if (newArrSize >= arrSize)
    {
        int *newArr(new int[newArrSize]);
        for (size_t i = 0; i < arrSize; i++)
            newArr[i] = arr[i];
 
        if (arr)
        {
            delete[] arr;
        }
 
        arr = newArr;
        arrSize = newArrSize;
    }
}
Now we create a class with the same functionality as intArray but all data will be sorted:

download
class intSortedArray : public intArray
{
public:
    intSortedArray();
    intSortedArray(const int &size);
    int insert(const int &num, const unsigned int &index);
    int biggest() const;
private:
    int biggestElem;
};
downloadtoggle
15 lines ...
intSortedArray::intSortedArray() : intArray(), biggestElem(std::numeric_limits<int>::min()) { }
 
intSortedArray::intSortedArray(const int &size) : intArray(size), biggestElem(std::numeric_limits<int>::min()) { }
 
int intSortedArray::insert(const int &num, const unsigned int &index)
{
    if (biggestElem < num)
        biggestElem = num;
 
    //std::cout << "now the variable arrSize is accessible from derived classes! arrSize = " 
    //    << arrSize << std::endl;
 
    return intArray::insert(num, index);
}
 
int intSortedArray::biggest() const { return biggestElem; }
The relationship is better achieved with inheritance: intSortedArray is derived from intArray, in which intArray is the base class of the derivation. By post-fixing the class name intSortedArray in its definition by public intArray the derivation is defined: the class intSortedArray now contains all the functionality of its base class intArray plus its own specific information.

The extra functionality consists here of a CTORs and interface functions to access the biggest item in array.

Types of Inheritance
In cpp there are 3 levels of inheriting a base class. In the example above, we used a public inheritance. There are also protected and private inheritance. The table below summarizes the access level (public, protected or private) that a derived class gets, to the base class members according to each inheritance level
Inheritance levels

Public Inheritance Example
Example: The previous code
download
class intSortedArray : public intArray {...}
 
intSortedArray sortArray;
std::cout << sortArray.isEmpty() << std::endl;
std::cout << sortArray.insert(10, 5) << std::endl;
std::cout << sortArray.biggest() << std::endl;
This example shows three features of derivation.

isEmpty() - is no direct member of a intSortedArray. This member function is an implicit part of the class, inherited from its "parent" class.
biggest() - derived class intSortedArray now adds to functionality of intArray.
insert() - is overloaded in the class intSortedArray.
Protected access control
In C++ we can declare a variable or a function that seems private to outside users of a class but seems public to classes that inherit the class. To do so, we use the protected section of the base class. Note that the derived class must use either public or protected level of inheritance.
For example, say that the intSortedArray class needed direct access to the arrSize variable in intArray in order to improve its performance, but we still want to keep normal instances of intArray and intSortedArray from accessing the arrSize directly.

By declaring arrSize as a protected member in intArray, it would be accessible by the derived class intSortedArray but not by normal instances of intArray or intSortedArray:

download
class intArray {
public:
    intArray();
    int insert(const int*);
protected:
    int arrSize;
private:
    ...
}
Private Inheritance Example
As mentioned earlier, it is also possible to use a protected/private inheritance using private/protected keyword instead of public, however this is generally not recommended (we will not elaborate on that).
Bellow is an example of private inheritance use. Private inheritance makes all of the public members of the base class private in the derived class. This means that they can be used in order to implement the derived class without being accessible to the outside world.

Unlike a public inheritance, a private inheritance defines a "has-a" relation with its parent, and not an "is-a" relation like public inheritance.

For example:

downloadtoggle
11 lines ...
class Person {};
class Student : private Person {};    // private
void eat(const Person& p){}            // anyone can eat
void study(const Student& s){}        // only students study
 
void main()
{
    Person p;    // p is a Person
    Student s;    // s is a Student
    eat(p);        // Fine, p is a Person
    eat(s);        // Compilation error! s isn't a Person
}
The CTOR of a derived class
A derived class inherits the functionality of its base class. How effects the inheritance on the CTOR of a derived class ?
As can be seen from the definition of the class intSortedArray, a CTOR exists to set the biggestElem of an object. The implementations of this CTORs could be:

download
intSortedArray::intSortedArray() : biggestElem(std::numeric_limits<int>::min()) { }
Memory Layout Of Derived Class
Memory layout of derived class:
|*****************|

| Base part |

|*****************|

| Derived part |

|*****************|

The C++ compiler will generate code to call the default CTOR of a base class from each CTOR in the derived class, unless explicitly instructed otherwise.

Calling CTOR of the Base Class
The better solution is of course to directly call the CTOR of intArray which expects an int argument. The syntax to achieve this, is to place the CTOR to be called (supplied with an argument) following the argument list of the CTOR of the derived class:
download
intSortedArray::intSortedArray(const int &size) : intArray(size), biggestElem(std::numeric_limits<int>::min()) { }
Virtual Functions
Virtual functions are member functions whose behavior can be overridden in derived classes. As opposed to non-virtual functions, the overridden behavior is preserved even if there is no compile-time information about the actual type of the class.
The access level to a virtual member function is determined by the declared type of the variable that is used to access it, regardless the actual variable type, since it is determined at compilation time. This means that overridden functions that are defined with a different access level than the base class (this is legal), their access level might not be respected.

For example:

downloadtoggle
22 lines ...
class Base {
public:
    virtual void foo(){ std::cout << "foo of Base\n"; }
};
 
class Derived : public Base {
public:
    …
private:
    virtual void foo() { std::cout << "foo of derived\n"; } 
};
 
 
void callFoo(Base* bp) { bp->foo(); } //Can access publicly both functions, public Base::foo() and private Derived::foo() even though Derived::foo() is private
 
void main() {
    Base b;
    Derived d;
    b.foo();      // OK
    d.foo();      // compilation error since d is declared "Derived" which has no access to foo()
    callFoo(&b);  // OK, will print "foo of base" 
    callFoo(&d);  // OK, will print ""foo of derived"
}
Running this example (after commenting out d.foo() line) produces the following output:

download
foo of Base
foo of Base
foo of derived
Virtual Destructor
Without a garbage collector, polymorphism can be problematic when attempting to release memory. This is best explained with an example. We first look at what happens when not using virtual destructor:
downloadtoggle
19 lines ...
class Base
{
public:
    Base(){ std::cout << "Constructing Base"; }
    // this is a non virtual destructor:
    ~Base(){ std::cout << "Destroying Base"; }
};
 
class Derived : public Base
{
public:
    Derived(){ std::cout << "Constructing Derived"; }
    ~Derived(){ std::cout << "Destroying Derived"; }
};
 
void main()
{
    Base *basePtr = new Derived();
    delete basePtr;
}
Since Derived class inherits from Base, we can treat a Derived object as a Base pointer. What will be the output of these lines?

download
Constructing Base  
Constructing Derived 
Destroying Base
Since we told the compiler that basePtr is a pointer of type Base, the destructor of the Base class was used. This means that if Derived allocated memory, it would not have been deleted (since the destructor of Derived was not called) and we would have a memory leak.

Now lets take a look at what happens when we use a virtual destructor. We now change the destructor's definition in the Base class to be virtual:

download
virtual ~Base(){ cout<<"Destroying Base";}
And repeat the same function. The output will now be:

download
Constructing Base  
Constructing Derived 
Destroying Derived
Destroying Base
Now, when the compiler see that we assigned a Derived class to a Base class with a virtual destructor, it knows that in order to delete this pointer, we need to call the destructor of Derived before calling the destructor of Base.

The best way to avoid such cases is to use virtual destructor whenever possible.

Pure Virtual Functions & Abstract Base Classes
Let’s say we want to create a set of classes that implement different kinds of shapes ( circle, rectangle … ). The functionality of them is similar. We’ll not want to rewrite the same code many times and we would like to create some class that will contain common code but would be impossible to make instantiation of such class.
Example is class "shape" which will be a base class of all kinds of real shapes as circle, ellipse, rectangle etc. In C++ this is possible to do by declaration of one or more member functions as pure virtual.

download
class shape {
public:
    virtual void draw() = 0;
    …
};
Now compiler will fail compiling following code:

download
shape shp; // compilation error
The only type of object that can be declared is an object of derived class in which virtual member function draw() is implemented

Until now the base class shape contained its own, concrete, implementations of the virtual functions draw(). In C++ it is also possible only to mention virtual functions in a base class, and not define them. The functions are concretely implemented in a derived class. The special feature of only declaring functions in a base class, and not defining them, is that derived classes must take care of the actual definition: C++ compiler will not allow the definition of an object of a class which doesn't concretely define the function.

The base class enforces a protocol by declaring a function prototype, but the derived classes must care of the actual implementation. The base class itself is therefore only a model, to be used for the derivation of other classes. Such base classes are also called abstract classes.

A function is made pure virtual by preceding its declaration with the keyword virtual and by post-fixing it with =0. The functions are called pure virtual functions.

Namespace
C++ namespace is similar to Java packages only in the sense that it helps identifying names. The C++ compiler will not create a folder hierarchy of the namespace.
downloadtoggle
37 lines ...
// namespace.cpp   namespaces are typically defined in header files
//                 usually one namespace per file
//                 usually the "using namespace xxx;" is after the  #include's
#include <iostream>
using namespace std;
 
namespace mine
{
  int i, j;          // without 'static' or 'namespace' these are seen by linker
  //   all declarations including 'namespace' and 'using' allowed
}
 
namespace yours
{
  int i, k, n;      // this could give a linker error about "i" doubly defined
}                   // with 'namespace' each "i" is unique to the linker
 
 
int global_name_known_to_linker;  // other files use  extern int global_nam...
static int global_to_this_file_but_not_to_linker;
 
int main(void)
{
  int n = 1;
  mine::i = 2;            // same notation used for class variables
  yours::i = 3;
  yours::n = 4;
  using namespace mine;
  j = 5;
  using namespace yours;
  k = 6;
 
  // i = 7;              error, ambiguous now that 'using' both namespaces
  n = 8;                 // local 'n' not overridden or ambiguous
  cout << mine::i << yours::i << j << k << n << yours::n << '\n';
  return 0;
}
// results are:   235684
More information can be found here.

References
Excellent C++ reference page
C++ String Examples

Programming in C includes C/C++ Program Compilation

Moving from Java to C++

Header files and the preprocessor (Microsoft discussion)
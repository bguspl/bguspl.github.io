Practical Session #3 C++ Classesshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
Goalshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>The main goal for this practical session is learning Object Oriented programming in C++ with emphasis on classes that hold resources, specifically classes with pointer members. We will learn basic syntax definitions and explore the differences between C++ conventions and Java conventions.
Parameter passingshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
Correct parameter passing can help reduce the number of bugs and generate more efficient code. Passing parameters can be done in 3 distinct methods:
Passing by value
Passing by pointer
Passing by reference
The following examples will refer to passing a parameter into a function. the same principles apply to the return value.

Passing by value - exampleshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
int power(int i) 
{
  i = i*i;
  return i;
}
int main (int argc, char **argv) 
{
  int number = 5;
  power(number);
}
In this example the primitive integer value is passed to the power function by value. This means that the value of number is being copied to the argument list of the called function. Passing by value is usually done for primitive variables. Passing by value also works with objects, it will cause the compiler to add a call to the copy constructor making it a slow option. In general, unless a new copy of the object is needed, we will never pass an object by value.
Passing by pointer - exampleshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
void emphasize(std::string *word) 
{
  if (word == nullptr)
    return;
  word->append(3, '!'); // same as (*word).append(3,'!');
}
int main (int argc, char **argv) 
{
  std::string mySentence = "Everything is awesome";
  emphasize(&mySentence);
}
In this code the function emphasize is used to add 3 exclamation marks to the end of a string. we notice, that the function must work on the same string that was sent to it, if we used passing by value the change would not impact the string that was sent from the main function. Notice that when passing by pointer we can also pass pointers to NULL (which is 0). Having the possibility of an uninitialized object is one of the reasons to use this method.
Passing by pointer is not actually real. A pointer is just a numerical value, similar to int, that holds the address of a parameter. When using such a method of parameter passing we are actually passing the address by value.

Passing by reference - exampleshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
void emphasize(std::string &word) 
{
  word.append(3,'!');
}
int main (int argc, char **argv) 
{
  std::string mySentence = "Everything is awesome";
  emphasize(mySentence);
}
We use the same scenario as above, only now we pass by reference. notice that while passing by reference, except for the "&" that declares word as a reference, the syntax is the same as passing by value. Passing by reference allows us to send a link to the original object without using pointers, this means that any change in the object word will also apply on mySentence. This is usually a much safer method since pointers might suggest responsibility for the memory allocation and release and might confuse when attempting to search for bugs. References must be initialized at start, this means that they cannot have a NULL value.
Behind the scenes, the compiler actually looks at references as constant pointers (cannot have their addresses changed). the compiler passes the address by value, similarly to passing by pointer, but it does not allow the programmer access to the address nor does it allow the user to change it. This method should be used in most instances of object parameter passing.

Classesshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>We shall start with a simple class example and move on later to classes that hold a resource.
C++ simple class exampleshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>The class syntax in C++ is somewhat different than in Java. Here is an example of a Point class:
// THIS IS THE DECLARATION FILE OF THE CLASS POINT (Point.h) 
  class Point 
  {
  public:
     Point();
     Point(double xval, double yval);
     void move(double dx, double dy);
     double getX() const;
     double getY() const;
     
  private:
     double x_;
     double y_;
  };               // semicolon at the end of the class declaration
// THIS IS THE IMPLEMENTATION FILE OF CLASS POINT (Point.cpp) 
#include "Point.h"
 
Point::Point():x_(0), y_(0){ }
 
Point::Point(double xval, double yval):x_(xval),y_(yval){}
 
void Point::move(double dx, double dy) 
{
  x_ = x_ + dx;
  y_ = y_ + dy;
}
double Point::getX() const
{
  return x_;
}
double Point::getY() const
{  
  return y_;
}
C++ class syntax vs Java class syntaxshow_sections_edit()){?>name,"edit","action=edit§ion=9")?>
Declarations and implementation are separated - The class definition only contains the declarations of the methods. The actual implementation are listed separately, where each method name is prefixed by the class name. The :: operator separates class name and method name. Note that the separation of declaration and implementation is usually done in cases where you want to export or share the class with another class, and is not required for inner, local private or helper classes.
Semicolon at the end of the class - There is a semicolon at the end of the class declaration. Not placing it will lead to unclear compilation error.
Public and private section - In C++, there are public and private sections, started by the keywords public and private. In Java, each individual item must be tagged with public or private.
Const methods - const functions do not change the state of an object. Good candidates for const are accessors functions (getters). Furthermore const functions cannot be used in a way that would allow you to use them to modify const data. This means that when const functions return references or pointers to members of the class, they must also be const. Also notice, that const methods can only access const methods.

Once declared a const variable, one can only use methods declared const on that variable.
...
const Point p(0,0);
p.getY();         //this is O.K since getY is declared const
p.move(1,1);      //compilation error since move is not declared const
...
Member Initialization Listshow_sections_edit()){?>name,"edit","action=edit§ion=12")?>In C++, we use a member initialization list to initialize class members. The initial value can be any expression. The member initialization list is executed before the body of the function. It is possible to initialize data members inside the constructor body but not advised for the following two reasons:
Implicit call to default constructor - when a data member is itself a class object, not initializing it via the initialization list means implicitly calling its default constructor! If you do initialize it in the body of the constructor you are actually initializing it twice. If your data member is a class with no default constructor, meaning you supplied some constructor that has parameters, you will not be able to pass compilation - think why.
Const members - Const members of a class can only be initialized via member initialization list.
The order the initialization happens is according to the order the member vars are declared (not the order in the member initialization list). It is hence a convention to keep the order of the list as the order of the declaration.
Project Handling and Division into Filesshow_sections_edit()){?>name,"edit","action=edit§ion=12")?>As mentioned above, the declarations and the implementation are defined separately in C++. We place the class declaration file in a header file (e.g X.h) and the class implementation in file X.cpp.
To avoid including a Header twice, we check whether a pre-compiler unique variable is defined. If not we define it and include the header. A convention is to use as a variable the name of the header class. for example:

//Header file of Point class
#IFNDEF _POINT_H_
#DEFINE _POINT_H_
// now the header class declaration as before
//...//
#ENDIF
To compile a project you need to compile each class separately to an object file then to link them. If you are not familiar with those terms please refer to previous practical sessions as they are VERY important to understand ! Make is a tool that can do that work for you. It is the equivalent of ANT if you wish.

In this course, we do not allow writing any implementation in the H file. It is possible but requires some care which we leave to experienced C++ programmers.

Here is an example of a project with 3 cpp files + 2 H file + makefile.
Objectsshow_sections_edit()){?>name,"edit","action=edit§ion=12")?>In C++, object variables hold values, not object references. You simply supply the construction parameters after the variable name.
Example: Point p(1, 2); /* construct p */

If you do not supply construction parameters, then the object is constructed with the default constructor. Time now; /* construct now with Time::Time() */

This is very different from Java. In Java, this command would merely create an uninitialized reference. In C++, it constructs an actual object.

When one object is assigned to another, a copy of the actual values is made. In Java, copying an object variable merely establishes a second reference to the object. Copying a C++ object is just like calling clone in Java. Modifying the copy does not change the original.

Point q = p; /*invokes copy constructor*/
q.move(1, 1); /* moves q but not p */
In most cases, the fact that objects behave like values is very convenient. There are, however, a number of situations where this behavior is undesirable.

When modifying an object in a function, you must remember to use call by reference
Two object variables cannot jointly access one object. If you need this effect in C++, then you need to use references
An object variable can only hold values of a particular type. If you want a variable to hold objects from different subclasses, you need to use pointers
If you want a variable to point to either null or to an actual object, then you need to use pointers in C++
The "this" Pointershow_sections_edit()){?>name,"edit","action=edit§ion=20")?>
Note that, as in Java, this is a pointer to the active object. So for example, when L1 = L2 is executed, L1's member function copy assignment operator is called, so this is a pointer to L1.
We also make use of this for the returned value in the copy assignment operator; the type to be returned is List& so we dereference this, a.k.a return *this.
We would like to establish a way to distinguish between parameters and class variables.This can be done by the this pointer. It is used as a pointer to the class object instance by the member functions.
this pointer stores the address of the object instance, to enable pointer access of the object.
this pointers are not accessible for static member functions.
this pointers are not modifiable.
Operator -> and Operator .show_sections_edit()){?>name,"edit","action=edit§ion=20")?>The pointer-to-member operators, .* and –>*, return the value of a specific class member for the object specified on the left side of the expression. The right side must specify a member of the class. The following example shows how to use these operators:
class Point{.....
}
int main( ){
    Point *p1 = new Point(0,0);
    Point p2(0,0);
    p1->getX();
    (*p1).getX();
    p2.getX();
    (&p2)->getX();
}
The Rule of 3 (Classes that Hold Pointers)show_sections_edit()){?>name,"edit","action=edit§ion=20")?>
A thumb rule in C++ (prior to C++11) for classes that own a resource (such as allocated memory on the heap and files). This rule requires such class to define and implement the following functions:
A destructor
A copy constructor
A copy assignment operator

The Rule of 3 Example
We learned that a class that own a resource must explicitly define three functions, according to the rule of 3. We will now see an example of such classes, a List class and a Link that together implement a linked list object. The List class includes a pointer to a dynamically allocated Link.
The List class own the resource head_, and the responsibility for allocating memory, deleting it and making changes to it belongs to the class. In a similar way, the Link class own the resource for the next_ object in the list.

We will demonstrate declarations of the class's destructor, copy constructor, and assignment operator:

#ifndef LINKED_LIST_H_
#define LINKED_LIST_H_
#include <string>
 
class Link {
/**
 * The linked list is made of Link objects
 * each Link holds a pointer to the next link and its data (a string)
 * The Link object only knows about a single link - it never iterates through the next
 * No deep copy or deep delete - these are the responsibility of the LinkedList.
 */
  private:
    Link *next_;
    std::string data_;
 
  public:
    Link(const std::string &data,  Link *link);   // data passed by reference and link by pointer
    Link(const Link &aLink);
    virtual ~Link();
    void setNext(Link *link);
    Link * getNext() const;
    const std::string & getData() const;
    // Note - ‘const’ in Functions Return Values: to prevent   list.getData()[2] = 'b'
};
 
class List {
  private:
    Link *head_;
    Link *copy() const;
    void clear();
 
  public:
    List(); // constructor
 
    const Link * getHead() const;
    void insertData(const std::string &data);
    void removeFirst();
 
    List(const List &aList);
    virtual ~List();
    List & operator=(const List &L);
};
#endif /*LINKED_LIST_H_*/
#include "LinkedList.h"
 
/*****************************************************************
 * Link Implementation [Internal class]
 ****************************************************************/
 
/** constructor
 * A link is a single link in the list - this class never iterates beyond the single link.
 * No deep copy, deep delete - these are all responsibilities of the List.
 */
Link::Link(const std::string &data,  Link *link) : data_(data)
{
   setNext(link);
}
void Link::setNext(Link *link)
{
   next_=link;
}
Link * Link::getNext() const
{
   return next_;
}
const std::string & Link::getData() const
{
   return data_;
}
 
/** destructor
 *  No deep deletion for a simple link - this allows us to remove a single link from a list
 */
Link::~Link() { }
 
/** copy constructor
 *  This only copies the data of the link - no deep copy and no copy of next to avoid sharing Links between lists.
 */
Link::Link(const Link &aLink)
{
   data_=aLink.getData();
   next_=nullptr;
}
/*****************************************************************
 * List Implementation
 ****************************************************************/
 
/**
 * Constructor.
 * Builds an empty list.
 */
List::List() : head_(nullptr)
{
}
const Link * List::getHead() const
{
   return head_;
}
void List::insertData(const std::string &data)
{
   head_ = new Link(data, head_);
}
/**
 * removes the current head
 * After the method head_ points to the next element
 */
void List::removeFirst()
{
  if (head_ != nullptr) {
    Link *tmp = head_;
    head_ = head_->getNext();
    delete tmp;
  }
}
/**
 * Destructor: "deep delete"
 */
List::~List()
{
  clear();
}
/**
 * Clear all content (delete all links)
 */
void List::clear()
{
  while (head_ != nullptr) {
    removeFirst();
  }
}
/**
 * deep copy of this list (allocates links)
 */
Link * List::copy() const
{
  if (getHead() == nullptr) {
    return nullptr;
  } else {
    Link *head = new Link(*getHead());
    Link *next = head;
    // @inv: next points to last node in new list
    // origPtr points to node in original list that is not yet copied
    for (Link *origPtr = getHead()->getNext(); origPtr != nullptr; origPtr = origPtr->getNext()) {
      next->setNext(new Link(*origPtr));
      next = next->getNext();
    }
    return head;
  }
}
/**
 * Copy Constructor:deep copy of aList
 */
List::List(const List &aList)
{
    head_ = aList.copy();
}
/**
 * Assignment Operator
 */
List & List::operator=(const List &aList)
{
  // check for "self assignment" and do nothing in that case
  if (this == &aList) {
    return *this;
  }
  clear();
  head_ = aList.copy();
  // return this List
  return *this;
}
 
int main() {
  return 0;
}
Destructor show_sections_edit()){?>name,"edit","action=edit§ion=20")?>An object's destructor function is called when that object is about to "go away"; i.e., when:
A class instance (a value parameter or a local variable) goes out of scope, or
The dynamically allocated storage pointed to by the pointer is freed by the programmer using the delete operator

The main purpose of the destructor function is to free any dynamically allocated storage pointed to only by a data member of that object. Note that it is up to the programmer to ensure that no other pointers are pointing to that storage.
For example, consider the following function,

void f(List list) {// list is passed by value only for sake of this example
  List *p = new List();
  while (...) {
    List list1;
    ...
  }
  delete p;
}
In this example, the scope of value parameter list is the whole function; list goes out of scope at the end of the function (line 8). So when function f ends, list's destructor function is called. (Note: if f had one or more return statements, list's destructor function would be called when a return was executed). Note we passed list by value only for the sake of the example as it is not recommended. It should have been passed by reference.

The scope of variable list1 is the body of the while loop (lines 4 to 6). list1's constructor function is called at the beginning of every iteration of the loop, and its destructor function is called at the end of every iteration of the loop. Note that if the loop included a break or continue statement, the destructor would still be called.

Variable p is a pointer to a List. When a List object is allocated using new at line 2, that object's constructor function is called. When the storage is freed at line 7, the object's destructor function is called (and then the memory for the List itself is freed).

Question: Is a destructor function of a reference parameter called at the end of the function?
A reference parameter's destructor function is not called at the end of the function because the corresponding actual parameter refers to the same object. The object does not "go away" when the function ends, so its dynamically allocated storage should not be freed.

Destructor functions are defined using a syntax similar to that used for the constructor function (the name of the class followed by a double colon followed by the name of the function) and are always declared as virtual. For example, the definition of the List destructor function would look like this:

/**
  * Destructor: "deep delete"
  */
 List::~List()
 {
   clear();
 }
/**
 * removes the current head
 * After the method head_ points to the next element
 */
void List::removeFirst()
{
  if (head_ != nullptr) {
    Link *tmp = head_;
    head_ = head_->getNext();    
    delete tmp;
  }
}
/**
 * Clear all content (delete all links)
 */
void List::clear()
{
  while (head_ != nullptr) {
    removeFirst();
  }
}
If you don't write a destructor function for a class that includes pointers to dynamically allocated storage, your code will still work, but you will probably have some storage leaks.
Copy Constructor show_sections_edit()){?>name,"edit","action=edit§ion=20")?>An object's copy constructor is called (automatically, not by the programmer) when it is created, and needs to be initialized to be a copy of an existing object. This happens when an object is:
passed as a value parameter to a function,
....
Point q(2,2);
Point p(0,0);
p.MoveTo(q);
...
returned (by value) as a function result,
declared with initialization from an existing object of the same class.
...
A(Point p){
Point temp=p;    //copy constructor called here to copy p
Point temp2(p);  //copy constructor called here to copy p
...

Here are two functions that illustrate when copy constructors are called:
List f( List &list ) {
   List tmp1 = list;    // copy constructor called here to copy list
   List tmp2(list);     // copy constructor called here to copy list
   ...
   return tmp1;      // copy constructor called here 
                     // to copy tmp1 to list2
}
 
int main() {
   List list1;
   ...
   List list2 = f(list1);
}
On line 12, variable list1 is passed as a reference parameter to function f. The corresponding formal parameter is list.

On line 2, variable tmp1 is declared to be a List, initialized to be the same as variable list. When line 2 is executed, tmp1's copy constructor is called to initialize tmp1 to be a copy of list. Similarly, when line 3 is executed, tmp2's copy constructor is called to initialize tmp2 to be a copy of list.

On line 5, variable tmp1 is returned as the result of calling function f. When line 5 is executed, the copy constructor is called to make a copy of tmp1 to be returned. (Later, that copy is used as the right-hand side of the assignment on line 12.)

A Copy Constructor must exist on classes that manages resources
If you don't write a copy constructor, the compiler will provide one that just copies the value of each data member (this is sometimes called a shallow copy). If some data member is a pointer, this causes aliasing (both the original pointer and the copy point to the same location), and may lead to trouble.

The Copy Constructor Declaration
The copy constructor has one argument: its type is the class, and it is a const reference parameter. The argument is the object that the copy constructor is supposed to copy. For example:
class List {
   public:
   ....
   List(const List &aList);// copy constructor
   ...
};
The Copy Constructor Definition
The definition of the copy constructor (the actual code for the function) should be put in a ".cpp" file, along with the code for the other class member functions. The copy constructor should copy the values of all non-pointer data members, and should copy the objects pointed to by all pointer data members (this is sometimes called a deep copy). For example:
/**
 * Copy Constructor:deep copy of aList
 */
List::List(const List &aList)
{
  head_ = aList.copy();
}
/**
 * deep copy of this list (allocates links)
 */
Link * List::copy() const
{
  if (getHead() == nullptr) {
    return nullptr;
  } else {
    Link *head = new Link(*getHead());
    Link *next = head;
    // @inv: next points to last node in new list
    // origPtr points to node in original list that is not yet copied
    for (Link *origPtr = getHead()->getNext(); origPtr != nullptr; origPtr = origPtr->getNext()) {
      next->setNext(new Link(*origPtr));
      next = next->getNext();
    }
    return head;
  }
}
Copy assignment operatorshow_sections_edit()){?>name,"edit","action=edit§ion=20")?>Consider this example:
List list1, list2;
...
list1 = list2;  // this assignment is OK
By default, class assignment is just a field-by-field assignment (i.e., a shallow copy is done). For example, the above assignment is equivalent to:
list1.data_ = list2.data_;
list1.next_ = list2.next_;
Of course, the field assignments could not be written outside a List member function, since they are private fields; however, they illustrate the effect of the assignment list1 = list2.
If a class includes pointer fields, the default assignment operator causes aliasing, which lead to trouble! The default assignment can also cause storage leaks when the class has a pointer field.

To prevent these problems, you should always override the assignment operator as a class member function for a class with a pointer field You may either define the operator to do a deep copy or make it private. The declaration of the member function looks like this for the List class:

List & operator=(const List &aList);
When the assignment list1=list2 is executed, list1's member function operator is called, and list2 is passed as the argument to that function. It can be actually be written like this list1.operator=(list2).

Note that List's operator function returns a List. This is to permit chained assignment, for example: list1 = list2 = list3; When this statement is executed, the expression list2 = list3 is evaluated first; the result of evaluating that expression is used as the right-hand side of the assignment to list1. The operator function returns its result by reference (that's what the ampersand means). This is done for efficiency, to prevent the List copy constructor being called to make a copy of the returned value. So this can be written as list1.operator=(list2.operator=(list3)).

Note that copy assignment operator differs from the copy constructor in three important ways:

The object being assigned to has already been initialized; therefore, if it has a pointer field, the storage pointed to must be freed to prevent a storage leak.
It is possible for a programmer to assign from a variable into itself; for example: list1 = list1. The copy assignment operator code must check for this case, and do nothing.
The copy assignment operator code must return a value.
Here is the definition of copy assignment operator for the List class. It should always include the following 4 sections:

check assignment to self
clear existing data members
copy data member from other
return this
/**
 * Assignment Operator
 */
List & List::operator=(const List &aList)
{
  // check for "self assignment" and do nothing in that case
  if (this == &aList) {
    return *this;
  }
  clear();
  head_ = aList.copy();
  // return this List
  return *this;
}
Exception safety in copy assignment operatorshow_sections_edit()){?>name,"edit","action=edit§ion=20")?>Suppose the aList.copy() expression yields an exception (either because there is insufficient memory for the allocation or because Link's copy constructor throws one), the List will end up holding a pointer to a deleted Link. Such pointers are toxic. You can't safely delete them. You can't even safely read them. About the only safe thing you can do with them is spend lots of debugging energy figuring out where they came from.
Happily, making copy assignment operator exception-safe typically renders it self-assignment-safe, too. In many cases, a careful ordering of statements can yield exception-safe (and self-assignment-safe) code. Here, for example, we just have to be careful not to delete the List pointed by head_ until l.copy() completed successfully.

/**
 * Assignment Operator
 */
List & List::operator=(const List &aList)
{    
    // check for "self assignment" and do nothing in that case
    if (this == &aList) {
        return *this;
    }
    Link *temp = aList.copy();        // make temp point to a copy of aList           
    clear();            // delete the original List
    head_ = temp;    
  
  // return this List
  return *this;
}
Now, if aList.copy() throws an exception, head_ remains unchanged. Even without the identity test, this code handles assignment to self, because first we make a copy of the new List, then delete the original List and finally point to the copy we made. It may not be the most efficient way to handle self-assignment, but it does work.

If you're concerned about efficiency, you could put the identity test back at the top of the function. Before doing that, however, ask yourself how often you expect self-assignments to occur, because the test isn't free. It makes the code (both source and object) a bit bigger, and it introduces a branch into the flow of control, both of which can decrease runtime speed. The effectiveness of instruction prefetching, caching, and pipelining can be reduced, for example.

Additional links:show_sections_edit()){?>name,"edit","action=edit§ion=20")?>
examples1
examples2
Pointers and Functions
The C++ 'const' declaration
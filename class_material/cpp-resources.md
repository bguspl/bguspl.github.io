Resource Allocation and Ownership

Objects are usually complex entities; when they are constructed, they sometime acquire a resource. If designed carefully they might do with one resource, otherwise (and this is sometime unavoidable) they will acquire several resources. When an object is destructed, the resources associated with it must be released.

What is a Resource
Anything that respects the Release/Acquire protocol. For example, consider files (open/close), memory allocation (allocate/free), locks (acquire/release).
Construction and destruction of objects
Construction of an object is the process of:
Allocating - Allocating enough memory to hold the object's state
Initialization - Initializing the internal state of the object. This involves 2 sub-steps:
Acquiring Resources - Acquiring the necessary resources which the object requires
Consistent state - Making sure that the initial state of the object is valid according to the class invariants.

When a new object is instantiated, the RTE allocates the needed memory and calls the constructor of the class. It is not different from a call to a method. However, the constructor is somewhat different from a normal method invocation, as we don't consider the object to exist until the construction is finished. For example, we do not want to access this of the object in the constructor. In the same manner, we do not want to throw exceptions from the constructor.
When an object is constructed, resources are allocated (either explicitly, by the object's constructor, or implicitly – the memory allocated to hold the object's state). As in all resource allocations, these resources should be deallocated. We say should, since in C++ it is the programmer's responsibility to do so. The process of releasing the resources acquired by a process is called the destruction of the object.

Destructing an object includes:

Releasing resources - Releasing all of the resources acquired by the object during the object's lifetime and construction.
Releasing memory - releasing all the memory allocated for the object

The process of destructing an object is shared between the RTE and the programmer. The programmer is responsible to release any resources acquired by the constructor (and during the life time of the object), and the RTE is responsible to release the memory allocated to hold the state of the object.
Destructing an object can be done either implicitly or explicitly. An object is implicitly destructed if the object is allocated on the stack. When the program exits the scope of this object, the RTE will destruct the object. On the other hand, an object allocated on the heap is destructed explicitly, by the programmer, by using the delete operator on a pointer to the object.

The destruction of an object is performed in the following order:

The RTE calls a special function of the object, namely, the destructor. It is the responsibility of the programmer to implement such a function, and release all resources acquired by the object during the object's lifetime.
The RTE releases the memory used to hold the object's state (either from the stack, or the memory blocked on the heap returned by new).
The LifeCycle of an Object
We say that an object is "born" after its constructor (which is in charge of the object's construction) successfully terminated. We say that an object "retired" after its destructor successfully terminated.
We further denote an object as "alive" during the time frame following its construction and just before its destruction.

→ We consider an object as valid only as long as the object is alive.

Classes Which Own Resources
In the following example, the IntArray holds an array of ints. The array is dynamically allocated by the object during its construction which normally means that the array is owned by the object (i.e., it is the object responsibility to manage the memory of this array and make sure that no memory leaks will happen).

The following code shows an implementation of the IntArray class. For brevity reasons, the class is both declared and implemented on the same file - note that usually it will not be the case (as will be described in the next lecture)

downloadtoggle
33 lines ...
#include <iostream>
 
using namespace std;
 
class IntArray {
    int *array_;
    int length_;
    
public: 
    IntArray(int length) {
        array_ = new int[length];
        length_ = length;
    }
    
    void set(int i, int v) {
        array_[i] = v;
    }
    
    int get(int i) const {
        return array_[i];
    }
    
    int length() const {
        return length_;
    }
    
    void print() const {
        for (int i=0; i<this->length(); i++) {
            cout << get(i) << " ";
        }
        
        cout << "\n";
    }
};
Although this class seems trivial, it has some very serious problems.

The Rule of 3
The rule of 3 is is a rule of thumb in C++ (prior to C++11) that claims that each class that own a resource must explicitly define all three:
destructor
copy constructor
copy assignment operator
Destructors
Each time we will use an object of this class we will have a memory leak as the array owned by the class is never freed:

downloadtoggle
15 lines ...
void use_int_array() {
   IntArray array = IntArray(10);
    
   for (int i=0; i<array.length(); i++) {
       array.set(i, i);
   }
   
   array.print();
}
 
int main(int argc, char** argv) {
  use_int_array();
  use_int_array();
  use_int_array();
  return 0;
}
The resulting code will create a memory leak of 30 ints - make sure you understand why!

Since the class owned the array resource it is responsible for releasing it. In order to do so we need to add an appropriate destructor to our IntArray class:

download
class IntArray {
  //...
  public: 
  //...
  virtual ~IntArray() {
      delete[] array_;
  }
}
Remarks:

Destructors should always be declared virtual. We will discuss this when talking about inheritance.
If you don't find a good reason why not to, declare each method as virtual. (We will discuss this again when talking about inheritance.)
Copy Constructors And The Copy Assignment Operator
Unless explicitly defined, C++ will automatically generate for each class a default copy constructor and copy assignment operator .

Copy constructor is a special type of constructor which receives a single argument of the same type of the class and copies the values from this argument into this. The default copy constructor perform shallow copying i.e., in our class we will have implicitly the following constructor:

download
class IntArray {
  //...
  public: 
  //...
  IntArray(const IntArray& rhs) {
    array_ = rhs.array_;
    length_ = rhs.length_;
  }
}
Copy Assignment Operator is a method that tells c++ what to do when someone attempt to assign a new value to an already initialized value. Like in the case of copy constructor, C++ will automatically generate a default copy assignment operator for us that perform shallow copy:

downloadtoggle
9 lines ...
class IntArray {
  //...
  public: 
  //...
  IntArray& operator=(const IntArray& rhs) {
    array_ = rhs.array_;
    length_ = rhs.length_;
    return *this;
  }
}
C++ uses the copy constructor and the copy assignment operator automatically:

downloadtoggle
12 lines ...
IntArray return_int_array() { ... }
void accept_int_array(IntArray array) { ... }
 
IntArray test() {
  IntArray x(10); //initialization - calls regular constructor
  IntArray y = return_int_array(); //initialization - no additional constructor is called here
  x = y; //calls the assignment operator
 
  accept_int_array(x); //calls the copy constructor to create a copy to send to the method
                       //then when the method completed calls the destructor for the copy
 
  return x; //copy constructor may get called
}
This behavior can lead to the Memory Addresses Aliasing problem which is a memory leak problem that happens when an object leaks the pointers to the resources it owns to other objects. This problem happen on the code above in line 3 of the test function. Note that after line 4 both x and y internal arrays point to an invalid (already freed) address - make sure you understand why!

This behavior can be solve by creating a custom copy constructor and copy assignment operator:

downloadtoggle
30 lines ...
class IntArray {
  //...
 
  void clean() { 
    delete[] array_; 
    array_ = nullptr;
    length_ = -1;
  }
  
  void copy(const IntArray& other) {
        length_ = other.length_;
        array_ = new int[other.length_];
        
        for (int i=0; i<length(); i++) {
            array_[i] = other.array_[i];
        }
  }
  public: 
  //...
  IntArray(const IntArray& rhs) {
    copy(rhs);
  }
 
  IntArray& operator=(const IntArray& rhs) {
    if (&rhs != this) {
      clean()
      copy(rhs);
    }
    return *this;
  }
}
Move semantics (or The rule of 5)
Following the rule of 3 is required in order to create a "correct" program (i.e., without the inherent bugs that were explained above), starting from 2011 and the introduction of rvalue references the rule of 3 was updated with two additional steps and therefore became the rule of 5. The rule of 5 claims that if a class defines one (or more) of the following it should probably explicitly define all five:
destructor
copy constructor
move constructor
copy assignment operator
move assignment operator

The additional required methods are the move constructor and the move assignment operator, note that while the rule of 3 is required for the correctness of the program, the additional 2 methods are required for the efficiency of the program and therefore can be considered optional.
The Move Constructor and The Move Assignment Operator
Consider the following example code:
download
IntArray return_int_array() { ... }
void accept_int_array(IntArray array) { ... }
 
void test() {
  accept_int_array(return_int_array()); 
  IntArray x(10);
  x = return_int_array();
}
In the above code, on the first line of the test function - the function return_int_array create an int array and return it, this int array is then kept to a temporary location (which means it is an r-value) and then in order to pass the created int array to the accept_int_array function c++ creates a copy of the temporary variable using the copy constructor, sends it to the function and then when the function returns c++ will destruct both the temporary and the temporary copy. This is really inefficient as it allocate the internal array twice, fill it and then free it for no good reason. The third line of the test method demonstrate the same process only now with the copy assignment operator.

Can we do better? Yes! using C++'s rvalue references.

rvalue References
rvalue references are, like the name suggested - references to rvalues. They are defined using &&, i.e., int&& i is an rvalue reference to int. They allow us to create functions that can use the temporary variables without making a copy of them. functions receiving rvalue references should take (steal) what they want from the referenced object but not the referenced object itself (as it will be immediately destructed once the method completes)
Since we know that the temporary variable that is sent to the copy constructor/assignment operator can never be referenced again we could have just "steal" or "move" its resources instead of fully copy them. In order to accomplish that C++11 introduce the concept of Move constructors and Move assignment operators. These methods typically "steal" the resources held by their argument rather than make copies of them, for example in order to make the code above more performant, we could have add the following methods to our class:

downloadtoggle
23 lines ...
class IntArray {
  //...
  void steal(IntArray& other) {
    length_ = other.length_;
    array_ = other.array_;
 
    //the next line complete the stealing, without this other can free the array_ 
    //when it will get out of scope
    other.array_ = nullptr;
  }
  public: 
  //...
  IntArray(IntArray&& rhs) {
    steal(rhs);
  }
 
  IntArray& operator=(IntArray&& rhs) {
    if (&rhs != this) {
       clean();
       steal(rhs);
    }
    return *this;
  }
}
C++ Templates
Templates are the foundation of generic programming in C++. You can think of a template as a way to tell the compiler how to generate a class, struct or a function given some compile time variables. The most common compile time variables are types. Templates are a compile time mechanism in C++ that is Turing-complete, meaning that any computation expressible by a computer program can be computed, in some form, by a template metaprogram prior to runtime. This makes them both powerful and complex, therefore, for the purpose of this course we will learn only the basic of templates in C++.
Templates are defined using the template keyword, afterwhich are a list of compile time variables surrounded by angle brackets (e.g., template <typename A, typename B> defines a template with two compile time variables A and B both are name of types like int, long or Foo). Following the template declaration line you may write a class, struct or function which are the subject of this template.

downloadtoggle
34 lines ...
template <typename T>
T argmax(T a, T b) {
  if (a > b) return a;
  return b;
}
 
template <typename X, typename Y> 
class Pair {
private:
  X &x_;
  Y &y_;
public: 
  Pair(X x, Y y): x_(x), y_(y) {}
  const X& getX() const {
    return x_;
  }
  const Y& getY() const {
    return y_;
  }
};
 
//usage example
int main() {
  int i1=10, i2=20, i3;
  double d1=1.23, d2=0.07, d3;
 
  i3 = argmax(i1, i2); //the compiler will generate a max variant which has T = int
  d3 = argmax(d1, d2); //the compiler will generate a max variant which has T = double
  
  Pair<int,int> p_int(i1, i2); //the compiler will generate a Point variant with X = int, Y = int;
  Pair<double,double> p_double(d1, d2); //the compiler will generate a Point variant with X = double, Y = double;
  Pair<int,double> p_mixed(i3, d3); //the compiler will generate a Point variant with X = int, Y = double;
  
  return 0;
}
RAII - Resource Acquisition Is Initialization
Resource Acquisition Is Initialization or RAII, is a C++ programming technique which binds the life cycle of a resource to the lifetime of an object. RAII guarantees that the resource is available to any function that may access the object (resource availability is a class invariant). It also guarantees that all resources are released when the lifetime of their controlling object ends, in reverse order of acquisition. Likewise, if resource acquisition fails (the constructor exits with an exception), all resources acquired by every fully-constructed member and base subobject are released in reverse order of initialization. This leverages the core language features (object lifetime, scope exit, order of initialization and stack unwinding) to eliminate resource leaks and guarantee exception safety.
RAII can be summarized as follows:

encapsulate each resource into a class
the constructor acquires the resource and establishes all class invariants or throws an exception if that cannot be done
the destructor releases the resource and never throws exceptions
always use the resource via an instance of a RAII-class
Move semantics make it possible to safely transfer resource ownership between objects, across scopes while maintaining resource safety.
Classes with open()/close(), lock()/unlock(), or init()/copyFrom()/destroy() member functions are typical examples of non-RAII classes

The following is an example of a simple RAII smart pointer template that uses reference counting
downloadtoggle
93 lines ...
#include <iostream>
using namespace std;
 
template <typename T>
class SmartPointer {
    T *ptr_;
    int *refc_;
    
    void link(const SmartPointer<T>& rhs) {
        ptr_ = rhs.ptr_;
        refc_ = rhs.refc_;
        (*refc_)++;
    }
    
    void steal(SmartPointer<T>& rhs) {
        ptr_ = rhs.ptr_;
        refc_ = rhs.refc_;
        rhs.ptr_ = nullptr;
        rhs.refc_ = nullptr;
    }
    
    void clean() {
      if (refc_==nullptr) return;
        
      if (*refc_ == 1){
          delete ptr_;
          delete refc_;
          ptr_ = nullptr;
          refc_ = nullptr;
      }else {
          (*refc_)--;
      }
    }
 
public:
    
    SmartPointer(T *ptr) {
        ptr_ = ptr;
        refc_ = new int(1);
    }
    
    SmartPointer(const SmartPointer<T>& rhs) {
        link(rhs);
    }
    
    SmartPointer<T>& operator=(const SmartPointer<T>& rhs) {
        if (&rhs != this) {
          clean();
          link(rhs);
        }
        return *this;
    }
    
    SmartPointer(SmartPointer<T>&& rhs) {
        steal(rhs);
    }
    
    SmartPointer& operator=(SmartPointer<T>&& rhs) {
        clean();
        steal(rhs);
        return *this;
    }
    
    virtual ~SmartPointer() {
        clean();
    }
    
    T& get() {
        return *ptr_;
    }
    
    void set(const T &v) {
        *ptr_ = v;
    }
    
};
 
SmartPointer<int> create() {
  SmartPointer<int> p(new int(42));
  return p;
}
 
void inc(SmartPointer<int> p) {
  p.set(p.get() + 1);  
  //when leaving this function p's refc_ will be decreased using the destructor
}
 
void use_int_pointer() {
  SmartPointer<int> pointer = create(); //move constructor will get called stealing the returned pointer - keeping refc = 1
  cout << "the pointer value is " << pointer.get() << endl;
  inc(pointer); //when entering the function the refc will be increased via the copy constructor
  cout << "and now the pointer value is " << pointer.get() << endl;
} // <-- the pointer will automatically get free'd when leaving this scope
  // even when exception is thrown!!!
As shown in the use_int_pointer example we define the SmartPointer on the stack but it manages a pointer on the heap, the SmartPointer<int> class wraps the raw int pointer and manages a reference count to the number of SmartPointer<int> instances that holds the same raw pointer. The pointer will automatically get free'd when there is no function referencing it anymore - even if an exception is thrown;

Smart Pointers
The example we saw above is so important and idiomatic that C++ added support for these kind of pointers (a.k.a., Smart Pointers) directly into its standard library:
Smart pointers are wrappers around raw pointers that act much like the raw pointers they wrap, but that avoid many of their pitfalls. You should therefore prefer smart pointers over raw pointers. Smart pointers can do virtually everything raw pointers can, but with far fewer opportunities for error.

There are several smart-pointer types available in C++ (11+) but we will only discuss the most common one: the std::shared_ptr (but you are encouraged to read about std::unique_ptr too).

std::shared_ptr
The shared_ptr type is a smart pointer in the C++ standard library that is designed for scenarios in which more than one owner might have to manage the lifetime of the object in memory. After you initialize a shared_ptr you can copy it, pass it by value in function arguments, and assign it to other shared_ptr instances. All the instances point to the same object, and share access to one "control block" that increments and decrements the reference count whenever a new shared_ptr is added, goes out of scope, or is reset. When the reference count reaches zero, the control block deletes the memory resource and itself.
Its usage have some similarities to the SmartPointer we saw above:

download
void find_cat(shared_ptr<Dog> d) {...}
 
  shared_ptr<Dog> bingo_p(new Dog("Bingo"));
  bingo_p->bark(); //whoof whoof
  find_cat(bingo_p); //bingo_p will be free'd after this line
Arrays and Objects
We have seen how to allocate arrays of primitive types in C++, and saw that each cell of the array is initialized using the default constructor. This holds also for objects. Consider the following code:
download
I shape_array[42];
 
for (int i=0; i<42; i++)
{
        shape_array[i] = Shape();
}
Each cell of the array is initialized twice! The first time is when the array is constructed, by using the default constructor, and the second time by using the assignment operator. Moreover, there are a total of $42*2=84$ Shape objects created in total! $42$ for the array's cells, and $42$ inside the loop.
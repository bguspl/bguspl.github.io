Practical Session #02 – C++ Memory Handling
General
The main goal for this practical session is to learn how C++ types are represented in memory, how to use pointers and how to keep out of trouble.
Memory Model
Each process running on the system has it's own memory space. This memory space is the range of addresses that can be described in a single word. For example, In 32 bit computers, each process can access any address from 0 to 2^32 - 1. A total of 2^32 bytes. In this memory space the process puts any information it needs. This include compiled code, constants and variables it needs.
In this course we will focus As on two "kinds" of memory available to a process:

Stack: Used for static memory allocation. Rather small temporary storage for function-local data. Every variable you declare inside a function lives on the stack. Once you leave the scope of the function, the stack variables are discarded. In each activation frame (function scope) one can access only memory allocated within that scope.
Heap: Used for dynamic memory allocation and it has a big storage for data. Variables allocated on the heap have their memory allocated at run time and accessing this memory is a bit slower. You can allocate a block at any time and free it at any time. This makes it much more complex to keep track of which parts of the heap are allocated or free at any given time

ps2_image1.gif
Pointers
Pointers are variables whose values are memory locations – addresses of some data.
Java's references
download
String s = new String("a string");
The actual value of the object (the string "a string" and some additional data)

is on the heap. The value of the variable 's' is the location of the object on the

heap (But you as a programmer don't have access to this value).

download
String s = null;
Here 's' is a variable which knows how to hold a value of the form 'address of String', but

it's actual value is 'null', which means "I don't have the address of any meaningful data".

Primitive types are not pointers:

download
int i = 5;

The value of the variable 'i' is the number 5.

You can not have a variable that 'points' to a primitive type.

C++ pointers
C++ is different. In C++ you have a choice. In C++ you have access to actual memory locations. C++ is dangerous. It's like the second amendment, which gives you the power to bear arms, but does not prevent you from shooting your own leg.
The null value in C++ is the number 0 (memory address 0).

Declaring a pointer:

download
int *p_i;
float *p_f;
Accessing the values of a pointer:

download
int j = 5; 
int *p_j = &j;  // p_j value is the address of the variable j
 
cout << p_j << endl;  // prints the address of variable j
cout << *p_j << endl; // prints the value of variable j (5)
Example of bad but possible assigning to pointers

download
int *p_i = (int *) 923451; // p_i points to the int at memory location 923451
cout << p_i << endl;  // prints the number (address) 923451
cout << *p_i << endl; // prints the value of the integer at address 923451 
                      //(might be garbage, might crash the program).
The operators * and &
When dealing with pointers, the operator * has 2 different meanings, depending on the context in which it appears.
In variable definition, it means "this variable is a pointer".
Once the variable is defined, it means "the value at the address pointed to by the variable".

The operator & is "the address of the given variable".
[Note that in different contexts, these operators can represent completely different operations. For example, * is also the multiplication operator, and & is the bitwise and operator (as well as the reference operator, which we'll learn later). In addition, C++ supports operator overloading, which allow you to redefine these and other operators for the classes you create].

Comparing pointers:

download
int j = 5; 
int i = 5;
int *p_j = &j;  // p_j value is the address of the variable j
int *p_j2 = &j; // p_j2 value is the address of the variable j also
int *p_i = &i;  // p_i value is the address of variable i
 
if (p_j == p_j2) { cout << "1" << endl; } 
if (p_j == p_i)  { cout << "2" << endl; } 
if (*p_j == *p_i) { cout << "3" << endl; }
(what will be printed??)
You can also have a pointer to a pointer:

downloadtoggle
9 lines ...
int num = 5; 
int *x = &num;
int **y = &x; 
 
double num2 = 2.5;
double* z = &num2;
 
cout << y<< endl;  // this is the memory location of x
cout << *y << endl; // this is the memory location of num
cout << **y << endl; // 5
p2p.jpg
Illustration of the stack as a result of the above code
example_stack.bmp
Pointers arithmetic
A pointer is just a 32bit (or 64bit) number. So why do we need different types of pointers (a.k.a: pointer to int, pointer to char, etc.) ?
Type safety: telling the compiler to check that we meant what we wrote is always a good idea (recall the use of generics in java).

As a side effect, we need to discuss pointer arithmetic.

The operators +,+=,++,-,-=,-- are all defined for pointers. But what does it mean to add a number to a pointer? Let's see:

download
int j = 5;
int *i = &j;
cout << "i   = " << i << endl;
i+=1;
cout << "i+1 = " << i << endl;
i = 8048900
i+1 = 8048904

The difference is 4. Why?

Adding 1 to a pointer means "make it point to the next element of the same type", so adding 1 to a pointer to int actually adds 1*sizeof(int) (which happens to equal 4 on my computer). Similarly, adding 5 to a pointer of type float actually adds 5*sizeof(float). To add exactly 1 (byte) to a pointer use the type char since sizeof(char) == 1.

Arrays
Arrays in C++ are const pointers with a special behavior.
The special thing about them is that they point to the start of a memory block, and you can not make an array pointer point to anywhere else. This memory block is on the stack. (However, you can treat "non-special" pointers as arrays as well, and we'll see that below. We also strongly recommend you not to use arrays. Use vector instead of arrays)

Here is how you declare an array of 10 integers in C++:

download
int A[10];
Let's understand what this line means:

A memory block big enough for holding 10 integers (10*sizeof(int)) is allocated on the stack
A holds the address of the start of this block.

Here is how you access an array element:
download
int A[10];
A[3] = 9;
int j = A[3];
The meaning of these lines is:
The first line declares an array of 10 integers. A is a pointer holds the address of the first element.
The second line puts the value 9 in the 4th place in the array. Note that the location of the 4th place in the array is "the value of A plus 3*sizeof(int)", or using pointers arithmetic:A+3. So this line could be written as *(A+3) = 9;.
The third line reads a value from the 4th place in the array. Note that this line could be written as int j = *(A+3).

Notice that unlike Java, C++ arrays don't have a "length" properties: C++ arrays are just special pointers, and they don't know the size of the memory block they are pointing to. You need to track this information yourself if need it.
For this reason, C++ arrays will not give you an "array out of bound" exception if you try to access elements beyond the allocated memory. This is very dangerous: the operation will probably work, but write or read things you didn't intend to.

For example, what does the following lines do?

download
int A[10];
A[100] = 9;
A[-2]  = 500;
Regular pointers can also be used like arrays. You just have to make sure you actually try and access a reasonable memory location (Assume we have a 32 bit address length).:

downloadtoggle
13 lines ...
double A[10];
double s=0;  //just to clear size of a box
double *pi;
double j;
 /* The stack at this point is depicted in the image below.*/
 
pi = &j;
pi[3] = 5;   /* Unreported problem. What will happen here?
 
pi = A;
pi[3] = 5; // OK.  Accessing the 4th element of A.
pi++;      // now pi points to the second element of A
pi[0] = 1; // assign 1 to the second element of A.
/* Question: What will happen, if s is an int and not double */
Illustration of the stack as a result of the above code
stack3.jpg

The file Pointers.cpp has some examples of using pointers and arrays (It also introduces "structs", which we didn't learn yet. structs are almost classes, and we'll get to them later in the course).

Strings vs. char*
In the previous practical session we've seen the std::string object.
This is similar to a Java string: it's an instance of a class representing a string.

C++ has another way of representing string: an array of characters, where the last character in the string has the value 0.

download
const char *c = "abcdefg";
std::cout << c[3] << endl; // what will be printed?
std::cout << &(c[3]) << endl; // what will be printed? this case is special, as cout has a different behavior when it comes to chars. 
                                                 //Instead of printing the address, it will print the string from the 4th cell.
Next, we see how to convert std::strings() to char* strings and back:

downloadtoggle
14 lines ...
std::string s("abcdefg");   // what goes on in this line?
 
// c_str() returns a char* representation of the string
// NOTE: it returns a pointer to an internal representation.
// this is GOOD: because you don't have to manage this memory yourself
// (more on memory management below)
// this is also BAD: there's no guarantee you will keep pointing 
//to a valid c-string after additional calls
//      to methods of the string
 
// c_str() returns a char* representation of the string.
const char *cs = s.c_str();             
 
// this is actually very similar to the first line.
std::string s2 = std::string(cs);
The commandline arguments to a C++ programs are passed as c-strings (char* string) to the main function,
as in the following code sample: commandline.cpp.

c++ References
C++ supports a cencept called references. There are two types of references: lvalue reference, which is commonly used, and rvalue references.
lvalue and rvalue
Every expression in C++ is either an lvalue or an rvalue.
lvalue refers to an object that persists beyond a single expression.
rvalue refers to a temporary value that does not persist beyond the expression that uses it.

A good indication should be the location of the expression. The rvalue can not appear on the left side of an expression. Example if lvalues and rvalues.
download
int main(int argc, char** argv)
{
  int i = 5; // i is lvalue, 5 is rvalue
  int j = 4; // j is lvalue, 4 is rvalue
  char *s = "test"; // s is lvalue, "test" is rvalue
  j = 5 * 22; // j is lvalue, 5 * 22 is rvalue
  i = i + j; // i is lvalue, i + j is rvalue
}
The compiler might use the lvalue / rvalue definitions to help you identify issues in your code. For example, the following code:

download
int foo() {return 2;}
 
int main(int argc, char** argv)
{
  foo() = 2;
  return 0;
}
Will return the following error by the compiler:

download
test.c: In function 'main':
test.c:8:5: error: lvalue required as left operand of assignment
This indicates that you can only pass values to lvalue.

lvalue reference
These are the basic references. They behave like a const pointer, this means that they point to a specific location but you cannot change the location. With references, you cannot even access that location. This means that lvalue references must be initialized with a real value (not null). lvalue references allows the coder to pass objects similarly to java.
downloadtoggle
12 lines ...
int main (int argc, char** argv)
{
  //example1:
  int X = 5;
  int & rX = X;  // rX will hold the value 5
  rX++;   // rX and X will hold the value 6
 
  //example2:
  int* i = &argc;
  int& j = *i; // j is a reference to int, (*i) is of type int.
  std::cout << i << "," << *i << "," << j << std::endl;
  // Will print: pointer address,1,1
}
Memory Handling
Up till now we allocated memory on the stack. Now we learn how to allocate memory on the heap. This is done with the new operator.
new allocates memory on the heap, initializes it and returns a pointer to it.

Allocating primitives:

download
int *i = new int;       // i points to a heap location containing 0
int *i2 = new int(2);   // i2 points to a heap location containing 2
// if this was a complete code, it would have been terrible! we don't free the memory.
Allocating objects:
download
string *s = new string("I'm on the heap");
cout << "s=" << s << endl;
cout << "s=" << *s << endl;
delete s;  // memory on the heap must be freed!
All the memory allocated by new must be freed, or you will get a memory leak. The delete operator takes a pointer to an allocated space on the heap, and frees it: the memory is returned to the OS. However, you need to be very carefull: don't try to delete a pointer not allocated by new!.

Safe delete
download
if(p!=nullptr){
    delete p;
    p= nullptr;
}
Safe delete is recommended when a pointer is shared in many functions.
Memory allocation may fail. There might not be enough space on the heap, in which case new will throw the std:bad_alloc exception. You should only catch it if the program is able to recover from such a state.
Arrays on the heap
We've seen before that arrays are just pointers to a block of memory. You can use the new[] operator to allocate a block of memory on the heap. Use delete[] to free the allocated memory.
download
int *A = new int[10]; // allocate 10 integers on the heap
int *B = A + 1;
A[0] = 1;
A[1] = 2;
B[1] = 3;
delete[] A;           // free the allocated memory
Illustration of the stack and the heap as a result of the above code
arrayheap.jpg
2-D array on the heap: A 2-dimensional array is basically an array of pointers to arrays. You should initialize it using a loop, like this:

download
int** arry = new int*[rowCount];
for(int i = 0; i < rowCount; ++i)
    arry[i] = new int[colCount];
Illustration of 2D array on the heap
ps2_image2.png
Pointers Dangers
When you use pointers, beware of the following situations:
Uninitialized pointers
download
int *p; // p is an uninitialized pointer
*p = 3; // bad!
Dereferencing a null pointer
download
int *p = nullptr;  // p is a nullptr
*p = 3;         // bad!
Dereferencing a deleted pointer
download
int *p = new int;
delete p;
*p = 5;              // bad!
Dereferencing a dangling pointer
download
int *p, *q;
p = new int;
q = p;           // p and q point to the same location
delete q;        // now p is a dangling pointer
*p = 3;          // bad!
If p is the only pointer that points to dynamically allocated memory, and you reassign p without first deleting it, that memory will be lost (your code will have a storage leak)
download
int *p = new int;
p = nullptr;            // reassignment to p without freeing the storage
                       // it pointed to -- storage leak!
use delete for freeing memory allocated with new and delete[] for memory allocated with new[]
Memory leak
Memory leak occurs when the program fails to release memory that is no longer needed. A memory leak can diminish the performance of the computer by reducing the amount of available memory. Eventually, in the worst case, too much of the available memory may become allocated and all or part of the system or devices stops working correctly, the application fails, or the system slows down.
Use valgrind with –leak-check=yes option if your implementation has memory leaks.

Valgrind User Manual, The Valgrind Quick Start Guide, Graphical User Interfaces

The code in Memory1.cpp illustrates the life-cycle of objects in heap and stack memory.
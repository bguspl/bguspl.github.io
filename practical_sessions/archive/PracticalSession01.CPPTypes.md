C++ Types
The following is taken from Moving from Java to C++.
Example of some of the primitive (built-in) types
int
double
long
bool
char

download
int days = 0;
Constants
C++ uses the const keyword instead of Java's final.
download
const int DAYS_PER_YEAR = 365;
C++ Standard library
C++ standard library is a collection of classes and function written in the core language and is part of the C++ standard.
This basic libraries are based on STL (Standard template library), That were developed by SGI. It contains 4 major components:

Algorithms - Such as binary search or lower bound
Containers - Such as vectors, set, stack, map, queue etc…
Functional - Set of class templates for function objects (Function wrappers). We will discuss templates in a later class.
Iterators - Iterators for STL containers. including random access iterator, bidirectional iterators ect…
Each component implements some of the basic objects and functionalities needed in most programs (printing, files, hash maps, linked lists, strings etc…).
If you wish to find specific items or how to use specific algorithms and container, cplusplus is a great source of knowledge with a good search option.

String
The C++ string type is called string. It comes under the namespace std (i.e. std::string), included from string. Namespaces are somewhat similar to Java packages (we will elaborate on this later).
ASCII vs. Unicode
Both are storage formats. UTF-8 is usually 8-bit, giving 256 characters - unable to represent any languages besides English. In Unicode, each logical character is represented by a 32-bit integer, giving a total 65535 characters that can represented.
download
#include <string>
using namespace std;
 
string str = "We go step by step to the target";
It is quite similar to the Java String type. However, pay attention to these differences:

C++ strings store characters regardless of Encoding (might be UTF-8 or Unicode characters strings) and the reading of the strings is OS dependent)
download
string hebrew = "רכילות";  // might show something else on different OS
C++ strings can be modified, whereas Java strings are immutable.
download
str.insert(str.end(),3,'!');  // result str is: "We go step by step to the target!!!"
The substring operation in C++ is called substr. The command s.substr(i, n) extracts a substring of length n starting at position i.
download
int n = str.find("step");
string s = str.substr(n, 12);  // = step by step
You can only concatenate strings with other strings, not with arbitrary objects.
To compare strings, use the relational operators == != < <= > >=. The last four operators perform lexicographic comparison. This is actually more convenient than the use of equals and compareTo in Java.
More information and examples, here.
Templates
Before we continue to Vector, a word about C++ templates. Class templates are special classess that can operate with generic types. This allows us to create a class template whose functionality can be adapted to more than one type or class without repeating the entire code for each type. It is conceptually similar to Java Generics introduced in 1.5.
Vector
Vector comes under the namespace std (i.e. std::vector), included from .
download
#include <vector>
using namespace std;
The C++ vector construct combines the best features of arrays and vectors in Java. A C++ vector has convenient element access (i.e. random access), and it can grow dynamically. If T is any type, then vector<T> is a dynamic array of elements of type T. Read the explanations in the comments:
downloadtoggle
20 lines ...
vector<int> a;       // makes an initially empty vector that can handle only int types. 
vector<int> b(100);  // makes a vector that has initially 100 elements, each one a zero.
 
a.push_back(7);      // will add the element 7 to the end of the vector 
 
int n = a.back();   // gets the last element from a.
 
a.pop_back(); // removes the last element from a. 
 
size_t size = a.size();  // find the current number of elements in a.
 
//You access the elements with the familiar [] operator:
for (size_t i = 0; i < a.size(); ++i) {
   sum = sum + a[i];
}
 
// You access the elements by an iterator
std::vector<int>::iterator it;
for(it = a.begin(); it != a.end(); ++it) {
  sum = sum + *(it);    // You will learn about * next time!!
}
Iterator pattern
In object-oriented programming, the Iterator pattern is a design pattern in which iterators are used to aggregate object sequentially without exposing its underlying representation. An Iterator object encapsulates the internal structure of how the iteration occurs.
No runtime check in c++
As in Java, array indexes are between 0 and (a.size() - 1). However, unlike Java, there is no runtime check for legal array indexes. Accessing an illegal index can cause very serious unreported errors. You can use external tools, such as valgrind, for runtime error detection.
Use C++ arrays with care and only if you know what you are doing. If you're looking for a safer data-structure which is closer to the java array, use vector instead.
Deep copy semantic defined by the operator =
Just like all other C++ objects, vectors are values. If you assign one vector to another, all elements are copied. This is specified in vector's operator = definition.
download
vector<int> b = a; /* all elements are copied */
Contrast that with the situation in Java. In Java, an array variable is a reference to the array. Making a copy of the variable just yields a second reference to the same array.
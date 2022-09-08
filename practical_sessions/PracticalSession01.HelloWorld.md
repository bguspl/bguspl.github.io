Introduction to C++
The main goal for this part is to "get the hands on" C++.
We will go over the basic C++ rules, using a text editor and do a short exercise. The goal of this PS is to teach you how to have 2 C++ files, written and run under Linux.
As an editor you may try vi, pico, kate.
vi - Type `vi [filename]` to run it. You may refer to [this page](http://www.cs.colostate.edu/helpdocs/vi.html) or this [cheat sheet](http://www.viemu.com/vi-vim-cheat-sheet.gif)
pico - Just run the command `pico [filename]` from the command line and look at the bottom for options.
kate - A graphical text editor like notepad. You can find it in the K-menu.
This practical session has a lot of technical information regarding C++. We will not go over all these information. Make sure to read & understand it all.

Basic Terminology and C++ Functions

We will go over C++ terminology using the example in the next part.
Free (rather than in a class) functions
Functions in C++ are like Methods in Java. However, C++ functions can exist both in a class and as standalone.
In C++, the function void myFunc(int i) can be declared outside of the class. In this case we say it is in the global scope. Same can be done with variables. You should avoid declaring functions (or variables) in the global scope

The main function
Like in Java, main is a unique free function name that will be executed.
It returns an int value, and receives the command line arguments:int main(int argc, char**argv).

Using cout
cout is a reference to the standard output stream. Java's default System.out object, wraps the same output stream.
Java: System.out.println("Hello") should have the same result as C++: std::cout << "Hello" << std::endl

Function decleration and definition
A function definition contains both a prototype and a body, or implementation. For example:
download
int add(int a, int b) { return a + b; };
A function declaration contains only the prototype followed by a semicolon:
download
int add(int, int);
You can have multiple declarations of the same function but only one definition in the entire program.

Forward declaration and header files
Forward declaration is a declaration of a variable a function or a class for which the programmer has not yet given a complete definition. The compiler uses this declaration to determine how to generate the object code.
A header file (*.h) commonly contains forward declarations of subroutines, variables, classes and other identifiers (names). Identifiers that need to be declared in more than one source file can be placed in one header file, which is then included whenever its contents are required.
A C++ source file (*.cpp) will define behavior for identifiers.

#include
#include has the same purpose as the Java'simport. However unlike java, #include "copy-pastes" contents of the included file into the current file.
This is why you must be careful as to what you include. As a rule, include only header files.

Header files
download
//Always start your header file with
#ifndef   uniqueName_H_
#define   uniqueName_H_
 
//and end it with
#endif  
We will soon see what these #command means and why to use it.

Common mistake
Don't include every h file you have on your hard disk. Include only the files you really need. Failing to do so might cause a big headache latter on…

Our first C++ program

Create the project folder for the renowned Hello-World example: ~/workspace/HelloWorld Inside the project directory create the following three folders: src, include, bin.
Inside src create text files "HelloWorld.cpp" and "Run.cpp"
Inside include create a text file named "HelloWorld.h"
Use a simple editor such as kate. This is a GUI editor. There are also editors that work entirely in the console, for examplepico. This is useful when you are working in a gui-less environment, such as when connecting from home via ssh.
project_structure.jpg

H file - declaration
HelloWorld.h
download
#ifndef HELLOWORLD_H_
#define HELLOWORLD_H_
 
void printHello();   // This is a free function declaration
 
#endif
CPP file - definition
HelloWorld.cpp
download
#include "../include/HelloWorld.h"
#include <iostream>


void printHello() {
    std::cout << "Hello World!" << std::endl;  
}

Run.cpp
download
#include "../include/HelloWorld.h"

int main(int argc, char *argv[]) {
  printHello();
}
Now we have C++ files, and we have to first compile them, and then link them into an executable. To remind you, the src directory has source code, the include directory has header files, and the bin directory is a working directory, where machine produced files will be.

Run the following from the console:

Compiling the 1st file: g++ -g -Wall -Weffc++ -c -Iinclude -o bin/HelloWorld.o src/HelloWorld.cpp
Compiling the 2nd file: g++ -g -Wall -Weffc++ -c -Iinclude -o bin/Run.o src/Run.cpp
Linking them together: g++ -o bin/hello bin/HelloWorld.o bin/Run.o -L/usr/lib
Run the program: bin/hello

Now this is nice for 2 source files and one header file, but your homework assignments will have more. And for every ; you forget you have to start the process all over, this will be not practical. For that purpose we have a special file called makefile and a program make that looks for that file, and together they automate this process. So lets get started with makefiles.

Extra notes on include
Splitting to H and CPP files
If we would include a CPP, we would put declared source in several places in our code. This will cause duplicity compilation errors.
The header files function as an interface. Declared codes is not considered duplicated. But it might contredict.
The #include in depth including the #ifndef derivative
Let's say we have: A.h
download
#include "B.h"
class A {
};
And B.h

download
#include "A.h"
class B {
};
Then writing:

download
#include "A.h"
 
int main() {...}
Would pre-process into:

download
#include "B.h"
class A {
};
 
int main() {...}
download
#include "A.h"
class B {
};
class A {
};
 
int main() {...}
download
#include "B.h"
class A {
};
class B {
};
class A {
};
 
int main() {...}
Etc. etc.

Using #define and #ifndef will prevent that.

download
#ifndef UNIQU_ID_class_A
#define UNIQU_ID_class_A
#include "B.h"
class A {
};
#endif   /*  UNIQU_ID_class_A */
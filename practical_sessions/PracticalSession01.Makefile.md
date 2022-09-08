Compiling and Linking C++ program
Java Vs. C++
There are several fundamental different between Java and C++ in the compilation process. Make sure that you understand that.
http://users.cs.cf.ac.uk/Dave.Marshall/C/cmodel.gif
(Diagram and explanations by Dave Marshall)
The Preprocessor
The Preprocessor accepts source code (*.cpp) as input and is responsible for
removing comments
interpreting special preprocessor directives denoted by #.
For example:
#include – paste in a named file. Usually used for header files. e.g
#include <math.h> – paste in the standard library math file.
#include <stdio.h> – paste in standard library I/O file
#include "My.h" – paste in the file My.h from the same directory (relative path)
#include "/user/spl111/Base.h" – paste in a file from absolute path
C++ Compiler
The C++ compiler translates source to assembly code. The source code is received from the preprocessor.
Assembler
The assembler creates object code. On a UNIX system you may see files with a *.o suffix (*.OBJ on MS-DOS) to indicate object code files.
Linker
If a source file references a functions defined in another source files the link editor (in short, the linker) combines the code to create an executable file. External Variable references are resolved by the linker as well. The linker also looks for the main() function to mark it as the starting point of the executable.
The linker combines multiple object files (*.o) and libraries (*.a) into one executable.

How C++ compilation process is different from Java
C/C++ compilation produces machine specific binary files whereas Java compilation produces byte-code that is specific to the Java Virtual Machine.
Where C++ decides the location of each class member at compile time, Java defers this decision until you attempt to load and run the class.

To be more technical:

C++ compiler (by default) will do: pre-compilation, compilation and assembly
Java compiler (by default) will do: compilation. It has no assembly step, since it is not machine specific.
C++ linker will link the object files to an executable.
In Java, linking is done at runtime. Whenever execution of Java bytecode requires the presence of a new class then a class loader is invoked to fetch the class material.

Compilation.png

Compiling C++ (and makefile)
Manually
After we discussed the compiler and linker, see how they run:
Open terminal.
Go to project root, e.g. cd ~/workspace/HelloWorld
Clean: rm -f bin/*
Compile: g++ -g -Wall -Weffc++ -std=c++11 -c -Iinclude -o bin/HelloWorld.o src/HelloWorld.cpp
Compile: g++ -g -Wall -Weffc++ -std=c++11 -c -Iinclude -o bin/Run.o src/Run.cpp
Link: g++ -o bin/hello bin/HelloWorld.o bin/Run.o
Run: ./bin/hello

Notice we do not compile a Header file.
As we can see, If we have 2 code files (.c/.cpp) we can easily do this manually.
For large a project, this will take a long time.

g++ parameters
-o fileName Place output in file fileName. By default is will create a file called: a.out
-c Compile or assemble the source files, but do not link. The compiler output is an object file corresponding to each source file.

-Ldir Add directory dir to the list of directories to be searched for -l.

-llib Use the library named lib when linking. (C++ programs often require g++ -l for successful linking.)

-O Optimize. Use this to yield optimized code. (for example that runs faster).

-g Add debug information.

-W What warning level would you like to receive.

Eclipse compiler
If you check your project properties in Eclipse, you can find that the compiler options are: g++ -O0 -g3 -Wall -c -fmessage-length=0
Makefiles
Makefiles (and the program which uses them) are used to specify dependencies between abstract targets, and what needs to be done to achieve each target. More specifically, a Makefile is a text file containing a description of our targets (the most common case of a target is an object), followed by their dependencies and next by the actions which are necessary to build these targets.
Target names and dependencies are assumed to be files. Consider a simple target, $T_1$, which depends on $D_1$ and $D_2$ (all of which are files). To build $T_1$, the make utility will work as follows.

If either $D_1$ or $D_2$ do not exist, build them (recursively).
Check if both $D_1$ and $D_2$ are up to date. If not, build them recursively.
Check the date of $T_1$ (the modification time). If $T_1$ is at least as new as BOTH $D_1$ and $D_2$, we are done, as $T_1$ is up to date.
Otherwise, follow the instructions given to build $T_1$.

When you will type "make" in your shell, the script will look for a file "makefile" in the same directory and will execute it, using the rules defined in the "makefile" file.
By default, make will only execute the first target in the makefile. So, make sure the first target will cause a complete build.

makefile

downloadtoggle
23 lines ...
# All Targets
all: hello

# Tool invocations
# Executable "hello" depends on the files hello.o and run.o.
hello: bin/HelloWorld.o bin/Run.o
	@echo 'Building target: hello'
	@echo 'Invoking: C++ Linker'
	g++ -o bin/hello bin/HelloWorld.o bin/Run.o
	@echo 'Finished building target: hello'
	@echo ' '

# Depends on the source and header files
bin/HelloWorld.o: src/HelloWorld.cpp
	g++ -g -Wall -Weffc++ -std=c++11 -c -Iinclude -o bin/HelloWorld.o src/HelloWorld.cpp

# Depends on the source and header files 
bin/Run.o: src/Run.cpp
	g++ -g -Wall -Weffc++ -std=c++11 -c -Iinclude -o bin/Run.o src/Run.cpp

#Clean the build directory
clean: 
	rm -f bin/*
Tabs in makefile
Important - the space you see to the left of some lines are tabs, and not space characters.
There are many cases where you may wish to change compiler options or even compilers when building a piece of software. For example, you may need to specify '-g' to turn on debugging, or '-O2' to add some extra optimization. If those flags are hard-coded as they are in the simple example, then each instance must be edited in the Makefile. There is, however, an easier way to handle this - with variables.

Traditionally, makefile variables are all upper-case, and are referenced using ${VAR_NAME}. The previous makefile is re-written using variables.

advanced makefile (not obligatory)

downloadtoggle
29 lines ...
# define some Makefile variables for the compiler and compiler flags
# to use Makefile variables later in the Makefile: $()
CC = g++
CFLAGS  = -g -Wall -Weffc++ -std=c++11
LFLAGS  = -L/usr/lib

# All Targets
all: hello

# Tool invocations
# Executable "hello" depends on the files hello.o and run.o.
hello: bin/HelloWorld.o bin/Run.o
	@echo 'Building target: hello'
	@echo 'Invoking: C++ Linker'
	$(CC) -o bin/hello bin/HelloWorld.o bin/Run.o $(LFLAGS)
	@echo 'Finished building target: hello'
	@echo ' '

# Depends on the source and header files
bin/HelloWorld.o: src/HelloWorld.cpp
	$(CC) $(CFLAGS) -c -Iinclude -o bin/HelloWorld.o src/HelloWorld.cpp

# Depends on the source and header files 
bin/Run.o: src/Run.cpp
	$(CC) $(CFLAGS) -c -Iinclude -o bin/Run.o src/Run.cpp

#Clean the build directory
clean: 
	rm -f bin/*
Create the above makefile file. Two ways:
Download the above makefile from here. Right click and "Save Link As…": ~/workspace/HelloWorld/makefile
pico makefile and copy-paste or write the above makefile. Pay attention to the tabs.
Type: make
(Optionally) type: make clean to delete the executable file and all the object files.
Executing C++ programs
Type: ./bin/hello (created by the makefile) or ./Debug/hello (created by eclipse)
Remember: execute the software from a console and from eclipse works slightly deferent. Make sure to run your application through the console.
As we discussed, C++ compilation produces machine specific binary files. So it can be executed by the operating-system (double-click the file or run from the shell command line). You cannot run the executable on an OS that is not compatible with OS that compiled the code.

In the same way, Java classes can only be executed by a JVM that is compatible with the Java-Compiler that compiled the code. E.g. you cannot run Java1.5 specific code on Java1.4-JVM.
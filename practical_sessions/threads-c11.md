# C++ 11 Multithreaded Programs

## Table of Contents
- [C++ 11 Multithreaded Programs](#c-11-multithreaded-programs)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Creating a Task](#creating-a-task)
  - [Basic Example](#basic-example)
  - [Basic Synchronization](#basic-synchronization)
  - [Further reading](#further-reading)


## Introduction
The C++ 11 standard library provides several classes that can be used to write multi-threaded programs in C++. The process of writing a multi-threaded program is similar to what you already know from Java (There are few minor differences). 

## Creating a Task
You need to perform the following steps:
1. Create a Task class with a run function (or any other name you like). The function can receive arguments. Notice that if you override `operator()` (similar to `operator=` just for ()) then it will be called by default.
1. On the second example, we give the function's address but since it belongs to a class it has a first hidden parameter (the "this" pointer).
1. Create an instance of this class.
1. Create a thread, and start the tasks using the constructor `std::thread t(&MyClass::MyFunction, &instance [, parameter1, parameter2 ...])`.
1. You can use the method `join()` of `std::thread` to wait for single thread.

To complete linking of a code that uses C++ 11 threads you must link the threads library using `-pthread`.


## Basic Example
```cpp
#include <iostream>
#include <thread>
 
class Task {
private:
    int _id;
public:
    Task (int number) : _id(number) {}
 
    void operator()(){
        for (int i= 0; i < 100; i++){
           std::cout << i << ") Task " << _id << " is working" << std::endl; 
        }
        std::this_thread::yield(); //Gives up the remainder of the current thread's time slice, to allow other threads to run. 
    }
};
 
int main(){
    Task task1(1);
    Task task2(2);
    
    std::thread th1(std::ref(task1)); // we use std::ref to avoid creating a copy of the Task object
    std::thread th2(std::ref(task2)); 
    th1.join();
    th2.join();    
    
    return 0;
}
```

## Basic Synchronization
Basic synchronization is performed using the mutex object `std::mutex`. If several threads share a resource, they should also share a lock. \
When a thread needs to access the shared resources, it should acquire the lock first by creating a `std::lock_guard<std::mutex>` instance (it is not recommended to use mutex directly).  \
The constructor acquire the lock on the mutex for the current scope. Upon exiting the scope, the lock is released automatically. This methodology is called [RAII (Resource Acquisition Is Initialization)] (https://en.cppreference.com/w/cpp/language/raii) and it is a common technique we saw also in smart pointers. \

As in Java, only one thread is allowed to acquire the lock in any point in time. Other threads trying to acquire the lock when it is already acquired by some thread, will wait until it is released.

The next example corrects the synchronization problem from the previous example. (The shared resource is the output stream.)

```cpp
#include <iostream>
#include <mutex>
#include <thread>
 
class Task{
private:
    int _id;
    std::mutex & _mutex;
public:
    Task (int id, std::mutex& mutex) : _id(id), _mutex(mutex) {}
 
    void run(){
        for (int i= 0; i < 100; i++){
            std::lock_guard<std::mutex> lock(_mutex); // constructor locks the mutex while destructor (out of scope) unlocks it
            std::cout << i << ") Task " << _id << " is working" << std::endl;
        }
    }
};
 
int main(){
    std::mutex mutex;
    Task task1(1, mutex);
    Task task2(2, mutex);
 
    std::thread th1(&Task::run, &task1);
    std::thread th2(&Task::run, &task2);
    th1.join();
    th2.join();
    return 0;
}
```

## Further reading

We recommend following this tutorial: [C++11 Multithreading Tutorial](https://thispointer.com/c-11-multithreading-part-1-three-different-ways-to-create-threads/)


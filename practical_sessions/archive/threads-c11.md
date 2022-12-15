C++ 11 standard library provides several classes that can be used to write multi-threaded programs in C++. The process of writing a multi-threaded program is similar to what you already know from Java. (There are few minor differences.) You need to perform the following steps.
There is no Runnable interface. Instead, create a Task class with a run function (or any other name you like). The function can receive arguments. Notice that if you override operator() (similar to operator= just for ()) then it will be called by default.
On the second example, we give the function's address but since it belongs to a class it has a first hidden parameter (the "this" pointer).
Create an instance of this class.
Create a thread, and start the tasks using the constructor std::thread t(&MyClass::MyFunction, &instance [, parameter1, parameter2 ...]).
You can use the method join() of std::thread to wait for single thread.

To complete linking of a code that uses c++11 threads you must link the threads library using -pthread
Here is a simple example.

downloadtoggle
27 lines ...
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
from file: threadsc11.cpp
Basic synchronization
Basic synchronization is performed using the mutex object std::mutex. If several threads share a resource, they should also share a lock. When a thread needs to access the shared resources, it should acquire the lock first by creating a std::lock_guard<std::mutex> instance. The constructor acquire the lock on the mutex for the current scope. Upon exiting the scope, the lock is released automatically. As in java, only one thread is allowed to acquire the lock in any point in time. Other threads trying to acquire the lock when it is already acquired by some thread, will wait until it is released.
The next example corrects the synchronization problem from the previous example. (The shared resource is the output stream.)

downloadtoggle
29 lines ...
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
from file: threadsac11.cpp
References
Toturial for multithreading in c++ 11 can be found here
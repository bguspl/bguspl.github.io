Application Persistence Design Patterns
Objectives
In this lecture we will learn about persistence design patterns. i.e, how to construct an application that uses data persistence (usually via databases). We will demonstrate the patterns you will learn using the python programming language and the sqlite database.
Since you will need to know a little python in order to complete this lesson here is a quick guide to python.

Some parts of this section are based on the python for java programmers book/tutorial you are encouraged to read it if you wish to learn more about python.

The python language have many different implementation, in this lecture we are only referring to the standard one which is CPython V2.7.
Python in ~1 hour
Python is an open-source, general purpose programming language that is dynamic, strongly-typed, object-oriented, functional, and memory-managed. Python uses an interpreter to translate and run its code and that's why it's called a scripting language.
A scripting language is a programming language that uses an Interpreter to translate its source code. The interpreter reads and executes each line of code one at a time, just like a SCRIPT for a play or an audition, hence the the term "scripting language".
We know that Java is a statically typed language, i.e. the types are checked and enforced at compile time. In contrast, Python is dynamic which means that the types are checked only at runtime. But Python is also strongly typed, just like Java. You can only execute operations that are supported by the target type.

Another way to think about this is that while in Java, both variables and objects have types associated with them, in Python only objects have types, not the variables that they are bound to.

For example, in Java, when we declare:

download
MyType obj = new MyType()
The variable obj is declared of type MyType and then the newly instantiated object of type MyType is assigned to it. In contrast, in Python, the same declaration would read:

download
obj = MyType()
Ignoring the missing new keyword (which Python doesn’t have), obj is simply a name that is bound to the object on the right, which happens to be of type MyType. We can even reassign obj in the very next line – obj = MyOtherType() – and it wouldn’t be a problem. In Java, this re-assignment would fail to compile while in Python, the program will run and will only fail at runtime if we try to execute an operation via obj that is incompatible with the type assigned to it at that point in time.

Python is object oriented and supports all the standard OOP features that Java has like creation of types using classes, encapsulation of state, inheritance, polymorphism, etc. It even goes beyond Java and supports features such as multiple inheritance, operator overloading and meta-programming. Although these features were not added to java in purpose as most of the time they lead to unneeded complexity and confusing APIs.

It is often claimed that Java(or c++) were designed for easy code reading (which is important for large systems) while python designed for easy code writing (which is important for small prototyping).

Another similarity between the languages is in terms of manual memory management, in that there is none. The language runtime takes care of correctly allocating and freeing up memory, saving the programmer from the drudgery – and mistakes – of manually managing memory. Having said that, the JVM garbage collectors are much, much better performing than the Python GC. This can become a concern depending on the type of application you are building.

Finally, the dynamic and interpreted nature of python together with its object model (which mainly defines each object as a dictionary - which similar to Java's HashMap ) leads to very slow code execution (relative to java or C++). So why use Python? given this inherent inefficiency, why would we even think about using Python? Well, it comes down to this: Python is considered much easier to use than Java/C++. It's extremely flexible and forgiving, this flexibility leads to efficient use of development time, and on those occasions that you really need speedy code, Python offers hooks into compiled libraries. In other words, choose the right tool for the job. You can read more about the performance of python in Why Python is Slow: Looking Under the Hood.

The python language
Lets look on some code examples in python. This is by no means a complete set of examples for all python features, but only a short introduction to get you started.
downloadtoggle
85 lines ...
# defining a variable and assigning a value
x = 1 # int
x = 'hello' # string
x = "hello" # also string
x = """ A MULTI
        LINE 
        STRING """
 
# string format
x = 'hello {} have a {} day'.format('sammy', 'nice') # = "hello sammy have a nice day"
 
#declaring the numbers variable and assign it with a list of numbers containing 1-9
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9] 
 
odd_numbers = [] 
#iterating over the list, notes: no semicolon or barracks needed but indentation is extremely important
for num in numbers:                         
  if num % 2 != 0:                          
    odd_numbers.append(num)                 
 
print "the odd numbers are:", odd_numbers
 
#some other collections:
a_tuple = (1,2,3) # a tuple is an immutable list
a_dictionary = {'a':1, 'c':2, 'e':3} # a dictionary is similar to Java's HashMap
 
#working with lists
numbers = [0, 1, 2, 'three', 4, 5, 6, 7, 8, 9] 
print len(numbers) # 10
print numbers[0] # 0
print numbers[0:4] # [0, 1, 2, 'three']
print numbers.index('three') # 3 (return the index of the value 'three')
print [1]*5 # [1, 1, 1, 1, 1] 
 
#working with dictionaries
print a_dictionary.keys() # ['a', 'c', 'e']
print a_dictionary.values() # [1, 2, 3]
print a_dictionary['a'] # 1
print len(a_dictionary) # 3
 
#equality
copy = numbers[:] # create a shallow copy
print copy == numbers # prints 'True', "==" is the equivalent to Java's equals
print copy is numbers # prints 'False', "is" is the equivalent to Java's == 
 
#no for-index loops - can use xrange instead
for index in xrange(len(numbers)):
  print index
 
#special syntax to construct a list from other list values 
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] 
incremented = [x+1 for x in numbers] # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10] 
 
#defining a function
def inc(a, b):
  return a+b
 
#calling a function
inc(1,2)
 
#you can call a function with arguments taken 
#from a list or a tuple using the '*' operator  
args = [1,2]
inc(*args)
 
#defining a class
class Example(object): #you need to explicitly say that you inherit from object (lowercase)
  
  #constructor, each instance method must include self as first parameter, it is the equivalent to Java's "this"
  def __init__(self, x):
    self._x = x #_x is a field, the leading underscore is a convention for marking private fields in python
 
  #the equivalent to Java's toString
  def __str__(self):
    return "Example {x=" + self._x + "}"  
 
  def print_x(self):
    print self._x
 
#working with classes
example = Example('the best x') #no need for new, no need to pass self
example.print_x() # no need to pass self
 
#each object in python is actually a dictionary inside, we can get access to
#the internal dictionary using the vars builtin function
print vars(example) #{'_x': 'the best x'}
Organizing Code
The most basic unit of source code organization in Python is the module which is just a .py file with Python code in it. This code could be functions, classes, statements or any combination thereof. Several modules can be collected together into a package. Python packages are just like Java packages with one difference: the directory corresponding to a package must contain an initializer file named __init__.py. This file can either be empty or can optionally contain some bootstrap code that is executed when the package is first imported.
Here is an example of a phonebook application code base:

 .
 |-- phonebook_app.py
 +-- gui
 |   |-- __init__.py
 |   |-- windows.py
 |   |-- linux.py
 +-- model
 |   |-- __init__.py
 |   |-- phone.py
 |   |-- user.py
The main method
Now that we’ve organized our application’s source code into multiple files, you must be wondering, what defines the entry point for the application? As Java programmers, public static void main is forever burnt into our brains! What’s the equivalent in Python?
There is no formal notion of an application entry point in Python. Instead, when Python executes a file, e.g., when you run python foo.py, execution begins at the start of the file and all statements defined at the top-level are executed in order till we reach the end of the file.

Python on the other hand have the notation of the "main module". when invoking python in the following way:

python -m phonebook_app.py arg1 arg2
python will set a special variable called __name__ to "__main__" inside the phonebook_app.py which we can use in order to execute our main code as can be seen in the example phonebook_app.py

download
import sys
 
def main(args):
  print "Main method of phonebook_app is called with arguments:", args
 
if __name__ == '__main__':
    main(sys.argv)
SQL Lite
SQLite is an embedded SQL database engine. Unlike most other SQL databases, SQLite does not have a separate server process. SQLite reads and writes directly to ordinary disk files. A complete SQL database with multiple tables, indices, triggers, and views, is contained in a single disk file.
Python has a reach set of modules (some of which we will learn about in this class), one of these libraries is sqlite3. Here is what you need to know about the sqlite3 library (for this class)

downloadtoggle
29 lines ...
# importing the sqlite3 library will result in a sqlite3 variable with all the functions and classes of sqlite3
import sqlite3 
 
# will create a connection to a file base database stored in 'mydb.db' file
# if the file does not exist, it will be created  
conn = sqlite3.connect('mydb.db')
conn.text_factory = bytes # don't use unicode, python 2 has problematic support of unicode
 
# executing a statement that does not return a value, note that the location of values for the statement
# are marked using '?' and the values themself are given as the second argument to execute
conn.execute("""
  INSERT INTO some_table (col1, col2) VALUES (?,?)
""", [val1, val2])
 
# executing a statement that return a value 
cur = conn.cursor()
cur.execute("""
  SELECT * FROM some_table 
  WHERE col1 = ? 
""", 42)
 
#the result is stored inside the curser we can retrieve it as a list of tuples using:
all = cur.fetchall()
 
#or if we know that there is a single value
val = cur.fetchone()
 
# connections must be closed when you done with them
conn.commit() # commit any changes not yet written to the database
conn.close() # close the connection
The code above contains most of the sqlite3 methods that you will need to know for this class.

Application Persistence Patterns
There are many ways to construct an application that needs to store and retrieve data. In this section we will create a simple application that uses sqlite for data persistence. We will then examine this application and show different design patterns and best practices.
'Vicious' - The Assignment Tester
Lets assume that we want to create an assignment tester for python assignments in the SPL course. This application will include a method that will check assignments and store their grades in the database. We will assume the following:
Students has a unique id
Assignments are numbered (e.g., assignment 1, 2,..)
Each submitted assignment contains only a single *.py file with a method run_assignment that accept no arguments and returns a string.
The submitted file name will be of the form <submitter student id>.py

Our program will use an sqlite database with the following tables:
students which includes the id of the student and its name
assignments which includes the number of the assignment and a string representing the expected output of the run_assignment method of the corresponding assignment
grades which includes the grade of each student on a specific assignment

Lets examine a simple python script that provides the behavior defined above
downloadtoggle
99 lines ...
#file: vicious.py
 
import os
import sqlite3
import imp
import atexit
 
#connect to the database
_conn = sqlite3.connect('grades.db')
 
#register a function to be called immediately when the interpreter terminates
def _close_db():
    _conn.commit()
    _conn.close()
 
atexit.register(_close_db)
 
#our application API:
 
def create_tables():
    _conn.executescript("""
        CREATE TABLE students (
            id      INT         PRIMARY KEY,
            name    TEXT        NOT NULL
        );
 
        CREATE TABLE assignments (
            num                 INT     PRIMARY KEY,
            expected_output     TEXT    NOT NULL
        );
 
        CREATE TABLE grades (
            student_id      INT     NOT NULL,
            assignment_num  INT     NOT NULL,
            grade           INT     NOT NULL,
 
            FOREIGN KEY(student_id)     REFERENCES students(id),
            FOREIGN KEY(assignment_num) REFERENCES assignments(num),
 
            PRIMARY KEY (student_id, assignment_num)
        );
    """)
 
 
def insert_student(id, name):
    _conn.execute("""
        INSERT INTO students (id, name) VALUES (?, ?)
    """, [id, name])
 
 
def insert_assignment(num, expected_output):
    _conn.execute("""
        INSERT INTO assignments (num, expected_output) VALUES (?, ?)
    """, [num, expected_output])
 
 
def insert_grade(student_id, assignment_num, grade):
    _conn.execute("""
        INSERT INTO grades (student_id, assignment_num, grade) VALUES (?, ?, ?)
    """, [student_id, assignment_num, grade])
 
 
def grade(assignments_dir, assignment_num):
 
    query_data = _conn.cursor()
    query_data.execute("""
        SELECT expected_output FROM assignments WHERE num = ?
    """, [assignment_num])
    
    # the expected value is the first element in the fetched row
    expected_output = query_data.fetchone()[0]
 
    #loop over the files in the assignment_dir
    for assignment in os.listdir(assignments_dir):
 
        #the os.path.splitext split a file name that looks like: 'file.ext' into ['file', 'ext']
        (student_id, ext) = os.path.splitext(assignment)
 
        #imp.load_source loads python source code (module) into a variable 
        #the first argument will be the __name__ that the module will see when it loads
        #and the second is the path to the module 
        code = imp.load_source('test', assignments_dir + '/' + assignment)
 
        #calculating the grade 
        grade = 100 if code.run_assignment() == expected_output else 0
 
        insert_grade(student_id, assignment_num, grade)
 
 
def print_grades():
    cur = _conn.cursor()
 
    cur.execute("""
        SELECT name as student_name, assignment_num, grade
        FROM students INNER JOIN grades ON students.id = grades.student_id
    """)
 
    print 'grades:'
    for row in cur.fetchall():
        print 'grade of student {} on assignment {} is {}'.format(*row)
Assume that we have 3 students in our class: Alice, Bob and Chris, with the corresponding ids: 1111, 2222 and 3333. In addition, assume that they submitted the following files that we collected and put inside the 'assignments' library:

downloadtoggle
10 lines ...
#1111.py
def run_assignment():
  return 'spl'
 
#2222.py
def run_assignment():
  return 'spl!'
 
#3333.py
def run_assignment():
  return 'spl'
We can now open the python interpreter in the directory that contains our script ('vicious.py') and execute our tester as follows:

> python

>>> from vicious import *
>>> create_tables()
>>> insert_student(1111, 'Alice')
>>> insert_student(2222, 'Bob')
>>> insert_student(3333, 'Christ')
>>> insert_assignment(1, 'spl')

>>> grade('assignments', 1)
>>> print_grades()
grades:
grade of student Alice on assignment 1 is 100
grade of student Bob on assignment 1 is 0
grade of student Christ on assignment 1 is 100
This code works but the logic of the tester is coupled with the logic of the database, this coupling has the following drawbacks:

If we will change some part of our database (e.g., some table name) we will have to modify our application logic in multiple functions (and maybe multiple files)
If we will change our database type (e.g., from sqlite3 to sqlite4) we will have to go over all our code and adapt it
If we add more modules that wants to use our database, they will have to import the 'vicious' module, even if they does not need to 'grade' assignments
Database queries are a relatively long operation. Many times, we want to cache some of our queried so that we will not have to re-query the database if we need it again. In the way our application is designed now, our application logic will have to include cache related logic (everywhere) which will make our application code much more complicated.

These drawbacks will only get worse when our code will grow and more modules will be added. An obvious solution is to decouple the persistence related code from our application logic as we will see next.
Data Persistence Layer
The obvious way to solve the issues above is to divide our code into:
Persistence Layer: a group of files which is used to communicate between the application and DB.
The rest of the application logic - which will use the persistence layer to store and query data.

The following is a rewrite of our 'Vicious' assignment checker with separated Persistence layer (in the persistence.py file)
downloadtoggle
44 lines ...
#file persistence.py
 
import sqlite3
import atexit
 
#we want to make the persistence layer singleton so we are marking the following class as private
# by prepending an underscore to its name.
class _PersistenceLayer(object):
    def __init__(self):
        self._conn = sqlite3.connect('grades.db')
 
    def _close(self):
        self._conn.close()
 
    def create_tables(self):
        #see code in previous version...
 
    def insert_student(self, id, name):
        #see code in previous version...
 
    def insert_assignment(self, num, expected_output):
        #see code in previous version...
 
    def insert_grade(self, student_id, assignment_num, grade):
        #see code in previous version...
 
    def get_assignment_expected_output(self, assignment_num):
        c = self._conn.cursor()
        c.execute("""
            SELECT expected_output FROM assignments WHERE num = ?
        """, [assignment_num])
 
        return str(c.fetchone()[0])
 
    def get_all_grades(self):
        c = self._conn.cursor()
        return c.execute("""
            SELECT name as student_name, assignment_num, grade
            FROM students INNER JOIN grades ON students.id = grades.student_id
        """).fetchall()
 
#persistence layer is a singleton
psl = _PersistenceLayer()
 
atexit.register(psl._close)
Given the above persistence layer, our application code became much simpler:

downloadtoggle
24 lines ...
#file: vicious.py
 
from persistence import psl
import os
import imp
 
 
def grade(assignments_dir, assignment_num):
    expected_output = psl.get_assignment_expected_output(assignment_num)
 
    for assignment in os.listdir(assignments_dir):
        (student_id, ext) = os.path.splitext(assignment)
 
        code = imp.load_source('test', assignments_dir + '/' + assignment)
 
        grade = 100 if code.run_assignment() == expected_output else 0
 
        psl.insert_grade(student_id, assignment_num, result)
 
 
def print_grades():
 
    print 'grades:'
    for grade in psl.get_all_grades():
        print 'grade of student {} on assignment {} is {}'.format(*grade)
DAO, DTO and the Repository design patterns
Lets re-examine our application code, in vicious.py we can spot two "problems"
In the first line of the grade method we getting the expected_output of an assignment from the persistence layer. What if we add more fields to the assignment, should we create a new method to get each field? what if we want all the fields?
Inside the function print_grades we can see that we assuming that get_all_grades returns a list of tuples such that for each tuple the student name is the first, the assignment number is second and the grade is third. If we add fields to the grades table this may break our assumption. What if we will write another method that need all the grades but does not need the student name, instead it needs the student id - should we create another method get_all_grades2?

The problems above can be solved using the DTO (Data Transfer Objects) and DAO (Data Access Objects) design pattern. This pattern defines two types of objects
DTOs: these are objects that are passed to and from the persistence layer. When passed from the persistence layer to the application logic, they contains the data retrieved from the database. When passed from the application logic to the persistence layer, they contains the data that should be written to the database. In most cases, these objects represents a single table. In our case, we need to construct classes for the following DTOs: Student, Grade and Assignment.
DAOs: these objects contains methods for retrieving and storing DTOs, In most cases, each DAO is responsible for a single DTO. In our case we need 3 DAOs: one that knows how to handle students, one for assignments and one for grades.

In many cases, DTO and DAOs are not sufficient, since each DAO only knows how to handle a specific DTO, where should we put our create_tables method? and what if we want to have queries that span multiple tables (using join)? which DAO should hold the methods for these queries?
The repository design pattern can help us with these questions. A "repository" is similar to a DAO but it manages group of related DTOs. In our case, we can rewrite our presentation layer as follows:

downloadtoggle
97 lines ...
#file: persistence.py
 
import sqlite3
import atexit
 
# Data Transfer Objects:
class Student(object):
    def __init__(self, id, name):
        self.id = id
        self.name = name
 
 
class Assignment(object):
    def __init__(self, num, expected_output):
        self.num = num
        self.expected_output = expected_output
 
 
class Grade(object):
    def __init__(self, student_id, assignment_num, grade):
        self.student_id = student_id
        self.assignment_num = assignment_num
        self.grade = grade
 
 
# Data Access Objects:
# All of these are meant to be singletons
class _Students:
    def __init__(self, conn):
        self._conn = conn
 
    def insert(self, student):
        self._conn.execute("""
               INSERT INTO students (id, name) VALUES (?, ?)
           """, [student.id, student.name])
 
    def find(self, student_id):
        c = self._conn.cursor()
        c.execute("""
            SELECT id, name FROM students WHERE id = ?
        """, [student_id])
 
        return Student(*c.fetchone())
 
 
class _Assignments:
    def __init__(self, conn):
        self._conn = conn
 
    def insert(self, assignment):
        self._conn.execute("""
                INSERT INTO assignments (num, expected_output) VALUES (?, ?)
        """, [assignment.num, assignment.expected_output])
 
    def find(self, num):
        c = self._conn.cursor()
        c.execute("""
                SELECT num,expected_output FROM assignments WHERE num = ?
            """, [num])
 
        return Assignment(*c.fetchone())
 
 
class _Grades:
    def __init__(self, conn):
        self._conn = conn
 
    def insert(self, grade):
        self._conn.execute("""
            INSERT INTO grades (student_id, assignment_num, grade) VALUES (?, ?, ?)
        """, [grade.student_id, grade.assignment_num, grade.grade])
 
    def find_all(self):
        c = self._conn.cursor()
        all = c.execute("""
            SELECT student_id, assignment_num, grade FROM grades
        """).fetchall()
 
        return [Grade(*row) for row in all]
 
#The Repository
class _Repository(object):
    def __init__(self):
        self._conn = sqlite3.connect('grades.db')
        self.students = _Students(self._conn)
        self.assignments = _Assignments(self._conn)
        self.grades = _Grades(self._conn)
 
    def _close(self):
        self._conn.commit()
        self._conn.close()
 
    def create_tables(self):
        #see code in previous version...
 
# the repository singleton
repo = _Repository()
atexit.register(repo._close)
We can then make some small adaption to our main program logic as follows:

downloadtoggle
26 lines ...
from persistence import repo
 
import os
import imp
 
def grade(assignments_dir, assignment_num):
    expected_output = repo.assignments.find(assignment_num).expected_output
 
    for assignment in os.listdir(assignments_dir):
        (student_id, ext) = os.path.splitext(assignment)
 
        code = imp.load_source('test', assignments_dir + '/' + assignment)
 
        student_grade = Grade(student_id, assignment_num, 0)
        if code.run_assignment() == expected_output:
            student_grade.grade = 100
 
        repo.grades.insert(student_grade)
 
 
def print_grades():
    print 'grades:'
    for grade in repo.grades.find_all():
        student = repo.students.find(grade.student_id)
 
        print 'grade of student {} on assignment {} is {}'\
            .format(student.name, grade.assignment_num, grade.grade)
Automating things: ORMs and generic DAOs
If we examine the persistence code that we saw above we may notice several repeating patterns in the different DAOs. The different insert methods look the same but operates on different tables, the same observation holds for the different find methods. As good programmers, if we spot a repeated code, we should think how to unify it.
The work that is done by the different DAOs involves converting object to and from database records, the technical name for this process is called ORM - Object Relational Mapping. If we can create a generic ORM that knows how to convert data to any DTO than we can use it in order to create a generic DAO class. In order to create such generic ORM we need to make some assumptions about our DTO classes:

Each DTO class represents a single table
The different DTO classes are obeying a common naming conventions: A DTO class named Foo will represents a table named foos
The different DTO classes has a constructor that accept all their fields, the name of the constructor arguments is the same as the name of the fields
The name of the fields of each DTO class is the same as the column names in the database

If you examine these restrictions closely you will find out that by examining the code of a DTO class that follow these restrictions we can easily infer how the corresponding database table looks like. Since python is an interpreted language, it actually contains modules that help us query the interpreter about the structure of any class (including its methods, constructors etc.). One of this modules is called inspect and it contains the method getargspec that returns the names of the arguments of a function or a constructor.
As our generic ORM need to convert from database data (that is retrieved by a cursor) to a DTO, we can use the getargspec method in order to get the name of the arguments in the constructor of the DTO that we want to convert our database data to. Recall that we assume that these arguments names are the same as the columns names of the data to convert.

But getting the structure of the DTO class is only half of the information our ORM needs, the other half is getting the structure of the data that it needs to convert from. Recall that this data came from a cursor object, luckily for us, the cursor object has a function named description which returns the column names of the last query. it returns a list which contains a 7-tuple for each column where the first item in this tuple is the column name.

Using the information above, we can now create a generic ORM method, the method will receive a cursor and a DTO type, it will examine the constructor arguments of the DTO class and the column names inside the cursor. It can then create a mapping array col_mapping such that for each constructor argument in position i, col_mapping[i] is the index of the corresponding column inside the data stored in the cursor. Finally it will loop over the data inside the cursor and use the col_mapping in order to construct and return one DTO object per data row.

downloadtoggle
51 lines ...
#file dbtools.py
 
import inspect
 
def orm(cursor, dto_type):
 
    #the following line retrieve the argument names of the constructor
    args = inspect.getargspec(dto_type.__init__).args
    
    #the first argument of the constructor will be 'self', it does not correspond 
    #to any database field, so we can ignore it.
    args = args[1:]  
 
    #gets the names of the columns returned in the cursor
    col_names = [column[0] for column in cursor.description]
    
    #map them into the position of the corresponding constructor argument
    col_mapping = [col_names.index(arg) for arg in args]
    return [row_map(row, col_mapping, dto_type) for row in cursor.fetchall()]
 
 
def row_map(row, col_mapping, dto_type):
    ctor_args = [row[idx] for idx in col_mapping]
    return dto_type(*ctor_args)
 
 
#we can use our method above in order to start writing a generic Dao
#note that this class is not complete and we will add methods to it next
class Dao(object):
    def __init__(self, dto_type, conn):
        self._conn = conn
        self._dto_type = dto_type
 
        #dto_type is a class, its __name__ field contains a string representing the name of the class.
        self._table_name = dto_type.__name__.lower() + 's'
 
    def insert(self, dto_instance):
        ins_dict = vars(dto_instance)
 
        column_names = ','.join(ins_dict.keys())
        params = ins_dict.values()
        qmarks = ','.join(['?'] * len(ins_dict))
 
        stmt = 'INSERT INTO {} ({}) VALUES ({})'\
               .format(self._table_name, column_names, qmarks)
 
        self._conn.execute(stmt, params)
 
    def find_all(self):
        c = self._conn.cursor()
        c.execute('SELECT * FROM {}'.format(self._table_name))
        return orm(c, self._dto_type)
The code above used the join function of the type string. This method receive a sequence (e.g., list or tuple) and returns a string, which is the concatenation of the strings in the sequence. The separator between elements is the string providing this method. For example, 'x'.join(['a', 'b', 'c']) = 'axbxc'.
The code above contains both an implementation of a generic orm function and a generic Dao class that use it. The generic Dao class is not completed. It only supports getting all the items in the table. What if we want to get a specific item? (in our example application, we want to get an assignment with a specific num).

We can support such queries by creating a find function that receives a dictionary with column/value entries. We can then construct a select statement that includes a WHERE clause which represents the conjunction of the keys in the dictionary. For example, if we want to get an assignment with num=7, we will be able to do so by calling find({'num':7}).

Python actually has a special syntax for methods that receives dictionary like our find method using the ** operator.

download
def foo(dict):
  print dict
 
def cool_foo(**dict):
  print dict
 
foo({'a':1, 'b':2}) #prints {'a':1, 'b':2}
cool_foo(a=1, b=2) #prints {'a':1, 'b':2}
With this knowledge, we can add the find method to our generic Dao class:

downloadtoggle
12 lines ...
class Dao(object):
  #previous code ...
 
   def find(self, **keyvals):
        column_names = keyvals.keys()
        params = keyvals.values()
 
        stmt = 'SELECT * FROM {} WHERE {}' \
               .format(self._table_name, ' AND '.join([col + '=?' for col in column_names]))
 
        c = self._conn.cursor()
        c.execute(stmt, params)
        return orm(c, self._dto_type)
This conclude our generic Dao class, we can now adapt our persistence layer as follows:

downloadtoggle
43 lines ...
#file: persistence.py
 
import sqlite3
import atexit
from dbtools import Dao
 
# Data Transfer Objects:
class Student(object):
    def __init__(self, id, name):
        self.id = id
        self.name = name
 
 
class Assignment(object):
    def __init__(self, num, expected_output):
        self.num = num
        self.expected_output = expected_output
 
 
class Grade(object):
    def __init__(self, student_id, assignment_num, grade):
        self.student_id = student_id
        self.assignment_num = assignment_num
        self.grade = grade
 
 
#Repository
class Repository(object):
    def __init__(self):
        self._conn = sqlite3.connect('grades.db')
        self._conn.text_factory = bytes
        self.students = Dao(Student, self._conn)
        self.assignments = Dao(Assignment, self._conn)
        self.grades = Dao(Grade, self._conn)
 
    def _close(self):
        #see code in previous version...
 
    def create_tables(self):
        #see code in previous version...
 
# singleton
repo = Repository()
atexit.register(repo._close)
And to so, our assignment checker has the following minor changes:

downloadtoggle
26 lines ...
from persistence import *
 
import os
import imp
 
def grade(assignments_dir, assignment_num):
    expected_output = repo.assignments.find_one(num=assignment_num).expected_output #CHANGED
 
    for assignment in os.listdir(assignments_dir):
        (student_id, ext) = os.path.splitext(assignment)
 
        code = imp.load_source('test', assignments_dir + '/' + assignment)
 
        student_grade = Grade(student_id, assignment_num, 0)
        if code.run_assignment() == expected_output:
            student_grade.grade = 100
 
        repo.grades.insert(student_grade)
 
 
def print_grades():
    print 'grades:'
    for grade in repo.grades.find_all():
        student = repo.students.find(id = grade.student_id)[0] #CHANGED
 
        print 'grade of student {} on assignment {} is {}'\
            .format(student.name, grade.assignment_num, grade.grade)
# Persistence Layer

## What Is Persistence Layer

Assume we have an application that is storing and accessing data to/from a DB, as seen in class, if the application accesses the DB directly, each change in the DB structure, table format or application needs will require changes in several parts of the code. Persistence layer is a design pattern for managing the storing and accessing of permanent data. it manages the communication between the application and the DB and creates a separation between the application logic and the DB access which allows for better code stability.

In order to demonstrate an implementation of the persistence layer design pattern we will review the implementation of the assignment tester as seen in class. Reminder, the assignment tester is a Python application designed to grade students' assignments. it uses a DB with the following tables:

* **students** - which includes the id of the student and its name
* **assignments** - which includes the number of the assignment and a string representing the expected output of the run_assignment method of the corresponding assignment
* **grades** - which includes the grade of each student on a specific assignment

the implementation of a persistence later revolves around 3 types of objects:
* **DTO** - Data Transfer Object
* **DAO** - Data Access Object
* **Repository**

the full working code can be found in the class materials.

### DTO - Data Transfer Object

A DTO is an object that represents a record in most cases from a single table. Its variables represent the columns of the table.
DTOs are passed to and from the persistence layer. When passed from the persistence layer to the application logic, they contain the data retrieved from the database. When passed from the application logic to the persistence layer, they contain the data that should be written to the database.

**DTO naming convention**
The DTO naming convention is that a DTO named 'Abc' represents a table named 'abcs'. we will use this convention in the future to map a DTO object to the table it 

** Data Transfer Objects**
```python
class Student:
    def __init__(self, id, name):
        self.id = id
        self.name = name
 
 
class Assignment:
    def __init__(self, num, expected_output):
        self.num = num
        self.expected_output = expected_output
 
 
class Grade:
    def __init__(self, student_id, assignment_num, grade):
        self.student_id = student_id
        self.assignment_num = assignment_num
        self.grade = grade
```
       
### DAO - Data Access Object

These objects contain methods for retrieving and storing DTOs, In most cases, each DAO is responsible for a single DTO.

** Data Access Objects:**

```python
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
 
        return [Grade(*row) for row in all]]
```
        
### Repository

In many cases, DTO and DAOs are not sufficient, since each DAO only knows how to handle a specific DTO, where should we put methods that aren't related to just a single DTO(table)? for example:
* create_tables method?
* queries that span multiple tables (using join)?
* which DAO should hold these methods? The answer is in the repository. the repository is similar to a DAO but it manages a group of related DTOs.

```python
#The Repository
class _Repository:
    def __init__(self):
        self._conn = sqlite3.connect('grades.db')
        self.students = _Students(self._conn)
        self.assignments = _Assignments(self._conn)
        self.grades = _Grades(self._conn)
 
    def _close(self):
        self._conn.commit()
        self._conn.close()
 
    def create_tables(self):
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
 
# the repository singleton
repo = _Repository()
atexit.register(repo._close)
```

**Application logic**

```python
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
    print('grades:')
    for grade in repo.grades.find_all():
        student = repo.students.find(grade.student_id)
 
        print('grade of student {} on assignment {} is {}'.format(student.name, grade.assignment_num, grade.grade))
supporting additional features
Join query
looking at the method print_grades at the application logic implementation above, we can see that the method is inefficient. in order to find the name of the student to whom belongs the grade, the method goes over all the students one by one. this could have been prevented if we used a join query to match between the students grade and it's name. but where will we place such a query? the answer is obviously in the repository.
downloadtoggle
29 lines ...
class StudentGradeWithName:
    def __init__(self, name, assignment_num, grade):
        self.name = name
        self.assignment_num = assignment_num
        self.grade = grade
 
#The Repository
class _Repository:
    def __init__(self):
        # see code above...
        
    def _close(self):
        # see code above...
 
    def create_tables(self):
        # see code above...
 
    def get_grades_with_names(self):
        c = self._conn.cursor()
        all = c.execute("""
            SELECT students.name, grades.assignment_num, grades.grade 
            FROM grades
            JOIN students ON grades.student_id = students.student_id
        """).fetchall()
 
        return [StudentGradeWithName(*row) for row in all]
 
# the repository singleton
repo = _Repository()
atexit.register(repo._close)
```

With that addition to the repository our print_grades function can simply be:

```python
def print_grades():
    print('grades:')
    for studentGradeWithName in repo.get_grades_with_names():
        print('grade of student {} on assignment {} is {}'.format(studentGradeWithName.name, studentGradeWithName.assignment_num, studentGradeWithName.grade))
```

**Update**

Suppose we want our assignment tester to also support in appeals, we will need a method to update a student's assignment grade. The DAOs we have seen so far only supported 2 functions, insert and find. lets try and add support for the appeals in the *_Grades* class:

```python
class _Grades:
    def __init__(self, conn):
        # see code above...
 
    def insert(self, grade):
        # see code above...
 
    def find_all(self):
        # see code above...
 
    def update(self, grade):
        self._conn.execute("""
               UPDATE grades SET grade=(?) WHERE student_id=(?) AND assignment_num=(?)
           """, [grade.grade, grade.student_id, grade.assignment_num])
```

Now we have support for appeals, but what if we want to support additional update functions such as update student name (in case of a marriage :))? In class we have seen the ORM and generic DAO classes designed to implement generic DAO methods. lets first revise the ORM and then try and add an update method to the generic DAO class.

### ORM - Object Relational Mapping

The ORM is a method for mapping between a certain DTO object and its related table in a manner that suits any given DTO. reminder, the DTO uses several assumptions in order to function properly:
* Each DTO class represents a single table
* The different DTO classes are obeying a common naming conventions: A DTO class named Foo will represents a table named foos
* DTO constructor parameters names == DTO fields names == table represented by the DTO column names

```python
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
```

Lets go over the code above line by line to make sure we understand it.

line number 1: using the inspect module, we get the arguments of the constructor of the dto_type received. dto_type is a class type, __init__ is the constructor.
line number 2: the first argument of the constructor (or any method) is 'self' so we remove it cause it does not represent a column in the table.
line number 3: gets the name of the columns from the table th cursor last executed on. reminder, the ORM method will be called after a select has been executed.
line number 4: creates an array that at each index i, it will hold j where j is the column number in the table of the ith element in the constructor arguments.
line number 5: using the col_mapping created the previous line and the row_map method, it will create a DTO object for each record returned from the select command executed before the ORM method was called.

Using the ORM, we can construct a generic DAO:

```python
class Dao:
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
 
        stmt = 'INSERT INTO {} ({}) VALUES ({})'.format(self._table_name, column_names, qmarks)
 
        self._conn.execute(stmt, params)
 
    def find_all(self):
        c = self._conn.cursor()
        c.execute('SELECT * FROM {}'.format(self._table_name))
        return orm(c, self._dto_type)
 
    def find(self, **keyvals):
        column_names = keyvals.keys()
        params = keyvals.values()
 
        stmt = 'SELECT * FROM {} WHERE {}'.format(self._table_name, ' AND '.join([col + '=?' for col in column_names]))
 
        c = self._conn.cursor()
        c.execute(stmt, params)
        return orm(c, self._dto_type)
```

**Generic delete**

Now after we have learned about the ORM and generic DAO, we can add more generic methods. before we try to add a generic update, lets start with something simpler and add a generic delete. notice that the SQL delete command has a structure similar to that used in our find method. so adding delete will be just a minor modification to our find method that instead of returning the DTO objects using the ORM, it will simply execute the delete.

```python
class Dao:
    def __init__(self, dto_type, conn):
        # see code above...
 
    def insert(self, dto_instance):
        # see code above...
 
    def find_all(self):
        # see code above...
 
    def find(self, **keyvals):
        # see code above...
 
    def delete(self, **keyvals):
        column_names = keyvals.keys()
        params = keyvals.values()
 
        stmt = 'DELETE FROM {} WHERE {}'.format(self._table_name, ' AND '.join([col + '=?' for col in column_names]))
 
        c = self._conn.cursor()
        c.execute(stmt, params)
```

**Generic Update**

After we have seen how to add generic delete, lets finally try and add our generic update. in order to add generic update, we must allow for both a number of set values and a complex where condition. that will require us to use two different dictionaries, one containing the set values, and the other the condition.

```python
class Dao:
    def __init__(self, dto_type, conn):
        # see code above...
 
    def insert(self, dto_instance):
        # see code above...
 
    def find_all(self):
        # see code above...
 
    def find(self, **keyvals):
        # see code above...
 
    def delete(self, **keyvals):
        # see code above...
 
    def update(self, set_values, cond):
        set_column_names = set_values.keys()
        set_params = set_values.values()
 
        cond_column_names = cond.keys()
        cond_params = cond.values()
 
        params = list(set_params) + list(cond_params)
 
        stmt = 'UPDATE {} SET {} WHERE {}'.format(self._table_name,
                                                      ', '.join([set + '=?' for set in set_column_names]),
                                                      ' AND '.join([cond + '=?' for cond in cond_column_names]))
 
        self._conn.execute(stmt, params)
```

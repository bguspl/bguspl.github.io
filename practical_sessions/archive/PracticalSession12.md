Basic SQL
SQL - Structured Query Language
We would like to have a system that stores data that will allow us to:
Store and access data without having to deal with the low level implementation details of storing it (like the syntax of XML files).
Access specific records without having to read entire files.
Define relations between different "files".

SQL is the standard way to interact with relational databases. It will allow us to interact with a wide variety of relational database management systems (RDBMS) that solve those issues (like MySQL, PostgreSQL, Microsoft SQL Server, and more).
SQL consists of two parts:

Data Definition Language
Data Manipulation Language

The first is used to define tables, including their layout, fields, what type each field has, what foreign keys they have. The second is used to insert/update/delete data. All data inserted with DML has to be 100% consistent with the former decisions of the DDL.
Definitions
Table - A table in the database, it holds only one kind of records. In the following example each record (row) is comprised of (int, string, string).
TEACHING_ASSISTANTS table:

ID	Name	Office hours
1	Majeed	Sun 10:00-12:00
2	Ben	Tue 17:00-19:00
3	Matan	Wed 16:00-18:00
4	Dan	Mon 11:15-13:15
5	Hagit	Thu 14:15-16:15
6	Eran	Thu 14:00-16:00
7	Hussein	Mon 18:15-20:15
8	Irit	Tue 12:00-14:00
9	Morad	Mon 18:00-20:00
10	Yair	Tue 16:00-18:00

Record - a row from the table. This represents a single entry in a table.
A record from that table:
4	Dan	Mon 11:15-13:15

Primary key - A field that is unique in a table.
The field id is the primary key in this example.

Foreign key - A "pointer" to another a record in another table. This will allow us to define relations between tables.
In the following table, the field TA_ID is a foreign key, pointing at records in the TEACHING_ASSISTANTS table:
TA_ID	GroupNum	Location	Time
3	11	90-234	Sun 14-16
3	12	28-103	Sun 18-20
4	13	90-141	Wed 14-16
6	21	72-123	Thu 12-14
1	22	90-239	Sun 12-14
Data Definition Language
The Data Definition Language (DDL) is used to create and destroy databases and database objects. These commands will primarily be used by database administrators during the setup and removal phases of a database project.
Create table
Creates a tables in memory:
Syntax
download
CREATE [TEMP] [CACHED|MEMORY|TEXT] TABLE name 
( columnDefinition [, ...] ) 
 
columnDefinition: 
column DataType [ [NOT] NULL] [PRIMARY KEY] 
DataType: 
{ INTEGER | DOUBLE | VARCHAR | DATE | TIME |... }
Example
download
CREATE TABLE STORE
(id INTEGER, Name VARCHAR(30), Type VARCHAR(30))
Primary Key
A primary key is used to uniquely identify each row in a table. It can either be part of the actual record itself , or it can be an artificial field (one that has nothing to do with the actual record). A primary key can consist of one or more fields on a table. When multiple fields are used as a primary key, they are called a composite key.
Example:
download
CREATE TABLE address
(id INTEGER, Street VARCHAR(50), City VARCHAR(30), HouseNumber INTEGER, ZipCode INTEGER,
primary key (id))
The part primary key (id) defines the field id to be unique in the whole table. Meaning if we have one field where id = 5 then no other record in that table can have the value 5 in the id field.

Try inserting the same record twice to 'address' table. Can this succeed? What do you expect?
Foreign Key
A foreign key is a field (or fields) that points to the primary key of another table. The purpose of the foreign key is to ensure referential integrity of the data. In other words, only values that are supposed to appear in the database are permitted. For example, say we have two tables, like before. A TEACHING_ASSISTANTS table, and a PRACTICAL_SESSIONS table. In the practical session you may insert only TAs that are already in the TEACHING_ASSISTANTS table. In this case, we will place a foreign key on the PRACTICAL_SESSIONS table and have it relate to the primary key of the TEACHING_ASSISTANTS table. This way, we can ensure that all practical sessions in the PRACTICAL_SESSIONS table are related to a TA in the Teaching Assistant table. In other words, the PRACTICAL_SESSIONS table cannot contain information on a TA that is not in the TEACHING_ASSISTANTS table.
The structure of these two tables will be as follows:

Column	type	characteristic
ID	INTEGER	Primary Key
Name	VARCHAR(50)	
Office hours	VARCHAR(9)	

Column	type	characteristic
TA_ID	INTEGER	Foreign Key
GroupNum	INTEGER	Primary Key
Location	VARCHAR(50)	
Time	VARCHAR(9)	
In the above example, the TA_ID column in the PRACTICAL_SESSIONS table is a foreign key pointing to the ID column in the TEACHING_ASSISTANTS table.
Syntax:
download
CREATE TABLE TEACHING_ASSISTANTS
(
    ID INTEGER PRIMARY KEY,
    Name VARCHAR(50),
    OfficeHours VARCHAR(9)
)
download
CREATE TABLE PRACTICAL_SESSIONS
(
    TA_ID INTEGER REFERENCES TEACHING_ASSISTANTS(ID),
    GroupNum INTEGER PRIMARY KEY,
    Location VARCHAR(50),
    Time VARCHAR(9)
)
Data Manipulation Language
The Data Manipulation Language (DML) is used to retrieve, insert and modify database information. These commands will be used by all database users during the routine operation of the database.
Insert / Delete / Update
Insert
Adds one or more new rows of data into a table.
Syntax:

download
INSERT INTO table [ (column [,...] ) ] 
{ VALUES(Expression [,...]) | SelectStatement }
Write a statement that adds a row to the PRACTICAL_SESSIONS table.

Delete
Removes rows in a table. (And not the table itself!)
Syntax:
download
DELETE FROM table [WHERE Expression]
Delete one of the records you just added.

Update
The UPDATE command can be used to modify information contained within a table, either in bulk or individually.
Syntax:
download
UPDATE table SET column = Expression [, ...] 
[WHERE Expression]
Example: Our university decided to add an additional practical session to Morad. The following SQL command would do that:

download
INSERT INTO PRACTICAL_SESSIONS (TA_ID, Group, Location, Time)
VALUES (9, 43, "72-218", "Sun 10-12")
Later there turns out to be a mistake and the Time should be "Thu 10-12". The following SQL command corrects that:

download
UPDATE PRACTICAL_SESSIONS
SET Time = "Thu 10-12"
WHERE GroupNum = 13
Simple Select
The SELECT command is the most commonly used command in SQL. It allows database users to retrieve the specific information they desire from an operational database.
Syntax:
download
SELECT [DISTINCT] 
{ selectExpression | table.* | * } [, ... ] 
[INTO [CACHED|TEMP|TEXT] newTable] 
FROM tableList 
[WHERE Expression] 
[GROUP BY Expression [, ...] ]
[ORDER BY selectExpression [{ASC | DESC}] [, ...] ] 
[LIMIT n m]
[UNION [ALL] selectStatement]
The command shown below retrieves all of the information contained within the TEACHING_ASSISTANT table. Note that the asterisk is used as a wildcard in SQL. This literally means "Select everything from the TEACHING_ASSISTANT table."

download
SELECT *
FROM TEACHING_ASSISTANT
Alternatively, users may want to limit the attributes that are retrieved from the database. For example, you may require a list of the last names of all TAs . The following SQL command would retrieve only that information:

download
SELECT Name
FROM TEACHING_ASSISTANT
Finally, the WHERE clause can be used to limit the records that are retrieved to those that meet specified criteria. We might be interested in reviewing the records of TAs that have office hours at Sunday. The following command retrieves all of the data contained within TEACHING_ASSISTANT for records that have Office hours at Sunday.

download
SELECT *
FROM TEACHING_ASSISTANT
WHERE OfficeHours LIKE 'Sun%'

Join operation

Now that we have seen a simple select operation from a single table, we would like to access information across a few tables.
The join action will allow us to do so.
The following code will retrieve all a Cartesian product of the information from TEACHING_ASSISTANTS and PRACTICAL_SESSIONS.
download
SELECT *
FROM TEACHING_ASSISTANTS,PRACTICAL_SESSIONS

Most of the time we would like to use the join operation to connect related information from given tables.
for example both TEACHING_ASSISTANTS and PRACTICAL_SESSIONS have a column referring to the ID of a given teaching assistant.
The following code will retrieve a table containing the name of each TA and his practical session time.
download
SELECT ta.Name, ps.GroupNum, ps.Location, ps.Time
FROM TEACHING_ASSISTANTS as ta
JOIN PRACTICAL_SESSIONS as ps
ON ta.ID = ps.TA_ID

The ON operation lets us choose what is the connection between the two tables we would like to connect on.
Multiple attributes can be compared by using the AND / OR keyword between comparisons.
* The AS keyword can be used to give a temporary name to a table or selected attribute.
The basic JOIN operation (also called inner join) will ignore any line from TEACHING_ASSISTANTS that does not fit any line in PRACTICAL_SESSIONS.
For example if TEACHING_ASSISTANTS had another TA that had no PRACTICAL_SESSIONS, he would not appear in an inner join.
We can force the join to ignore such lines and show them any way (with NULL values in the values concerning PRACTICAL_SESSIONS columns) by using the LEFT JOIN.
The following code will retrieve a table containing all TAs with their PS info, or NULL if no PS exists.

download
SELECT ta.Name, ps.GroupNum, ps.Location, ps.Time
FROM TEACHING_ASSISTANTS as ta
LEFT JOIN PRACTICAL_SESSIONS as ps
ON ta.ID = ps.TA_ID

Name	GroupNum	Location	Time
Majeed	23	90-239	Sun 12-14
Ben	NULL	NULL	NULL
Matan	13	90-141	Wed 14-16
Dan	22	72-123	Thu 12-14
Hagit	NULL	NULL	NULL
Eran	NULL	NULL	NULL
Hussein	NULL	NULL	NULL
Irit	NULL	NULL	NULL
Morad	12	28-103	Sun 18-20
Morad	11	90-234	Sun 14-16
Yair	NULL	NULL	NULL
Basic Python
What is Python?
Python is an open-source, general purpose programming language that is dynamic, strongly-typed, object-oriented, functional, and memory-managed.
Python is an interpreted language, meaning that it uses an interpreter to translate and run its code.
The interpreter reads and executes each line of code one at a time, just like a script, and hence the term "scripting language"
Python is dynamic which means that the types are checked only at runtime. But Python is also strongly typed, just like Java. You can only execute operations that are supported by the target type.
Note that this means that variable names do not have a type in Python, their values do.
Coding Python
Being an interpreted language, there are two ways to code Python: using Python's REPL (Read-Eval-Print-Loop), or using files. The REPL is used for "exploratory coding", when we just want to check how something works without the hassle of creating a new file, etc. For example, run python3 in the labs:
~> python3                                                    
Python 3.5.2 (default, Nov 23 2017, 16:37:01)                         
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print('Hello, Python!')
Hello, Python!
>>> import time
>>> time.time()
1514924068.7659469
>>> exit()
~>
Python source files use the ".py" extension and are called "modules." With a Python module hello.py, the easiest way to run it is with the shell command python3 hello.py Morad which calls the Python interpreter to execute the code in hello.py, passing it the command line argument "Morad", Command line arguments will reside in the argv structure of the standard module sys:

downloadtoggle
13 lines ...
# This is a comment.
# Import sys module
import sys
 
# Gather our code in a main() function
def main():
    print('Hello,', sys.argv[1])
    # Command line args are in sys.argv[1], sys.argv[2] ...
    # sys.argv[0] is the script name itself and can be ignored
 
# Standard boilerplate to call the main() function to begin
# the program.
if __name__ == '__main__':
    main()
Special variables
Notice the part if __name__ == '__main__': main(). Remember that we said that Python is a 'scripting' language. If we omit this part, then Python will execute the code line by line in a top-bottom manner. But since we want a definitive start point, we can use __name__ and compare it to '__main__'. Before executing the code, Python will define a few special variables. For example, if the Python interpreter is running that module (the source file) as the main program, it sets the special __name__ variable to have a value "__main__". If this file is being imported from another module, __name__ will be set to the module's name.
Python indentation
One Python feature is that the whitespace indentation of a piece of code affects its meaning. A logical block of statements such as the ones that make up a function should all have the same indentation, set in from the indentation of their parent function or "if" or whatever. If one of the lines in a group has a different indentation, it is flagged as a syntax error.
Strings
String literals can be enclosed by either double or single quotes, although single quotes are more commonly used:
download
myStr = 'hello.'
mySecondStr = "hello."
Python strings are "immutable" - they cannot be changed after they are created. For example:

download
str1 = 'hello'
str2 = str1 + ' world'
In the above code, the expression str2 = str1 + ' world' takes the two strings str1 and ' world' and builds a new string out of them. We can use the print function to print to the screen. We can also use the str(_) function to convert non-string variables to string:

download
s = 'hello'
print(s[1])          # e
print(len(s))        # 5
print(s + ' there')  # hello there
pi = 3.14
text = 'The value of pi is ' + str(pi)
print(text)          # The value of pi is 3.14
There are alot of helpful string methods:

s.lower() – returns the lowercase version of a string
s.upper() – returns the upper version of a string
s.strip() – returns a string with the whitespaces removed from the start to the end
s.replace('old', 'new') – returns a new string where all occurrences of 'old' have been replaced by 'new'
s.split('delim') – returns a list of substrings separated by the given delimter.
For example: 'aaa,bbb,ccc'.split(',') gives ['aaa', 'bbb', 'ccc'].

Python also has a printf()-like facility to put together a string: The % operator.
download
text = "SPL %d is the most awesome course ever. %s and I take it this year!" % (181, 'Mary')
'If' statement
download
if x >= 0.5 and y >= 0.5:
  print('All above half')
elif x >= 0.5 or y >= 0.5:
  print('One of them is above half.')
else:
  print('None of them is above half.')
Notice the : at the end of the if, elif, else lines. elif is basically else if in Java. Notice also the operators or and and – these are the logical operators in Python.
Python Lists
Python has lists. Lists are written within square brackets []. Lists work similarly to strings - use the len() function and square brackets [] to access data. As in Java, the first element is at index 0.
download
cars = ['Ford', 'Honda', 551]
print(cars[0])   # Ford
print(cars[1])   # Honda
print(cars[2])   # 551
print(len(cars)) # 3
Yes. you can mix types in a list in Python. But that is not recommended for your own type-safety.

List methods
list.append(elem) - adds a single element to the end of the list. Does not return the new list, just modifies the original
list.insert(index, elem) - inserts the element at the given index, shifting elements to the right
list.remove(elem) - searches for the first instance of the given element and removes it
list.extend(list2) - adds the elements in list2 to the end of the list. You can also use += on a list for the same effect
list.sort() - sorts the list in place (does not return it)
list.pop(index) - removes the returns the element at the given index
For, in, while, if in
There are no loops of the syntax for (i=0;i<....;....) in Python.
download
for car in cars:
  print(car)
This is how we run on the elements of a list. Alternatively:

download
i = 0
while i < len(cars):
  print(cars[i])
  i += 1
Or, we can create an iterator over a range of numbers using range(start, end) (from start until end, excluding end):

download
for i in range(0, len(cars)):
  print(cars[i])
In all cases, the output will be:

download
Ford
Honda
551
Notice that Python does not have the support for the syntax i++. Only the i = i + someNumber or i += someNumber syntax is accepted.
This is how you check if an element is a member of a list:

download
if 'Ford' in cars:
  print('Yay!')
List and String slicing
In Python, lists and strings support slicing: it is a useful way to refer to sub-parts of sequences.
s[x:y] is the elements of s starting at index x until index y EXCLUDING element at index y.
If x is omitted, then it assumes you want the elements starting from the first element until index y exclusive.
If y is omitted, then it assumes you want the elements from index x until the end of s.
The example is given on strings, but the same applies to lists:
download
s = "Morad"
str1 = s[1:4]   # str1 = "ora"
str2 = s[1:]    # str2 = "orad"
str3 = s[:]     # str3 = "Morad"
str4 = s[1:100] # str4 = "orad" - an index that is too big is truncated down to the string length
Python also supports negative indexes in slicing. If the string is s = "Morad", then s[-1] is d, s[-2] is a, etc. An index -n means "the n-th to last element".

Handling standard user input
Here is an echo example, that exits whenever we type exit
downloadtoggle
11 lines ...
def main():
    while True:
        print("Input: ")
        inputline = input()  # read a line from the user
        if inputline == 'exit':
            return
        else:
            outline = inputline + ".." + inputline[-2:] + ".." + inputline[-2:]
            print("Got echo: %s" % (outline,))
 
if __name__ == '__main__':
    main()
Output example:

Input: 
hello
Got echo: hello..lo..lo
Input: 
Morad
Got echo: Morad..ad..ad
Input: 
exit
File operations
We can open a file given the filename using open(filename). Then we can run on each line easily. Here is a short example that opens a file given in the command-line arguments:
downloadtoggle
9 lines ...
import sys
 
def main(args):
    inputfilename = args[1]
    with open(inputfilename) as inputfile:
        for line in inputfile:
            print(line)
 
if __name__ == '__main__':
    main(sys.argv)
The with keyword states that Python should automatically close the resources for us (ie, the filestream) when:

An exception occurs
The scope ends. In this case, the scope is the whole program run.

It is very similar to the try-with-resources concept of Java.
A useful method that checks whether a file exists is:

download
import os
os.path.isfile(filename)
Tuples and Dicts
Tuples and Dicts are very strong structures in Python which we are interested in.
Read about tuples here: https://www.tutorialspoint.com/python/python_tuples.htm
Read about dicts here: https://www.tutorialspoint.com/python/python_dictionary.htm
Basic SQLite
SQLite is a library that provides a lightweight disk-based database that doesn't require a separate server process and allows accessing the database using a nonstandard variant of the SQL query language. There exists a module called sqlite3 that provides an SQLite interface for python.
Connection and access
To use the module we must first create a connection to the database.
If the given database name does not exist, it will be created.
download
import sqlite3 
dbcon = sqlite3.connect('example.db')
Once we have a connection, we would like to execute commands. In order to be able to do so, we must create a cursor object.
A cursor object is an object that lets you traverse records in a database. Cursors facilitate subsequent processing in conjunction with the traversal, such as retrieval, addition and removal of database records.

download
import sqlite3
dbcon = sqlite3.connect('example.db')
with dbcon:
    cursor = dbcon.cursor()
Executing commands
Now that we have a cursor, we can execute SQL commands:
download
import sqlite3
dbcon = sqlite3.connect('example.db')
with dbcon:
    cursor = dbcon.cursor()
    cursor.execute("CREATE TABLE Students(ID INTEGER PRIMARY KEY, NAME TEXT NOT NULL)") # create table students
    cursor.execute("INSERT INTO Students VALUES(?,?)", (1, 'Morad')) # add entry 'id = 1, name = Morad' into the table.
    cursor.execute("INSERT INTO Students VALUES(?,?)", (2, 'Harry Potter'))
Notice that (1, 'Morad',) is a Python tuple. Note the syntax of ?; these are fields that are replaced one by one using the given tuple.

Queries with results
As we have seen, some SQL queries yield result(s), like the SELECT query. These can be fetched from the cursor after executing the query.
Cursors have two methods that interest us regarding this:
cursor.fetchone()
fetches one of the results of the query in the form of a tuple, or None if none is available
cursor.fetchall()
fetches all of the results of the query, returning a list of tuples

Let's see this by example:
downloadtoggle
31 lines ...
import sqlite3
import os
 
databaseexisted = os.path.isfile('example.db')
 
dbcon = sqlite3.connect('example.db')
 
with dbcon:
    cursor = dbcon.cursor()
    if not databaseexisted: # First time creating the database. Create the tables
        cursor.execute("CREATE TABLE Students(ID INTEGER PRIMARY KEY, NAME TEXT NOT NULL)") # create table students
        cursor.execute("INSERT INTO Students VALUES(?,?)", (1, 'Morad',)) # add entry 'id = 1, name = Morad' into the table.
        cursor.execute("INSERT INTO Students VALUES(?,?)", (2, 'Harry Potter',))
 
    # let's get all students and print their entries
    cursor.execute("SELECT * FROM Students");
    studentslist = cursor.fetchall()
    print("All students as list:")
    print(studentslist)
    print("All students one by one:")
    for student in studentslist:
        print("Student name: " + str(student))
 
    # let's get the name of the student of id 1
    cursor.execute("SELECT NAME FROM Students WHERE ID=(?)", (1,))
    studentwithid1 = cursor.fetchone()
    print("Student with id 1: " + str(studentwithid1))
 
    # let's get the name of the student of id 5
    cursor.execute("SELECT NAME FROM Students WHERE ID=(?)", (5,))
    studentwithid5 = cursor.fetchone()
    print("Student with id 5: " + str(studentwithid5))
Note the commas in (1,) and (5,) to make them a single element tuple, e.g. (1) would be evaluated as 1.

Here is the output of the above script:

All students as list:
[(1, 'Morad'), (2, 'Harry Potter')]
All students one by one:
Student name: (1, 'Morad')
Student name: (2, 'Harry Potter')
Student with id 1: ('Morad',)
Student with id 5: None
Notes:

In order not to try to create the tables and put the entries each time, we checked if the file existed. If it did, we know that we have created the database, the tables, and entered the data in the past, so we don't need to do that again. This was achieved using os.path.isfile
Note how the last example gave us None because no student exists with id 5
Why are we so sure that we should use fetchone and not fetchall in the second and third calls to the db? Because we select rows by the field ID which is a primary key, so we are sure it is unique, so at most one row exists with such an ID.
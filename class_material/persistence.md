# **Persistence Data Storage**

## **Objectives**

This lecture explains how the RTE provides services to processes to support persistent data storage. Up to this point, we studied how memory is managed within the virtual memory space of a process, and how messages can be exchanged between processes, using inter-process communication services from the RTE. We now turn to the issue of storing data such that it can remain unchanged beyond the end of the process execution. This allows us to compute and store data in one run of a program, then retrieve the same data in another run. It also allows us to share data among several processes - as long as they are provided access to the same storage location.
Persistent storage has several properties:

* It is persistent
* It can be shared among several processes

We discuss in this lecture two types of persistent storage services: the file system and database services.

## **File System Services**

RTEs often provide access to a file system. The file system provides the following interface to processes:

* Map names to storage locations: a storage location consists of storage capacity on a disk. The file system maps abstract names (such as "/usr/local/file.ext") to concrete locations in the disk. The name mapping interface is exposed through system calls similar to: open(filename), create(filename), delete(filename) and operations to query the space of names getfiles(filename-pattern). The file system generally organizes the space of file names in a hierarchical structure of embedded directories.

* Provide a stream interface to read and write the content of a file as a stream of bytes. The file system does not provide any structure over the content of files - it just allows the process to arbitrarily seek any location within the file, read its content and write new content. The file system deals with allocation of space to files and automatic increase of the storage space required when new write operations are performed. This is exposed through system calls similar to seek, read, write.

* Provide locking over files: to allow multiple processes to synchronize access on shared files, the RTE provides services of file locking. Generally, one can lock a whole file or just a segment of bytes within the content of a file.

The file system is organized as an abstract interface of system calls (open, read, write, seek etc) which are implemented by specific drivers which know how to interact with specific hardware devices.

In recent years, file systems have been implemented in a distributed manner, so that one process can request access to files stored on a remote machine, through an implementation of the file system in the RTE which passes the system calls (open, read, etc) to a file system server on a server machine. Such distributed file systems (for example, NFS and SAMBA) have complex locking mechanisms.

### **File systems vs. DBMS**

A Database Management System (DBMS) is a different way of storing persistent data than using a file system. A DBMS is present as a service, to which processes connect through an inter-process communication protocol (generally over TCP). The DBMS manages the persistent storage requirements of its clients, often by accessing the file system on its side.

How is a DBMS different from the file system (FS)?

* **Data Model** – The data model employed by the FS is very simple: a stream of bytes with the open/close, read/write and lock operation. A DBMS defines a rich data model and manages serialization (encoding and decoding of complex data types between persistent storage and the process runtime objects). The programmer must define the specific data model for the data he is interested in storing.

* **Data Independence** – DBMSs provide an abstract interface for data storage/access. The programmer does not need to define in which file, at which offset data is to be stored. Instead, one can query the DBMS by content (e.g., "retrieve all content that satisfies certain criteria").

* **Efficient concurrency** – DBMSs are built to support thousands of concurrent users. They must ensure that data is kept consistent and provide efficient management of such high concurrency.

* **Reduced application development time** – DBMSs allow data manipulation using a simple API. The DBMS manages storage, query optimization, concurrency and integrity management, and, therefore, the application builder does not have to.

### **DBMS Services**

A DBMS server provides the following services to its clients:

* Security management: verify that users are authorized to access data. Specific rights can be granted for each part of the data and each group of users.

* Session and Transaction management: once users are authenticated on the server, they can request the execution of data transactions. A transaction is a complex data operations which may read and modify many different objects, but which is viewed from the outside as a single atomic operation, which either completely succeeds or is not performed at all (that is, one cannot perform "half a transaction").
* 
Query optimization and execution: clients interact with the DBMS by submitting complex queries, in a language supported by the DBMS. The DBMS decides how to execute these queries in the most efficient manner possible. This form of query compilation has been developed into extremely powerful techniques in the world of relational database servers.

**Back-end services** (not directly visible to the user but performed by the DBMS as part of its implementation):
* File and Access methods
* Buffer management
* Disk space management

### **Transaction**

A database transaction is a unit of work performed against a database management system or similar system that is treated in a coherent and reliable way independent of other transactions. A database transaction, by definition, must be atomic, consistent, isolated and durable (ACID acronym). A single transaction is composed of one or more independent units of work, each reading and/or writing information to a database or other data store.

Transactions provide an "all-or-nothing" proposition stating that work units performed in a database must be completed in their entirety or take no effect whatsoever. Further, transactions must be isolated from other transactions, results must conform to existing constraints in the database and transactions that complete successfully must be committed to durable storage.

A simple transaction is usually issued to the database system in a language like SQL wrapped in a transaction, using a pattern similar to the following:

1. Begin the transaction
2. Execute several data manipulations and queries
3. If no errors occur then commit the transaction
4. If errors occur then rollback the transaction

Transactions provide a different perspective on providing safety for concurrent systems than the locking methods we studied in Chapter 1. To implement transactions, the runtime environment employs the following logic:
* Identify the resources that will be accessed during the transaction. In the case of SQL, resources are full tables or parts of tables (sets of rows).
* Acquire locks on all the resources before starting the transaction. The locks indicate the intent of the transaction system to read or write the value of data held in the resources.
* Perform the code of the transaction. In general, modifications are not performed directly to the data, but a snapshot of all modification operations is kept in memory. The property of isolation means that the data read by the transaction is not affected by modifications performed concurrently by other threads and that the partial modifications which are not yet committed are also not visible by other threads as long as the transaction is not committed.
* When the transaction is committed, the modifications are merged to persistent storage and all other threads will see the modifications.
* If the transaction is rollbacked, the modifications are deleted and persistent storage is not modified.

## **Data Models**

A *data model* determines which data values can be sent for storage to a persistent storage service. The type of complex data values can be described in different ways. Up to this point, we have used the Object Oriented method of describing data models. This model is based on: primitive data types (int, bool, char etc) and methods to construct complex data types given existing data types. Complex data types in the object-oriented model are constructed using either: arrays, references (a reference to another data type is a new data type) and objects (a collection of named fields, each with its own type). Importantly, the object-oriented model assumes that objects have a distinct identity (independent of the value of their fields).

While the object-oriented model dominates in the field of programming languages, it has not achieved the same status in the world of DBMSs. The relational model introduced by E. F. Codd. in the 1970s is more popular. Most commercial and open source databases currently in use are based on the relational model and we will focus on it in the rest of the lecture. Hybrid models combining object and relational models exist as well but are not gaining much popularity.

### **Case Study: A Data Model for Colleges**

In the following, we will use as a running example the data model of an academic college. A college has different departments, each specializing in teaching a certain domain. For example the Computer Science (CS) department and the Physics department. A department has a department head and a geographical location. Students may be enrolled in one department and registered to courses given by it.

### **The Relational Model**

DBMSs allow the user to define data in terms of a data model instead of as physical data. "Relation" is a mathematical term for "table", and thus "relational" roughly means "based on tables". The relational model of data permits the database designer to create a consistent, logical representation of information. Consistency is achieved by including declared constraints in the database design, which is usually referred to as the logical schema. The theory includes a process of database normalization whereby a design with certain desirable properties can be selected from a set of logically equivalent alternatives.

The main building blocks of the relational model are:

* **Relation** - Think of it as a set of records, all with the same fields. The header of a table if you wish. In our example: students, departments, etc.

* **Attribute** - A single field within a relation, characterized by a data type and a name. In our example: student name, department head, etc.

* **Domain** - The definition of a new data type as a restriction of existing data types (for example, range of integers between 20 and 40). In our example: student ID is 9 digits 0-9 each.

* **Records** - Also known as tuple, is a single line within a relation.

* **Integrity constraints** - A constraint employed on an attribute with regard to all the records in the relation. For example - an ID field must remain unique in its relation. In our example: Student ID is unique.

### **Relational Algebra**
The relational algebra defines operators on the relations defined in a relational schema (set of relations). The main operators are:

* **Selection** - select specific rows from a relation.

* **Projection** - projects specific columns from a relation.

* **Set operations** - union, intersection, cross-product, set-difference and more on relations that have the same structure (same attributes).

* **Join** - cross product followed by selection and projection.

* **Renaming operation** - change name of columns.

These operations take existing relations as arguments and return new relations as a value.

Note that in the relational data model, the columns of tables are restricted to being simple values - one cannot have a column whose value is an embedded table or an array. Columns must be primitive data types. To describe complex data models with complex relations, one must model the objects using the notion of primary and foreign keys as described below.

## **SQL**

SQL (Structured Query Language) is a query language designed to help applications interface with a relational DBMS. It is closely related to the relational model. SQL provides two sub-languages:

* DDL: Data Definition Language - used to define schemas, relations and domains.
* DML: Data Manipulation Language - used to perform data queries, insertions, updates and deletions in a defined relational schema.

A client interacts with an SQL server by sending SQL queries (in general through a TCP protocol) and receiving as answers a result-set. The result set is not sent all at once to the client (since it could be pretty large). Instead, the client requests the rows from the result set as it needs it.

This means that the communication protocol between an SQL server and its clients is stateful and session oriented.

### **Keys**

Given a relation, one can define a key as a subset of the attributes of the relation which uniquely identify rows. For example, if we have a relation for students with attributes name, birthday, address and Social Security Number (SSN). Name is not a key (two students could have the same name). By definition, SSN is a key (this is why it was invented:). There could be several attributes in a key. For example, if the relation for courses has attributes (course name, department, semester, lecturer) then course name is not a key (the same course could be given in 2 different semesters) but the pair of attributes (course name, semester) could be a key.

### **Database Normalization and Foreign Keys**

When designing a database relational data model, it is critical to employ database normalization techniques. The idea is very simple: **minimize duplication of information** (also known as "Don't Repeat Yourself" - the DRY principle).

Duplication means the same data could appear twice in the domain of the problem modeled. For example in our case study assume we choose to define a relation student – with attributes: ID, Name and Course. To reflect that a student is enrolled to three courses we will need three records duplicating the ID and Name of the student.

To avoid this, we define in our model that student data will be stored only in a single relation. For other relations that refer to students, we rely on keys to remove data repetition: in the relation that stores information about courses taken by students, we use a single attribute SSN to refer to a student.

To retrieve the information on the student given a tuple from the courses relation, we need to operate a join operation on the SSN attribute (which is shared by the 2 relations) with a query similar to the following in SQL:

```sql
select student.ssn, student.name, course.coursename from
       student left join course on student.ssn = course.studentSSN
```

Note that in the the student relation, SSN is a key. In the course relation, StudentSSN is not a key - there can be several courses taken by a single student. But StudentSSN refers to the key of another table. This is the way in the relational model to define that one object **refers** to another: it is called a foreign key. StudentSSN is a *foreign key* in the course relation that refers to the student relation.

### **Data Integrity Constraints**

When foreign keys are defined, the DBMS server takes the responsibility to enforce data integrity constraints.

Example of such constraints are: one cannot create a tuple in the course relation that refers to a student that does not exist. As a consequence, the following operations must be verified by the server:

* Insert a new row in the course relation: check that the foreign key exists in the student relation.
* Update the SSN field of the course relation: check that the new value for StudentSSN exists in the student relation.
* Delete a row in the student relation: verify that there are no rows in course that refer to the row to be deleted.
* Update a row in the student relation: verify that there are no rows in course that refer to the old value to be updated.

When defining a data model on a server, the user declares which constraints are part of the model: keys and foreign keys. The DBMS server then enforces the corresponding constraints and verifies each insert, update and delete operations so that they do not conflict with existing constraints.

#### **NULL**
NULL is a special value in SQL which belongs to all domains (a column of domain int, string, date etc can all have the value NULL). NULL has a very special meaning: it indicates lack of knowledge. That is, when a column is set to NULL, it means we do not know what its value is (which is very different from saying that we know that its value is NULL).

For example (```1 = NULL```) is false and (```1 != NULL```) is also false! (```NULL = NULL```) is also false. The presence of NULL complicates queries significantly, but it is an important tool to in data modeling.

As a consequence, when we compare NULL with any other value, the result of the comparison is always false.

### **Indexing**

Indexing in the DBMS languages means representing the records within a relation in an query efficient data structure with respect to certain attributes. B-Tree and hash-tables are examples of such data structures. While it is very query efficient to have indexes when querying the data, indexes also cost time when inserting or deleting data.

Indexing may occur implicitly or explicitly. Implicit indexing is employed by the DBMS for attributes defined as unique (keys). Explicit indexing of an attribute is possible via the DBMS API.

### **Designing a Data Model**

Among the many different considerations that should be kept in mind while designing a data model, the following two should be the most prominent ones:
A normalized data model (i.e., no data reduction).
A data model that provides efficient access to data with respect to the most typical scenarios for a given problem.

#### **Design Steps**
We will explain the design in constructive steps using our running example.

**Step A – Identify simple correlation among attributes**
A Student has a name and ID. We ignore Courses and Department for now as they are not simple since they hold correlation to other attributes as well: A department has a name, location and head. A Course has a name.

**Step B – Identify unique attributes**
The most important feature of an attribute is whether it must be unique in a table. We would like to have at least one attribute in a relation that is unique, meaning there cannot be two records with the same value for this attribute. A specific case of the unique constraint is the Primary Key - a unique attribute that must have a value (NULL is not allowed).

In the student relation the ID should be used as a primary key. In the courses relation there is no such unique ID already existing, so we should create an artificial one: a course number. The same holds for department ID.

In some problems, it is natural to define several attributes as a unique key and it is also possible. The main concern should be that any relation that should be later queried efficiently must have a primary key.

In some rare cases, there is no need to define a primary key. For example if the the logs of an application are stored in the DB, the data is not queried during the application run but insertions are performed occasionally based on a time range. There is no requirement to uniquely identify a single row of a log - so there is no need to define a key on such a table.

**Step C – Associate simple relations**
We want to identify associations between tables. The first association we shall consider is where one table refers to another: this is implemented by adding in a table an attribute that refers to rows in another relation. In our example, a student is registered to a department (assume only one). We add the primary key of the department relation as an attribute in the students relation. This is called a foreign key and it denotes a many-to-one relationship (many students to one department).

A foreign key identifies a column or a set of columns in one (referencing) table that refers to a column or set of columns in another (referenced) table. The values in one row of the referencing columns must occur in a single row in the referenced table. A row in the referencing table cannot contain values that don't exist in the referenced table (except potentially NULL - which means that we do not know what the value of the column is).

**step D – Represent Collections**
In many cases, we would like to describe an attribute as a collection. In our example, the courses to which a student is registered is a collection of values. To model this association, we need to create another table that correlates the courses and the students with foreign keys.

Lets define student2course with attributes: courseId as a foreign key to courses.ID and studentId as a foreign key to Students.ID. Such relations model many-to-many associations, and are often called "cross tables".

#### **The College model**

Tables: departments, students, courses, student2course
departments: ID integer as a PRIMARY KEY. Name short string that must be UNIQUE, Building integer

students: ID integer as a PRIMARY KEY, Name short string, DepID integer as a FOREIGN KEY to departments.ID

courses: ID integer as a PRIMARY KEY, Name short string that must not be NULL

student2course: SID integer as a FOREIGN KEY to students.ID , CID integer as a FOREIGN KEY to courses.ID and Grade integer.

We add an explicit index on Students.Name and on Courses.Name as we know many queries will be performed on those attributes.

The following SQL Code Example shows how the model is encoded in SQL. You can execute this SQL code by using the software discussed in the practical session.

```sql
CREATE TABLE departments (
    ID integer PRIMARY KEY,
    Name VARCHAR(30),
    Building INTEGER );
 
ALTER TABLE Departments ADD CONSTRAINT unique_department_name UNIQUE(Name)
 
CREATE TABLE students (
    ID INTEGER PRIMARY KEY,
    Name VARCHAR(30),
    DepID INTEGER);
 
ALTER TABLE students 
    ADD FOREIGN KEY (DepID) 
    REFERENCES departments(ID);
 
CREATE INDEX index_students_name ON Students(Name)
 
CREATE TABLE courses 
    (ID INTEGER PRIMARY KEY,
    Name VARCHAR(30) NOT NULL);
 
CREATE INDEX index_courses_name ON Courses(Name)
 
CREATE TABLE student2course 
    (SID INTEGER,  CID INTEGER);
 
ALTER TABLE student2course ADD COLUMN Grade INTEGER;
 
ALTER TABLE student2course 
    ADD CONSTRAINT fk_student2course_sid
    FOREIGN KEY (SID) 
    REFERENCES students(ID);
 
ALTER TABLE student2course 
    ADD FOREIGN KEY (CID) 
    REFERENCES courses(ID);
 
-----------------------------------------------------------------------------
 
INSERT INTO departments (ID, Name, Building) VALUES (1, 'computer science', 37);
INSERT INTO departments (ID, Name, Building) VALUES (10, 'electrical engineering', 33);
INSERT INTO departments (ID, Name, Building) VALUES (11, 'biology', 21);    

INSERT INTO students (ID, Name, DepID) VALUES (2020, 'studenta', 1);
INSERT INTO students (ID, Name, DepID) VALUES (3030, 'studentb', 1);
INSERT INTO students (ID, Name, DepID) VALUES (4040, 'studentc', 1);
INSERT INTO students (ID, Name, DepID) VALUES (5050, 'studentd', 10);
  
INSERT INTO courses (ID, Name) VALUES (20210232, 'spl');
INSERT INTO courses (ID, Name) VALUES (20223040, 'compilation');
INSERT INTO courses (ID, Name) VALUES (20113090, 'discrete math');
INSERT INTO courses (ID, Name) VALUES (30312020, 'blah');
 
INSERT INTO student2course (SID, CID, GRADE) VALUES (2020, 20210232, 100);
INSERT INTO student2course (SID, CID, GRADE) VALUES (2020, 20223040, 100);
INSERT INTO student2course (SID, CID, GRADE) VALUES (2020, 20113090, 25);
INSERT INTO student2course (SID, CID, GRADE) VALUES (3030, 20223040, 56);
INSERT INTO student2course (SID, CID, GRADE) VALUES (3030, 20113090, 78);
INSERT INTO student2course (SID, CID, GRADE) VALUES (4040, 20113090, 100);
INSERT INTO student2course (SID, CID, GRADE) VALUES (4040, 20210232, 67);
 
 
UPDATE    Students
SET    Name = 'Gabby'
WHERE    ID     = 2020
 
 
DELETE    FROM Students
WHERE    Name = 'studentd'
 
-----------------------------------------------------------------------------
 
//show all grades in spl, given spl course number:
SELECT student2course.Grade 
    FROM student2course 
    WHERE student2course.CID = 20210232;
 
//now, also show student names :
SELECT students.Name,student2course.Grade 
    FROM student2course JOIN students 
        ON students.ID = student2course.SID
    WHERE student2course.CID = 20210232;
 
// all of the students registered to cs
SELECT students.Name FROM 
    students JOIN departments 
        ON students.DepID = departments.ID
    WHERE departments.Name = 'computer science'
    ORDER BY students.Name;
        
// count how many students are registered to ee:
SELECT  COUNT(*) FROM 
    students JOIN departments 
        ON students.DepID = departments.ID
    WHERE departments.Name = 'computer science';

//calculate average grade in discrete math:
SELECT AVG(student2course.Grade) 
    FROM student2course JOIN students
        ON students.ID = student2course.SID
    WHERE student2course.CID = 20113090;
 
//show how many students are registered to each course:
//first try:
SELECT courses.Name 
    FROM student2course JOIN courses
    on student2course.CID = courses.ID;
 
SELECT courses.Name, COUNT(*) 
    FROM student2course JOIN courses 
        on student2course.CID = courses.ID
    GROUP BY courses.Name;
 
 //show all courses which have no one enrolled in:
SELECT courses.ID,courses.Name 
    FROM courses LEFT OUTER JOIN student2course 
    ON courses.ID = student2course.CID
    WHERE student2course.CID IS NULL;
 
//show grade average of students in all courses:
SELECT students.ID, AVG(student2course.Grade)
    FROM students join student2course 
        ON students.ID = student2course.SID
    GROUP BY students.ID;
 
-----------------------------------------------------------------------------
 
-- in an extension to basic SQL this is also possible 

//show all grades, using student name, course name:
SELECT students.Name, courses.Name, student2course.Grade
    FROM (students JOIN (
        courses JOIN student2course 
           ON courses.ID = student2course.CID)
       ON (students.ID = student2course.SID))
    ORDER BY courses.Name, students.Name;
 
 
SELECT courses.Name as 'Course Name', COUNT(*) as 'Number'
    FROM (student2course JOIN courses 
        on student2course.CID = courses.ID)
    GROUP BY courses.Name;
    
 // show all courses. For courses with students, the students and their grades will be shown as well:
SELECT students.Name  AS Student, courses.Name AS Course, student2course.Grade  AS Grade
    FROM (students LEFT OUTER JOIN (
       courses JOIN student2course
        ON courses.ID = student2course.CID) 
     ON (students.ID = student2course.SID));
UNION
SELECT students.Name AS Student, courses.Name AS Course, student2course.CID AS Grade 
    FROM (courses LEFT OUTER JOIN (
       students JOIN student2course
        ON students.ID = student2course.SID) 
     ON (courses.ID = student2course.CID));
    WHERE student2course.SID IS NULL 
```
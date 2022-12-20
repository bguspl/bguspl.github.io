# Java Packages

Packages are Java's mechanism for dealing with namespaces and for defining logical-code-units which are bigger than a single class. \
For more on packages see here.

## Table of Contents
- [Java Packages](#java-packages)
  - [Table of Contents](#table-of-contents)
  - [Why use packages?](#why-use-packages)
    - [code organization](#code-organization)
    - [controlling visibility](#controlling-visibility)
    - [namespace](#namespace)
  - [Using packages](#using-packages)
  - [Packages and directory structure](#packages-and-directory-structure)
  - [The CLASSPATH](#the-classpath)


## Why use packages?
### code organization
Software projects are big and hopefully modular. You want to organize your code such that related classes are kept together, and unrelated classes are kept somewhere else.

### controlling visibility
Sometimes, you want some piece of code to be visible outside of its class, but still not visible to the whole world. Packages help you do that: the protected access level is visible from within the package (and to subclasses). If you don't supply any access level, then the visibility is from within the package (but not from subclasses).

### namespace
Imagine you write a cool data-structures library. A part of it is a linked list, and you have a class named Link. Someone else is writing a web-browser, and he wants to use your cool data-structure library. He is willing to buy it from you for lots of money! But then he realizes his project also has a Link class, for representing url-links on the web. This is sad, because now he can't use your library because of name collissons. You just lost a lot of money. In order for him to use your library, there are two possibilities: (1) he will change the name of his Link class to WebLink, or (2) you will change the name of your Link class to LinkedListLink. These changes take time.. alternatively, you can use packages and avoid such problem to begin with.

## Using packages
Packages are like "family names" for classes and interfaces. \
A package is a unit of code organization. Every file you write (Class, Interface, …) belongs to a package. \ If you don't specify a package, the code belongs to the default package. This is a bad habbit.

So, assuming you want to have a Link class, representing a link in your linked-list implementation in your cool data-structure package. Instead of putting it in the default package, we'll put it in a package named: `yourname.datastructure.lists`

```java
package yourname.datastructure.lists;
 
class Link {
   // ...
}
```

Now suppose you want to use your Link in code. If it's in the same package, then there is no problem:

```java
package yourname.datastructure.lists;
 
class LinkedList {
   private Link aLink; 
   // ...
}
```

Notice how LinkedList, in the package `yourname.datastructure.lists` just use Link without any special effort.

Now let's assume you want to use the Link class from a class in a different package. Now, you have to use the FULL NAME of the Link class – the name including the package.

```java
package othername.web;
 
class WebBrowser {
   private yourname.datastructure.lists.Link aLink;
   // ...
}
```
This works fine, but it is a bit long to write. If you use yourname.datastructure.lists.Link a lot, you will have a lot to type, and your code will look ugly.

Instead, if you don't have a name collison problem (there are no other classes you use in the same file with the same "first-name" Link, you can use the import mechanism to avoid all the typing:

```java
package othername.web;
 
import yourname.datastructure.lists.Link;
 
class WebBrowser {
   private Link aLink;
   // ...
}
```
The import line tell the java compiler: whenver you see Link in this file, it actually means yourname.datastructure.lists.Link.

If you plan to use many classes from the same package, you can import all of them together:

```java
package othername.web;
 
import yourname.datastructure.lists.*;
 
class WebBrowser {
   private Link aLink;
   // ...
}
```
This star tells the compiler: when there is a name you don't know in this file, try appending yourname.datastructure.lists before it, and see if you can find it.

Note: you can use any class installed on your system without using import. You will just have to type more.

## Packages and directory structure
In Java, there is a correspondence between package names and directory structure. If you have a class named `yourname.datastructure.lists.Link` its source file should be in a file named yourname/datastructure/lists/Link.java and the compiled file should be in a file named yourname/datastructure/lists/Link.class (which means, a file named `Link.class` inside a directory named lists, inside a directory named datastructure, inside a directory called yourname). \
When you create a package in IDE, it will make this directory structure for you. \
 The package directory structure will start under the source folder the package belongs to in Eclipse.

## The CLASSPATH
When the compiler or the runtime system sees a line like:
```java
import the.pkg.name.ClassA;
```

or:

```java
the.pkg.name.ClassA something = new the.package.name.ClassA();
```
it knows it should look for a .class file named: the/pkg/name/ClassA, but where should it start looking?

The answer is, the compiler and the runtime system have a variable named "the classpath" which contains a list of directories in which to look for .class files.

Usually, Eclipse manages the classpath of the project for you. If you don't use Eclipse, you should specify the classpath manually. This is done either with the use of an environment variable or with the -classpath commandline switch to javac and java.

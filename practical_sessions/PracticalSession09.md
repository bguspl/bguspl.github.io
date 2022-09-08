Practical Session 9
In the first part of this practical session we will learn about Exceptions in Java.
In the second part, we will see an introduction to Networking.
Exceptions
Java uses exceptions to handle errors and other exceptional events. Examples:
Reading a file that does not exist or is inaccessible.
Dividing a number by zero.
Network is down, URL does not exist.
Accessing an array element at a certain index that is not within the bounds of the array.

Code Example:
downloadtoggle
16 lines ...
public class ExceptionExample {
    public static void main(String[] args) {
        int[] a = {2, 4, 6, 8};
        for(int j = 0; j <= a.length ; j++)
            System.out.println(a[j]);
    }
}
 
/*
output:
2
4
6
8
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 4
    at spl.examples.exceptions.ExceptionExample.main(ExceptionExample.java:7)
*/
What Is an Exception?
An exception is an event that occurs during the execution of a program that disrupts the normal flow of instructions.
When an error occurs within a method, the method creates an object and hands it off to the runtime system. The object, called an exception object, contains information about the error, including its type and the state of the program when the error occurred.

Creating an exception object and handing it to the runtime system is called throwing an exception.

After a method throws an exception, the runtime system attempts to find something to handle it. The set of possible "somethings" to handle the exception is the ordered list of methods that had been called to get to the method where the error occurred. The list of methods is known as the call stack (see the next figure).

http://docs.oracle.com/javase/tutorial/figures/essential/exceptions-callstack.gif

The runtime system searches the call stack for a method that contains a block of code that can handle the exception. This block of code is called an exception handler. The search begins with the method in which the error occurred and proceeds through the call stack in the reverse order in which the methods were called.

If no handler is found, the exception is thrown to the JVM, which terminates the program and prints the exception together with its call stack to the console.

Exception Types
ExceptionHierarchy.png
There are 2 different types of exceptions:

Unchecked Exceptions:
Errors: exceptional conditions that are external to the application, and that the application usually cannot anticipate or recover from. They are indicated by Error and its subclasses.
- Example: OutOfMemoryError - signals that the application ran out of memory
Runtime Exceptions: exceptional conditions that are internal to the application, and that the application usually cannot anticipate or recover from. These usually indicate programming bugs, such as logic errors or improper use of an API. Runtime exceptions are those indicated by RuntimeException and its subclasses.
- Examples:
NullPointerException - signals that the application is trying to access a null pointer (an uninitialized object).
ArrayIndexOutOfBoundsException - signals that the application is trying to access an array outside of its boundaries.
Checked Exceptions: exceptional conditions that a well-written application should anticipate and recover from. They are inherited from the core Java class Exception. Checked exceptions must be handled in your code (catch it), or passed to the calling code for handling (declare that your method might throw it using the throws keyword at the method signature). - Example: FileNotFoundException - signals that the application is trying to access a file that does not exist.
Throwing an Exception
If a method wants to signal that something went wrong during its execution, it throws an exception; exceptions may be caught and handled by another part of the program.
Throwing an exception involves:

Creating an exception object that encloses information about the problem that occurred.
Using the throw statement to notify about the exception.

Code Example:
downloadtoggle
13 lines ...
public class Clock {
    private int hours, minutes, seconds;
 
    // constructors and methods
    public void setTime(int h,int m,int s){
        if (h<1 || h>12 || m<0 || m>59 || s<0 || s>59)
           throw new IllegalArgumentException(“Invalid time”);
       
        //this code does not run if an exception occurs.
        hours = h;
        minutes = m;
        seconds = s;
    }
}
There is no need to catch IllegalArgumentException or to declare that the function setTime throws it since it's a RuntimeException (unchecked exception). However, note that even though the method setTime is not declared that it throws an IllegalArgumentException, you can still try and catch an IllegalArgumentException when you call the function setTime if you want and behave accordingly. If you do declare that setTime throws such an exception, it will be documented in the javadoc.
Creating New Exceptions
You can create a new exception by extending Exception, or one of its subclasses.
Code Example: Creating a new runtime exception:

downloadtoggle
20 lines ...
//StackUnderflowException inherits all of its parent's methods and does not add any of its own
//You can also add methods to StackUnderflowException or override existing methods to change 
//printing format, for example
public class StackUnderflowException extends RuntimeException {}
 
//a class that uses our new exception
public class Stack {
    int elements[];
    int top;
    
    //next empty location in the elements array
    // ... constructor, push(), isEmpty()
 
    //There is no need to declare StackUnderflowException  as thrown in the method signature 
    //since it's a RuntimeException (unchecked exception)
    public int pop() {
        if (isEmpty())
             throw new StackUnderflowException();
        return elements[--top];
    }
}
Catching Exceptions
A method catches an exception using a combination of the try and catch keywords. A try/catch block is placed around the code that might generate an exception. Each catch block is an exception handler and handles the type of exception indicated by its argument, which must inherits from the Throwable class.
Code Example:

downloadtoggle
19 lines ...
import java.io.*;
public class FirstLine {
    public static void main(String[] args){
        String name = args[0];
        
        try{
            BufferedReader file = new BufferedReader(new FileReader(name));
            String line = file.readLine();
            System.out.println(line);
 
        //handle FileNotFoundException 
        }catch (FileNotFoundException e) {
            System.out.println("File not found: "+ name);
 
        //handle IOException
        }catch (IOException e) {
            System.out.println("Problem: " + e);
        }
    }
}
If an exception extends another exception and you would like to handle each separately, then the catch block that handles the sub exception should be before the catch block that handles the base exception.
Exceptions don't have to be handled in the method in which they occurred. You can let a method further up the call stack handle the exception. This is done by adding a throws clause to the method declaration.

download
public BufferedReader openFileForReading(String filename) throws IOException{
     return new BufferedReader(new FileReader(filename));
}
Handling Exceptions
When handling exceptions you should restore stable state and do one of the following:
Change conditions and retry. For example, when using a fail-fast iterator.
Clean up and fail (re-throw exception). For example, when trying to open a file that does not exist.

A code example of the first case is given here:
downloadtoggle
54 lines ...
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
 
public class FailFastExample
{
    public static void main(String[] args) throws InterruptedException
    {
        Map<String,String> premiumPhone = new HashMap<String,String>();
        premiumPhone.put("Apple", "iPhone");
        premiumPhone.put("HTC", "HTC one");
        premiumPhone.put("Samsung","S5");
        boolean success = false;
        new Thread(new MapModifier(premiumPhone)).start();
                
        while (!success)
        {
            try
            {
                Iterator<String> iterator = premiumPhone.keySet().iterator();
                while (iterator.hasNext())
                {
                    System.out.println(premiumPhone.get(iterator.next()));
                    Thread.sleep(100);
                }
                success = true;
            }
            catch (java.util.ConcurrentModificationException e)
            {
                System.out.println("Caught a ConcurrentModificationException, trying again!");
            }
        }
        
    }    
}
 
class MapModifier implements Runnable
{
    private Map<String,String> m_premiumPhone;
    
    MapModifier(Map<String,String> premiumPhone)
    {
        m_premiumPhone = premiumPhone;
    }
    public void run()
    {
        try {
            Thread.sleep(50);
            m_premiumPhone.put("Sony", "Xperia Z");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
    }
}
The Finally Block
The finally block comes right after all catch blocks in a try-catch clause. The code found in the finally block is always executed before exiting the try/catch block, even if an exception occurred, or a return/continue/break statement were encountered. It is usually used for cleanup code, since it allows the programmer to avoid having cleanup code accidentally bypassed by a return, continue, or break.
downloadtoggle
28 lines ...
import java.io.*;
public class FirstLine {
    public static void main(String[] args){
        String name = args[0];
        BufferedReader file = null;
        try{
            file = new BufferedReader(new FileReader(name));
            String line = file.readLine();
            System.out.println(line);
 
        //handle FileNotFoundException 
        }catch (FileNotFoundException e) {
            System.out.println("File not found: "+ name);
 
        //handle IOException
        }catch (IOException e) {
            System.out.println("Problem: " + e);
        }finally{
            if (file != null) { 
                System.out.println("Closing the file:" + name);
                try {
                    file.close(); // May throw an exception by itself
                } catch (IOException ignore) {}
            } else { 
                System.out.println("Unable to open file:" + name);
            } 
        } 
    }
}
Try-With-Resource
While the code in the previous section seems to handle all possible exceptions - it does overlook the possibility that the close() function of the BufferedReader will produce an IOException! Should such an exception occur within the finally block - it would not be caught by the previous try/catch block, but rather thrown upwards.
This could obviously be solved by a second set of try/catch statements, but fortunately, Java SE 7 provides a more elegant solution in the form of try-with-resource statements. The code in the previous section can be replaced with the following equivalent:

downloadtoggle
9 lines ...
try (BufferedReader file = new BufferedReader(new FileReader(name));) {
     String line = file.readLine();
     System.out.println(line);
//handle FileNotFoundException 
}catch (FileNotFoundException e) {
    System.out.println("File not found: "+ name);
//handle IOException
}catch (IOException e) {
    System.out.println("Problem: " + e);
}
In the above code, a BufferedReader is declared as part of the "try" statement. If an exception occurs, either during the creation of the BufferedReader, or its use within the "try" statement - the corresponding .close() function will be called automatically and no further exceptions will be thrown.
Suppressed Exceptions
In Java 7, the concept of suppressed exceptions was introduced. If in the previous code everything worked but the automatically called-upon close() method causes an exception, it will be caught by the closest exception to it (IOException in this case). However, if something did not work (opening the file or reading a line) and also the automatically called-upon close() did not work then only the file-open or read-line exception will be caught, and the rest of the exceptions will be suppressed, ie will not be caught but will rather be "hidden". You can retrieve suppressed exceptions by calling the getSuppressed() method from the exception thrown by the try block.
Java Sockets
Introduction
Internet Protocol: IP Addresses
The Internet is a global system of interconnected computer networks, it is a network of networks that consists of millions of private and public, academic, business, and government networks of local to global scope that are linked by a broad array of electronic and optical networking technologies.
Every machine on the Internet has a unique identifying number, called an IP Address. The IP stands for Internet Protocol, which is the language that computers use to communicate over the Internet. The original designers of TCP/IP defined an IP address as a 32-bit number and this system, known as Internet Protocol Version 4 or IPv4, is still in use today. However, due to the enormous growth of the Internet and the resulting depletion of available addresses, a new addressing system (IPv6), using 128 bits for the address, was developed in 1995.

IPv4 addresses are usually represented in dot-decimal notation (four numbers, each ranging from 0 to 255). Each part represents 8 bits of the address, and is therefore called an octet.

Protocol – rules of conversation.
A typical IP address (decimal format) - 216.27.61.137
A typical IP address (binary format) - 11011000.00011011.00111101.10001001
TCP/IP model
The TCP/IP model is a description framework for computer network protocols. TCP/IP provides end-to-end connectivity specifying how data should be formatted, addressed, transmitted, routed and received at the destination. TCP/IP is generally described as having four abstraction layers:
Link Layer
Internet Layer
Transport Layer
Application Layer

IP_stack_connections.png
Restricted IP addresses
0.0.0.0 - is reserved for the default network
255.255.255.255 - is used for broadcasts (Messages that are intended for all computers on a network).
127.0.0.1 (Loopback) - is used as the loopback address. This means that it is used by the host computer to send a message back to itself. Also aliased as localhost.
Domain Name System (DNS)
Remembering IP addresses is hard for humans.
The DNS service maps text names to IP addresses automatically. For example: www.cs.bgu.ac.il —> xxx.xxx.xxx.xxx

An often used analogy to explain the Domain Name System is that it serves as the "phone book" for the Internet by translating human-friendly computer hostnames into IP addresses.

Internet Servers and Clients
All of the machines on the Internet are either servers or clients (or both). The machines that provide services to other machines are servers. And the machines that are used to connect to those services are clients.
A server has a static IP address that does not change very often.

A home machine that is dialing up through a modem, typically has a dynamic IP address - A session unique IP address assigned by the ISP (usually assigned by a server using DHCP) every time you dial in (could be different the next time you dial in).

Ports
Any server machine makes its services available using numbered ports - one for each service that is available on the server. Think of the IP:Port combo as this: The IP is the building address, where the Port denotes which apartment in that building. In our world, IP is the computer machine, where the Port is the application that listens to it and can send and receive messages through it.
For example, if a server machine is running a Web server and a file transfer protocol (FTP) server, the Web server would typically be available on port 80, and the FTP server would be available on port 21. Clients connect to a service at a specific IP address and on a specific port number. List of TCP and UDP port numbers

Once a client has connected to a service on a particular port, it accesses the service using a specific protocol. Protocols are often text and simply describe how the client and server will have their conversation.

Network console tools
ipconfig (Microsoft Windows console application) – displays all current TCP/IP network configuration values.
ifconfig (Unix-like console application) - configure, control, and query TCP/IP network interface parameters.
ping (Unix-like & Windows) - utility used to test whether a particular host is reachable across an Internet Protocol (IP) network and to measure the round-trip time for packets sent from the local host to a destination computer.
Telnet client (Unix-like & Windows) - Telnet is a network protocol used to provide a bidirectional interactive communications facility. Typically, telnet provides access to a command-line interface on a remote host via a virtual terminal connection. Putty can be used on recent Windows-es which don't have telnet.
cURL is a computer software project providing a library and command-line tool for transferring data using various protocols.
Sockets
Sometimes your applications require low-level communication, for example connecting to a database or implementing instant messaging. Networking allows processes running on different hosts to exchange messages. Message destinations are specified as socket addresses; each socket address is a communication identifier that consists of a port number and an Internet address.
When messages are sent, they are queued at the sending socket until the underlying network protocol has transmitted them. When they arrive, the messages are queued at the receiving socket until the receiving process makes the necessary calls to receive them.

Datagram vs. Stream
Transport Layer is a group of methods and protocols within a layered architecture of network components. There are two communication protocols that one can use for socket programming: datagram communication (UDP) and stream communication (TCP). In this practical session we focus on session-oriented, reliable connections implemented by the TCP protocol and used through the socket interface.
Datagram Communication
The datagram communication protocol, known as UDP (user datagram protocol), is a connectionless protocol. It means that a datagram can be sent at any moment without prior connection preparation as in TCP. You just send the datagram and hope the receiver is able to handle it.
There is absolutely no guarantee that the datagram will be delivered to the destination host. In reality, the failure rate is very low on the Internet and nearly null on a LAN unless the bandwidth is full. Not only the datagram can be undelivered, but it can be delivered in an incorrect order. It means you can receive a packet before another one, even if the second has been sent before the first you just received. You can also receive the same packet twice.

The main advantages for UDP are that you can broadcast, and it is fast. The main disadvantage is unreliability and therefore complicated to program at the application level.

Stream Communication
The stream communication protocol is known as TCP (transfer control protocol). Unlike UDP, TCP is a connection-oriented protocol. In order to do communication over the TCP protocol, a connection must first be established between the pair of sockets. While one of the sockets listens for a connection request (server), the other asks for a connection (client). Once two sockets have been connected, they can be used to transmit data in both (or either one of the) directions.

Being stream oriented means that the data is a plain bytes sequence. The receiver has no means of knowing how the data was actually transmitted. The sender can send many small data chunks and the receiver receive only one big chunk, or the sender can send a big chunk, the receiver receiving it in a number of smaller chunks. The only thing that is guaranteed is that all data sent will be received without any error and in the correct order. Should any error occur, it will automatically be corrected (retransmitted as needed) or the error will be notified if it can’t be corrected.

The following diagram presents the way we will handle text streams in java:
JavaStreams.JPG

This illustrates the duplex communication model:
tcp_sockets.JPG

The main advantages for TCP are that it guarantees three things: correctness, order and no duplication. The main disadvantage is that it cannot be used for broadcast or multicast transmission. In addition it might have a bad throughput on high latency link.

TCP/UDP Overhead
The following image illustrates the header size for TCP and UDP packets:
 (source: microchip.wikidot)

It is quite evident that the TCP header (sent along with each data packet!) is significantly larger than the UDP header. This creates additional overhead, reducing throughput of TCP communication.

The Client-Server Model
The client-server is a very common model in many networking applications. The server provides some service, such as processing database queries or sending out current stock prices. The client uses the service provided by the server, either displaying database query results to the user or making stock purchase recommendations to an investor.
A socket connection based on top of a TCP-connection is symmetric between the two ends - except for the connection establishment stage. To establish a connection, the model determines that:

The server waits for connection requests from clients.
The server only reacts to the initiative of the client.
To remain available for further connections, the server does not open the connection directly with the client on the incoming socket. Instead, it creates a private socket which the client can keep connected for the whole period of the session.

Servers must be built to maximize availability – that is, they must handle requests for connections as fast as possible and then return to the mode where they can receive and handle more requests from other clients.
Examples
There are four examples. In each example you will find code for client and for server. Compile both files and open two console windows. Place them so you can see them both. First run the server and supply the server with a port to listen at, for example
> java LPServer 4444

Then run the client, provide it with the host name and the port of the server. For example, if you use the same machine for both, then

> java LPClient localhost 4444

If the server runs on different machine replace "localhost" (or 127.0.0.1) with the server computer name or IP.

1. Simple Line Printer
A very simple client-server application. The server begins listening for a single client. Once the client connects it establishes a UTF-8 text based encoder decoder that returns a message upon reception of a newline symbol ('n').
The LP protocol is that any message from the client is printed on the screen by the server. If the message 'bye' is received, the server closes the connection.
Server code
downloadtoggle
55 lines ...
import java.io.*;
import java.net.*;
 
class LPServer {
    
    public static void main(String[] args) throws IOException
    {
        ServerSocket lpServerSocket = null;
 
        // Get port
        int port = Integer.decode(args[0]).intValue();
        
        // Listen on port
        try {
            lpServerSocket = new ServerSocket(port);
        } catch (IOException e) {
            System.out.println("Couldn't listen on port " + port);
            System.exit(1);
        }
        
        System.out.println("Listening...");
        
        // Waiting for a client connection
        Socket lpClientSocket = null;
        try {
            lpClientSocket = lpServerSocket.accept();
        } catch (IOException e) {
            System.out.println("Failed to accept...");
            System.exit(1);
        }
        
        System.out.println("Accepted connection from client!");
        System.out.println("The client is from: " + lpClientSocket.getInetAddress() + ":" + lpClientSocket.getPort());
        
        // Read messages from client
        BufferedReader in = new BufferedReader(new InputStreamReader(lpClientSocket.getInputStream()));
        String msg;
 
        while ((msg = in.readLine()) != null)
        {
            System.out.println("Received from client: " + msg);
            if (msg.equals("bye"))
            {
                System.out.println("Client sent a terminating message");
                break;
            }
        }
        
        System.out.println("Client disconnected - bye bye...");
        
        lpServerSocket.close();
        lpClientSocket.close();
        in.close();
        
    }
}
Client code
downloadtoggle
48 lines ...
import java.io.*;
import java.net.*;
 
public class LPClient {
    
    public static void main(String[] args) throws IOException
    {
        Socket lpSocket = null; // the connection socket
        PrintWriter out = null;
 
        // Get host and port
        String host = args[0];
        int port = Integer.decode(args[1]).intValue();
        
        System.out.println("Connecting to " + host + ":" + port);
        
        // Trying to connect to a socket and initialize an output stream
        try {
            lpSocket = new Socket(host, port); // host and port
              out = new PrintWriter(lpSocket.getOutputStream(), true);
        } catch (UnknownHostException e) {
              System.out.println("Unknown host: " + host);
              System.exit(1);
        } catch (IOException e) {
            System.out.println("Couldn't get I/O to " + host + " connection");
            System.exit(1);
        }
        
        System.out.println("Connected to server!");
 
        String msg;
        BufferedReader userIn = new BufferedReader(new InputStreamReader(System.in));
        
        while ((msg = userIn.readLine())!= null)
        {
            out.println(msg);
            if(msg.indexOf("bye") >= 0){
                  break;
            }
        }
        
        System.out.println("Exiting...");
        
        // Close all I/O
        out.close();
        userIn.close();
        lpSocket.close();
    }
}
2. Encoding and Decoding
Different computers, run time environment languages and compilers can use different symbols representation. If we want to communicate we need uniform coding for the symbols. Since we are transferring objects larger than a single byte (integers, chars, strings) over streams of bytes, we need to encode them.
Even for Strings: Should we run on one side

java -Dfile.encoding=US-ASCII LPServer 4500
and on another

java LPClient localhost 4500 (assuming default UTF-8)

any text which has characters not included in the similar 127 first characters in the encodings is not transferred correct (e.g. Hebrew). We can notice other problems using UTF-16 and UTF-32 (for example line endings no longer recognized as message ends since they have different numerical values).

We can also see that unicode strings have different lengths when represented in different encodings:

downloadtoggle
12 lines ...
import org.junit.*;
import static org.junit.Assert.*;
 
public class TestEncodings {
 
    @Test public void testAll() throws Exception {
         String  s = "dד𝚫"; //MATHEMATICAL BOLD CAPITAL DELTA 1D6AB
         assertEquals(4, s.length()); //length of a UTF-32 string in UTF-16
         assertEquals(3, s.codePointCount(0, s.length())); //length by codepoints (actual letters)
         assertEquals(7, s.getBytes("UTF-8").length); //length in bytes (based on given encoding UTF-8)
    }
 
}
In the next example the client and the server are communicate using the "UTF-8" format. You will learn more about Encoding in the lectures. This time, the server sends the client back the message it received using UTF-8. We create the readers and writers with the encoding specified as follows:

download
...
in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream(),"UTF-8"));
...
out = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream(), "UTF-8"), true);
...
Note that "UTF-8" is the default encoding of InputStreamReader and OutputStreamWriter so we did not actually have to indicate that in the previous code.

Server Code: (EchoServer)
Same Encoding and Decoding as in LPServer. The protocol is slightly different, this time if the message isn't 'bye' the server will return the message sent by the client, back to him.
downloadtoggle
99 lines ...
import java.io.*;
import java.net.*;
 
class EchoServer {
    
    private BufferedReader in;
    private PrintWriter out;
    ServerSocket echoServerSocket;
    Socket clientSocket;
    int listenPort;
    
    public EchoServer(int port)
    {
        in = null;
        out = null;
        echoServerSocket = null;
        clientSocket = null;
        listenPort = port;
    }
    
    // Starts listening
    public void initialize() throws IOException
    {
        // Listen
        echoServerSocket = new ServerSocket(listenPort);
        
        System.out.println("Listening...");
        
        // Accept connection
        clientSocket = echoServerSocket.accept();
        
        System.out.println("Accepted connection from client!");
        System.out.println("The client is from: " + clientSocket.getInetAddress() + ":" + clientSocket.getPort());
        
        // Initialize I/O
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream(),"UTF-8"));
        out = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream(), "UTF-8"), true);
        
        System.out.println("I/O initialized");
    }
    
    public void process() throws IOException
    {
        String msg;
        
        while ((msg = in.readLine()) != null)
        {
            System.out.println("Received \"" + msg + "\" from client");
            
            if (msg.equals("bye"))
            {
                out.println("Ok, bye bye...");
                break;
            }
            else
            {
                out.print(msg + "\n");
                out.flush();
            }
        }
    }
    
    // Closes the connection
    public void close() throws IOException
    {
        in.close();
        out.close();
        clientSocket.close();
        echoServerSocket.close();
    }
    
    public static void main(String[] args) throws IOException
    {
        // Get port
        int port = Integer.decode(args[0]).intValue();
        
        EchoServer echoServer = new EchoServer(port);
        
        // Listen on port
        try {
            echoServer.initialize();
        } catch (IOException e) {
            System.out.println("Failed to initialize on port " + port);
            System.exit(1);
        }
        
        // Process messages from client
        try {
            echoServer.process();
        } catch (IOException e) {
            System.out.println("Exception in processing");
            echoServer.close();
            System.exit(1);
        }
        
        System.out.println("Client disconnected - bye bye...");
        
        echoServer.close();
    }
}
Client Code (EchoClient)
downloadtoggle
56 lines ...
import java.io.*;
import java.net.*;
 
public class EchoClient {
    
    public static void main(String[] args) throws IOException
    {
        Socket clientSocket = null; // the connection socket
        PrintWriter out = null;
        BufferedReader in = null;
 
        // Get host and port
        String host = args[0];
        int port = Integer.decode(args[1]).intValue();
        
        System.out.println("Connecting to " + host + ":" + port);
        
        // Trying to connect to a socket and initialize an output stream
        try {
            clientSocket = new Socket(host, port); // host and port
              out = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream(), "UTF-8"), true);
        } catch (UnknownHostException e) {
              System.out.println("Unknown host: " + host);
              System.exit(1);
        } catch (IOException e) {
            System.out.println("Couldn't get output to " + host + " connection");
            System.exit(1);
        }
        
        // Initialize an input stream
        try {
            in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream(),"UTF-8"));
        } catch (IOException e) {
            System.out.println("Couldn't get input to " + host + " connection");
            System.exit(1);
        }
        
        System.out.println("Connected to server!");
 
        String msg;
        BufferedReader userIn = new BufferedReader(new InputStreamReader(System.in));
        
        while ((msg = userIn.readLine())!= null)
        {
            out.println(msg);
            System.out.println(in.readLine());
        }
        
        System.out.println("Exiting...");
        
        // Close all I/O
        out.close();
        in.close();
        userIn.close();
        clientSocket.close();
    }
}
3. Protocol Interface
We don’t want to hold the implementation of a protocol inside the server’s code. It is complicated enough that it has to handle the communication, and not testable while a protocol usually follows a specification. We would like to have the protocol implemented somewhere else. For that purpose we define an interface MessagingProtocol which has a single methods for protocol processing: process - for processing the received message and construct a response message
Note that since we have an independent protocol processor we can hold the protocol’s state. Once again we have an Echo Server, but this time, in addition to sending back the message it is also numbered.

downloadtoggle
140 lines ...
import java.io.*;
import java.net.*;
 
interface ServerProtocol {
 
    String processMessage(String msg);
 
    boolean isEnd(String msg);
 
}
 
class EchoProtocol implements ServerProtocol {
 
    private int counter;
 
    public EchoProtocol()
    {
        counter = 0;
    }
 
    public String processMessage(String msg)
    {
        counter++;
 
        if (isEnd(msg))
        {
            return new String("Ok, bye bye...");
        }
        else
        {
            return new String(counter + ". Received \"" + msg + "\" from client");
        }
 
    }
 
    public boolean isEnd(String msg)
    {
        return msg.equals("bye");
    }
}
 
class ProtocolServer {
 
    private BufferedReader in;
    private PrintWriter out;
    ServerSocket echoServerSocket;
    Socket clientSocket;
    int listenPort;
    ServerProtocol protocol;
 
    public ProtocolServer(int port, ServerProtocol p)
    {
        in = null;
        out = null;
        echoServerSocket = null;
        clientSocket = null;
        listenPort = port;
        protocol = p;
    }
 
    // Starts listening
    public void initialize() throws IOException
    {
        // Listen
        echoServerSocket = new ServerSocket(listenPort);
 
        System.out.println("Listening...");
 
        // Accept connection
        clientSocket = echoServerSocket.accept();
 
        System.out.println("Accepted connection from client!");
        System.out.println("The client is from: " + clientSocket.getInetAddress() + ":" + clientSocket.getPort());
 
        // Initialize I/O
        in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream(), "UTF-8"));
        out = new PrintWriter(new OutputStreamWriter(clientSocket.getOutputStream(), "UTF-8"), true);
 
        System.out.println("I/O initialized");
    }
 
    public void process() throws IOException
    {
        String msg;
 
        while ((msg = in.readLine()) != null)
        {
            System.out.println("Received \"" + msg + "\" from client");
 
            String response = protocol.processMessage(msg);
            if (response != null)
            {
                out.println(response);
            }
 
            if (protocol.isEnd(msg))
            {
                break;
            }
 
        }
    }
 
    // Closes the connection
    public void close() throws IOException
    {
        in.close();
        out.close();
        clientSocket.close();
        echoServerSocket.close();
    }
 
    public static void main(String[] args) throws IOException
    {
        // Get port
        int port = Integer.decode(args[0]).intValue();
 
        ProtocolServer server = new ProtocolServer(port, new EchoProtocol());
 
        // Listen on port
        try {
            server.initialize();
        } catch (IOException e) {
            System.out.println("Failed to initialize on port " + port);
            System.exit(1);
        }
 
        // Process messages from client
        try {
            server.process();
        } catch (IOException e) {
            System.out.println("Exception in processing");
            server.close();
            System.exit(1);
        }
 
        System.out.println("Client disconnected - bye bye...");
 
        server.close();
    }
}
4. Http Client
Hypertext Transfer Protocol is an application protocol for distributed, collaborative, hypermedia information systems.
We can see a few attributes to many communication protocol designs. An HTTP request is defined as a header, followed by an empty line and an optional body. The request includes the protocol version. An HTTP response is defined as a header, followed by an empty line and an optional body. The response may end after the body, or in HTTP/1.1 each response may contain either Content-Length header field indicating the number of bytes for the response, or a response with Transfer-Encoding: chunked with each chunk containing the chunk size.

downloadtoggle
21 lines ...
import java.io.*; 
import java.net.*;
 
public class Http {
 
        public static void main(String[] args) throws Exception {
                String host = args[0];
                try (Socket lp = new Socket(host, 80);
                     PrintWriter out = new PrintWriter(new OutputStreamWriter(lp.getOutputStream(), "UTF-8"), true);
                     BufferedReader in = new BufferedReader(new InputStreamReader(lp.getInputStream(),"UTF-8"))) {
                    out.print ("GET / HTTP/1.0\r\n" +
                               "Host: " + host+ "\r\n" +
                               "\r\n");
                    out.flush();
                    String msg = in.readLine();
                    while (msg != null) {
                            System.out.println(msg);
                            msg = in.readLine();
                } catch (Exception e) { System.out.println("Error: " + e); }
        }
 
}
If we try to run java Http www.cs.bgu.ac.il we should receive a proper HTTP response.

Useful Links
sun tutorial on sockets
java work on sockets
Comparative analysis - TCP - UDP
TCP/IP Protocol Architecture
Firewall (Explanation for what to look out if you test with multiple computers and get no connection)
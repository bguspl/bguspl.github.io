Application Level Protocol Design
Objectives
In this lecture we will discuss issues related to application level protocol design. We start by discussing the atomic units used by any protocol (which we term "messages"), revisit our discussion of encoding and finish by presenting a reusable, protocol independent, TCP server, accompanied by a LinePrinting protocol implementation.
Protocol Definition
A protocol is a set of rules, governing the communication details between two parties (in our case, processes). Protocols exist in a myriad of different forms and levels; from protocols governing how to exchange bits across a wire to protocols governing administration of super computers. In this lecture we focus on application level protocols, which define the interaction between two computer applications.
Each protocol defines the following sets of rules:

The syntax of the communication: how do we phrase the information we exchange.
The semantics of the communication: what is the proper response for each information datum received.
The synchronization of the communication: whose turn it is to speak (given the above defined semantics).

Almost all protocols follow a very simple skeleton. The parties involve exchange information by using messages, which define the syntax. The communication usually begins when one party sends an initiation message (a hand-shake) to the other party. The synchronization used is usually very simple, where each party sends one message in a round robin fashion. The difference between most protocols is the syntax used for messages, and the semantics of the protocol.
An example for a protocol is HTTP, which stands for Hyper Text Transfer Protocol, and govern the details of exchanging special text files over the network. We give a brief and simple (but not complete) description of the protocol:

synchronization: the client initiates the connection, sends a single request, and receive the reply from the server.
syntax: text based, see rfc2616.
semantics: the server either sends to the client the page asked for, or returns an error.

In the rest of this lecture we will discuss the syntax and semantics aspects of protocols. We will assume that synchronization works in a round robin fashion, e.g., each party sends one message at a time.
Message Format
To define the syntax of a protocol, we employ the concept of messages. A message is the atomic unit of data exchanged throughout the protocol. We can think of a message as a letter, which contain all the information one party needs to send to the other party. We leave the discussion of what exactly is written inside the letter to later, and first concentrate on the delivery mechanism.
Framing
When using a streaming protocol such as TCP, the need to separate between different messages arises; as all messages are sent on the same stream, one after the other, the receiver should be able to distinguish between different messages. The usual solution is to use message framing. Framing a message is taking the content of the message, and encapsulating it in a frame (continuing our metaphor of a letter, think of an envelop). The sender and receiver agree on the framing method beforehand (the framing is part of the message format/protocol). The framing used should enable the receiver to easily discover, given a stream of bytes, where a message starts and where it ends.
The simplest example of a framing protocol, when sending strings, is using a special character, the FRAMING character (e.g., a line break). In other words, each message is framed by two FRAMING characters, one at the beginning and one at the end. Care should be taken that each message will not contain a FRAMING character in its text!

A different framing protocol may be achieved by adding a special tag at the start of the message, and a special tag at the end. When sending strings of text, a message can be framed using <begin> and <end> strings. Again, care must be taken to avoid having <begin> and <end> as part of the message body.

Yet another framing protocol can be achieved by employing a variable length message format, namely a special tag or character are used to mark the start of a frame while the message itself contains the information on the message's length.

Binary Data
Many standard protocols exchange data in textual form – that is, they send and receive strings of characters, in an agreed upon character encoding, often UTF-8. The advantage of using text in a protocol is that it is very easy to document and to debug - just print the messages exchanged by the client and server and you understand what is happening. The limitation of text-based protocols is that it becomes difficult to send non-textual data. For example: how do we send a picture? a video? an audio file? In this context, non-textual data is called binary data. (Of course, all data is eventually encoded in "binary" format, as a sequence of bits - so this usage of "binary data" is special – it means data that cannot be encoded as a readable string of characters).
Sending binary data in raw binary format in a stream protocol such as TCP is dangerous. As the binary data may contain any byte sequence, the binary data may corrupt our framing protocol. There are at least two solutions you may employ:

Encoding: you can encode the binary data by using a well established encoding algorithm. For example, Base64 encoding is one of the better alternatives. Base64 encoding encodes binary data into a string, converting every 2 bytes sequence from the binary data into 3 ASCII characters. Base64 is used by many "standard" protocols (such as email protocols to encode file attachments which may contain any type of data).
In C++, the Boost library includes a module to perform encoding/decoding of arbitrary byte arrays into/from Base64 encoded ASCII data. This functionality is modeled as a sort of stream "filter" that performs encode/decode on all data flowing through the stream in the serialization package.

In Java, the standard java library includes a Base64 encoder/decoder you can use.

The advantage of this encoding is that any stream of bytes can now be "framed" as ASCII data - regardless of the character encoding used by the protocol. The disadvantage is that there is a cost in the size of the message, which is increased by 50%.

Devising a variable length message format.
Encoding
An integral part of the message format is the encoding used to send strings. There are many standard ways to encode a string into a byte sequence. However, in this course we will use UTF-8 as our encoding scheme.
Protocol and Server Separation
We have already established that when designing complex systems, code reuse is one of our design goals. In the context of networking, it would be especially useful to have a generic implementation of a server, which handles all the communication details, and a generic protocol interface, which handles incoming messages, implements the protocol's semantics and generates the reply messages. It follows that the protocol object is the object in charge of implementing the expected behavior of our server, namely what actions should be performed upon the arrival of a request. Note that requests may be correlated one to another, meaning the protocol should save an appropriate state per client.
For example, protocols often require user authentication (login), so that only authorized users can perform certain actions. In this case, the protocol is stateful - this means that the protocol serving the requests of a client can be in at least 2 distinct states: authenticated (user has already logged in) or non-authenticated (user has not yet provided his login and password). Depending on the state of the protocol object, the behavior of the protocol object will be different (if an authenticated user asks the protocol object to perform an action, it will be done, while a non-authenticated user asking the same action will receive an error code).

The key to producing such a generic server implementation is to carefully separate the different tasks the server must perform. The following actions can be distinguished:

Accept new connections from new clients.
Receive new bytes from connected clients.
Parse incoming bytes from connected clients into separate messages (an operation known as "de-serialization" or "unframing" or "decoding").
Dispatch a message to the right method on the server side to execute the requested operation.
Send back an answer to a connected client after an action has been executed.

We now describe a software architecture that separates these various tasks into separate interfaces:
protocol.gif

The key participants in this architecture are:

the MessageEncoderDecoder, which implements the protocol's syntax, encoding and decoding messages from and to bytes.
the MessagingProtocol, which implements the protocol's semantics; e.g., handling the received messages and generating the appropriate responses.

Using these interfaces, we can create a generic server - i.e, a protocol agnostic server. Note though, that for simplicity the general protocol that we defined above assumes that clients will send messages to the server and the server can respond. We can think about protocols which has the server sending messages to the clients and the clients respond or even mixed protocols. Extending the interfaces to support these types of protocols is fairly simple once you understand their implementation and usage.
Interfaces
We implement the separation between the protocol and the server by employing the following design. First, we define a message. A message can be encoded in various ways: Base64 message, XML message or text message. For simplicity, we will describe messages encoded as plain UTF-8 text.
Next, we define the framing of messages, that is, the delimiters between messages when messages are sent through a common stream.

Finally, we define the protocol interface which handles each individual message.

MessageEncoderDecoder
The MessageEncoderDecoder interface is in charge of parsing a stream of bytes into a stream of messages and backwords.
The context of this interface is the following: the server accepted a new connection from a client. The server creates an instance of a BlockingConnectionHandler object that will handle all incoming messages from this client. The ConnectionHandler object maintains the state of the connection for the specific client which it serves (for example, if the user performed "login", the ConnectionHandler object will remember this in its state). The ConnectionHandler also has access to the Socket connecting the server to the client process.

Since we are describing a TCP server, the Socket connection is viewed as a pair of InputStream and OutputStream. These streams are streams of bytes – that is, as far as TCP is concerned, the client and the server exchange a bunch of bytes.

The MessageEncoderDecoder interface is a filter that we put between the Socket input stream and the protocol. The protocol does not access the input/output stream directly - it only handle application level messages while the MessageEncoderDecoder responsible to translate them to and from bytes. This way, one can use the same protocol under different message formats and reuse message formats for different protocols.

The decoding process of the MessageEncoderDecoder works in a byte-by-byte fashion. Every time we receive a new byte from the server we will give it to the MessageEncoderDecoder if this byte, together with the previous bytes which were passed to the MessageEncoderDecoder represents a full message, the MessageEncoderDecoder will return it to us and the decoding process will restart.

downloadtoggle
17 lines ...
public interface MessageEncoderDecoder<T> {
 
    /**
     * add the next byte to the decoding process
     *
     * @param nextByte the next byte to consider for the currently decoded message
     * @return a message if this byte completes one or null if it doesnt.
     */
    T decodeNextByte(byte nextByte);
 
    /**
     * encodes the given message to bytes array
     * @param message the message to encode
     * @return the encoded bytes
     */
    byte[] encode(T message);
 
}
Messaging Protocol
We define next the protocol interface. The MessagingProtocol interface operates in the following context:
A ConnectionHandler instance wraps together: the socket connected to the client; the MessageEncoderDecoder which splits incoming bytes from the socket into messages. The next step is to pass the incoming messages from the client to the MessagingProtocol which will now execute the action requested by the client. The task of the MessagingProtocol is to look at the message and decide what should be done. This decision may depend on the state of the connection (remember the example of the "authenticated" protocol). Once the action is performed, we will need to send an answer to the client. So we expect to get an answer back from the MessagingProtocol.

We model this behavior in the following interface:

downloadtoggle
14 lines ...
public interface MessagingProtocol<T> {
 
    /**
     * process the given message 
     * @param msg the received message
     * @return the response to send or null if no response is expected by the client
     */
    T process(T msg);
 
    /**
     * @return true if the connection should be terminated
     */
    boolean shouldTerminate();
 
}
Note that we allow the protocol to use message any type of message (the type argument T). This means that the operation of Serialization and Deserialization (encode/decode complex parameters to/from Strings) will be performed by the MessageEncoderDecoder - which yield a good separation of concerns.

Implementations
The Connection Handler
We now put things together into the ConnectionHandler. ConnectionHandler is designed to run by its own thread. It handles one connection to one client for the whole period during which the client is connected (from the moment the connection is accepted, until one of the sides decides to close the connection). It therefore is modeled as a Runnable class.
The ConnectionHandler holds references to the TCP socket connected to the client, a MessageEncoderDecoder and an instance of the MessagingProtocol.

The following runnable connection handler is generic, and works flawlessly with any implementation of a messaging protocol.

downloadtoggle
35 lines ...
public class ConnectionHandler<T> implements Runnable {
 
    private final MessagingProtocol<T> protocol;
    private final MessageEncoderDecoder<T> encdec;
    private final Socket sock;
 
    public ConnectionHandler(Socket sock, MessageEncoderDecoder<T> reader, MessagingProtocol<T> protocol) {
        this.sock = sock;
        this.encdec = reader;
        this.protocol = protocol;
    }
 
    @Override
    public void run() {
        
        try (   Socket sock = this.sock; //just for automatic closing
                BufferedInputStream in = new BufferedInputStream(sock.getInputStream());
                BufferedOutputStream out = new BufferedOutputStream(sock.getOutputStream())) {
 
            int read;
            while (!protocol.shouldTerminate() && (read = in.read()) >= 0) {
                T nextMessage = encdec.decodeNextByte((byte) read);
                if (nextMessage != null) {
                    T response = protocol.process(nextMessage);
                    if (response != null) {
                        out.write(encdec.encode(response));
                        out.flush();
                    }
                }
            }
 
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
}
To implement a TCP server on the basis of this design, we now only need to implement our specific framing handler (the message encoder/decoder) and the specific protocol we wish to use. To continue our example of echo server, we will now illustrate how to implement the echo printing protocol in this architecture.

MessageEncoderDecoder
For our protocol, we use a framing method based on a single character delimiter. We assume that we have a stream of messages, delimited by FRAMING, specifically, we will use the character '\n' (newline).
downloadtoggle
37 lines ...
public class LineMessageEncoderDecoder implements MessageEncoderDecoder<String> {
 
    private byte[] bytes = new byte[1 << 10]; //start with 1k
    private int len = 0;
 
    @Override
    public String decodeNextByte(byte nextByte) {
        //notice that the top 128 ascii characters have the same representation as their utf-8 counterparts
        //this allow us to do the following comparison
        if (nextByte == '\n') { 
            return popString();
        }
 
        pushByte(nextByte);
        return null; //not a line yet
    }
 
    @Override
    public byte[] encode(String message) {
        return (message + "\n").getBytes(); //uses utf8 by default
    }
 
    private void pushByte(byte nextByte) {
        if (len >= bytes.length) {
            bytes = Arrays.copyOf(bytes, len * 2);
        }
 
        bytes[len++] = nextByte;
    }
 
    private String popString() {
        //notice that we explicitly requesting that the string will be decoded from UTF-8
        //this is not actually required as it is the default encoding in java.
        String result = new String(bytes, 0, len, StandardCharsets.UTF_8);
        len = 0;
        return result;
    }
}

EchoProtocol
We now implement a specific protocol on the server side. This server, when it receives a message, prints it on the screen (on the server side) together with the time it received and then return it back to the sender while repeating the last two chars a couple of times. That is, if a client send to the server the line "hello" it will be responded with the line "hello .. lo .. lo .."
The protocol also support the "bye" message which causes the server to close its connection to the client.

downloadtoggle
20 lines ...
public class EchoProtocol implements MessagingProtocol<String> {
 
    private boolean shouldTerminate = false;
 
    @Override
    public String process(String msg) {
        shouldTerminate = "bye".equals(msg);
        System.out.println("[" + LocalDateTime.now() + "]: " + msg);
        return createEcho(msg);
    }
 
    private String createEcho(String message) {
        String echoPart = message.substring(Math.max(message.length() - 2, 0), message.length());
        return message + " .. " + echoPart + " .. " + echoPart + " ..";
    }
 
    @Override
    public boolean shouldTerminate() {
        return shouldTerminate;
    }
}

A Client
Before we see how to put together the ConnectionHandler into a running server process, let us review the code of a very basic compatible TCP client for the protocol we have just described.
downloadtoggle
28 lines ...
public class EchoClient {
 
    public static void main(String[] args) throws IOException {
 
        if (args.length == 0) { //set default values
            args = new String[]{"localhost", "hello"};
        }
 
        if (args.length < 2) {
            System.out.println("you must supply two arguments: host, message");
            System.exit(1);
        }
 
        //BufferedReader and BufferedWriter automatically using UTF-8 encoding
        try (   Socket sock = new Socket(args[0], 7777);
                BufferedReader in = new BufferedReader(new InputStreamReader(sock.getInputStream())); 
                BufferedWriter out = new BufferedWriter(new OutputStreamWriter(sock.getOutputStream()))) {
 
            System.out.println("sending message to server");
            out.write(args[1]);
            out.newLine();
            out.flush();
 
            System.out.println("awaiting response");
            String line = in.readLine();
            System.out.println("message from server: " + line);
        }
    }
}
By now we created the protocol, message encoder/decoder, and client for our echo protocol. We know that our client will get treated in the server using its own connection handler, all that is left is to write the actual server - which is just an object that listen to new connection and assigned them to connection handlers.

Concurrency Models of TCP Servers
We now address the question of the concurrency model the TCP server should implement. A TCP server should strive to optimize the following quality criteria:
Scalability: the capability to server a large number of concurrent clients.
Low accept latency: do not make clients a long time before they are accepted.
Low reply latency: send a reply to the client as fast as possible after it has been received.
High efficiency: for a given number of concurrent connections and a given level of latency, use as little resources on the server host as possible (as measured by RAM, number of threads and CPU usage).

We can actually define an abstract server that allow its derivatives to implement different concurrency models Since we want the server to be generic we will supply it with suppliers of MessageEncoderDecoder and MessagingProtocol, as you can see in the following code.
downloadtoggle
38 lines ...
public abstract class BaseServer {
 
    private final int port;
    private final Supplier<MessagingProtocol> protocolFactory;
    private final Supplier<MessageEncoderDecoder> encdecFactory;
 
    public BaseServer(
            int port,
            Supplier<MessagingProtocol> protocolFactory,
            Supplier<MessageEncoderDecoder> encdecFactory) {
 
        this.port = port;
        this.protocolFactory = protocolFactory;
        this.encdecFactory = encdecFactory;
    }
 
    public void serve() {
        try (ServerSocket serverSock = new ServerSocket(port)) {
 
            while (!Thread.currentThread().isInterrupted()) {
 
                Socket clientSock = serverSock.accept();
                ConnectionHandler handler = new ConnectionHandler(
                        clientSock,
                        encdecFactory.get(),
                        protocolFactory.get());
 
                execute(handler);
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
 
        System.out.println("server closed!!!");
    }
 
 
    protected abstract void execute(ConnectionHandler handler);
}
First note that Supplier is an interface in java that has one non default function called get get. A factory is a supplier of objects. Our TCP server needs to create a new Protocol and EncoderDecoder for every connection it receives but since it is a generic server, it does not know what and how to create such objects. This problem is solved using factories, the server receives factories in its constructor that create those objects for it.

To obtain good quality, a TCP server will most often use multiple threads. We will now investigate three simple models of concurrency for servers, i.e., three implementations of preparing the ServerConcurrencyModel interface.

Server Model 1: Single Thread
The following server uses the same (main) thread for accepting a new client and for dealing its requests, by applying the run method of the passive ConnectionHandler object. Clearly, this implementation will have:
No scalability: at any given moment in time, it can serve at most one client.
Very high accept latency (a second client must wait until the first client disconnects to be served).
Very low reply latency: all the resources of the server are concentrated on serving one client.
Good efficiency: the server uses exactly the resources needed to serve one client at a time.

Such a solution may be considered if the time it takes to the server to process a full connection from one client is guaranteed to remain very small. For example, consider the case of a server that provides clients with the date and time value on the server machine. Such a server simply sends one string to the client then disconnects. In this case, it is perfectly fine to use this concurrency model (single thread). For any other type of connections, the model is not appropriate.
downloadtoggle
15 lines ...
public class SingleThreadedServer extends BaseServer {
 
    public SingleThreadedServer(
            int port,
            Supplier<MessagingProtocol> protocolFactory,
            Supplier<MessageEncoderDecoder> encoderDecoderFactory) {
 
        super(port,protocolFactory,encoderDecoderFactory);
    }
 
    @Override
    protected void execute(ConnectionHandler handler) {
        handler.run();
    }
 
}
Server Model 2: Thread per Client
The following server assigns a new thread, for each connected client, by invoking the 'start' method over the runnable ConnectionHandler object.
This implementation will have:

Scalability: the server can serve several concurrent clients, up to the point where there are too many threads running in the process. This happens when there are so many threads that all the RAM of the host is used (each thread allocates a stack and thus consumes RAM) and the scheduler of the host is overwhelmed. Practically, this happens when about 500 to 1000 threads become active within a single process. After this limit, the process itself does not defend itself – it keeps creating new threads for new incoming connections, and thus may become dangerous for the host.
Low accept latency: the time from one accept to the next is approximately the time it takes to create a new thread - which is short compared to the delay between incoming client connections. Thus accept latency is good.
Reply latency: the resources of the server are spread among all the concurrent connections. As long as a reasonable number of connections are active (no more than a few hundreds), and that the load requested by each connection is relatively low in CPU and RAM, then the server architecture will produce good reply latency.
Low efficiency: the server creates a full thread for each connection, even if each connection may be mainly bound to Input/Output operations. That is, most of the time the ConnectionHandler thread will be blocked waiting for input data, but will still use the resources of the thread (RAM and Thread). We therefore expect efficiency to be low - and will find that the Reactor architecture will provide better efficiency.

downloadtoggle
15 lines ...
public class ThreadPerClientServer extends BaseServer {
 
    public ThreadPerClientServer(
            int port,
            Supplier<MessagingProtocol> protocolFactory,
            Supplier<MessageEncoderDecoder> encoderDecoderFactory) {
 
        super(port, protocolFactory, encoderDecoderFactory);
    }
 
    @Override
    protected void execute(ConnectionHandler handler) {
        new Thread(handler).start();
    }
 
}
Server Model 3: Constant Number of Threads
The following server uses a constant number of threads, instead of a thread per client, by adding the runnable ConnectionHandler object to the task queue of a thread pool executor.
The key advantage of this design is that it avoids the danger described above of the server causing a complete host crash when too many clients connect at the same time to the server. The difference is that up to N concurrent client connections, the server behaves the same way as the "thread-per-connection" one; above this number, accept latency will grow. In other words, scalability is limited on purpose to the amount of concurrent connections we believe we can support.

downloadtoggle
24 lines ...
public class FixedThreadPoolServer extends BaseServer {
 
    private final ExecutorService pool;
 
    public FixedThreadPoolServer(
            int numThreads,
            int port,
            Supplier<MessagingProtocol> protocolFactory,
            Supplier<MessageEncoderDecoder> encoderDecoderFactory) {
 
        super(port, protocolFactory, encoderDecoderFactory);
        this.pool = Executors.newFixedThreadPool(numThreads);
    }
 
    @Override
    public void serve() {
        super.serve();
        pool.shutdown();
    }
 
    @Override
    protected void execute(ConnectionHandler handler) {
        pool.execute(handler);
    }
}
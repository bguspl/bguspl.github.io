The Command Invocation Protocol
In the previous sections we learnt about a very simple echo protocol. In this section we are going to create a new generic protocol that will allow clients to execute remote commands on the server. Many complex applications can be described using this generic protocol and we will see an example of a news feed server that allow clients to publish and read news in multiple channels.
The Protocol
download
public interface Command<T> extends Serializable {
 
    Serializable execute(T data);
}
A command is a generic interface with one method: execute. A command is sent by a client but executed by the server (on the server process). Depending on the server type, the server may pass a single argument to the command. This argument can hold different services that the command can interract with (as we will see in the later example). This process can be easily expressed by our MessagingProtocol

downloadtoggle
18 lines ...
public class RemoteCommandInvocationProtocol<T> implements MessagingProtocol<Serializable> {
    
    private T data;
 
    public RemoteCommandInvocationProtocol(T data) {
        this.data = data;
    }
 
    @Override
    public Serializable process(Serializable msg) {
        return ((Command) msg).execute(data);
    }
 
    @Override
    public boolean shouldTerminate() {
        return false;
    }
 
}
Java-Serialization
Java provides a mechanism, called object serialization where an object can be represented as a sequence of bytes that includes the object's data as well as information about the object's type and the types of data stored in the object.
A serialized object (i.e., a byte[]) can be deserialized back into a copy of the original object that is, the type information and bytes that represent the object and its data can be used to recreate the object in memory.

Most impressive is that the entire process is JVM independent, meaning an object can be serialized on one platform and deserialized on an entirely different platform.

Classes ObjectInputStream and ObjectOutputStream are high-level streams that contain the methods for serializing and deserializing any object, they are able to deserialize any Serializable object. A class is Serializable if it:

Implements the Serializable interface or it super class is Serializable
Its first non-serializable super class has a no-args constructor
All its non-transient fields must be serializable

For the purpose of this lecture this information is sufficient, you can read more about the serialization mechanism in the following links: serializable javadoc, basic serialization tutorial and java object serialization.
With the above knowledge we can actually design a message encoder decoder that can handle arbitrary serializable objects and especially our commands:

downloadtoggle
85 lines ...
public class ObjectEncoderDecoder<> implements MessageEncoderDecoder<Serializable> {
 
    private final byte[] lengthBytes = new byte[4];
    private int lengthBytesIndex = 0;
    private byte[] objectBytes = null;
    private int objectBytesIndex = 0;
 
    @Override
    public Serializable decodeNextByte(byte nextByte) {
        if (objectBytes == null) { //indicates that we are still reading the length
            lengthBytes[lengthBytesIndex++] = nextByte;
            if (lengthBytesIndex == lengthBytes.length) { //we read 4 bytes and therefore can take the length
                int len = bytesToInt(lengthBytes);
                objectBytes = new byte[len];
                objectBytesIndex = 0;
                lengthBytesIndex = 0;
            }
        } else {
            objectBytes[objectBytesIndex++] = nextByte;
            if (objectBytesIndex == objectBytes.length) {
                Serializable result = deserializeObject();
                objectBytes = null;
                return result;
            }
        }
 
        return null;
    }
 
    private static void intToBytes(int i, byte[] b) {
        b[0] = (byte) (i >> 24);
        b[1] = (byte) (i >> 16);
        b[2] = (byte) (i >> 8);
        b[3] = (byte) i;
    }
 
    private static int bytesToInt(byte[] b) {
        //this is the reverse of intToBytes, 
        //note that for every byte, when casting it to int, 
        //it may include some changes to the sign bit so we remove those by anding with 0xff
 
        return ((b[0] & 0xff) << 24)
                | ((b[1] & 0xff) << 16)
                | ((b[2] & 0xff) << 8)
                | (b[3] & 0xff);
    }
 
    @Override
    public byte[] encode(Serializable message) {
        return serializeObject(message);
    }
 
    private Serializable deserializeObject() {
        try {
            ObjectInput in = new ObjectInputStream(new ByteArrayInputStream(objectBytes));
            return (Serializable) in.readObject();
        } catch (Exception ex) {
            throw new IllegalArgumentException("cannot desrialize object", ex);
        }
 
    }
 
    private byte[] serializeObject(Serializable message) {
        try {
            ByteArrayOutputStream bytes = new ByteArrayOutputStream();
 
            //placeholder for the object size
            for (int i = 0; i < 4; i++) {
                bytes.write(0);
            }
 
            ObjectOutput out = new ObjectOutputStream(bytes);
            out.writeObject(message);
            out.flush();
            byte[] result = bytes.toByteArray();
 
            //now write the object size
            intToBytes(result.length - 4, result);
            return result;
 
        } catch (Exception ex) {
            throw new IllegalArgumentException("cannot serialize object", ex);
        }
    }
 
}
The ObjectEncoderDecoder is the first binary encoder decoder that you encountered in this course. It is actually much more simple than it looks like, the encoding of a message that contains the object O which can be serialized to a byte array b1,b2,b3,...,bN will be N,b1,b2,b3,...,bn (i.e., the message will start with the number of bytes that need to be read in order to deserialize an object). The number of bytes N is sent using a binary representation - note byteToInt and intToByte. Finally, when an object received, the decodeNextByte method first checks if it belongs to N or that we already started to read b1,b2,...,bN and fill the correct byte arrays.

We can now create a generic client for our generic protocol

downloadtoggle
39 lines ...
public class RCIClient implements Closeable{
    
    private final ObjectEncoderDecoder encdec;
    private final Socket sock;
    private final BufferedInputStream in;
    private final BufferedOutputStream out;
 
    public RCIClient(String host, int port) throws IOException {
        sock = new Socket(host, port);
        encdec = new ObjectEncoderDecoder();
        in = new BufferedInputStream(sock.getInputStream());
        out = new BufferedOutputStream(sock.getOutputStream());
    }
 
   
    public void send(Command<?> cmd) throws IOException {
        out.write(encdec.encode(cmd));
        out.flush();
    }
 
    public Serializable receive() throws IOException {
        int read;
        while ((read = in.read()) >= 0) {
            Serializable msg = encdec.decodeNextByte((byte) read);
            if (msg != null) {
                return msg;
            }
        }
 
        throw new IOException("disconnected before complete reading message");
    }
 
    @Override
    public void close() throws IOException {
        out.close();
        in.close();
        sock.close();
    }
 
}
These 4 classes (the Command, RemoteCommandInvocationProtocol, ObjectEncoderDecoder and RCIClient ) can be serve as the basis for many advanced servers as we will see next.

NewsFeed Server
Lets utilize our generic command invocation protocol to create a NewsFeed server. This server will allow clients to execute two commands:
Publish news to a category by its name
Fetch all the news which were published to a specific category

The main object that is manipulated by the server is the NewsFeed
download
public interface NewsFeed {
 
    void clear();
 
    List<String> fetch(String category);
 
    void publish(String category, String news);
    
}
The client commands receives the NewsFeed and manipulate it

downloadtoggle
13 lines ...
public class FetchNewsCommand implements Command<NewsFeed> {
 
    private String category;
 
    public FetchNewsCommand(String category) {
        this.category= category;
    }
 
    @Override
    public Serializable execute(NewsFeed feed) {
        return feed.fetch(category);
    }
 
}
downloadtoggle
16 lines ...
public class PublishNewsCommand implements Command<NewsFeed> {
 
    private String category;
    private String news;
 
    public PublishNewsCommand(String category, String news) {
        this.category= category;
        this.news = news;
    }
 
    @Override
    public Serializable execute(NewsFeed feed) {
        feed.publish(category, news);
        return "OK";
    }
 
}
Note that the client works with the interface of news feed while the server will have the actual implementation. Since the news feed can be manipulated by different connection handlers in the server on the same time (as they will respond to concurrent client requests) it must be implemented as a thread safe object.

downloadtoggle
24 lines ...
public class NewsFeedImpl implements NewsFeed {
 
    private ConcurrentHashMap<String, ConcurrentLinkedQueue<String>> newsPerCategory = new ConcurrentHashMap<>();
 
    @Override
    public List<String> fetch(String category) {
        ConcurrentLinkedQueue<String> queue = newsPerCategory.get(category);
        if (queue == null) {
            return new ArrayList<>(0); //empty
        } else {
            return new ArrayList<>(queue); //copy of the queue, arraylist is serializable
        }
    }
 
    @Override
    public void publish(String category, String news) {
        ConcurrentLinkedQueue<String> queue = newsPerCategory.computeIfAbsent(category, (k) -> new ConcurrentLinkedQueue<>());
        queue.add(news);
    }
 
    @Override
    public void clear() {
        newsPerCategory.clear();
    }
}
This is actually all that is needed in order to implement our protocol, we can now start the server as follows:

downloadtoggle
11 lines ...
public class NewsFeedServerMain {
 
    public static void main(String[] args) {
        NewsFeed feed = new NewsFeedImpl(); //one shared object
 
        new ThreadPerClientServer(
                7777, //port
                () -> new RemoteCommandInvocationProtocol<>(feed), //protocol factory
                () -> new ObjectEncoderDecoder<>()                 //message encoder decoder factory
        ).serve();
    }
}
And we can use the following code to test our server

downloadtoggle
51 lines ...
public class NewsFeedClientMain {
 
    public static void main(String[] args) throws Exception {
        if (args.length == 0) {
            args = new String[]{"localhost"};
        }
 
        System.out.println("running clients");
 
        runFirstClient(args[0]);
        runSecondClient(args[0]);
        runThirdClient(args[0]);
    }
 
    private static void runFirstClient(String host) throws Exception {
        try (RCIClient c = new RCIClient(host, 7777)) {
            c.send(new PublishNewsCommand(
                    "jobs",
                    "System Programmer, knowledge in C++, Java and Python required. call 0x134693F"));
 
            c.receive(); //ok
 
            c.send(new PublishNewsCommand(
                    "headlines",
                    "new SPL assignment is out soon!!"));
 
            c.receive(); //ok
 
            c.send(new PublishNewsCommand(
                    "headlines",
                    "THE CAKE IS A LIE!"));
 
            c.receive(); //ok
 
        }
 
    }
 
    private static void runSecondClient(String host) throws Exception {
        try (RCIClient c = new RCIClient(host, 7777)) {
            c.send(new FetchNewsCommand("jobs"));
            System.out.println("second client received: " + c.receive());
        }
    }
 
    private static void runThirdClient(String host) throws Exception {
        try (RCIClient c = new RCIClient(host, 7777)) {
            c.send(new FetchNewsCommand("headlines"));
            System.out.println("third client received: " + c.receive());
        }
    }
}
which will print:

second client received: [System Programmer, knowledge in C++, Java and Python required. call 0x134693F]
third client received: [new SPL assignment is out soon!!, THE CAKE IS A LIE!]
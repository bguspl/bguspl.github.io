Practical Session 11 - Reactor
In this practical session we'll discuss the implementation of the Reactor pattern, learned in class.
Why and What is the Reactor Pattern?
Most servers need to handle many clients simultaniously.
One way of achieving this is using a Multi-Threaded server, as seen in Tirgul 10. In this design, one thread is in charge of accepting new connections, and opens a new thread for each new client. Thus, handling 30 clients requires 31 threads.

This kind of a Multi-Threaded server is problematic for several reasons:

It's wasteful
Creating a new Thread is relatively expensive.
Each thread requires a fair amount of memory.
Threads are blocked most of the time waiting for network IO.
It's not scalable
A Multi-Threaded server can't grow to accommodate tens of thousands of concurrent requests.
It's also extremely vulnerable to Denial Of Service attacks.
Poor availability – it takes a long time to create a new thread for each new client. Moreover, the response time degrades as the number of clients rises.

The Reactor Pattern is another (better) design for handling several concurrent clients.
It's based on the observation that if a thread does not need to wait for Network IO, a single thread could easily handle tens of client requests.

Based on this observation, the Reactor Pattern:

Use Non-Blocking IO, so Threads don't waste time waiting for Network.
Have one thread in charge of the Network: accepting new connections and handling IO. As the Network is non-blocking, read, write and accept operations "take no time", and a single thread is enough for all the clients.
Have a fixed number of threads (for example in a thread pool), which are in charge of the protocol. These threads perform the message framing (tokenizations) and message processing (what to do with each message). Note that unlike the Multi-Threaded server, in this design a single thread may handle many clients.
Selector
When we perform non-blocking IO, read(), for example, read the numbers of currently available bytes, and returns. This means we can read or write 0 bytes, and this is what will probably happen most of the time. We would like a way for our thread to wait until any of our channels is ready, and only then perform the read(), write() or accept(), only on the ready channel. This is what the Selector class does.
The Channel registers itself to a Selector, adding a new selection key. Registration means that the channel informs the Selector it is interested in specific events. This event can be "the channel is ready to be read", "the channel can now be written to" or "the channel can now accept()". During registration, the selector is told which events to monitor. In addition, when registering a channel to a Selector, one can add an "attachment". An attachment is an object that we will have access to when the event occurs (usually, this object is associated with a task that should be performed).

Registration:

	Selector selector = Selector.open(); // a new Selector
	Object anAttachment = new Object();
	socketChannel.register(selector, SelectionKey.OP_READ, anAttachmemt);
https://www.cs.bgu.ac.il/~spl201/wiki.files/Selector.png

Bitmasks
SelectionKey.OP_READ is a number representing the READ event.
Similarly, SelectionKey.OP_WRITE is a number representing the WRITE event.
How would we represent a "READ or WRITE" event? One option would be to define another constant: SelectionKet.OP_READ_OR_WRITE, but this is cumbersome. When checking for a value we would need to write something like: if (e = SelectionKey.OP_READ || e = SelectionKey.OP_READ_OR_WRITE). Now think about an OP_READ_OR_ACCEPT event, etc.
A better solution is to use a Bitmask. We'll define the values of the flags so that their 1-bits won't overlap. For example: OP_READ=1, OP_WRITE=4, OP_ACCEPT=16. We could then use bitwise operations. The "read or write" event will be: OP_READ | OP_WRITE, and it's value will be 5. Then, when checking for a read event, it's enough to ask: if (e & OP_READ != 0)

select()
The Selector holds 3 lists of selection keys
key - A set of all the selection keys in the selector
selected-key - A set of all the selection keys with a ready operation in the selector
cancelled-key - A set of all the selection keys that were cancelled but not yet unregistered in the selector
Once a Selector is registered to some channels, one can use the "select()" method. This method blocks until at least one of the channels is ready for the registered event. Then, a set of SelectionKeys is returned. Each Selectionkey is the set is associated with at least one ready event.
The Reactor
Now that we know how Java's non-blocking IO works, we'll take a closer look at the Reactor implementation which employs it.
It's the same implementation given in class.

We will concentrate on the "core" reactor classes: Reactor, NonBlockingConnectionHandler and the ActorThreadPool.

The Reactor is the main thread, which is in charge of reading, writing and accepting new connections. Reading and Writing are encapsulated in the NonBlockingConnectionHandler class, and accepting happens in the Reactor class.

The reactor thread also holds a thread-pool of workers. These worker-threads are in charge of the protocol handling: they receive the bytes read from the network, and let the protocol do something with them. The class which represents the thread pool is ActorThreadPool.

Once some bytes were read by the reactor thread, the NonBlockingConnectionHandler creates a lambda that is responsible for decoding, processing and encoding of a single read operation. The decoding  encoding and processing is done by using the MessagingProtocol and MessagingEncoderDecoder of the specific client. This task is then submitted to the ActorThreadPool. Each client may have at most one thread in the ActorThreadPool handling his own task at any given time. By using a list of incoming tasks for each client and synchronizing the submit function we make sure that the lambdas will run in correct order thus keeping the order in which the data was read from the socket. The synchronized code checks if a thread is currently running lambdas that belong to the current client submitting, if it is, it adds the new lambda to a list. If the actor has no running tasks at the moment, then the given lambda is wrapped so that after it is finished it will call the complete function. The complete function will also synchronize on the actor checking if new tasks are available to process for that same actor, if so it will push the next one into the executor.

We notice a couple of strange new collections we haven't used before:
1) WeakHashMap - We use it to map a client handler to its' list of lambdas to perform. In a WeakHashMap, the key is not counted as a reference. this means that if this is the only reference to the object, the garbage collector will still be able to clear it from memory. Thus the ActorThreadPool doesn't need to "clean after" a connection has been closed and ConnectionHandler discarded.
2) ReadWriteLock - we protect that hash map by using a ReadWriteLock. A ReadWriteLock allows for multiple simultaneous read locks but only a single writeLock while not allowing read and write at the same time. This reduces the synchronization needed when accessing the map. If we only want to get the list for the current actor we use a read lock. If we wish to add a new actor to the map, we use a write lock.
https://www.cs.bgu.ac.il/~spl201/wiki.files/reactor-fixed.png

Reactor.java
downloadtoggle
122 lines ...
public class Reactor<T> implements Server<T> {
 
    private final int port;
    private final Supplier<MessagingProtocol<T>> protocolFactory;
    private final Supplier<MessageEncoderDecoder<T>> readerFactory;
    private final ActorThreadPool<NonBlockingConnectionHandler> pool;
    private Selector selector;
 
    private Thread selectorThread;
    private final ConcurrentLinkedQueue<Runnable> selectorTasks = new ConcurrentLinkedQueue<>();
 
    public Reactor(
            int numThreads,
            int port,
            Supplier<MessagingProtocol<T>> protocolFactory,
            Supplier<MessageEncoderDecoder<T>> readerFactory) {
 
        this.pool = new ActorThreadPool<>(numThreads);
        this.port = port;
        this.protocolFactory = protocolFactory;
        this.readerFactory = readerFactory;
    }
 
    @Override
    public void serve() {
        selectorThread = Thread.currentThread();
 
        try (   Selector selector = Selector.open();
                ServerSocketChannel serverSock = ServerSocketChannel.open()) {
 
            this.selector = selector; //just to be able to close
 
            serverSock.bind(new InetSocketAddress(port));
            serverSock.configureBlocking(false);
            serverSock.register(selector, SelectionKey.OP_ACCEPT);
 
            while (!Thread.currentThread().isInterrupted()) {
 
                selector.select();
                runSelectionThreadTasks();
 
                for (SelectionKey key : selector.selectedKeys()) {
 
                    if (!key.isValid()) {
                        continue;
                    } else if (key.isAcceptable()) {
                        handleAccept(serverSock, selector);
                    } else {
                        handleReadWrite(key);
                    }
                }
 
                selector.selectedKeys().clear(); //clear the selected keys set so that we can know about new events
 
            }
 
        } catch (ClosedSelectorException ex) {
            //do nothing - server was requested to be closed
        } catch (IOException ex) {
            //this is an error
            ex.printStackTrace();
        }
 
        System.out.println("server closed!!!");
        pool.shutdown();
    }
 
    void updateInterestedOps(SocketChannel chan, int ops) {
        final SelectionKey key = chan.keyFor(selector);
        if (Thread.currentThread() == selectorThread) {
            key.interestOps(ops);
        } else {
            selectorTasks.add(() -> {
                if(key.isValid())
                    key.interestOps(ops);
            });
            selector.wakeup();
        }
    }
 
 
    private void handleAccept(ServerSocketChannel serverChan, Selector selector) throws IOException {
        SocketChannel clientChan = serverChan.accept();
 
        clientChan.configureBlocking(false);
        final NonBlockingConnectionHandler<T> handler = new NonBlockingConnectionHandler<>(
                readerFactory.get(),
                protocolFactory.get(),
                clientChan,
                this);
 
        clientChan.register(selector, SelectionKey.OP_READ, handler);
    }
 
    private void handleReadWrite(SelectionKey key) {
        @SuppressWarnings("unchecked")
        NonBlockingConnectionHandler<T> handler = (NonBlockingConnectionHandler<T>) key.attachment();
        if (key.isReadable()) {
            Runnable task = handler.continueRead();
            if (task != null) {
                pool.submit(handler, task);
            } else if (!key.isValid()) {
                // if no task was created key may no longer be valid and throw exception on key.isWritable
                return;
            }
        }
        if (key.isWritable()) {
            handler.continueWrite();
        }
    }
 
    private void runSelectionThreadTasks() {
        while (!selectorTasks.isEmpty()) {
            selectorTasks.remove().run();
        }
    }
 
    @Override
    public void close() throws IOException {
        selector.close();
    }
 
}


NonBlockingConnectionHandler .java
downloadtoggle
100 lines ...
public class NonBlockingConnectionHandler<T> implements ConnectionHandler<T> {
private static final int BUFFER_ALLOCATION_SIZE = 1 << 13; //8k
    private static final ConcurrentLinkedQueue<ByteBuffer> BUFFER_POOL = new ConcurrentLinkedQueue<>();
 
    private final MessagingProtocol<T> protocol;
    private final MessageEncoderDecoder<T> encdec;
    private final Queue<ByteBuffer> writeQueue = new ConcurrentLinkedQueue<>();
    private final SocketChannel chan;
    private final Reactor reactor;
 
    public NonBlockingConnectionHandler(
            MessageEncoderDecoder<T> reader,
            MessagingProtocol<T> protocol,
            SocketChannel chan,
            Reactor reactor) {
        this.chan = chan;
        this.encdec = reader;
        this.protocol = protocol;
        this.reactor = reactor;
    }
 
    public Runnable continueRead() {
        ByteBuffer buf = leaseBuffer();
 
        boolean success = false;
        try {
            success = chan.read(buf) != -1;
        } catch (IOException ex) {
            ex.printStackTrace();
        }
 
        if (success) {
            buf.flip();
            // Creating a task, Notice that this code is very similar to the BlockingConnectionHandler's main loop
            return () -> {
                try {
                    while (buf.hasRemaining()) {
                        T nextMessage = encdec.decodeNextByte(buf.get());
                        if (nextMessage != null) {
                            T response = protocol.process(nextMessage);
                            if (response != null) {
                                writeQueue.add(ByteBuffer.wrap(encdec.encode(response)));
                                reactor.updateInterestedOps(chan, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
                            }
                        }
                    }
                } finally {
                    releaseBuffer(buf);
                }
            };
        } else {
            releaseBuffer(buf);
            close();
            return null;
        }
 
    }
 
    public void close() {
        try {
            chan.close();
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }
 
    public void continueWrite() {
        while (!writeQueue.isEmpty()) {
            try {
                ByteBuffer top = writeQueue.peek();
                chan.write(top);
                if (top.hasRemaining()) {
                    return;
                } else {
                    writeQueue.remove();
                }
            } catch (IOException ex) {
                ex.printStackTrace();
                close();
            }
        }
 
        if (writeQueue.isEmpty()) {
            if (protocol.shouldTerminate()) close();
            else reactor.updateInterestedOps(chan, SelectionKey.OP_READ);
        }
    }
 
    private static ByteBuffer leaseBuffer() {
        ByteBuffer buff = BUFFER_POOL.poll();
        if (buff == null) {
            return ByteBuffer.allocateDirect(BUFFER_ALLOCATION_SIZE);
        }
 
        buff.clear();
        return buff;
    }
 
    private static void releaseBuffer(ByteBuffer buff) {
        BUFFER_POOL.add(buff);
    }}


ActorThreadPool.java
downloadtoggle
64 lines ...
public class ActorThreadPool<T> {
 
    private final Map<T, Queue<Runnable>> acts;
    private final ReadWriteLock actsRWLock;
    private final Set<T> playingNow;
    private final ExecutorService threads;
 
    public ActorThreadPool(int threads) {
        this.threads = Executors.newFixedThreadPool(threads);
        acts = new WeakHashMap<>();
        playingNow = ConcurrentHashMap.newKeySet();
        actsRWLock = new ReentrantReadWriteLock();
    }
 
    public void submit(T act, Runnable r) {
        synchronized (act) {
            if (!playingNow.contains(act)) {
                playingNow.add(act);
                execute(r, act);
            } else {
                pendingRunnablesOf(act).add(r);
            }
        }
    }
 
    public void shutdown() {
        threads.shutdownNow();
    }
 
    private Queue<Runnable> pendingRunnablesOf(T act) {
 
        actsRWLock.readLock().lock();
        Queue<Runnable> pendingRunnables = acts.get(act);
        actsRWLock.readLock().unlock();
 
        if (pendingRunnables == null) {
            actsRWLock.writeLock().lock();
            acts.put(act, pendingRunnables = new LinkedList<>());
            actsRWLock.writeLock().unlock();
        }
        return pendingRunnables;
    }
 
    private void execute(Runnable r, T act) {
        threads.execute(() -> {
            try {
                r.run();
            } finally {
                complete(act);
            }
        });
    }
 
    private void complete(T act) {
        synchronized (act) {
            Queue<Runnable> pending = pendingRunnablesOf(act);
            if (pending.isEmpty()) {
                playingNow.remove(act);
            } else {
                execute(pending.poll(), act);
            }
        }
    }
 
}


Server.java
downloadtoggle
46 lines ...
public interface Server<T> extends Closeable {
 
    /**
     * The main loop of the server, Starts listening and handling new clients.
     */
    void serve();
 
    /**
     *This function returns a new instance of a thread per client pattern server
     * @param port The port for the server socket
     * @param protocolFactory A factory that creats new MessagingProtocols
     * @param encoderDecoderFactory A factory that creats new MessageEncoderDecoder
     * @param <T> The Message Object for the protocol
     * @return A new Thread per client server
     */
    public static <T> Server<T>  threadPerClient(
            int port,
            Supplier<MessagingProtocol<T> > protocolFactory,
            Supplier<MessageEncoderDecoder<T> > encoderDecoderFactory) {
 
        return new BaseServer<T>(port, protocolFactory, encoderDecoderFactory) {
            @Override
            protected void execute(BlockingConnectionHandler<T>  handler) {
                new Thread(handler).start();
            }
        };
 
    }
 
    /**
     * This function returns a new instance of a reactor pattern server
     * @param nthreads Number of threads available for protocol processing
     * @param port The port for the server socket
     * @param protocolFactory A factory that creats new MessagingProtocols
     * @param encoderDecoderFactory A factory that creats new MessageEncoderDecoder
     * @param <T> The Message Object for the protocol
     * @return A new reactor server
     */
    public static <T> Server<T> reactor(
            int nthreads,
            int port,
            Supplier<MessagingProtocol<T>> protocolFactory,
            Supplier<MessageEncoderDecoder<T>> encoderDecoderFactory) {
        return new Reactor<T>(nthreads, port, protocolFactory, encoderDecoderFactory);
    }
 
}
# bio nio aio

```
1.同步指的是用户进程触发IO操作并等待或者轮询地去查看IO操作是否就绪。
2.异步是指用户进程触发IO操作以后便开始做自己的事情，而当IO操作已经完成的时候会得到IO完成的通知（异步的特点就是通知）。
3.阻塞是指, 当试图对该文件描述符进行读写时, 如果当时没有东西可读,或者暂时不可写, 线程就进入等待状态, 直到有东西可读或者可写为止。
4.非阻塞是指，如果没有东西可读, 或者不可写, 读写函数马上返回, 而不会等待。
```

Java中的IO方式通常分为几种，即：

同步阻塞的BIO、同步非阻塞的NIO、异步非阻塞的AIO。

IO为同步阻塞形式，NIO为同步非阻塞形式，NIO并没有实现异步，在JDK1.7后升级NIO库包，开始支持异步非阻塞(AIO)。

## 1.BIO

同步阻塞式IO，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

　　在JDK1.4出来之前，我们建立网络连接的时候采用BIO模式，需要先在服务端启动一个ServerSocket，然后在客户端启动Socket来对服务端进行通信。默认情况下服务端需要对每个请求建立一堆线程等待请求，而客户端发送请求后，先咨询服务端是否有线程响应，如果没有则会一直等待或者遭到拒绝请求，如果有的话，客户端线程会在等待请求结束后继续执行。

## 2.NIO即non-blocking IO

是指jdk1.4 及以上版本里提供的新api（New IO） ，为所有的原始类型（boolean类型除外）提供缓存支持的数据容器。

　　NIO(reactor模型，同步IO)：同步非阻塞式IO，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。

　　NIO主要想解决的是BIO的大并发问题： 在使用同步I/O的网络应用中，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理。这样做虽然可以达到我们的要求，但同时又会带来另外一个问题：由于每创建一个线程，就要为这个线程分配一定的内存空间（也叫工作存储器），而操作系统本身对线程的总数也会有一定的限制，如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，服务器甚至也可能会因此而瘫痪。NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应地通知应用程序进行处理，应用再将流读取到缓冲区或写入操作系统。    

　　NIO的最重要的地方是当一个连接创建后，不需要对应一个线程，这个连接会被注册到多路复用器上面，所以所有的连接只需要一个线程就可以搞定，当这个线程中的多路复用器进行轮询的时候，发现连接上有请求的话，才开启一个线程进行处理，也就是一个请求一个线程模式。

　　NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。传统IO基于字节流和字符流进行操作，而NIO基于Channel和Buffer(缓冲区)进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

　　NIO和传统IO（一下简称IO）之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。 Java IO面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方；此外，它不能前后移动流中的数据。如果需要前后移动从流中读取的数据，需要先将它缓存到一个缓冲区。NIO的缓冲导向方法略有不同。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动。这就增加了处理过程中的灵活性。但是，还需要检查该缓冲区中是否包含所有你需要处理的数据。而且，需确保当更多的数据读入缓冲区时，不要覆盖缓冲区里尚未处理的数据。
　　IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

　　NIO中的Channel和BIO中的Stream(流)基本上是一个等级的。只不过Stream是单向的，如InputStream, OutputStream；而Channel是双向的，既可以用来进行读操作，又可以用来进行写操作。

　　NIO中的Channel的主要实现有：

FileChannel
DatagramChannel
SocketChannel
ServerSocketChannel
　　见名知义，上述Channel分别可以对应文件IO、UDP和TCP（Server和Client）。

　　NIO中的Buffer实现有：ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer，它们分别对应基本数据类型: byte, char, double, float, int, long, short。另外，NIO中还有MappedByteBuffer, HeapByteBuffer, DirectByteBuffer等。

　　NIO中的Selector运行单线程处理多个Channel，如果你的应用打开了多个通道，但每个连接的流量都很低，使用Selector就会很方便。例如在一个聊天服务器中。要使用Selector, 得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件。

传统IO方式读取文件：

```

static void bioMethod() {
        InputStream in = null;
        try {
            in = new BufferedInputStream(new FileInputStream("src/com/itszt/test6/io.txt"));
            byte[] buf = new byte[1024];
            int bytesRead = in.read(buf);
            while (bytesRead != -1) {
                for (int i = 0; i < bytesRead; i++){
                    System.out.print((char) buf[i]);
                }
                bytesRead = in.read(buf);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    in.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

通过NIO的方式来处理上述文件读取问题：

```

static void nioMethod() {
        RandomAccessFile aFile = null;
        try {
            aFile = new RandomAccessFile("src/com/itszt/test6/io.txt", "rw");
            FileChannel fileChannel = aFile.getChannel();
            ByteBuffer buf = ByteBuffer.allocate(1024);
 
            int bytesRead = fileChannel.read(buf);//返回字符的数量
            System.out.println(bytesRead);
 
            while (bytesRead != -1) {
                buf.flip();
                while (buf.hasRemaining()) {
                    System.out.print((char) buf.get());
                }
 
                buf.compact();
                bytesRead = fileChannel.read(buf);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (aFile != null) {
                    aFile.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

使用Buffer时，一般遵循下面几个步骤：

```
分配空间（ByteBuffer buf = ByteBuffer.allocate(1024); ）
写入数据到Buffer(int bytesRead = fileChannel.read(buf);)
调用filp()方法（ buf.flip();）
从Buffer中读取数据（System.out.print((char)buf.get());）
调用clear()方法或者compact()方法
```

Buffer顾名思义是缓冲区的意思，它实际上是一个容器，相当于一个连续数组。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer。

向Buffer中写数据：

```
从Channel写到Buffer (fileChannel.read(buf))
通过Buffer的put()方法 （buf.put(…)）
```

从Buffer中读取数据：

```
- 从Buffer读取到Channel (channel.write(buf))
- 使用get()方法从Buffer中读取数据 （buf.get()）
```

可以把Buffer简单地理解为一组基本数据类型的元素列表，它通过几个变量来保存这个数据的当前位置状态：capacity, position, limit, mark：

| 索引     | 说明                                                    |
| -------- | ------------------------------------------------------- |
| capacity | 缓冲区数组的总长度                                      |
| position | 下一个要操作的数据元素的位置                            |
| limit    | 缓冲区数组中不可操作的下一个元素的位置：limit<=capacity |
| mark     | 用于记录当前position的前一个位置或者默认是0             |

处理大文件，一般用BufferedReader,BufferedInputStream这类带缓冲的IO类，不过如果文件超大的话，更快的方式是采用MappedByteBuffer。

　　MappedByteBuffer是NIO引入的文件内存映射方案，读写性能极高。NIO最主要的就是实现了对异步操作的支持。其中一种通过把一个套接字通道(SocketChannel)注册到一个选择器(Selector)中，不时调用后者的选择(select)方法就能返回满足的选择键(SelectionKey)，键中包含了SOCKET事件信息。这就是select模型。

　　SocketChannel的读写是通过一个类叫ByteBuffer来操作的。ByteBuffer有两种模式：直接/间接。间接模式最典型的就是HeapByteBuffer，即操作堆内存 (byte[])，但是内存毕竟有限,如果发送一个超大空间的文件（如1GB)，这时就必须使用”直接”模式,即 MappedByteBuffer,文件映射。

　　我们先来看操作系统的内存管理。一般操作系统的内存分两部分：物理内存；虚拟内存。虚拟内存一般使用的是页面映像文件，即硬盘中的某个(某些)特殊文件。操作系统负责页面文件内容的读写，这个过程叫”页面中断/切换”。 MappedByteBuffer 是一种特殊的ByteBuffer，是ByteBuffer的子类。 MappedByteBuffer 将文件直接映射到内存（这里的内存指的是虚拟内存，并不是物理内存）。通常，可以映射整个文件，如果文件比较大的话可以分段进行映射，只要指定文件的哪个部分即可。

　　FileChannel提供了map方法把文件影射为内存映像文件：

　　MappedByteBuffer map(int mode,long position,long size); 可以把文件从position开始的size大小的区域映射为内存映像文件，mode指出了 可访问该内存映像文件的方式：

```
READ_ONLY,（只读）： 试图修改得到的缓冲区将导致抛出 ReadOnlyBufferException.(MapMode.READ_ONLY)
READ_WRITE（读/写）： 对得到的缓冲区的更改最终将传播到文件；该更改对映射到同一文件的其他程序不一定是可见的。 (MapMode.READ_WRITE)
PRIVATE（专用）： 对得到的缓冲区的更改不会传播到文件，并且该更改对映射到同一文件的其他程序也不是可见的；相反，会创建缓冲区已修改部分的专用副本。
```

　MappedByteBuffer是ByteBuffer的子类，其扩充了三个方法：

```
force()：缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文件；
load()：将缓冲区的内容载入内存，并返回该缓冲区的引用；
isLoaded()：如果缓冲区的内容在物理内存中，则返回真，否则返回假；
```

采用MappedByteBuffer读取文件：

```

static void mapNIOMethod() {
        RandomAccessFile aFile = null;
        FileChannel fc = null;
        try {
            aFile = new RandomAccessFile("src/com/itszt/test6/io.txt",
                    "rw");
            fc = aFile.getChannel();
            long timeBegin = System.currentTimeMillis();
            MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_ONLY, 0, aFile.length());
            long timeEnd = System.currentTimeMillis();
            System.out.println("Read time: " + (timeEnd - timeBegin) + "ms");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (aFile != null) {
                    aFile.close();
                }
                if (fc != null) {
                    fc.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

scatter和gather

分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

　　聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel。因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

　　scatter / gather经常用于将需要传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

```

static void gather() {
        ByteBuffer header = ByteBuffer.allocate(10);
        ByteBuffer body = ByteBuffer.allocate(10);
 
        byte[] b1 = {'0', '1'};
        byte[] b2 = {'2', '3'};
        header.put(b1);
        body.put(b2);
 
        ByteBuffer[] buffs = {header, body};
 
        try {
            FileOutputStream os = new FileOutputStream("src/com/itszt/test6/io.txt");
            FileChannel channel = os.getChannel();
            channel.write(buffs);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中：

```
static void transferFrom() {
        RandomAccessFile fromFile = null;
        RandomAccessFile toFile = null;
        try {
            fromFile = new RandomAccessFile("src/com/itszt/test6/fromFile.xml", "rw");
            FileChannel fromChannel = fromFile.getChannel();
            toFile = new RandomAccessFile("src/com/itszt/test6/toFile.txt", "rw");
            FileChannel toChannel = toFile.getChannel();
 
            long position = 0;
            long count = fromChannel.size();
            System.out.println(count);
            toChannel.transferFrom(fromChannel, position, count);
 
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fromFile != null) {
                    fromFile.close();
                }
                if (toFile != null) {
                    toFile.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

　　transferTo()方法将数据从FileChannel传输到其他的channel中：

```

static void transferTo() {
        RandomAccessFile fromFile = null;
        RandomAccessFile toFile = null;
        try {
            fromFile = new RandomAccessFile("src/com/itszt/test6/fromFile.xml", "rw");
            FileChannel fromChannel = fromFile.getChannel();
            toFile = new RandomAccessFile("src/com/itszt/test6/toFile.txt", "rw");
            FileChannel toChannel = toFile.getChannel();
 
            long position = 0;
            long count = fromChannel.size();
            System.out.println(count);
            fromChannel.transferTo(position, count, toChannel);
 
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fromFile != null) {
                    fromFile.close();
                }
                if (toFile != null) {
                    toFile.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

NIO 管道Pipe是2个线程之间的单向数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。如下：

```
static void pipeMethod() {
        Pipe pipe = null;
        ExecutorService exec = Executors.newFixedThreadPool(2);
        try {
            pipe = Pipe.open();
            final Pipe pipeTemp = pipe;
 
            exec.submit(new Callable<Object>() {
                @Override
                public Object call() throws Exception {
                    Pipe.SinkChannel sinkChannel = pipeTemp.sink();//向通道中写数据
                    while (true) {
                        TimeUnit.SECONDS.sleep(1);
                        String newData = "Pipe Test At Time " +
                                System.currentTimeMillis();
                        ByteBuffer buf = ByteBuffer.allocate(1024);
                        buf.clear();
                        buf.put(newData.getBytes());
                        buf.flip();
                        while (buf.hasRemaining()) {
                            System.out.println(buf);
                            sinkChannel.write(buf);
                        }
                    }
                }
            });
 
            exec.submit(new Callable<Object>() {
                @Override
                public Object call() throws Exception {
                    Pipe.SourceChannel sourceChannel = pipeTemp.source();//向通道中读数据
                    while (true) {
                        TimeUnit.SECONDS.sleep(1);
                        ByteBuffer buf = ByteBuffer.allocate(1024);
                        buf.clear();
                        int bytesRead = sourceChannel.read(buf);
                        System.out.println("bytesRead=" + bytesRead);
                        while (bytesRead > 0) {
                            buf.flip();
                            byte b[] = new byte[bytesRead];
                            int i = 0;
                            while (buf.hasRemaining()) {
                                b[i] = buf.get();
                                System.out.printf("%X", b[i]);
                                i++;
                            }
                            String s = new String(b);
                            System.out.println("=================||" + s);
                            bytesRead = sourceChannel.read(buf);
                        }
                    }
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            exec.shutdown();
        }
    }
```

NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。如下：

```

static void recive() {
        DatagramChannel channel = null;
        try {
            channel = DatagramChannel.open();
            channel.socket().bind(new InetSocketAddress(8888));
            ByteBuffer buf = ByteBuffer.allocate(1024);
            buf.clear();
            channel.receive(buf);
            buf.flip();
            while (buf.hasRemaining()) {
                System.out.print((char) buf.get());
            }
            System.out.println();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (channel != null) {
                    channel.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 
    static void send() {
        DatagramChannel channel = null;
        try {
            channel = DatagramChannel.open();
            String info = "I'm the Sender!";
            ByteBuffer buf = ByteBuffer.allocate(1024);
            buf.clear();
            buf.put(info.getBytes());
            buf.flip();
 
            int bytesSent = channel.send(buf, new InetSocketAddress("127.0.0.1",
                    8888));
            //System.out.println(bytesSent);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (channel != null) {
                    channel.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 
public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(2);
        exec.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                while (true){
                    recive();
                }
            }
        });
        exec.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                while (true){
                    send();
                }
            }
        });
 
    }
```

## TCP的NIO实现

NIO的强大功能部分来自于Channel的非阻塞特性，套接字的某些操作可能会无限期地阻塞。例如，对accept()方法的调用可能会因为等待一个客户端连接而阻塞；对read()方法的调用可能会因为没有数据可读而阻塞，直到连接的另一端传来新的数据。总的来说，创建/接收连接或读写数据等I/O调用，都可能无限期地阻塞等待。慢速的，有损耗的网络，或仅仅是简单的网络故障都可能导致任意时间的延迟。然而不幸的是，在调用一个方法之前无法知道其是否阻塞。NIO的channel抽象的一个重要特征就是可以通过配置它的阻塞行为，以实现非阻塞式的信道。

```
channel.configureBlocking(false)
```

在非阻塞式信道上调用一个方法总是会立即返回。这种调用的返回值指示了所请求的操作完成的程度。例如，在一个非阻塞式ServerSocketChannel上调用accept()方法，如果有连接请求来了，则返回客户端SocketChannel，否则返回null。

　　Selector类可以用于避免使用阻塞式客户端中很浪费资源的“忙等”方法。例如，考虑一个IM服务器。像QQ或者旺旺这样的，可能有几万甚至几千万个客户端同时连接到了服务器，但在任何时刻都只是非常少量的消息。

　　这就需要一种方法阻塞等待，直到至少有一个信道可以进行I/O操作，并指出是哪个信道。NIO的选择器就实现了这样的功能。
　　一个Selector实例可以同时检查一组信道的I/O状态，它像是一个多路开关选择器，因为一个选择器能够管理多个信道上的I/O操作。
　　如果用传统的方式来处理众多客户端，使用的方法是循环地一个一个地去检查所有的客户端是否有I/O操作，如果当前客户端有I/O操作，则可能把当前客户端扔给一个线程池去处理，如果没有I/O操作则进行下一个轮询，当所有的客户端都轮询过了又接着从头开始轮询。这种方法是非常耗时而且浪费资源的，因为大部分客户端是没有I/O操作的。

　　Selector就不一样了，它在内部可以同时管理多个I/O，当一个信道有I/O操作的时候，它会通知Selector，于是，Selector就记住这个信道有I/O操作，并且知道是何种I/O操作（读，写，或者接受新的连接）。因此，如果使用Selector，它返回的结果只有两种，一种是0，即在你调用的时刻没有任何客户端需要I/O操作；另一种结果是一组需要I/O操作的客户端。这种通知的方式比主动轮询的方式要高效得多。

```

NIO的通信步骤：
①创建ServerSocketChannel，为其配置非阻塞模式。
②绑定监听，配置TCP参数，录入backlog大小等。
③创建一个独立的IO线程，用于轮询多路复用器Selector。
④创建Selector，将之前创建的ServerSocketChannel注册到Selector上，并设置监听标识位SelectionKey.OP_ACCEPT。
⑤启动IO线程，在循环体中执行Selector.select()方法，轮询就绪的通道。
⑥当轮询到处于就绪状态的通道时，需要进行操作位判断，如果是ACCEPT状态，说明是新的客户端接入，则调用accept方法接收新的客户端。
⑦设置新接入客户端的一些参数，如非阻塞，并将其继续注册到Selector上，设置监听标识位等。
⑧如果轮询的通道标识位是READ，则进行读取，构造Buffer对象等。
⑨更细节的问题还有数据没发送完成继续发送的问题...... 
```

要使用选择器（Selector），需要创建一个Selector实例（使用静态工厂方法open()）并将其注册（register）到想要监控的信道上（注意，这要通过channel的方法实现，而不是使用selector的方法）。最后，调用选择器的select()方法。该方法会阻塞等待，直到有一个或更多的信道准备好了I/O操作或等待超时。select()方法将返回可进行I/O操作的信道数量。现在，在一个单独的线程中，通过调用select()方法就能检查多个信道是否准备好进行I/O操作。如果经过一段时间后仍然没有信道准备好，select()方法就会返回0，并允许程序继续执行其他任务。　　

```
public class TCPNIOtest {
    private static final int BUF_SIZE = 1024;
    private static final int PORT = 8080;
    private static final int TIMEOUT = 3000;
 
    public static void main(String[] args) {
        ExecutorService exec = Executors.newFixedThreadPool(2);
        /*exec.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                client();
                return null;
            }
        });*/
        exec.submit(new Callable<Object>() {
            @Override
            public Object call() throws Exception {
                serverSelector();
                return null;
            }
        });
 
        /*Callable<String> callable = new Callable<String>() {
            @Override
            public String call() throws Exception {
                return "haha";
            }
        };
        try {
            String call = callable.call();
            System.out.println("call = " + call);
        } catch (Exception e) {
            e.printStackTrace();
        }*/
 
    }
 
    static void client() {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        SocketChannel socketChannel = null;
        try {
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);
            socketChannel.connect(new InetSocketAddress("127.0.0.1", PORT));
 
            if (socketChannel.finishConnect()) {
                int i = 0;
                while (true) {
                    TimeUnit.SECONDS.sleep(1);
                    String info = "I'm " + i++ + "-th information from client";
                    buffer.clear();
                    buffer.put(info.getBytes());
                    buffer.flip();
                    while (buffer.hasRemaining()) {
                        System.out.println(buffer);
                        socketChannel.write(buffer);
                    }
                }
            }
        } catch (IOException | InterruptedException e) {
            e.printStackTrace();
        } finally {
            try {
                if (socketChannel != null) {
                    socketChannel.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
 
    public static void handleAccept(SelectionKey key) throws IOException {
        ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();
        SocketChannel sc = ssChannel.accept();
        sc.configureBlocking(false);
        sc.register(key.selector(), SelectionKey.OP_READ, ByteBuffer.allocateDirect(BUF_SIZE));
    }
 
    public static void handleRead(SelectionKey key) throws IOException {
        SocketChannel sc = (SocketChannel) key.channel();
        ByteBuffer buf = (ByteBuffer) key.attachment();
        long bytesRead = sc.read(buf);
        while (bytesRead > 0) {
            buf.flip();
            while (buf.hasRemaining()) {
                System.out.print((char) buf.get());
            }
            System.out.println();
            buf.clear();
            bytesRead = sc.read(buf);
        }
        if (bytesRead == -1) {
            sc.close();
        }
    }
 
    public static void handleWrite(SelectionKey key) throws IOException {
        ByteBuffer buf = (ByteBuffer) key.attachment();
        buf.flip();
        SocketChannel sc = (SocketChannel) key.channel();
        while (buf.hasRemaining()) {
            sc.write(buf);
        }
        buf.compact();
    }
 
    public static void serverSelector() {
        Selector selector = null;
        ServerSocketChannel ssc = null;
        try {
            selector = Selector.open();
            ssc = ServerSocketChannel.open();
            ssc.socket().bind(new InetSocketAddress(PORT));
            ssc.configureBlocking(false);
            ssc.register(selector, SelectionKey.OP_ACCEPT);
 
            while (true) {
                if (selector.select(TIMEOUT) == 0) {
                    System.out.println("==");
                    continue;
                }
                Iterator<SelectionKey> iter = selector.selectedKeys().iterator();
                while (iter.hasNext()) {
                    SelectionKey key = iter.next();
                    if (key.isAcceptable()) {
                        handleAccept(key);
                    }
                    if (key.isReadable()) {
                        handleRead(key);
                    }
                    if (key.isWritable() && key.isValid()) {
                        handleWrite(key);
                    }
                    if (key.isConnectable()) {
                        System.out.println("isConnectable = true");
                    }
                    iter.remove();
                }
            }
 
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (selector != null) {
                    selector.close();
                }
                if (ssc != null) {
                    ssc.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

总的来说，NIO主要通过通道和缓冲区来操作数据，期间使用Native函数库直接分配堆外内存（这个堆外内存不在JVM虚拟机定义的内存范围），然后通过一个存储于Java堆中的DirectByteBuffer对象作为这块内存的引用来对数据进行操作。

　　其本质就是阻塞和非阻塞的区别。

　　同步时，应用程序会直接参与IO读写操作,并且我们的应用程序会直接阻塞到某一个方法上,直到数据准备就绪；或者采用轮训的策略实时检查数据的就绪状态,如果就绪则获取数据.
　　异步时,则所有的IO读写操作交给操作系统,与我们的应用程序没有直接关系，我们程序不需要关系IO读写，当操作系统完成了IO读写操作时,会给我们应用程序发送通知,我们的应用程序直接拿走数据极即可。　

　　3.AIO(proactor模型)：异步非阻塞式IO，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。

　　与NIO不同，当进行读写操作时，只须直接调用API的read或write方法即可。这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，并通知应用程序；对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序。  即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数。

　[Java](http://lib.csdn.net/base/javase).nio.channels包下增加了下面四个异步信道：

```
AsynchronousSocketChannel
AsynchronousServerSocketChannel
AsynchronousFileChannel
AsynchronousDatagramChannel
```

　总的来说，BIO，NIO，AIO可以简述如下：

　BIO是同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。

　　NIO是同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。

　　AIO是异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。

 BIO、NIO、AIO适用场景分析:

```
BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，是JDK1.4以前的唯一选择，程序直观简单易理解。

NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。

AIO方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。
```

一般来说，I/O属于底层操作，需要操作系统支持，并发也需要操作系统的支持，所以性能方面不同的操作系统差异会比较明显。在高性能的I/O设计中，有两个比较著名的模式Reactor和Proactor模式，其中Reactor模式用于同步I/O，而Proactor运用于异步I/O操作。

　　Reactor 和 Proactor 是基于事件驱动，在网络编程中经常用到这两种设计模式。

　　Reactor，即反应堆。Reactor 的一般工作过程是首先在 Reactor 中注册（Reactor）感兴趣事件，并在注册时指定某个已定义的回调函数（callback）；当客户端发送请求时，在 Reactor 中会触发刚才注册的事件，并调用对应的处理函数。

Reactor包含如下角色：

```
Handle 句柄；用来标识socket连接或是打开文件；
Synchronous Event Demultiplexer：同步事件多路分解器：由操作系统内核实现的一个函数；用于阻塞等待发生在句柄集合上的一个或多个事件；
Event Handler：事件处理接口
Concrete Event HandlerA：实现应用程序所提供的特定事件处理逻辑；
Reactor：反应器，定义一个接口，实现以下功能：
1）供应用程序注册和删除关注的事件句柄；
2）运行事件循环；
3）有就绪事件到来时，分发事件到之前注册的回调函数上处理。
```

Proactor借助操作系统的异步读写，在调用的时候可以传递回调函数或者回送信号，当异步操作完毕，内核会自动调用回调函数或者发送信号，所以很依赖操作系统。

Proactor模式包含如下角色

```
Handle 句柄；用来标识socket连接或是打开文件；
Asynchronous Operation Processor：异步操作处理器；负责执行异步操作，一般由操作系统内核实现；
Asynchronous Operation：异步操作
Completion Event Queue：完成事件队列；异步操作完成的结果放到队列中等待后续使用
Proactor：主动器；为应用程序进程提供事件循环；从完成事件队列中取出异步操作的结果，分发调用相应的后续处理逻辑；
Completion Handler：完成事件接口；一般是由回调函数组成的接口；
Concrete Completion Handler：完成事件处理逻辑；实现接口定义特定的应用处理逻辑；
```

Reactor与Proactor对比：

```
（1）以主动写为例：
　　Reactor将handle放到select()，等待可写就绪，然后调用write()写入数据；写完处理后续逻辑；
　　Proactor调用aoi_write后立刻返回，由内核负责写操作，写完后调用相应的回调函数处理后续逻辑。

　　可以看出，Reactor被动的等待指示事件的到来并做出反应；它有一个等待的过程，做什么都要先放入到监听事件集合中等待handler可用时再进行操作；。
　　Proactor直接调用异步读写操作，调用完后立刻返回。

　　（2）Reactor实现了一个被动的事件分离和分发模型，服务等待请求事件的到来，再通过不受间断的同步处理事件，从而做出反应。

　　Proactor实现了一个主动的事件分离和分发模型；这种设计允许多个任务并发的执行，从而提高吞吐量；并可执行耗时长的任务（各个任务间互不影响）。

　　（3）优点：

　　Reactor实现相对简单，对于耗时短的处理场景处理高效； 操作系统可以在多个事件源上等待，并且避免了多线程编程相关的性能开销和编程复杂性； 事件的串行化对应用是透明的，可以顺序的同步执行而不需要加锁； 在事务分离上，可以将与应用无关的多路分解和分配机制和与应用相关的回调函数分离开来。

　　Proactor性能更高，能够处理耗时长的并发场景。

　　（4）缺点：

　　Reactor处理耗时长的操作会造成事件分发的阻塞，影响到后续事件的处理。

　　Proactor实现逻辑复杂；依赖操作系统对异步的支持，目前实现了纯异步操作的操作系统少，实现优秀的如windows IOCP，但由于其windows系统用于服务器的局限性，目前应用范围较小；而Unix/Linux系统对纯异步的支持有限，应用事件驱动的主流还是通过select/epoll来实现。

　　（5）适用场景。

　　Reactor：同时接收多个服务请求，并且依次同步的处理它们的事件驱动程序；。
　　Proactor：异步接收和同时处理多个服务请求的事件驱动程序。
```


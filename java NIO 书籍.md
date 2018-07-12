> 
>
> 这并不是说使用传统的 I/O 模型无法移动大量数据——当然可以(现在依然可以)。具体地说，RandomAccessFile 类在这方面的效率就不低，只要坚持使用基于数组的 read( )和 write( )方法。这些方法与底层操作系统调用相当接近，尽管必须保留至少一份缓冲区拷贝。
>
> 
>
> 传统的文件 I/O 是通过用户进程发布 read( )和 write( )系统调用来传输数据的。为了在内核空间的文件系统页与用户空间的内存区之间移动数据，一次以上的拷贝操作几乎总是免不了的。这是因为，在文件系统页与用户缓冲区之间往往没有一一对应关系。但是，还有一种大多数操作系统都支持的特殊类型的 I/O 操作，允许用户进程最大限度地利用面向页的系统 I/O 特性，并完全摒弃缓冲区拷贝。这就是内存映射 I/O，如图 1-6 所示。
>
> 
>
> 通道引入了一些与关闭和中断有关的新行为。如果一个通道实现 InterruptibleChannel 接口(参见图 3-2)，它的行为以下述语义为准:如果一个线程在一个通道上被阻塞并且同时被中断(由调用该被阻塞线程的 interrupt( )方法的另一个线程中断)，那么该通道将被关闭，该被阻塞线程也会产生一个 ClosedByInterruptException 异常。
>
> 此外，假如一个线程的 interrupt status 被设置并且该线程试图访问一个通道，那么这个通道将立即被关闭，同时将抛出相同的 ClosedByInterruptException 异常。线程的 interrupt status 在线程的interrupt( )方法被调用时会被设置。我们可以使用 isInterrupted( )来测试某个线程当前的 interruptstatus。当前线程的 interrupt status 可以通过调用静态的 Thread.interrupted( )方法清除。
>
> Character set 字符集 字符的集合
>
> coded character set 编码字符集  一个数值赋给一个字符的集合
>
> Character-encoding scheme(字符编码方案)  编码字符集成员到八位字节(8 bit 字节)的映射。编码方案定义了如何把字符编 码的序列表达为字节序列。字符编码的数值不需要与编码字节相同，也不需要是 一对一或一对多个的关系。原则上，把字符集编码和解码近似视为对象的序列化 和反序列化。
>
> Charset(字符集)
> 术语 charset 是在 RFC2278(http://ietf.org/rfc/rfc2278.txt)中定义的。它是编码字符集 和字符编码方案的集合。java.nio.charset 包的锚类是 Charset，它封装字符 集抽取。
>
> 字符集名称不区分大小写，也就是，当比较字符集名称时认为大写字母和小写字母相同。
>
> 

​			
​	

### Buffer	

>buffer API
>
>```java
>abstract Object array()
>abstract int    arrayOffset()
>abstract boolean	isDirect()
>abstract boolean	hasArray()    
>int capacity()
>Buffer clear()
>Buffer  flip()
>boolean	hasRemaining()
>abstract boolean	isReadOnly()
>int	limit()
>Buffer	limit(int newLimit)
>Buffer	mark()
>int	position()
>Buffer	position(int newPosition)
>int	remaining()
>Buffer	reset()
>Buffer	rewind()
>```
>
>

### ByteBuffer

>ByteBuffet API
>
>````java
>//创建buffer
>static ByteBuffer	allocate(int capacity)  //创建一个buffer
>static ByteBuffer	allocateDirect(int capacity) //创建一个内存映射buffer
>
>abstract CharBuffer	asCharBuffer()
>abstract DoubleBuffer	asDoubleBuffer()
>abstract FloatBuffer	asFloatBuffer()
>abstract IntBuffer	asIntBuffer()
>abstract LongBuffer	asLongBuffer()
>abstract ByteBuffer	asReadOnlyBuffer()
>abstract ShortBuffer	asShortBuffer()
>    
>abstract byte	get()
>   //bulk 读取buffer中元素
>ByteBuffer	get(byte[] dst)
>ByteBuffer	get(byte[] dst, int offset, int length)
>abstract byte	get(int index)
>abstract char	getChar()
>abstract char	getChar(int index)
>abstract double	getDouble()
>abstract double	getDouble(int index)
>abstract float	getFloat()
>abstract float	getFloat(int index)
>abstract int	getInt()
>abstract int	getInt(int index)
>abstract long	getLong()
>abstract long	getLong(int index)
>abstract short	getShort()
>abstract short	getShort(int index)
>  
>abstract ByteBuffer	put(byte b)
>ByteBuffer	put(byte[] src)
>ByteBuffer	put(byte[] src, int offset, int length)
>ByteBuffer	put(ByteBuffer src)
>abstract ByteBuffer	put(int index, byte b)
>abstract ByteBuffer	putChar(char value)
>abstract ByteBuffer	putChar(int index, char value)
>abstract ByteBuffer	putDouble(double value)
>abstract ByteBuffer	putDouble(int index, double value)
>abstract ByteBuffer	putFloat(float value)
>abstract ByteBuffer	putFloat(int index, float value)
>abstract ByteBuffer	putInt(int value)
>abstract ByteBuffer	putInt(int index, int value)
>abstract ByteBuffer	putLong(int index, long value)
>abstract ByteBuffer	putLong(long value)
>abstract ByteBuffer	putShort(int index, short value)
>abstract ByteBuffer	putShort(short value)
>    
>//浅拷贝当前的buffer
>abstract ByteBuffer	duplicate()
>boolean	equals(Object ob)
>abstract ByteBuffer	slice()
>abstract ByteBuffer	compact()   //压缩这个buffer
>boolean	hasArray()   //测试当前Bytebuffer是否拷贝到一个数组
>byte[]	array()    //返回当前buffer的拷贝
>int	arrayOffset()  //返回buffer中第一个元素的偏移量，也就是第一个可读的元素
>abstract boolean	isDirect()  //测试buffer是否是direct
>ByteOrder	order()
>Retrieves this buffer's byte order.
>ByteBuffer	order(ByteOrder bo)
>Modifies this buffer's byte order.
>int	hashCode()
>int	compareTo(ByteBuffer that)
>String	toString()
>    //利用byte类型数组创建byteBuffer
>static ByteBuffer	wrap(byte[] array)
>static ByteBuffer	wrap(byte[] array, int offset, int length)
>````
>
>

### Charset

>
>
>```Java
>//构建一个charset
>static Charset	defaultCharset()    //JVM 默认的charset
>static Charset	forName(String charsetName)  //返回指定的charset
>    
>    
>static SortedMap<String,Charset>	availableCharsets() //返回标准的charset集合
>static boolean	isSupported(String charsetName) //测试是否支持当前编码
>
>Set<String>	aliases()   //返回当前编码集的别名
>boolean	canEncode()     //测试当前charset是否支持encode
>
>
>abstract boolean	contains(Charset cs) //测试是否当前charset是否包含指定的charset    
>String	displayName()
>String	displayName(Locale locale)
>String	name()  //Returns this charset's canonical name.  
>  
>//Convenience method that decodes bytes in this charset into Unicode characters.
>CharBuffer	decode(ByteBuffer bb)
>//Convenience method that encodes Unicode characters into bytes in this charset.
>ByteBuffer	encode(CharBuffer cb)
>//Convenience method that encodes a string into bytes in this charset.
>ByteBuffer	encode(String str)
>  
>abstract CharsetDecoder	newDecoder() //Constructs a new decoder for this charset.
>abstract CharsetEncoder	newEncoder() //Constructs a new encoder for this charset.
>    
>    
>int	compareTo(Charset that)  //比较俩个charset是否相同
>//Tells whether or not this charset is registered in the IANA Charset Registry.
>boolean	isRegistered()
>String	toString()
>
>```
>
>
>
>

### CharEncode

>
>
>````java
>protected	CharsetEncoder(Charset cs, float averageBytesPerChar,
>                           float maxBytesPerChar)
>
>protected	CharsetEncoder(Charset cs, float averageBytesPerChar, 
>                           float maxBytesPerChar, byte[] replacement)
>
>
>
>float	averageBytesPerChar()  //返回平均每个字符的字节数
>
>boolean	canEncode(char c)
>boolean	canEncode(CharSequence cs)
>    
>Charset	charset()
>Returns the charset that created this encoder.
>ByteBuffer	encode(CharBuffer in)
>Convenience method that encodes the remaining content of a single input character buffer into a newly-allocated byte buffer.
>CoderResult	encode(CharBuffer in, ByteBuffer out, boolean endOfInput)
>Encodes as many characters as possible from the given input buffer, writing the results to the given output buffer.
>protected abstract CoderResult	encodeLoop(CharBuffer in, ByteBuffer out)
>Encodes one or more characters into one or more bytes.
>CoderResult	flush(ByteBuffer out)
>Flushes this encoder.
>protected CoderResult	implFlush(ByteBuffer out)
>Flushes this encoder.
>protected void	implOnMalformedInput(CodingErrorAction newAction)
>Reports a change to this encoder's malformed-input action.
>protected void	implOnUnmappableCharacter(CodingErrorAction newAction)
>Reports a change to this encoder's unmappable-character action.
>protected void	implReplaceWith(byte[] newReplacement)
>Reports a change to this encoder's replacement value.
>protected void	implReset()
>Resets this encoder, clearing any charset-specific internal state.
>boolean	isLegalReplacement(byte[] repl)
>Tells whether or not the given byte array is a legal replacement value for this encoder.
>CodingErrorAction	malformedInputAction()
>Returns this encoder's current action for malformed-input errors.
>float	maxBytesPerChar()
>Returns the maximum number of bytes that will be produced for each character of input.
>CharsetEncoder	onMalformedInput(CodingErrorAction newAction)
>Changes this encoder's action for malformed-input errors.
>CharsetEncoder	onUnmappableCharacter(CodingErrorAction newAction)
>Changes this encoder's action for unmappable-character errors.
>byte[]	replacement()
>Returns this encoder's replacement value.
>CharsetEncoder	replaceWith(byte[] newReplacement)
>Changes this encoder's replacement value.
>CharsetEncoder	reset()
>Resets this encoder, clearing any internal state.
>CodingErrorAction	unmappableCharacterAction()
>````
>
>
>
>

### 直接缓冲区

>字节缓冲区跟其他缓冲区类型最明显的不同在于，它们可以成为通道所执行的 I/O 的源 头和/或目标。如果您向前跳到第三章(喂!喂!)，您会发现通道只接收 ByteBuffer 作为 参数。 
>
>如我们在第一章中所看到的那样，操作系统的在内存区域中进行 I/O 操作。这些内存区 域，就操作系统方面而言，是相连的字节序列。于是，毫无疑问，只有字节缓冲区有资格参与 I/O 操作。也请回想一下操作系统会直接存取进程——在本例中是 JVM 进程的内存空间，以 传输数据。这也意味着 I/O 操作的目标内存区域必须是连续的字节序列。在 JVM 中，字节数 组可能不会在内存中连续存储，或者无用存储单元收集可能随时对其进行移动。在 Java 中， 数组是对象，而数据存储在对象中的方式在不同的 JVM 实现中都各有不同。 
>
>出于这一原因，引入了直接缓冲区的概念。直接缓冲区被用于与通道和固有 I/O 例程交 互。它们通过使用固有代码来告知操作系统直接释放或填充内存区域，对用于通道直接或原始 存取的内存区域中的字节元素的存储尽了最大的努力。 
>
>直接字节缓冲区通常是 I/O 操作最好的选择。在设计方面，它们支持 JVM 可用的最高效 I/O 机制。非直接字节缓冲区可以被传递给通道，但是这样可能导致性能损耗。通常非直接缓 冲不可能成为一个本地 I/O 操作的目标。如果您向一个通道中传递一个非直接 ByteBuffer 对象用于写入，通道可能会在每次调用中隐含地进行下面的操作: 
>
>1. 创建一个临时的直接 ByteBuffer 对象。 
>2. 将非直接缓冲区的内容复制到临时缓冲中。 
>3. 使用临时缓冲区执行低层次 I/O 操作。 
>4. 临时缓冲区对象离开作用域，并最终成为被回收的无用数据。 
>
>这可能导致缓冲区在每个 I/O 上复制并产生大量对象，而这种事都是我们极力避免的。 不过，依靠工具，事情可以不这么糟糕。运行时间可能会缓存并重新使用直接缓冲区或者执行 其他一些聪明的技巧来提高吞吐量。如果您仅仅为一次使用而创建了一个缓冲区，区别并不是 很明显。另一方面，如果您将在一段高性能脚本中重复使用缓冲区，分配直接缓冲区并重新使 用它们会使您游刃有余。 
>
>直接缓冲区时 I/O 的最佳选择，但可能比创建非直接缓冲区要花费更高的成本。直接缓 冲区使用的内存是通过调用本地操作系统方面的代码分配的，绕过了标准 JVM 堆栈。建立和 销毁直接缓冲区会明显比具有堆栈的缓冲区更加破费，这取决于主操作系统以及 JVM 实现。 直接缓冲区的内存区域不受无用存储单元收集支配，因为它们位于标准 JVM 堆栈之外。 
>
>使用直接缓冲区或非直接缓冲区的性能权衡会因JVM，操作系统，以及代码设计而产生巨 大差异。通过分配堆栈外的内存，您可以使您的应用程序依赖于JVM未涉及的其它力量。当加 入其他的移动部分时，确定您正在达到想要的效果。我以一条旧的软件行业格言建议您:先使 其工作，再加快其运行。不要一开始就过多担心优化问题;首先要注重正确性。JVM实现可能 会执行缓冲区缓存或其他的优化，5这会在不需要您参与许多不必要工作的情况下为您提供所 需的性能。 
>
>直接 ByteBuffer 是通过调用具有所需容量的 ByteBuffer.allocateDirect()函数 产生的，就像我们之前所涉及的 allocate()函数一样。注意用一个 wrap()函数所创建的被 包装的缓冲区总是非直接的。 
>
>```java
> public abstract class ByteBuffer
>        extends Buffer implements Comparable
> {
>        // This is a partial API listing
>public static ByteBuffer allocate (int capacity) 
>public static ByteBuffer allocateDirect (int capacity) 
>public abstract boolean isDirect( );
>
>}
>```
>
>
>
>所有的缓冲区都提供了一个叫做 isDirect()的 boolean 函数，来测试特定缓冲区是否 为直接缓冲区。虽然 ByteBuffer 是唯一可以被直接分配的类型，但如果基础缓冲区是一个 直接 ByteBuffer，对于非字节视图缓冲区，isDirect()可以是 true。这将我们带到 了...... 

### 内存映射缓冲区

>映射缓冲区是带有存储在文件，通过内存映射来存取数据元素的字节缓冲区。映射缓冲区 通常是直接存取内存的，只能通过 FileChannel 类创建。映射缓冲区的用法和直接缓冲区类 似，但是 MappedByteBuffer 对象可以处理独立于文件存取形式的的许多特定字符。出于这 个原因，我将讨论映射缓冲区的内容留到 3.4 节，在那里我们也会讨论文件锁。 

### channel

>Channel 是最顶层的接口，对所有通道来说只有两种共同的操作:检查一个通道是否 打开(IsOpen())和关闭一个打开的通道(close())。 
>
>InterruptibleChannel 是一个标记接口，当被通道使用时可以标示该通道是可以中断的 (Interruptible)。 
>
>通道分为File I/O 和 Stream I/O ，通道有的最后实现 FileChannel 和 SocketChannel，ServerSocketChannel 和DatagramChannel
>
>如果一个线程在某个文件上获得了一个独占锁，然后第二个线程利用一个单独打开的通道来请 求该文件的独占锁，那么第二个线程的请求会被批准。但如果这两个线程运行在不同的 Java 虚拟 机上，那么第二个线程会阻塞，因为锁最终是由操作系统或文件系统来判优的并且几乎总是在进程 级而非线程级上判优。锁都是与一个文件关联的，而不是与单个的文件句柄或通道关联。 
>
>锁与文件关联，而不是与通道关联。我们使用锁来判优外部进程，而不是判 优同一个 Java 虚拟机上的线程。 
>
>文件锁旨在在进程级别上判优文件访问，比如在主要的程序组件之间或者在集成其他供应商的 组件时。如果您需要控制多个 Java 线程的并发访问，您可能需要实施您自己的、轻量级的锁定方 案。那种情形下，内存映射文件(本章后面会进行详述)可能是一个合适的选择。 

### selector

>这就是为什么传统的监控多个 socket 的 Java 解决方案是为每个 socket 创建一个线程并使得线 程可以在read( )调用中阻塞，直到数据可用。这事实上将每个被阻塞的线程当作了socket监控器， 并将 Java 虚拟机的线程调度当作了通知机制。这两者本来都不是为了这种目的而设计的。程序员 和 Java 虚拟机都为管理所有这些线程的复杂性和性能损耗付出了代价，这在线程数量的增长失控 时表现得更为突出。 
>
>真正的就绪选择必须由操作系统来做。操作系统的一项最重要的功能就是处理 I/O 请求并通知 各个线程它们的数据已经准备好了。选择器类提供了这种抽象，使得 Java 代码能够以可移植的方 式，请求底层的操作系统提供就绪选择服务。 
>
>选择器(Selector) 
>
>  选择器类管理着一个被注册的通道集合的信息和它们的就绪状态。通道是和选择器一起被注册
>的，并且使用选择器来更新通道的就绪状态。当这么做的时候，可以选择将被激发的线程挂起，直
>到有就绪的的通道。
>
>可选择通道(SelectableChannel) 
>
>这个抽象类提供了实现通道的可选择性所需要的公共方法。它是所有支持就绪检查的通道类的 父类。FileChannel 对象不是可选择的，因为它们没有继承 SelectableChannel(见图 4-2)。 所有 socket 通道都是可选择的，包括从管道(Pipe)对象的中获得的通道。SelectableChannel 可以被注册到 Selector 对象上，同时可以指定对 那个选择器而言，那种操作是感兴趣的。一个通道可以被注册到多个选择器上，但对每个选择器而 言只能被注册一次。 
>
>选择键(SelectionKey) 
>
>选择键封装了特定的通道与特定的选择器的注册关系。选择键对象被 SelectableChannel.register( ) 返回并提供一个表示这种注册关系的标记。选择键包含了 两个比特集(以整数的形式进行编码)，指示了该注册关系所关心的通道操作，以及通道已经准备 好的操作。
>
> 
>
>已注册的键的集合(Registered key set) 
>
>与选择器关联的已经注册的键的集合。并不是所有注册过的键都仍然有效。这个集合通过 keys( )方法返回，并且可能是空的。这个已注册的键的集合不是可以直接修改的;试图这么做的话 将引 java.lang.UnsupportedOperationException。 
>
>已选择的键的集合(Selected key set) 
>
>已注册的键的集合的子集。这个集合的每个成员都是相关的通道被选择器(在前一个选择操作 中)判断为已经准备好的，并且包含于键的interest集合中的操作。这个集合通过selectedKeys( )方 法返回(并有可能是空的)。 
>
>不要将已选择的键的集合与 ready 集合弄混了。这是一个键的集合，每个键都关联一个已经准 备好至少一种操作的通道。每个键都有一个内嵌的 ready 集合，指示了所关联的通道已经准备好的 操作。 
>
>键可以直接从这个集合中移除，但不能添加。试图向已选择的键的集合中添加元素将抛出 java.lang.UnsupportedOperationException。 
>
>
>
>已取消的键的集合(Cancelled key set) 
>
>已注册的键的集合的子集，这个集合包含了 cancel( )方法被调用过的键(这个键已经被无效 化)，但它们还没有被注销
> 。这个集合是选择器对象的私有成员，因而无法直接访问。 
>
>在一个刚初始化的 Selector 对象中，这三个集合都是空的。 
>
>Selector 类的核心是选择过程。这个名词您已经在之前看过多次了——现在应该解释一下 了。基本上来说，选择器是对select( )、poll( )等本地调用(native call)或者类似的操作系统特定的系 统调用的一个包装。但是 Selector 所作的不仅仅是简单地向本地代码传送参数。它对每个选择 操作应用了特定的过程。对这个过程的理解是合理地管理键和它们所表示的状态信息的基础。 
>
>选择操作是当三种形式的 select( )中的任意一种被调用时，由选择器执行的。不管是哪一种形 式的调用，下面步骤将被执行: 
>
>1.已取消的键的集合将会被检查。如果它是非空的，每个已取消的键的集合中的键将从另外两 个集合中移除，并且相关的通道将被注销。这个步骤结束后，已取消的键的集合将是空的。 
>
>2.已注册的键的集合中的键的 interest 集合将被检查。在这个步骤中的检查执行过后，对 interest 集合的改动不会影响剩余的检查过程。 
>
>一旦就绪条件被定下来，底层操作系统将会进行查询，以确定每个通道所关心的操作的真实就 绪状态。依赖于特定的 select( )方法调用，如果没有通道已经准备好，线程可能会在这时阻塞，通常会有一个超时值。直到系统调用完成为止，这个过程可能会使得调用线程睡眠一段时间，然后当前每个通道的就 绪状态将确定下来。对于那些还没准备好的通道将不会执行任何的操作。对于那些操作系统指示至少已经准备好 interest 集合中的一种操作的通道，将执行以下两种操作中的一种: 
>
>a.如果通道的键还没有处于已选择的键的集合中，那么键的 ready 集合将被清空，然后表示操作系统发现的当前通道已经准备好的操作的比特掩码将被设置。 
>
>b.否则，也就是键在已选择的键的集合中。键的 ready 集合将被表示操作系统发现的当前已经 准备好的操作的比特掩码更新。所有之前的已经不再是就绪状态的操作不会被清除。事实上，所有 的比特位都不会被清理。由操作系统决定的 ready 集合是与之前的 ready 集合按位分离的，一旦键 被放置于选择器的已选择的键的集合中，它的 ready 集合将是累积的。比特位只会被设置，不会被 清理。 
>
>3.步骤 2 可能会花费很长时间，特别是所激发的线程处于休眠状态时。与该选择器相关的键可 能会同时被取消。当步骤 2 结束时，步骤 1 将重新执行，以完成任意一个在选择进行的过程中，键 已经被取消的通道的注销。 
>
>4.select 操作返回的值是 ready 集合在步骤 2 中被修改的键的数量，而不是已选择的键的集合中 的通道的总数。返回值不是已准备好的通道的总数，而是从上一个 select( )调用之后进入就绪状态 的通道的数量。之前的调用中就绪的，并且在本次调用中仍然就绪的通道不会被计入，而那些在前 一次调用中已经就绪但已经不再处于就绪状态的通道也不会被计入。这些通道可能仍然在已选择的 键的集合中，但不会被计入返回值中。返回值可能是 0。 
>
>使用内部的已取消的键的集合来延迟注销，是一种防止线程在取消键时阻塞，并防止与正在进 行的选择操作冲突的优化。注销通道是一个潜在的代价很高的操作，这可能需要重新分配资源(请 记住，键是与通道相关的，并且可能与它们相关的通道对象之间有复杂的交互)。清理已取消的 键，并在选择操作之前和之后立即注销通道，可以消除它们可能正好在选择的过程中执行的潜在棘 手问题。这是另一个兼顾健壮性的折中方案。 
>
>Selector 类的 select( )方法有以下三种不同的形式: 
>
>这三种 select 的形式，仅仅在它们在所注册的通道当前都没有就绪时，是否阻塞的方面有所不 同。最简单的没有参数的形式可以用如下方式调用: 
>
>这种调用在没有通道就绪时将无限阻塞。一旦至少有一个已注册的通道就绪，选择器的选择键 就会被更新，并且每个就绪的通道的 ready 集合也将被更新。返回值将会是已经确定就绪的通道的 数目。正常情况下，这些方法将返回一个非零的值，因为直到一个通道就绪前它都会阻塞。但是它 也可以返回非 0 值，如果选择器的 wakeup( )方法被其他线程调用。 
>
>有时您会想要限制线程等待通道就绪的时间。这种情况下，可以使用一个接受一个超时参数的 select( )方法的重载形式: 
>
>这种调用与之前的例子完全相同，除了如果在您提供的超时时间(以毫秒计算)内没有通道就 绪时，它将返回 0。如果一个或者多个通道在时间限制终止前就绪，键的状态将会被更新，并且方 法会在那时立即返回。将超时参数指定为 0 表示将无限期等待，那么它就在各个方面都等同于使用 无参数版本的 select( )了。 
>
>就绪选择的第三种也是最后一种形式是完全非阻塞的:
>  int n = selector.selectNow( );
>
>selectNow()方法执行就绪检查过程，但不阻塞。如果当前没有通道就绪，它将立即返回 0。 








```
package java.util.concurrent;

import java.io.ObjectStreamField;
import java.io.Serializable;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.AbstractMap;
import java.util.Arrays;
import java.util.Collection;
import java.util.Comparator;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Hashtable;
import java.util.Iterator;
import java.util.Map;
import java.util.NoSuchElementException;
import java.util.Set;
import java.util.Spliterator;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.locks.LockSupport;
import java.util.concurrent.locks.ReentrantLock;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.BinaryOperator;
import java.util.function.Consumer;
import java.util.function.DoubleBinaryOperator;
import java.util.function.Function;
import java.util.function.IntBinaryOperator;
import java.util.function.LongBinaryOperator;
import java.util.function.ToDoubleBiFunction;
import java.util.function.ToDoubleFunction;
import java.util.function.ToIntBiFunction;
import java.util.function.ToIntFunction;
import java.util.function.ToLongBiFunction;
import java.util.function.ToLongFunction;
import java.util.stream.Stream;

/**
 * 支持检索的完全并发性和更新的高预期并发性的哈希表。
 * 这个类服从相同功能规范如Hashtable ，并且包括对应于每个方法的方法版本Hashtable 。 
 * 不过，尽管所有操作都是线程安全的，检索操作并不意味着锁定，并没有为防止所有访问的方式锁定整个表的任何支持。
 * 这个类可以在依赖于线程安全性的程序中提供和Hashtable完全相同的功能 ，但不依赖于其同步锁synchronization。
 *
 * 检索操作(包括{@code get})通常不会阻塞，所以可能会与更新操作(包括{@code put}和{@code remove})重叠。
 * 检索反映最近完成更新操作开始时的结果。(更正式地说，对给定键的更新操作执行happens-before关系
 * ，并对报告更新值的键进行任何(非空)检索。)
 * 对于诸如{@code putAll}和{@code clear}这样的聚合操作，并发检索可能只反映插入或删除某些条目。
 * 类似地，迭代器、分流器和枚举返回元素，反映哈希表在迭代器/枚举创建时或创建后的某个时刻的状态。
 * 他们做不抛出{@link java.util。ConcurrentModificationException ConcurrentModificationException}。
 * 然而，迭代器被设计成一次只被一个线程使用。{@code size}、{@code isEmpty}和
 * {@code containsValue}通常只在映射不在其他线程中进行并发更新时有用。
 * 否则，这些方法的结果反映的瞬态状态可能足以用于监视或估计目的，但不能用于程序控制。
 *
 * 当冲突太多时，表会被动态扩展(例如，键具有不同的哈希码，但落在同一个槽内，对表大小进行模调)，
 * 预期的平均效果是每个Map维持大约2个bin（bin是指数组的元素，也就是说map初始的数组大小为2）
 * (和对应0.75负载因子阈值调整大小对应)。
 * 随着Map的添加和删除，这个平均值可能会有很大的差异，但总的来说，对于哈希表来说，
 * 这维护了一个普遍接受的时间/空间折衷可能是一个相对缓慢的操作。
 * 如果可能，最好在创建Map对象的时候，根据预计存入键值对的数量，
 * 在构造函数中提供【initialCapacity】，以免Map反复的扩容操作
 * 另外一个可选的构造函数参数【loadFactor】通过指定用于计算为给定元素分配的空间量的表密度（负载因子），
 * 提供了进一步定制初始表容量的方法。
 * 另外，为了与这个类以前的版本兼容，构造函数可以选择指定预期的{@code concurrencyLevel}作为内部大小调整的额外提示。
 * 注意，使用完全相同的{@code hashCode()}的多个键肯定会降低任何散列表的性能。
 * 为了改善影响，当键为{@link Comparable}时，这个类可以使用键之间的比较顺序来帮助断开连接。
 *
 * ConcurrentHashMap可以通过函数newKeySet()和newKeySet(int)创建一个只含有Maop的key值的Set（KeySetView）结构数据，
 * 或者keySet(Object mappedValue)通过给定value值或者所有值等于这个value的key的Set（KeySetView）结构数据
 *
 * ConcurrentHashMap可以通过使用 java.util.concurrent.atomic.LongAdder值并通过computeIfAbsent进行初始化，
 * 将其用作可缩放的频率映射（直方图或多集的形式）。 例如，要向 
 * ConcurrentHashMap<String,LongAdder> freqs添加计数，
 * 可以使用freqs.computeIfAbsent(k -> new LongAdder()).increment(); 
 *
 * 这个类及其视图和迭代器实现了{@link Map}和{@link Iterator}接口的所有可选方法。
 *
 * 这个类不允许{@code null}作为键或值。
 *
 * ConcurrentHashMaps支持一组顺序和并行批量操作，与大多数Stream方法不同，
 * 它们被设计为安全并且经常明智地应用，即使是由其他线程同时更新的映射; 
 * 例如，当计算共享注册表中的值的快照摘要时。 
 * 有三种操作，每种具有四种形式，接受键，值，条目和（键，值）参数和/或返回值的函数。 
 * 由于ConcurrentHashMap的元素不以任何特定的方式排序，并且可能会在不同的并行执行中以不同的顺序进行处理，
 * 因此提供的函数的正确性不应取决于任何排序，也不应依赖于可能瞬时变化的任何其他对象或值计算进行中;
 * 除了每一个行动，理想情况下都是无副作用的。 对Map.Entry对象的批量操作不支持方法setValue 。 
 *
 * forEach：对每个元素执行给定的操作。 变量形式在执行操作之前对每个元素应用给定的变换。 
 * search：返回在每个元素上应用给定函数的第一个可用非空结果; 当找到结果时跳过进一步的搜索。 
 * reduce：累积每个元素。 提供的减少功能不能依赖于排序（更正式地，它应该是关联和交换）。 有五种变体： 
 *      平原减少 （由于没有相应的返回类型，因此（key，value）函数参数没有这种方法的形式） 
 *      映射的减少积累了应用于每个元素的给定函数的结果。 
 *      使用给定的基础值减少到标量双，长和int。 
 *
 * 这些批量操作接受一个parallelismThreshold参数。 如果估计当前地图大小小于给定阈值，则方法依次进行。
 * 使用Long.MAX_VALUE的值Long.MAX_VALUE抑制所有的并行性。 使用1的值可以通过划分为足够的子
 * 任务来完全利用用于所有并行计算的ForkJoinPool.commonPool()来实现最大并行度。 
 * 通常，您最初将选择其中一个极值，然后测量使用中间值之间的性能，从而降低开销与吞吐量之间的关系。 
 * 
 * 批量操作的并发属性遵循ConcurrentHashMap的并发属性：
 * 从get(key)返回的任何非空结果和相关的访问方法与相关的插入或更新都有一个发生之前的关系。 
 * 任何批量操作的结果都反映了这些每个元素关系的组成（但是除非以某种方式已知是静态的，它们并不一定是相对于整个地图的原子）。
 * 相反，因为映射中的键和值从不为空，所以null作为目前缺乏任何结果的可靠原子指标。 
 * 为了保持此属性，null用作所有非标量缩减操作的隐含基础。
 * 对于double，long和int版本，基础应该是当与任何其他值组合时返回其他值（更正式地，它应该是减少的标识元素）。 
 * 大多数常见的减少具有这些属性; 例如，使用基数0或最小值与基准MAX_VALUE计算和。 
 * 
 * 作为参数提供的搜索和转换函数应该类似地返回null以指示缺少任何结果（在这种情况下不被使用）。
 * 在映射缩减的情况下，这也使得转换可以用作过滤器，如果不应该组合元素，返回null（或者在原始专业化的情况下，身份基础）。
 * 在使用它们进行搜索或减少操作之前，您可以通过在“null意味着现在没有任何内容”规则下自行构建复合转换和过滤。 
 * 
 * 接受和/或返回Entry参数的方法维护键值关联。 例如，当找到最大价值的钥匙时，它们可能是有用的。 
 * 请注意，可以使用new AbstractMap.SimpleEntry(k,v)提供“plain”Entry new AbstractMap.SimpleEntry(k,v) 。 
 * 
 * 批量操作可能突然完成，抛出在应用程序中遇到的异常。 
 * 在处理这样的异常时，请注意，其他并发执行的函数也可能引发异常，或者如果没有发生第一个异常，则会这样做。 
 * 
 * 与顺序形式相比，加速比是常见的，但不能保证。 如果并行计算的基础工作比计算本身更昂贵，
 * 则涉及小地图上的简短功能的并行操作可能比顺序形式执行得更慢。
 * 类似地，如果所有处理器正忙于执行不相关的任务，并行化可能不会导致太多的实际并行。 
 * 
 * 所有任务方法的所有参数都必须为非空值。 
 */
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
    private static final long serialVersionUID = 7249069246763182397L;

    /*
     * Overview:
     *
     * 这个哈希表的主要设计目标是在最小化更新争用同步锁导致阻塞的同时保持并发可读性(通常是方法get()，也包括迭代器和相关方法)。
     * 次要目标是保持与java.util.HashMap相同或更好的空间消耗，并支持在空表上由多个线程执行的高初始插入率。
     *
     * 这个映射通常充当一个bined（bucked）哈希表。
     * 每个键值映射都保存在一个Node对象中。大多数Node对象是具有hash、key、value和next字段的基本Node类的实例。
     * 如果相同hash的key太多，Node结构的链路存储会被红黑树替代TreeNodes
     * TreeBins含有一系列TreeNodes的root。
     * 在调整大小时，转发节点被放置在箱子的头部。
     * 在computeIfAbsent和相关方法中建立值时，ReservationNodes用作占位符。
     * TreeBin、ForwardingNode和ReservationNode类型不包含正常的key、value或hash，
     * 并且在搜索等过程中很容易区分，因为它们具有负哈希字段和空的键和值字段。
     * （这些特殊节点不是不常见的，就是暂时的，所以携带一些未使用的字段的影响是微不足道的。）
     *
     * 在第一次插入时，表被惰性地初始化为2级的大小。
     * 表中的每个bin通常包含一个Node列表(通常，该列表只有零个或一个Node)。
     * 表访问需要volatile/原子的读、写和用例。
     * 因为没有其他方法可以在不添加进一步间接的情况下进行安排
     * 所以我们使用intrinsics (sun.misc.Unsafe)操作。
     * 
     * 我们使用Node的hash字段的顶部(符号)位用于控制目的——由于寻址约束，无论如何它都是可用的。
     * 具有负散列字段的节点在映射方法中会被特殊处理或忽略。
     *
     * 将第一个Node插入（通过put或其变体）到一个空的bin中，只需将其放入bin即可。
     * 到目前为止，这是大多数key/hash分布下的put操作最常见的情况。
     * 其他更新操作（插入、删除和替换）需要锁定。
     * 我们不想浪费将不同锁对象与每个bin关联所需的空间，
     * 因此应该使用bin列表本身的第一个节点作为锁。
     * “同步锁”依赖于这些内置的锁。
     *
     * 使用列表的第一个节点作为锁本身是不够的，不足以解决线程安全问题:
     * 当一个节点被锁定时，任何更新必须首先验证它仍然是锁定后的第一个节点
     *（拿到锁喉，这个Node可能已经bin的第一个Node对象了），如果不是，则重试。
     * 因为新的节点总是被添加到列表中，所以一旦一个节点首先在bin中，它就会保持在首位，直到删除或者bin(在调整大小时)失效。
     * 而被删除或者调整大小后，被锁住的这个对象可能就不是bin的首个node，锁住它，
     * 别的操作还是会拿到第一个Node执行操作，会有线程安全问题
     *
     *
     * 每一个bin的首个Node作为锁的缺点：受同一锁保护的bin列表中其他Node上的其他更新操作可能会暂停
     * 然而，统计上，在随机哈希码下，这并不是一个常见的问题。
     * 理想情况下，如果调整阈值*为0.75，则箱中节点的频率遵循平均约为0.5的泊松分布参数，
     * 尽管由于调整粒度而具有较大的方差。忽略方差，
     * 列表大小k的预期出现次数为（exp（-0.5）*pow（0.5，k）/阶乘（k））。
     * 第一个值是：
     * 0:    0.60653066
     * 1:    0.30326533
     * 2:    0.07581633
     * 3:    0.01263606
     * 4:    0.00157952
     * 5:    0.00015795
     * 6:    0.00001316
     * 7:    0.00000094
     * 8:    0.00000006
     * 更多：少于一千万分之一
     *
     * 在随机哈希表下，访问不同元素的两个线程的锁争用概率大约是1 /(8 * #元素)。
     *
     * 实际中遇到的实际哈希码分布有时会明显偏离均匀随机性。
     * 这包括N>（1<<30）时的情况，因此某些关键点必须碰撞。
     * 类似地，对于一些愚蠢或恶意的用法，多个密钥被设计成具有相同的哈希码，
     * 或者只在隐藏的高位上有不同。
     * 因此，我们使用第二种策略，当bin中的节点数超过阈值时，该策略适用。
     * 这些树型图使用平衡树来保存节点（红黑树的一种特殊形式），将搜索时间限定为O（logn）。
     * TreeBin中的每个搜索步骤至少是常规列表中搜索步骤的两倍，但考虑到N不能超过（1<<64）（在地址用完之前），
     * 这就把搜索步骤、锁保持时间等限制在合理的常量（每个操作的最坏情况下大约检查100个节点），
     * 只要密钥是可比较的（这是非常常见的字符串，long，等等）。
     * TreeBin节点（TreeNodes）也维护与常规节点相同的“next”遍历指针，
     * 因此可以以相同的方式在迭代器中遍历。
     *
     * 当占用率超过百分比阈值(名义上为0.75，但见下文)时，将调整表的大小。
     * 在初始化线程分配和设置替换数组后，任何注意到容器过满的线程都可以帮助调整大小。
     * 但是，这些其他线程可以继续执行插入等操作，而不是停止。
     * TreeBins的使用可以保护我们在调整大小的过程中避免最坏的情况下过度填充的影响。
     * 调整大小的过程是通过一个接一个地将容器从表转移到下一个表。
     * 不过，线程在传输之前会要求传输一小块索引(通过field transferIndex)，
     * 从而减少争用。字段sizeCtl中的生成戳可确保大小不重叠。
     * 因为我们使用的是2的幂展开，所以每个容器中的元素必须保持在相同的索引中，或者以2的幂偏移量移动。
     * 我们通过捕捉由于下一个字段不更改而可以重用旧节点的情况来消除不必要的节点创建。
     * 平均而言，当一个表翻倍时，只有大约六分之一需要克隆。
     * 它们替换的节点一旦不再被并发遍历表中的任何读线程引用，就将成为垃圾回收。
     * 在传输时，旧的表bin只包含一个特殊的转发节点(哈希字段为“MOVED”)，
     * 它包含下一个表作为它的键。在遇到转发节点时，使用新表重新启动访问和更新操作。
     *
     * 每个容器传输都需要它的容器锁，在调整大小时，它可能会暂停等待锁。
     * 但是，由于其他线程可以加入并帮助调整大小，而不是争用锁，
     * 因此随着调整大小的进展，平均聚合等待时间会缩短。
     * 传输操作还必须确保旧表和新表中的所有可访问的容器在任何遍历中都可用。
     * 这部分是通过从最后一个bin(表)开始进行安排的。长度- 1)到第一个。
     * 在看到转发节点后，遍历(参见类遍历器)安排移动到新表，而无需重新访问节点。
     * 为了确保即使在无序移动时也不会跳过中间节点，
     * 在遍历过程中第一次遇到转发节点时将创建一个堆栈(参见类TableStack)，
     * 以便在以后处理当前表时保持其位置。
     * 对这些保存/恢复机制的需求相对较少，
     * 但当遇到一个转发节点时，
     * 通常会遇到更多的转发节点。
     * 因此，遍历器使用一个简单的缓存方案来避免创建如此多的新TableStack节点。
     * (感谢Peter Levart在这里建议使用堆栈。)
     *
     * 遍历模式还适用于对容器范围的部分遍历(通过另一个遍历构造函数)，
     * 以支持分区聚合操作。此外，如果只读操作被转发到一个空表，
     * 那么它将放弃，这提供了对关闭风格清除的支持，而这也是目前还没有实现的。
     *
     * Lazy表初始化可以最大限度地减少首次使用前的占用空间，
     * 并且在第一个操作来自putAll、带有map参数的构造函数或反序列化时也避免了调整大小。
     * 这些案例试图覆盖初始容量设置，但在种族的情况下无法生效。
     *
     * 使用LongAdder的专门化来维护元素计数。
     * 为了访问导致创建多个countercell的隐式竞争感知，我们需要合并一个专门化，
     * 而不是仅仅使用一个LongAdder。
     * 计数器机制避免了更新上的争用，但如果在并发访问期间读取过于频繁，可能会遇到缓存抖动。
     * 为了避免读取如此频繁，只有在添加到一个已经拥有两个或更多节点的bin时才尝试在争用下调整大小。
     * 在均匀哈希分布下，发生在阈值处的概率约为13%，
     * 这意味着只有大约1 / 8的人设置检查阈值(在调整大小后，这样做的人就更少了)。
     *
     * 树状结构为搜索和相关操作使用一种特殊的比较形式(这是我们不能使用树状结构等现有集合的主要原因)。
     * TreeBins包含可比较的元素，但可能包含其他元素，
     * 以及可比较但对相同的T不一定可比较的元素，因此我们不能在它们之间调用compareTo。
     * 为了处理这个问题，该树首先按照哈希值排序，然后按照Comparable.compareTo排序(如果适用的话)。
     * 在节点上查找时，如果元素不可比较或比较为0，那么在绑定散列值的情况下，可能需要同时搜索左子和右子。
     * (如果所有元素都是不可比较的，并且绑定了哈希值，那么这就对应了完整的列表搜索。)
     * 在插入时，为了在重平衡中保持一个完整的顺序(或者尽可能接近这里所要求的顺序)，
     * 我们比较类和identityhashcode作为切入点。
     * 红黑平衡代码是根据Cormen、Leiserson和Rivest的“算法介绍”(CLR)依次从pre-jdk集合
     * (http://gee.cs.oswego.edu/dl/classes/collections/RBCell.java)中更新的。
     *
     * TreeBins还需要额外的锁定机制。即使在更新期间，
     * 读者也可以遍历列表，但树遍历就不行了，
     * 这主要是因为树的旋转可能会改变根节点和/或它的链接。
     * TreeBins包括一个寄生在主链同步策略上的简单读写锁机制:
     * 与插入或删除相关的结构调整已经是链锁(因此不能与其他写器冲突)，
     * 但必须等待正在进行的读器完成。因为这样的服务员只能有一个，
     * 所以我们使用一个简单的方案，使用一个“waiter”字段来阻止写入器。
     * 然而，读者永远不需要阻塞。如果根锁被持有，它们将沿着缓慢的遍历路径(通过下一个指针)进行，
     * 直到锁可用或链表被耗尽(以最先到达的为准)。
     * 这些情况并不快，但最大限度地提高了预期总吞吐量。
     *
     * 维护API和序列化与以前版本的这个类的兼容性引入了一些奇怪的东西。
     * 主要内容:我们保留未修改但未使用的构造函数参数来引用concurrencyLevel。
     * 我们接受一个loadFactor构造函数参数，
     * 但仅将其应用于初始表容量(这是我们能够保证使用它的唯一时间)。
     * 我们还声明了一个未使用的“段”类，只有在序列化时才以最小形式实例化它
     *
     * 而且，仅仅为了与这个类以前的版本兼容，它扩展了AbstractMap，
     *  尽管它的所有方法都被覆盖了，因此它只是无用的包袱
     *  
     *  这个文件被组织使事情更容易遵循阅读时比他们可能:
     *  第一主静态声明和实用程序,然后字段,然后主要公共方法
     *  (有几把多个公共方法到内部的),然后分级方法,树、转盘和批量操作。
     */

    /* ---------------- Constants -------------- */

    /**
     * The largest possible table capacity.  This value must be
     * exactly 1<<30 to stay within Java array allocation and indexing
     * bounds for power of two table sizes, and is further required
     * because the top two bits of 32bit hash fields are used for
     * control purposes.
     */
    private static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * The default initial table capacity.  Must be a power of 2
     * (i.e., at least 1) and at most MAXIMUM_CAPACITY.
     */
    private static final int DEFAULT_CAPACITY = 16;

    /**
     * The largest possible (non-power of two) array size.
     * Needed by toArray and related methods.
     */
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * The default concurrency level for this table. Unused but
     * defined for compatibility with previous versions of this class.
     */
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;

    /**
     * The load factor for this table. Overrides of this value in
     * constructors affect only the initial table capacity.  The
     * actual floating point value isn't normally used -- it is
     * simpler to use expressions such as {@code n - (n >>> 2)} for
     * the associated resizing threshold.
     */
    private static final float LOAD_FACTOR = 0.75f;

    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2, and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     * The smallest table capacity for which bins may be treeified.
     * (Otherwise the table is resized if too many nodes in a bin.)
     * The value should be at least 4 * TREEIFY_THRESHOLD to avoid
     * conflicts between resizing and treeification thresholds.
     */
    static final int MIN_TREEIFY_CAPACITY = 64;

    /**
     * Minimum number of rebinnings per transfer step. Ranges are
     * subdivided to allow multiple resizer threads.  This value
     * serves as a lower bound to avoid resizers encountering
     * excessive memory contention.  The value should be at least
     * DEFAULT_CAPACITY.
     */
    private static final int MIN_TRANSFER_STRIDE = 16;

    /**
     * The number of bits used for generation stamp in sizeCtl.
     * Must be at least 6 for 32bit arrays.
     */
    private static int RESIZE_STAMP_BITS = 16;

    /**
     * The maximum number of threads that can help resize.
     * Must fit in 32 - RESIZE_STAMP_BITS bits.
     */
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;

    /**
     * The bit shift for recording size stamp in sizeCtl.
     */
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

    /*
     * Encodings for Node hash fields. See above for explanation.
     */
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

    /** Number of CPUS, to place bounds on some sizings */
    static final int NCPU = Runtime.getRuntime().availableProcessors();

    /** For serialization compatibility. */
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("segments", Segment[].class),
        new ObjectStreamField("segmentMask", Integer.TYPE),
        new ObjectStreamField("segmentShift", Integer.TYPE)
    };

    /* ---------------- Nodes -------------- */

    /**
     * Key-value entry.  This class is never exported out as a
     * user-mutable Map.Entry (i.e., one supporting setValue; see
     * MapEntry below), but can be used for read-only traversals used
     * in bulk tasks.  Subclasses of Node with a negative hash field
     * are special, and contain null keys and values (but are never
     * exported).  Otherwise, keys and vals are never null.
     */
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final K getKey()       { return key; }
        public final V getValue()     { return val; }
        public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
        public final String toString(){ return key + "=" + val; }
        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public final boolean equals(Object o) {
            Object k, v, u; Map.Entry<?,?> e;
            return ((o instanceof Map.Entry) &&
                    (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                    (v = e.getValue()) != null &&
                    (k == key || k.equals(key)) &&
                    (v == (u = val) || v.equals(u)));
        }

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }

    /* ---------------- Static utilities -------------- */

    /**
     * Spreads (XORs) higher bits of hash to lower and also forces top
     * bit to 0. Because the table uses power-of-two masking, sets of
     * hashes that vary only in bits above the current mask will
     * always collide. (Among known examples are sets of Float keys
     * holding consecutive whole numbers in small tables.)  So we
     * apply a transform that spreads the impact of higher bits
     * downward. There is a tradeoff between speed, utility, and
     * quality of bit-spreading. Because many common sets of hashes
     * are already reasonably distributed (so don't benefit from
     * spreading), and because we use trees to handle large sets of
     * collisions in bins, we just XOR some shifted bits in the
     * cheapest possible way to reduce systematic lossage, as well as
     * to incorporate impact of the highest bits that would otherwise
     * never be used in index calculations because of table bounds.
     */
    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }

    /**
     * Returns a power of two table size for the given desired capacity.
     * See Hackers Delight, sec 3.2
     */
    private static final int tableSizeFor(int c) {
        int n = c - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    /**
     * Returns x's Class if it is of the form "class C implements
     * Comparable<C>", else null.
     */
    static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
            if ((c = x.getClass()) == String.class) // bypass checks
                return c;
            if ((ts = c.getGenericInterfaces()) != null) {
                for (int i = 0; i < ts.length; ++i) {
                    if (((t = ts[i]) instanceof ParameterizedType) &&
                        ((p = (ParameterizedType)t).getRawType() ==
                         Comparable.class) &&
                        (as = p.getActualTypeArguments()) != null &&
                        as.length == 1 && as[0] == c) // type arg is c
                        return c;
                }
            }
        }
        return null;
    }

    /**
     * Returns k.compareTo(x) if x matches kc (k's screened comparable
     * class), else 0.
     */
    @SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
    static int compareComparables(Class<?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :
                ((Comparable)k).compareTo(x));
    }

    /* ---------------- Table element access -------------- */

    /*
     * Volatile access methods are used for table elements as well as
     * elements of in-progress next table while resizing.  All uses of
     * the tab arguments must be null checked by callers.  All callers
     * also paranoically precheck that tab's length is not zero (or an
     * equivalent check), thus ensuring that any index argument taking
     * the form of a hash value anded with (length - 1) is a valid
     * index.  Note that, to be correct wrt arbitrary concurrency
     * errors by users, these checks must operate on local variables,
     * which accounts for some odd-looking inline assignments below.
     * Note that calls to setTabAt always occur within locked regions,
     * and so in principle require only release ordering, not
     * full volatile semantics, but are currently coded as volatile
     * writes to be conservative.
     */

    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }

    /* ---------------- Fields -------------- */

    /**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    transient volatile Node<K,V>[] table;

    /**
     * The next table to use; non-null only while resizing.
     */
    private transient volatile Node<K,V>[] nextTable;

    /**
     * Base counter value, used mainly when there is no contention,
     * but also as a fallback during table initialization
     * races. Updated via CAS.
     */
    private transient volatile long baseCount;

    /**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
    private transient volatile int sizeCtl;

    /**
     * The next table index (plus one) to split while resizing.
     */
    private transient volatile int transferIndex;

    /**
     * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
     */
    private transient volatile int cellsBusy;

    /**
     * Table of counter cells. When non-null, size is a power of 2.
     */
    private transient volatile CounterCell[] counterCells;

    // views
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;


    /* ---------------- Public operations -------------- */

    /**
     * Creates a new, empty map with the default initial table size (16).
     */
    public ConcurrentHashMap() {
    }

    /**
     * Creates a new, empty map with an initial table size
     * accommodating the specified number of elements without the need
     * to dynamically resize.
     *
     * @param initialCapacity The implementation performs internal
     * sizing to accommodate this many elements.
     * @throws IllegalArgumentException if the initial capacity of
     * elements is negative
     */
    public ConcurrentHashMap(int initialCapacity) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException();
        int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
                   MAXIMUM_CAPACITY :
                   tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
        this.sizeCtl = cap;
    }

    /**
     * Creates a new map with the same mappings as the given map.
     *
     * @param m the map
     */
    public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
        this.sizeCtl = DEFAULT_CAPACITY;
        putAll(m);
    }

    /**
     * Creates a new, empty map with an initial table size based on
     * the given number of elements ({@code initialCapacity}) and
     * initial table density ({@code loadFactor}).
     *
     * @param initialCapacity the initial capacity. The implementation
     * performs internal sizing to accommodate this many elements,
     * given the specified load factor.
     * @param loadFactor the load factor (table density) for
     * establishing the initial table size
     * @throws IllegalArgumentException if the initial capacity of
     * elements is negative or the load factor is nonpositive
     *
     * @since 1.6
     */
    public ConcurrentHashMap(int initialCapacity, float loadFactor) {
        this(initialCapacity, loadFactor, 1);
    }

    /**
     * Creates a new, empty map with an initial table size based on
     * the given number of elements ({@code initialCapacity}), table
     * density ({@code loadFactor}), and number of concurrently
     * updating threads ({@code concurrencyLevel}).
     *
     * @param initialCapacity the initial capacity. The implementation
     * performs internal sizing to accommodate this many elements,
     * given the specified load factor.
     * @param loadFactor the load factor (table density) for
     * establishing the initial table size
     * @param concurrencyLevel the estimated number of concurrently
     * updating threads. The implementation may use this value as
     * a sizing hint.
     * @throws IllegalArgumentException if the initial capacity is
     * negative or the load factor or concurrencyLevel are
     * nonpositive
     */
    public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }

    // Original (since JDK1.2) Map methods

    /**
     * {@inheritDoc}
     */
    public int size() {
        long n = sumCount();
        return ((n < 0L) ? 0 :
                (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                (int)n);
    }

    /**
     * {@inheritDoc}
     */
    public boolean isEmpty() {
        return sumCount() <= 0L; // ignore transient negative values
    }

    /**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code key.equals(k)},
     * then this method returns {@code v}; otherwise it returns
     * {@code null}.  (There can be at most one such mapping.)
     *
     * @throws NullPointerException if the specified key is null
     */
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }

    /**
     * Tests if the specified object is a key in this table.
     *
     * @param  key possible key
     * @return {@code true} if and only if the specified object
     *         is a key in this table, as determined by the
     *         {@code equals} method; {@code false} otherwise
     * @throws NullPointerException if the specified key is null
     */
    public boolean containsKey(Object key) {
        return get(key) != null;
    }

    /**
     * Returns {@code true} if this map maps one or more keys to the
     * specified value. Note: This method may require a full traversal
     * of the map, and is much slower than method {@code containsKey}.
     *
     * @param value value whose presence in this map is to be tested
     * @return {@code true} if this map maps one or more keys to the
     *         specified value
     * @throws NullPointerException if the specified value is null
     */
    public boolean containsValue(Object value) {
        if (value == null)
            throw new NullPointerException();
        Node<K,V>[] t;
        if ((t = table) != null) {
            Traverser<K,V> it = new Traverser<K,V>(t, t.length, 0, t.length);
            for (Node<K,V> p; (p = it.advance()) != null; ) {
                V v;
                if ((v = p.val) == value || (v != null && value.equals(v)))
                    return true;
            }
        }
        return false;
    }

    /**
     * Maps the specified key to the specified value in this table.
     * Neither the key nor the value can be null.
     *
     * <p>The value can be retrieved by calling the {@code get} method
     * with a key that is equal to the original key.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with {@code key}, or
     *         {@code null} if there was no mapping for {@code key}
     * @throws NullPointerException if the specified key or value is null
     */
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                        

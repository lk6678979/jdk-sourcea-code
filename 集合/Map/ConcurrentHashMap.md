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
     * Actual hash code distributions encountered in practice
     * sometimes deviate significantly from uniform randomness.  This
     * includes the case when N > (1<<30), so some keys MUST collide.
     * Similarly for dumb or hostile usages in which multiple keys are
     * designed to have identical hash codes or ones that differs only
     * in masked-out high bits. So we use a secondary strategy that
     * applies when the number of nodes in a bin exceeds a
     * threshold. These TreeBins use a balanced tree to hold nodes (a
     * specialized form of red-black trees), bounding search time to
     * O(log N).  Each search step in a TreeBin is at least twice as
     * slow as in a regular list, but given that N cannot exceed
     * (1<<64) (before running out of addresses) this bounds search
     * steps, lock hold times, etc, to reasonable constants (roughly
     * 100 nodes inspected per operation worst case) so long as keys
     * are Comparable (which is very common -- String, Long, etc).
     * TreeBin nodes (TreeNodes) also maintain the same "next"
     * traversal pointers as regular nodes, so can be traversed in
     * iterators in the same way.
     *
     * The table is resized when occupancy exceeds a percentage
     * threshold (nominally, 0.75, but see below).  Any thread
     * noticing an overfull bin may assist in resizing after the
     * initiating thread allocates and sets up the replacement array.
     * However, rather than stalling, these other threads may proceed
     * with insertions etc.  The use of TreeBins shields us from the
     * worst case effects of overfilling while resizes are in
     * progress.  Resizing proceeds by transferring bins, one by one,
     * from the table to the next table. However, threads claim small
     * blocks of indices to transfer (via field transferIndex) before
     * doing so, reducing contention.  A generation stamp in field
     * sizeCtl ensures that resizings do not overlap. Because we are
     * using power-of-two expansion, the elements from each bin must
     * either stay at same index, or move with a power of two
     * offset. We eliminate unnecessary node creation by catching
     * cases where old nodes can be reused because their next fields
     * won't change.  On average, only about one-sixth of them need
     * cloning when a table doubles. The nodes they replace will be
     * garbage collectable as soon as they are no longer referenced by
     * any reader thread that may be in the midst of concurrently
     * traversing table.  Upon transfer, the old table bin contains
     * only a special forwarding node (with hash field "MOVED") that
     * contains the next table as its key. On encountering a
     * forwarding node, access and update operations restart, using
     * the new table.
     *
     * Each bin transfer requires its bin lock, which can stall
     * waiting for locks while resizing. However, because other
     * threads can join in and help resize rather than contend for
     * locks, average aggregate waits become shorter as resizing
     * progresses.  The transfer operation must also ensure that all
     * accessible bins in both the old and new table are usable by any
     * traversal.  This is arranged in part by proceeding from the
     * last bin (table.length - 1) up towards the first.  Upon seeing
     * a forwarding node, traversals (see class Traverser) arrange to
     * move to the new table without revisiting nodes.  To ensure that
     * no intervening nodes are skipped even when moved out of order,
     * a stack (see class TableStack) is created on first encounter of
     * a forwarding node during a traversal, to maintain its place if
     * later processing the current table. The need for these
     * save/restore mechanics is relatively rare, but when one
     * forwarding node is encountered, typically many more will be.
     * So Traversers use a simple caching scheme to avoid creating so
     * many new TableStack nodes. (Thanks to Peter Levart for
     * suggesting use of a stack here.)
     *
     * The traversal scheme also applies to partial traversals of
     * ranges of bins (via an alternate Traverser constructor)
     * to support partitioned aggregate operations.  Also, read-only
     * operations give up if ever forwarded to a null table, which
     * provides support for shutdown-style clearing, which is also not
     * currently implemented.
     *
     * Lazy table initialization minimizes footprint until first use,
     * and also avoids resizings when the first operation is from a
     * putAll, constructor with map argument, or deserialization.
     * These cases attempt to override the initial capacity settings,
     * but harmlessly fail to take effect in cases of races.
     *
     * The element count is maintained using a specialization of
     * LongAdder. We need to incorporate a specialization rather than
     * just use a LongAdder in order to access implicit
     * contention-sensing that leads to creation of multiple
     * CounterCells.  The counter mechanics avoid contention on
     * updates but can encounter cache thrashing if read too
     * frequently during concurrent access. To avoid reading so often,
     * resizing under contention is attempted only upon adding to a
     * bin already holding two or more nodes. Under uniform hash
     * distributions, the probability of this occurring at threshold
     * is around 13%, meaning that only about 1 in 8 puts check
     * threshold (and after resizing, many fewer do so).
     *
     * TreeBins use a special form of comparison for search and
     * related operations (which is the main reason we cannot use
     * existing collections such as TreeMaps). TreeBins contain
     * Comparable elements, but may contain others, as well as
     * elements that are Comparable but not necessarily Comparable for
     * the same T, so we cannot invoke compareTo among them. To handle
     * this, the tree is ordered primarily by hash value, then by
     * Comparable.compareTo order if applicable.  On lookup at a node,
     * if elements are not comparable or compare as 0 then both left
     * and right children may need to be searched in the case of tied
     * hash values. (This corresponds to the full list search that
     * would be necessary if all elements were non-Comparable and had
     * tied hashes.) On insertion, to keep a total ordering (or as
     * close as is required here) across rebalancings, we compare
     * classes and identityHashCodes as tie-breakers. The red-black
     * balancing code is updated from pre-jdk-collections
     * (http://gee.cs.oswego.edu/dl/classes/collections/RBCell.java)
     * based in turn on Cormen, Leiserson, and Rivest "Introduction to
     * Algorithms" (CLR).
     *
     * TreeBins also require an additional locking mechanism.  While
     * list traversal is always possible by readers even during
     * updates, tree traversal is not, mainly because of tree-rotations
     * that may change the root node and/or its linkages.  TreeBins
     * include a simple read-write lock mechanism parasitic on the
     * main bin-synchronization strategy: Structural adjustments
     * associated with an insertion or removal are already bin-locked
     * (and so cannot conflict with other writers) but must wait for
     * ongoing readers to finish. Since there can be only one such
     * waiter, we use a simple scheme using a single "waiter" field to
     * block writers.  However, readers need never block.  If the root
     * lock is held, they proceed along the slow traversal path (via
     * next-pointers) until the lock becomes available or the list is
     * exhausted, whichever comes first. These cases are not fast, but
     * maximize aggregate expected throughput.
     *
     * Maintaining API and serialization compatibility with previous
     * versions of this class introduces several oddities. Mainly: We
     * leave untouched but unused constructor arguments refering to
     * concurrencyLevel. We accept a loadFactor constructor argument,
     * but apply it only to initial table capacity (which is the only
     * time that we can guarantee to honor it.) We also declare an
     * unused "Segment" class that is instantiated in minimal form
     * only when serializing.
     *
     * Also, solely for compatibility with previous versions of this
     * class, it extends AbstractMap, even though all of its methods
     * are overridden, so it is just useless baggage.
     *
     * This file is organized to make things a little easier to follow
     * while reading than they might otherwise: First the main static
     * declarations and utilities, then fields, then main public
     * methods (with a few factorings of multiple public methods into
     * internal ones), then sizing methods, trees, traversers, and
     * bulk operations.
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
                        

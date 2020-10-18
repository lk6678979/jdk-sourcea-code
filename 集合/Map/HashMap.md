# HashMap
# 1. 首先看一下Map类
```java
import java.io.IOException;
import java.io.InvalidObjectException;
import java.io.Serializable;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Consumer;
import java.util.function.Function;
import sun.misc.SharedSecrets;

/**
 * Hash table based implementation of the <tt>Map</tt> interface.  This
 * implementation provides all of the optional map operations, and permits
 * <tt>null</tt> values and the <tt>null</tt> key.  (The <tt>HashMap</tt>
 * class is roughly equivalent to <tt>Hashtable</tt>, except that it is
 * unsynchronized and permits nulls.)  This class makes no guarantees as to
 * the order of the map; in particular, it does not guarantee that the order
 * will remain constant over time.
 *
 * <p>This implementation provides constant-time performance for the basic
 * operations (<tt>get</tt> and <tt>put</tt>), assuming the hash function
 * disperses the elements properly among the buckets.  Iteration over
 * collection views requires time proportional to the "capacity" of the
 * <tt>HashMap</tt> instance (the number of buckets) plus its size (the number
 * of key-value mappings).  Thus, it's very important not to set the initial
 * capacity too high (or the load factor too low) if iteration performance is
 * important.
 *
 * <p>An instance of <tt>HashMap</tt> has two parameters that affect its
 * performance: <i>initial capacity</i> and <i>load factor</i>.  The
 * <i>capacity</i> is the number of buckets in the hash table, and the initial
 * capacity is simply the capacity at the time the hash table is created.  The
 * <i>load factor</i> is a measure of how full the hash table is allowed to
 * get before its capacity is automatically increased.  When the number of
 * entries in the hash table exceeds the product of the load factor and the
 * current capacity, the hash table is <i>rehashed</i> (that is, internal data
 * structures are rebuilt) so that the hash table has approximately twice the
 * number of buckets.
 *
 * <p>As a general rule, the default load factor (.75) offers a good
 * tradeoff between time and space costs.  Higher values decrease the
 * space overhead but increase the lookup cost (reflected in most of
 * the operations of the <tt>HashMap</tt> class, including
 * <tt>get</tt> and <tt>put</tt>).  The expected number of entries in
 * the map and its load factor should be taken into account when
 * setting its initial capacity, so as to minimize the number of
 * rehash operations.  If the initial capacity is greater than the
 * maximum number of entries divided by the load factor, no rehash
 * operations will ever occur.
 *
 * <p>If many mappings are to be stored in a <tt>HashMap</tt>
 * instance, creating it with a sufficiently large capacity will allow
 * the mappings to be stored more efficiently than letting it perform
 * automatic rehashing as needed to grow the table.  Note that using
 * many keys with the same {@code hashCode()} is a sure way to slow
 * down performance of any hash table. To ameliorate impact, when keys
 * are {@link Comparable}, this class may use comparison order among
 * keys to help break ties.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a hash map concurrently, and at least one of
 * the threads modifies the map structurally, it <i>must</i> be
 * synchronized externally.  (A structural modification is any operation
 * that adds or deletes one or more mappings; merely changing the value
 * associated with a key that an instance already contains is not a
 * structural modification.)  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the map.
 *
 * If no such object exists, the map should be "wrapped" using the
 * {@link Collections#synchronizedMap Collections.synchronizedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the map:<pre>
 *   Map m = Collections.synchronizedMap(new HashMap(...));</pre>
 *
 * <p>The iterators returned by all of this class's "collection view methods"
 * are <i>fail-fast</i>: if the map is structurally modified at any time after
 * the iterator is created, in any way except through the iterator's own
 * <tt>remove</tt> method, the iterator will throw a
 * {@link ConcurrentModificationException}.  Thus, in the face of concurrent
 * modification, the iterator fails quickly and cleanly, rather than risking
 * arbitrary, non-deterministic behavior at an undetermined time in the
 * future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness: <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Doug Lea
 * @author  Josh Bloch
 * @author  Arthur van Hoff
 * @author  Neal Gafter
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     TreeMap
 * @see     Hashtable
 * @since   1.2
 */
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```
翻译：
```
基于哈希表的实现的Map接口。 此实现提供了所有可选的Map操作，并允许null的值和null键。 （ HashMap类大致相当于Hashtable ，除了它是不同步的，并允许null）。这个类不能保证地图的顺序; 特别是，它不能保证订单在一段时间内保持不变。 
假设哈希函数在这些存储桶之间正确分散元素，这个实现为基本操作（ get和put ）提供了恒定的时间性能。 收集视图的迭代需要与HashMap实例（桶数）加上其大小（键值映射数）的“容量” 成正比 。 因此，如果迭代性能很重要，不要将初始容量设置得太高（或负载因子太低）是非常重要的。 

HashMap的一个实例有两个影响其性能的参数： 初始容量和负载因子 。 容量是哈希表中的桶数，初始容量只是创建哈希表时的容量。 负载因子是在容量自动增加之前允许哈希表得到满足的度量。 当在散列表中的条目的数量超过了负载因数和电流容量的乘积，哈希表被重新散列 （即，内部数据结构被重建），使得哈希表具有桶的大约两倍。 

作为一般规则，默认负载因子（.75）提供了时间和空间成本之间的良好折中。 更高的值会降低空间开销，但会增加查找成本（反映在HashMap类的大部分操作中，包括get和put ）。 在设置其初始容量时，应考虑地图中预期的条目数及其负载因子，以便最小化重新组播操作的数量。 如果初始容量大于最大条目数除以负载因子，则不会发生重新排列操作。 

如果许多映射要存储在HashMap实例中，则以足够大的容量创建映射将允许映射的存储效率高于使其根据需要执行自动重新排序以增长表。 请注意，使用同一个hashCode()多个密钥是降低任何哈希表的hashCode()的一种方法。 为了改善影响，当按键是Comparable时，这个类可以使用键之间的比较顺序来帮助打破关系。 

请注意，此实现不同步。 如果多个线程同时访问哈希映射，并且至少有一个线程在结构上修改了映射，那么它必须在外部进行同步。 （结构修改是添加或删除一个或多个映射的任何操作;仅改变与实例已经包含的密钥相关联的值不是结构修改。）这通常通过对自然地封装映射的一些对象进行同步来实现。 如果没有这样的对象存在，地图应该使用Collections.synchronizedMap方法“包装”。 这最好在创建时完成，以防止意外的不同步访问地图： 

  Map m = Collections.synchronizedMap(new HashMap(...)); 所有这个类的“集合视图方法”返回的迭代器都是故障快速的 ：如果映射在迭代器创建之后的任何时间被结构地修改，除了通过迭代器自己的remove方法之外，迭代器将抛出一个ConcurrentModificationException 。 因此，面对并发修改，迭代器将快速而干净地失败，而不是在未来未确定的时间冒着任意的非确定性行为。 

请注意，迭代器的故障快速行为无法保证，因为一般来说，在不同步并发修改的情况下，无法做出任何硬性保证。 失败快速迭代器尽力扔掉ConcurrentModificationException 。 因此，编写依赖于此异常的程序的正确性将是错误的：迭代器的故障快速行为应仅用于检测错误。 
```
# 2. 常量
```
 /**
     * 默认初始容量-必须是2的幂，这里是16
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    /**
    *最大容量，在隐式指定更高的值时使用
    *由具有参数的构造函数之一。
    *必须是2的幂<=1<<30。
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 构造函数中未指定时使用的负载因子（默认值）
     * 当存储的数据超过map存储原始的75%时，会添加数据存储的位置并重新分配
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
      * 桶的树化阈值：即 链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
      * 桶的链表还原阈值：即 红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），
      * 在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表
     */
    static final int UNTREEIFY_THRESHOLD = 6;

    /**
     *最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
     *否则，若桶内元素太多时，则直接扩容，而不是树形化
     *为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
     */
    static final int MIN_TREEIFY_CAPACITY = 64;
```

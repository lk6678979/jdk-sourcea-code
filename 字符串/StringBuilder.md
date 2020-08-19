# StringBuilder
先看看类
```
/**
 * A mutable sequence of characters.  This class provides an API compatible
 * with {@code StringBuffer}, but with no guarantee of synchronization.
 * This class is designed for use as a drop-in replacement for
 * {@code StringBuffer} in places where the string buffer was being
 * used by a single thread (as is generally the case).   Where possible,
 * it is recommended that this class be used in preference to
 * {@code StringBuffer} as it will be faster under most implementations.
 *
 * <p>The principal operations on a {@code StringBuilder} are the
 * {@code append} and {@code insert} methods, which are
 * overloaded so as to accept data of any type. Each effectively
 * converts a given datum to a string and then appends or inserts the
 * characters of that string to the string builder. The
 * {@code append} method always adds these characters at the end
 * of the builder; the {@code insert} method adds the characters at
 * a specified point.
 * <p>
 * For example, if {@code z} refers to a string builder object
 * whose current contents are "{@code start}", then
 * the method call {@code z.append("le")} would cause the string
 * builder to contain "{@code startle}", whereas
 * {@code z.insert(4, "le")} would alter the string builder to
 * contain "{@code starlet}".
 * <p>
 * In general, if sb refers to an instance of a {@code StringBuilder},
 * then {@code sb.append(x)} has the same effect as
 * {@code sb.insert(sb.length(), x)}.
 * <p>
 * Every string builder has a capacity. As long as the length of the
 * character sequence contained in the string builder does not exceed
 * the capacity, it is not necessary to allocate a new internal
 * buffer. If the internal buffer overflows, it is automatically made larger.
 *
 * <p>Instances of {@code StringBuilder} are not safe for
 * use by multiple threads. If such synchronization is required then it is
 * recommended that {@link java.lang.StringBuffer} be used.
 *
 * <p>Unless otherwise noted, passing a {@code null} argument to a constructor
 * or method in this class will cause a {@link NullPointerException} to be
 * thrown.
 *
 * @author      Michael McCloskey
 * @see         java.lang.StringBuffer
 * @see         java.lang.String
 * @since       1.5
 */
public final class StringBuilder
    extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
```
翻译
```
一个可变的字符序列。 此类提供与StringBuffer的API，但不保证同步。 此类设计用作简易替换为StringBuffer在正在使用由单个线程字符串缓冲区的地方（如通常是这种情况）。 在可能的情况下，建议使用这个类别优先于StringBuffer ，因为它在大多数实现中将更快。 
StringBuilder的主要StringBuilder是append和insert方法，它们是重载的，以便接受任何类型的数据。 每个都有效地将给定的数据转换为字符串，然后将该字符串的字符附加或插入字符串构建器。 append方法始终在构建器的末尾添加这些字符; insert方法将insert添加到指定点。 

例如，如果z引用当前内容为“ start ”的字符串构建器对象，那么方法调用z.append("le")将导致字符串构建器包含“ startle ”，而z.insert(4, "le")会将字符串构建器更改为包含“ starlet ”。 

一般情况下，如果某人是指的一个实例StringBuilder ，则sb.append(x)具有相同的效果sb.insert(sb.length(), x) 。 

每个字符串构建器都有一个容量。 只要字符串构建器中包含的字符序列的长度不超过容量，则不需要分配新的内部缓冲区。 如果内部缓冲区溢出，则会自动变大。 

StringBuilder的StringBuilder不能安全使用多线程。 如果需要同步， 那么建议使用StringBuffer 。 

除非另有说明，否则将null参数传递给null中的构造函数或方法将导致抛出NullPointerException 。 

```

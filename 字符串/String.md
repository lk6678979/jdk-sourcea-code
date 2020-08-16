## String类
### 类的定义
```
/**
 * The {@code String} class represents character strings. All
 * string literals in Java programs, such as {@code "abc"}, are
 * implemented as instances of this class.
 * <p>
 * Strings are constant; their values cannot be changed after they
 * are created. String buffers support mutable strings.
 * Because String objects are immutable they can be shared. For example:
 * <blockquote><pre>
 *     String str = "abc";
 * </pre></blockquote><p>
 * is equivalent to:
 * <blockquote><pre>
 *     char data[] = {'a', 'b', 'c'};
 *     String str = new String(data);
 * </pre></blockquote><p>
 * Here are some more examples of how strings can be used:
 * <blockquote><pre>
 *     System.out.println("abc");
 *     String cde = "cde";
 *     System.out.println("abc" + cde);
 *     String c = "abc".substring(2,3);
 *     String d = cde.substring(1, 2);
 * </pre></blockquote>
 * <p>
 * The class {@code String} includes methods for examining
 * individual characters of the sequence, for comparing strings, for
 * searching strings, for extracting substrings, and for creating a
 * copy of a string with all characters translated to uppercase or to
 * lowercase. Case mapping is based on the Unicode Standard version
 * specified by the {@link java.lang.Character Character} class.
 * <p>
 * The Java language provides special support for the string
 * concatenation operator (&nbsp;+&nbsp;), and for conversion of
 * other objects to strings. String concatenation is implemented
 * through the {@code StringBuilder}(or {@code StringBuffer})
 * class and its {@code append} method.
 * String conversions are implemented through the method
 * {@code toString}, defined by {@code Object} and
 * inherited by all classes in Java. For additional information on
 * string concatenation and conversion, see Gosling, Joy, and Steele,
 * <i>The Java Language Specification</i>.
 *
 * <p> Unless otherwise noted, passing a <tt>null</tt> argument to a constructor
 * or method in this class will cause a {@link NullPointerException} to be
 * thrown.
 *
 * <p>A {@code String} represents a string in the UTF-16 format
 * in which <em>supplementary characters</em> are represented by <em>surrogate
 * pairs</em> (see the section <a href="Character.html#unicode">Unicode
 * Character Representations</a> in the {@code Character} class for
 * more information).
 * Index values refer to {@code char} code units, so a supplementary
 * character uses two positions in a {@code String}.
 * <p>The {@code String} class provides methods for dealing with
 * Unicode code points (i.e., characters), in addition to those for
 * dealing with Unicode code units (i.e., {@code char} values).
 *
 * @author  Lee Boynton
 * @author  Arthur van Hoff
 * @author  Martin Buchholz
 * @author  Ulf Zibis
 * @see     java.lang.Object#toString()
 * @see     java.lang.StringBuffer
 * @see     java.lang.StringBuilder
 * @see     java.nio.charset.Charset
 * @since   JDK1.0
 */

public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
```
翻译：
```
String类代表字符串。 Java程序中的所有字符串文字（例如"abc" ）都被实现为此类的实例。 
字符串不变; 它们的值在创建后不能被更改。 字符串缓冲区支持可变字符串。 因为String对象是不可变的，它们可以被共享。 例如： 

     String str = "abc";
 相当于： 

     char data[] = {'a', 'b', 'c'};
     String str = new String(data);
 以下是一些如何使用字符串的示例： 

     System.out.println("abc");
     String cde = "cde";
     System.out.println("abc" + cde);
     String c = "abc".substring(2,3);
     String d = cde.substring(1, 2);
 String类包括用于检查序列的各个字符的方法，用于比较字符串，搜索字符串，提取子字符串以及创建将所有字符翻译为大写或小写的字符串的副本。 案例映射基于Character类指定的Unicode标准版本。 

Java语言为字符串连接运算符（+）提供特殊支持，并为其他对象转换为字符串。 字符串连接是通过StringBuilder （或StringBuffer ）类及其append方法实现的。 字符串转换是通过方法来实现toString ，由下式定义Object和继承由在Java中的所有类。 有关字符串连接和转换的其他信息，请参阅Gosling，Joy和Steele， Java语言规范 。 

除非另有说明，否则传递null参数到此类中的构造函数或方法将导致抛出NullPointerException 。 

A String表示UTF-16格式的字符串，其中补充字符由代理对表示 （有关详细信息，请参阅Character课程中的Character部分）。 索引值是指char代码单元，所以补充字符在String中使用两个String 。 

String类提供处理Unicode代码点（即字符）的方法，以及用于处理Unicode代码单元（即char值）的方法。

```
* String 是 final 类型的，表示该类不能被其他类继承，同时该类实现了三个接口：java.io.Serializable Comparable<String> CharSequence
* String 字符串是常量，其值在实例创建后就不能被修改，但字符串缓冲区支持可变的字符串，因为缓冲区里面的不可变字符串对象们可以被共享（这个缓冲区就是String存储字符的char[]）
### 类的属性
```
/** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
 ```
* String 实际上是数据是一个final的字符数组，内容一旦被初始化后，其不能被修改的。
* String 的Hash默认是0

### 方法说明
* getByte()：将String中的char[]逐个字符根据指定的编码格式转为byte数，默认UTF-8，例如0转为48,1转为49....
* new String()：创建的是""字符串
* intern()：  
  常量池(constant pool)指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。它包括了关于类、方法、接口等中的常量，也包括字符串常量  
  当调用 intern 方法时，如果池已经包含一个等于此 String 对象的字符串（用 equals(Object) 方法确定），则返回池中的字符串。否则，将此 String 对象添加到池中(复制一个新的String对象)，并返回此 String 对象的引用。
* +号运算：
  如果+前后都是字符串，编译时会直接拼在一起  
  如果+前后是一个对象，会创建一个StringBuilder然后append2个值
* trim()：通过遍历String的时候char，判断第一个不是" "和最后一个不是" "的下标，然后使用substring()根据下标截取目标字符串
* compare():
```
    public int compareTo(String anotherString) {
        int len1 = value.length;
        int len2 = anotherString.value.length;
        int lim = Math.min(len1, len2);
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        return len1 - len2;
    }
```
  获取2个字符串的char[]，根据char[]较小的长度对比对应长度的char,从第一个char开始比大小，char大的字符串大，如果这些char都相等，则字符串更长的大
* equals()：长度不同则判定为false，长度相对for对比char[]只要有一个不同这false

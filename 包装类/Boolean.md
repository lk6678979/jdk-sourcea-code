# Boolean
布尔类将对象中的基元类型boolean的值进行包装。 类型为Boolean的对象包含一个单一字段value，其类型为boolean。  
此外，该类还提供了许多将boolean转换为String和String转换为boolean ，以及在处理boolean时有用的其他常量和方法。  

### 1. 如何确定Boolean对象对应的基础类型
Boolean的Class是Boolean.class，Boolean类中通过常来Type来记录对应的基础类型
```
/**
     * The Class object representing the primitive type boolean.
     * 表示原始类型的类对象布尔值。
     * Class.getPrimitiveClass:获取原始类型的Class:boolean，Boolean的Class是Boolean.class不是boolean
     * @since   JDK1.1
     */
    @SuppressWarnings("unchecked")
    public static final Class<Boolean> TYPE = (Class<Boolean>) Class.getPrimitiveClass("boolean");
```
测试
```
public static void main(String[] args) throws Exception {
        Boolean ss = true;
        System.out.println(ss.getClass());
        System.out.println(ss.TYPE);
        System.out.println(boolean.class == ss.getClass());
        System.out.println(boolean.class == ss.TYPE);
    }
```
输出结果：
```
class java.lang.Boolean
boolean
false
true
```
### 2. 字符串"true"、"false"和Boolean转换
* 字符串转Boolean：实际上就是通过字符串的equalsIgnoreCase("true"),所以只要字符串不是"true"则都是false
```java
    /**
     * Returns a {@code Boolean} with a value represented by the
     * specified string.  The {@code Boolean} returned represents a
     * true value if the string argument is not {@code null}
     * and is equal, ignoring case, to the string {@code "true"}.
     * TRUE 和 FALSE 是Boolean类才常量，分表是 new Boolean(true)和new Boolean(false)
     * @param   s   a string.
     * @return  the {@code Boolean} value represented by the string.
     */
    public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }
    
    /**
     * Parses the string argument as a boolean.  The {@code boolean}
     * returned represents the value {@code true} if the string argument
     * is not {@code null} and is equal, ignoring case, to the string
     * {@code "true"}. <p>
     * Example: {@code Boolean.parseBoolean("True")} returns {@code true}.<br>
     * Example: {@code Boolean.parseBoolean("yes")} returns {@code false}.
     *
     * @param      s   the {@code String} containing the boolean
     *                 representation to be parsed
     * @return     the boolean represented by the string argument
     * @since 1.5
     */
    public static boolean parseBoolean(String s) {
        return ((s != null) && s.equalsIgnoreCase("true"));
    }
 ```
 测试代码
 ```java
     public static void main(String[] args) throws Exception {
        System.out.println(Boolean.valueOf("true"));
        System.out.println(Boolean.valueOf("1111"));
    }
 ```
 输出结果：
 ```java
 true
 false
 ```
* Boolean转字符串：根绝boolan固定返回"true"或者"false"
```java
    /**
 * Returns a {@code String} object representing the specified
 * boolean.  If the specified boolean is {@code true}, then
 * the string {@code "true"} will be returned, otherwise the
 * string {@code "false"} will be returned.
 *
 * @param b the boolean to be converted
 * @return the string representation of the specified {@code boolean}
 * @since 1.4
 */
public static String toString(boolean b) {
    return b ? "true" : "false";
}

    /**
     * Returns a {@code String} object representing this Boolean's
     * value.  If this object represents the value {@code true},
     * a string equal to {@code "true"} is returned. Otherwise, a
     * string equal to {@code "false"} is returned.
     *
     * @return  a string representation of this object.
     */
    public String toString() {
        return value ? "true" : "false";
    }
```
* Boolean的hashcode()是固定int值。true：1231。false：1237
```
/**
     * Returns a hash code for this {@code Boolean} object.
     *
     * @return  the integer {@code 1231} if this object represents
     * {@code true}; returns the integer {@code 1237} if this
     * object represents {@code false}.
     */
    @Override
    public int hashCode() {
        return Boolean.hashCode(value);
    }

    /**
     * Returns a hash code for a {@code boolean} value; compatible with
     * {@code Boolean.hashCode()}.
     *
     * @param value the value to hash
     * @return a hash code value for a {@code boolean} value.
     * @since 1.8
     */
    public static int hashCode(boolean value) {
        return value ? 1231 : 1237;
    }
```
测试代码
```java
public static void main(String[] args) throws Exception {
        System.out.println(Boolean.valueOf("true").hashCode());
        System.out.println(Boolean.valueOf("true").hashCode());
        System.out.println(Boolean.valueOf("1111").hashCode());
        System.out.println(Boolean.valueOf("false").hashCode());
    }
```
输出结果
```
1231
1231
1237
1237
```

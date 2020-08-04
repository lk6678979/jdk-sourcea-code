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
### 2. API总结
* 字符串转Boolean：实际上就是通过字符串的equalsIgnoreCase("true"),所以只要字符串不是"true"则都是false  
* Boolean转字符串：根据boolan固定返回"true"或者"false"  
* Boolean的hashcode()是固定int值。true：1231。false：1237  
* compare()：ture>false  
* 迷之getBoolean(String name),这个方法并不是将“true”或者“false”转换为Boolean类型的API。
```
/**
     * Returns {@code true} if and only if the system property
     * named by the argument exists and is equal to the string
     * {@code "true"}. (Beginning with version 1.0.2 of the
     * Java<small><sup>TM</sup></small> platform, the test of
     * this string is case insensitive.) A system property is accessible
     * through {@code getProperty}, a method defined by the
     * {@code System} class.
     * <p>
     * If there is no property with the specified name, or if the specified
     * name is empty or null, then {@code false} is returned.
     *
     * @param   name   the system property name.
     * @return  the {@code boolean} value of the system property.
     * @throws  SecurityException for the same reasons as
     *          {@link System#getProperty(String) System.getProperty}
     * @see     java.lang.System#getProperty(java.lang.String)
     * @see     java.lang.System#getProperty(java.lang.String, java.lang.String)
     */
    public static boolean getBoolean(String name) {
        boolean result = false;
        try {
            result = parseBoolean(System.getProperty(name));
        } catch (IllegalArgumentException | NullPointerException e) {
        }
        return result;
    }
 ```
从源代码中可以看出Boolean.getBoolean首先会根据name从系统属性中获取name属性的值，然后对name属性对应的值进行转换  
正确的用法用法应该是：
```java
public static void main(String[] args) {
    String value = "true";
    String key= "key";
    System.setProperty(key, value);
    Boolean flag = Boolean.getBoolean(key);
    if(flag) {
        System.out.println("key is true");
    } else {
        System.out.println("key is not true");
    }
}
```
输出结果
```java
key is true
```

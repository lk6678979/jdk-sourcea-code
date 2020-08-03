# Clone克隆详解
## 1. 克隆是什么
克隆就是依据已经有的对象，创造一份新的数据完全完全一样的对象。

在Java中对象的克隆有深克隆和浅克隆之分。有这种区分的原因是Java中分为基本数据类型和引用数据类型，对于不同的数据类型在内存中的存储的区域是不同的。基本数据类型存储在栈中，引用数据类型存储在堆中。
## 2.如何实现克隆
在Object中clone()源码：
```java
/**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     * <p>
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown. Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code clone} method of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return a clone of this instance.
     * @throws CloneNotSupportedException if the object's class does not
     *                                    support the {@code Cloneable} interface. Subclasses
     *                                    that override the {@code clone} method can also
     *                                    throw this exception to indicate that an instance cannot
     *                                    be cloned.
     * @see java.lang.Cloneable
     */
    protected native Object clone() throws CloneNotSupportedException;
```
注释：
* 按照惯例，返回的对象应该通过调用super.clone获得。 如果一个类和它的所有超类（除了Object ）遵守这个惯例，那将是x.clone().getClass() == x.getClass()的情况。 
* 按照惯例，此方法返回的对象应该与此对象（正被克隆）无关。 为了实现这一独立性，可能需要修改super.clone返回的对象的一个或多个字段。 通常，这意味着复制构成被克隆的对象的内部“深层结构”的任何可变对象，并通过引用该副本替换对这些对象的引用。 如果一个类仅包含原始字段或对不可变对象的引用，则通常情况下， super.clone返回的对象中的字段通常不需要修改。 
* clone的方法Object执行特定的克隆操作。 首先，如果此对象的类不实现接口Cloneable ，则抛出CloneNotSupportedException 。 请注意，所有数组都被认为是实现接口Cloneable ，并且数组类型T[]的clone方法的返回类型是T[] ，其中T是任何引用或原始类型。 否则，该方法将创建该对象的类的新实例，并将其所有字段初始化为完全符合该对象的相应字段的内容，就像通过赋值一样。 这些字段的内容本身不被克隆。 因此，该方法执行该对象的“浅拷贝”，而不是“深度拷贝”操作。 
* Object类本身并不实现接口Cloneable ，因此在类别为Object的对象上调用clone方法将导致运行时抛出异常。 
测试clone方法，代码如下：
```
public class TestBean{
	
	public static void main(String[] args) throws Exception{
		
		TestBean testBean = new TestBean();
		
		TestBean copy = (TestBean)testBean.clone();
	}
	
}
```
执行结果
```
Exception in thread "main" java.lang.CloneNotSupportedException: com.test.TestBean
	at java.lang.Object.clone(Native Method)
	at com.test.TestBean.main(TestBean.java:11)
```
仔细查看clone方法的注释：表明此类不支持Cloneable接口的话就会报错。
```
@exception  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
```
Cloneable接口源码
```
package java.lang;

/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * <tt>Object.clone</tt> (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the <tt>clone</tt> method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   JDK1.0
 */
public interface Cloneable {
}
```
* 类实现了Cloneable接口后Object的clone()方法才可以合法地对该类实例进行按字段复制；
* 如果在没有实现Cloneable接口的实例上调用Object的clone()方法，则会导致抛出CloneNotSupporteddException；
* 实现此接口的类应该使用公共方法重写Object的clone()方法，Object的clone()方法是一个受保护的方法；
因此想实现clone的话，除了继承Object类外，还需要实现Cloneable接口;


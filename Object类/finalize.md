# Object的finalize
```
如果类中重写了finalize方法，当该类对象被回收时，finalize方法有【可能】会被触发。
```
### 源码
```java
 /**
     * Called by the garbage collector on an object when garbage collection
     * determines that there are no more references to the object.
     * A subclass overrides the {@code finalize} method to dispose of
     * system resources or to perform other cleanup.
     * <p>
     * The general contract of {@code finalize} is that it is invoked
     * if and when the Java&trade; virtual
     * machine has determined that there is no longer any
     * means by which this object can be accessed by any thread that has
     * not yet died, except as a result of an action taken by the
     * finalization of some other object or class which is ready to be
     * finalized. The {@code finalize} method may take any action, including
     * making this object available again to other threads; the usual purpose
     * of {@code finalize}, however, is to perform cleanup actions before
     * the object is irrevocably discarded. For example, the finalize method
     * for an object that represents an input/output connection might perform
     * explicit I/O transactions to break the connection before the object is
     * permanently discarded.
     * <p>
     * The {@code finalize} method of class {@code Object} performs no
     * special action; it simply returns normally. Subclasses of
     * {@code Object} may override this definition.
     * <p>
     * The Java programming language does not guarantee which thread will
     * invoke the {@code finalize} method for any given object. It is
     * guaranteed, however, that the thread that invokes finalize will not
     * be holding any user-visible synchronization locks when finalize is
     * invoked. If an uncaught exception is thrown by the finalize method,
     * the exception is ignored and finalization of that object terminates.
     * <p>
     * After the {@code finalize} method has been invoked for an object, no
     * further action is taken until the Java virtual machine has again
     * determined that there is no longer any means by which this object can
     * be accessed by any thread that has not yet died, including possible
     * actions by other objects or classes which are ready to be finalized,
     * at which point the object may be discarded.
     * <p>
     * The {@code finalize} method is never invoked more than once by a Java
     * virtual machine for any given object.
     * <p>
     * Any exception thrown by the {@code finalize} method causes
     * the finalization of this object to be halted, but is otherwise
     * ignored.
     *
     * @throws Throwable the {@code Exception} raised by this method
     * @jls 12.6 Finalization of Class Instances
     * @see java.lang.ref.WeakReference
     * @see java.lang.ref.PhantomReference
     */
    protected void finalize() throws Throwable {
    }
```
翻译：
```
protected void finalize() throws Throwable当垃圾收集确定不再有对该对象的引用时，垃圾收集器在对象上调用该对象。 一个子类覆盖了处理系统资源或执行其他清理的finalize方法。 
finalize的一般合同是，如果Java¢虚拟机已经确定不再有任何方法可以被任何尚未死亡的线程访问的方法被调用，除非是由于最后确定的其他对象或类的准备工作所采取的行动。 finalize方法可以采取任何行动，包括使此对象再次可用于其他线程; 然而， finalize的通常目的是在对象不可撤销地丢弃之前执行清除动作。 例如，表示输入/输出连接的对象的finalize方法可能会在对象被永久丢弃之前执行显式I / O事务来中断连接。 

所述finalize类的方法Object执行任何特殊操作; 它只是返回正常。 Object的Object可以覆盖此定义。 

Java编程语言不能保证哪个线程将为任何给定的对象调用finalize方法。 但是，确保调用finalize的线程在调用finalize时不会持有任何用户可见的同步锁。 如果finalize方法抛出未捕获的异常，则会忽略该异常，并终止该对象的定类。 

在为对象调用finalize方法之后，在Java虚拟机再次确定不再有任何方式可以通过任何尚未被死亡的线程访问此对象的任何方法的情况下，将采取进一步的操作，包括可能的操作由准备完成的其他对象或类别，此时可以丢弃对象。 

finalize方法从不被任何给定对象的Java虚拟机调用多次。 

finalize方法抛出的任何异常都会导致该对象的终止被停止，否则被忽略。 

异常 
Throwable - 这个方法提出的 异常 
另请参见： 
WeakReference ， PhantomReference 
See The Java™ Language Specification: 
12.6类实例的定稿 
```
下面通过一个例子说明finalize方法对垃圾回收有什么影响。

# Wait和Notify
![](https://github.com/lk6678979/image/blob/master/wait-notify.jpg) 
## 1. wait方法
* void wait() 导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法。  
* void wait(long timeout) 导致当前线程等待，直到另一个线程调用 notify()方法或该对象的 notifyAll()方法，或者指定的时间已过会自动唤醒。  
* void wait(long timeout, int nanos) 导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法，或者某些其他线程中断当前线程，或一定量的实时时间。  
### 来看看wait源码
```java
/**
     * Causes the current thread to wait until another thread invokes the
     * {@link java.lang.Object#notify()} method or the
     * {@link java.lang.Object#notifyAll()} method for this object.
     * In other words, this method behaves exactly as if it simply
     * performs the call {@code wait(0)}.
     * <p>
     * The current thread must own this object's monitor. The thread
     * releases ownership of this monitor and waits until another thread
     * notifies threads waiting on this object's monitor to wake up
     * either through a call to the {@code notify} method or the
     * {@code notifyAll} method. The thread then waits until it can
     * re-obtain ownership of the monitor and resumes execution.
     * <p>
     * As in the one argument version, interrupts and spurious wakeups are
     * possible, and this method should always be used in a loop:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait();
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @throws IllegalMonitorStateException if the current thread is not
     *                                      the owner of the object's monitor.
     * @throws InterruptedException         if any thread interrupted the
     *                                      current thread before or while the current thread
     *                                      was waiting for a notification.  The <i>interrupted
     *                                      status</i> of the current thread is cleared when
     *                                      this exception is thrown.
     * @see java.lang.Object#notify()
     * @see java.lang.Object#notifyAll()
     */
    public final void wait() throws InterruptedException {
        wait(0);
    }
```
翻译：
```
wait
public final void wait()throws InterruptedException
导致当前线程等待，直到另一个线程调用该对象的notify()方法或notifyAll()方法。 换句话说，这个方法的行为就好像简单地执行呼叫wait(0) 。 
当前的线程必须拥有该对象的监视器。 该线程释放此监视器的所有权，并等待另一个线程通知等待该对象监视器的线程通过调用notify方法或notifyAll方法notifyAll 。 
然后线程等待，直到它可以重新获得监视器的所有权并恢复执行。 

像在一个参数版本中，中断和虚假唤醒是可能的，并且该方法应该始终在循环中使用： 

  synchronized (obj) {
         while (<condition does not hold>)
             obj.wait();
         ... // Perform action appropriate to condition
     } 
该方法只能由作为该对象的监视器的所有者的线程调用。 有关线程可以成为监视器所有者的方式的说明，请参阅notify方法。 
异常：
IllegalMonitorStateException - 如果当前线程不是对象监视器的所有者。 
InterruptedException - 如果任何线程在当前线程等待通知之前或当前线程中断当前线程。 当抛出此异常时，当前线程的中断状态将被清除。 
另请参见： 
notify() ， notifyAll() 
```
### 来看看notify源码
```
  /**
     * Wakes up a single thread that is waiting on this object's
     * monitor. If any threads are waiting on this object, one of them
     * is chosen to be awakened. The choice is arbitrary and occurs at
     * the discretion of the implementation. A thread waits on an object's
     * monitor by calling one of the {@code wait} methods.
     * <p>
     * The awakened thread will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened thread will
     * compete in the usual manner with any other threads that might be
     * actively competing to synchronize on this object; for example, the
     * awakened thread enjoys no reliable privilege or disadvantage in being
     * the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. A thread becomes the owner of the
     * object's monitor in one of three ways:
     * <ul>
     * <li>By executing a synchronized instance method of that object.
     * <li>By executing the body of a {@code synchronized} statement
     * that synchronizes on the object.
     * <li>For objects of type {@code Class,} by executing a
     * synchronized static method of that class.
     * </ul>
     * <p>
     * Only one thread at a time can own an object's monitor.
     *
     * @throws IllegalMonitorStateException if the current thread is not
     *                                      the owner of this object's monitor.
     * @see java.lang.Object#notifyAll()
     * @see java.lang.Object#wait()
     */
    public final native void notify();

    /**
     * Wakes up all threads that are waiting on this object's monitor. A
     * thread waits on an object's monitor by calling one of the
     * {@code wait} methods.
     * <p>
     * The awakened threads will not be able to proceed until the current
     * thread relinquishes the lock on this object. The awakened threads
     * will compete in the usual manner with any other threads that might
     * be actively competing to synchronize on this object; for example,
     * the awakened threads enjoy no reliable privilege or disadvantage in
     * being the next thread to lock this object.
     * <p>
     * This method should only be called by a thread that is the owner
     * of this object's monitor. See the {@code notify} method for a
     * description of the ways in which a thread can become the owner of
     * a monitor.
     *
     * @throws IllegalMonitorStateException if the current thread is not
     *                                      the owner of this object's monitor.
     * @see java.lang.Object#notify()
     * @see java.lang.Object#wait()
     */
    public final native void notifyAll();
```
翻译：
```
public final void notify()
唤醒正在等待对象监视器的单个线程。 如果任何线程正在等待这个对象，其中一个被选择被唤醒。 选择是任意的，并且由实施的判断发生。 线程通过调用wait方法之一等待对象的监视器。 
唤醒的线程将无法继续，直到当前线程放弃此对象上的锁定为止。 唤醒的线程将以通常的方式与任何其他线程竞争，这些线程可能正在积极地竞争在该对象上进行同步; 例如，唤醒的线程在下一个锁定该对象的线程中没有可靠的权限或缺点。 

该方法只能由作为该对象的监视器的所有者的线程调用。 线程以三种方式之一成为对象监视器的所有者： 
通过执行该对象的同步实例方法。 
通过执行在对象上synchronized synchronized语句的正文。 
对于类型为Class,的对象，通过执行该类的同步静态方法。 
一次只能有一个线程可以拥有一个对象的显示器。 

异常 
IllegalMonitorStateException - 如果当前线程不是此对象的监视器的所有者。 
另请参见： 
notifyAll() ， wait() 

```
```
public final void notifyAll()
唤醒正在等待对象监视器的所有线程。 线程通过调用wait方法之一等待对象的监视器。 
唤醒的线程将无法继续，直到当前线程释放该对象上的锁。 唤醒的线程将以通常的方式与任何其他线程竞争，这些线程可能正在积极地竞争在该对象上进行同步; 例如，唤醒的线程在下一个锁定该对象的线程中不会有可靠的特权或缺点。 

该方法只能由作为该对象的监视器的所有者的线程调用。 有关线程可以成为监视器所有者的方法的说明，请参阅notify方法。 

异常 
IllegalMonitorStateException - 如果当前线程不是此对象的监视器的所有者。 
另请参见： 
notify() ， wait() 
```

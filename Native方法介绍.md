# Java中Native关键字
## 1. 了解Native的概念
  在JVM的体系结构中有一个Java Native Interface模块，称为Java本地库接口，它的作用是融合不同的编程语言为Java所用。Java是一个跨平台的语言，既然是跨了平台，所付出的代价就是牺牲一些对底层的控制，而Java要实现对底层的控制，就需要借助一些其他语言的方法，这个就是native的作用。
## 2. Java为什么要使用Native方法
  因为要兼顾到跨平台，所以很多对底层的操作就不能有太强的耦合。有时java需要与java外面的环境交互，比如底层的操作系统或者某些硬件，这也是本地方法存在的主要原因。通过使用本地方法，我们只需要调用它暴露给我们的接口，而不需要了解java应用之外的繁琐的细节。
## 3. 什么是native方法 
  一个Native method就是一个java调用非java代码的接口：该方法由非java语言实现，比如C、C++等。 
   Java中的本地方法栈就是这种功能，它与java虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行 Java 方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的 Native 方法服务。  
   Navtive 方法是 Java 通过 JNI（Java本地库接口） 直接调用本地 C/C++ 库，Native方法相当于 C/C++ 暴露给 Java 的一个接口，Java 通过调用这个接口从而调用到 C/C++ 方法。当线程调用 Java 方法时，虚拟机会创建一个栈帧并压入 Java 虚拟机栈。然而当它调用的是 native 方法时，虚拟机会保持 Java 虚拟机栈不变，也不会向 Java 虚拟机栈中压入新的栈帧，虚拟机只是简单地动态连接并直接调用指定的 native 方法。
   在定义一个native method时，并不提供实现体（有些像定义一个java interface），因为其实现体是由非java语言在外面实现的。下面给了一个示例：
```java
package java.lang;
 
public class Object { 
    
    ......
    
    public final native Class<?> getClass(); 
    
    public native int hashCode(); 
    
    protected native Object clone() throws CloneNotSupportedException; 
    
    public final native void notify(); 
    
    public final native void notifyAll(); 
    
    public final native void wait(long timeout) throws InterruptedException; 

    ......
 
} 
```
 native可以与所有其他的java标识符连用，abstract除外。这是合理的，因为native暗示这些方法室友实现体的，只不过它们不是用java实现的，但abstract却显然的指明这些方法无实现体。
 native修饰的方法并不会给调用它的类造成任何影响，它与其他method一样可以返回任何java类型。我们可以将它看做是一个正常的java方法来使用。
  
## 4. JAVA如何加载和使用native申明的外部方法
·  如果一个方法描述符内有native，那么这个方法将成为Java Native Interface中的一部分，这个描述符块将有一个指向该方法的实现的指针。  
·  这些实现在一些DLL文件内，DLL会被操作系统加载到java程序的地址空间，这个空间称为Java Native Method Libraries。  
·  当一个带有本地方法的类被加载时，其相关的DLL并未被加载，因此指向方法实现的指针并不会被设置。当本地方法被调用之前，这些DLL才会被加载，这是通过调用java.system.loadLibrary()实现的。

## 5. JAVA如何加载DLL


# 设计模式之单例模式详解（Java实现）

## 单例模式介绍

&lt;font color=&#34;red&#34;&gt;单例模式（Singleton Pattern）确保一个类只有一个实例，并提供一个全局访问点来访问这个唯一实例。&lt;/font&gt;

例如Windows任务管理器，在正常情况下只能打开唯一一个任务管理器。

![image-20220422171912798](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7673d2c6f2f6f6fb57a2cf40c1227fa6.png)

单例模式是一种对象创建型模式，&lt;font color=&#34;red&#34;&gt;其有三个要点：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。&lt;/font&gt;

&gt;**主要解决：** 一个全局使用的类频繁地创建与销毁。
&gt;
&gt;**何时使用：** 当您想控制实例数目，节省系统资源的时候。
&gt;
&gt;**如何解决：** 判断系统是否已经有这个单例，如果有则返回，如果没有则创建。
&gt;
&gt;**关键代码：** 构造函数是私有的。
&gt;
&gt;**应用实例：**
&gt;
&gt;- 1、一个班级只有一个班主任。
&gt;- 2、Windows 是多进程多线程的，在操作一个文件的时候，就不可避免地出现多个进程或线程同时操作一个文件的现象，所以所有文件的处理必须通过唯一的实例来进行。
&gt;- 3、一些设备管理器常常设计为单例模式，比如一个电脑有两台打印机，在输出的时候就要处理不能两台打印机打印同一个文件。
&gt;
&gt;**优点：**
&gt;
&gt;- 1、在内存里只有一个实例，减少了内存的开销，尤其是频繁的创建和销毁实例（比如管理学院首页页面缓存）。
&gt;- 2、避免对资源的多重占用（比如写文件操作）。
&gt;
&gt;**缺点：** 没有接口，不能继承，与单一职责原则冲突，一个类应该只关心内部逻辑，而不关心外面怎么样来实例化。
&gt;
&gt;**使用场景：**
&gt;
&gt;- 1、要求生产唯一序列号。
&gt;- 2、WEB 中的计数器，不用每次刷新都在数据库里加一次，用单例先缓存起来。
&gt;- 3、创建的一个对象需要消耗的资源过多，比如 I/O 与数据库的连接等。
&gt;
&gt;**注意事项：** getInstance() 方法中需要使用同步锁 synchronized (Singleton.class) 防止多线程同时进入造成 instance 被多次实例化。

## 单例模式详解

### 单例模式结构

单例模式是结构最简单的设计模式，它只包含一个类，即单例类。单例模式的结构图如下。

![image-20220422172423873](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/05a90834692be4f0aa7018c8c2ba283e.png)

由图可知，单例模式只包含一个单例角色，也就是Singleton。对于Singleton（单例），在单例类的内部创建它的唯一实例，并通过静态方法getInstance()让客户端可以使用它的唯一实例；&lt;font color=&#34;red&#34;&gt;为了防止在外部对单例类实例化，将其构造函数的可见性设为private；在单例类内部定义了一个Singleton类型的静态对象作为可供外部访问的唯一实例。&lt;/font&gt;

### 单例模式实现

典型的单例模式的实现代码如下：

```java
public class Singleton {
    // 静态私有成员变量
    private static Singleton instance = null;  
    // 私有构造函数
    private Singleton() {   
    }
    
    // 静态公有工厂方法，返回唯一实例
    public static Singleton getInstance() {
        if(instance == null)
            instance = new Singleton(); 
        return instance;
    }
}
```

### 单例模式应用举例

* **题目描述**

	&gt; 某软件公司承接了一个服务器负载均衡(Load Balance)软件的开发工作，该软件运行在一台负载均衡服务器上，可以将并发访问和数据流量分发到服务器集群中的多台设备上进行并发处理，提高了系统的整体处理能力，缩短了响应时间。由于集群中的服务器需要动态删减，且客户端请求需要统一分发，因此需要确保负载均衡器的唯一性，只能有一个负载均衡器来负责服务器的管理和请求的分发，否则将会带来服务器状态的不一致以及请求分配冲突等问题。如何确保负载均衡器的唯一性是该软件成功的关键，试使用单例模式设计服务器负载均衡器。

* **UML类图**

	![image-20220422195354566](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/489e751ac4d98fbc7dc8b14c07a778dd.png)

	其中将负载均衡器LoadBalance设计为单例类，其中包含一个存储服务器信息的集合serverList，每次在serverList中随机选择一台服务器来响应客户端的请求。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/singleton_pattern)

## 饿汉式单例与懒汉式单例

### 饿汉式单例

饿汉式单例（Eager Singleton）是实现起来最简单的单例类，饿汉式单例类结构图如下。

![image-20220422200347381](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/dac299c3b0047c5dfba0d9b77b892642-20231125210120603.png)

有图中我们可以看出，由于在定义静态变量的时候实例化单例类，因此在类加载时单例对象就已创建，代码如下：

```java
public class EagerSingleton { 
    private static final EagerSingleton instance = new EagerSingleton(); 
    private EagerSingleton() { } 

    public static EagerSingleton getInstance() {
        return instance; 
    }
}
```

当类被加载时，静态变量instance会被初始化，此时类的私有构造函数就会被调用，单例类的唯一实例将被创建。

### 懒汉式单例与双重检查锁定

与饿汉式单例相同的是，懒汉式单例（Lazy Singleton）的构造函数也是私有的；与饿汉式单例类不同的是，懒汉式单例类在第一次被引用时将自己实例化，在懒汉式单例类被加载时不会将自己实例化。懒汉式单例类的结构图如下。

![image-20220422200830831](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/6b48d0ed3518e55d11b9060f9b8f2156-20231125210127626.png)

&lt;font color=&#34;red&#34;&gt;但如果多个线程同时访问将导致创建多个单例对象！&lt;/font&gt;这个时候为了避免多个线程同时调用getInstance()方法，可以使用关键字synchronized，代码如下：

```java
public class LazySingleton { 
    private static LazySingleton instance = null; 

    private LazySingleton() { } 

    // 使用synchronized关键字对方法加锁，确保任意时刻只有一个线程可以执行该方法
    synchronized public static LazySingleton getInstance() { 
        if (instance == null) {
            instance = new LazySingleton(); 
        }
        return instance; 
    }
}
```

在上述懒汉式单例类中，在getInstance()方法前面增加了关键字synchronized进行线程锁定，已处理多个线程同时访问问题。但我们每次调用getInstance()时都需要进行线程锁定判断，在多线程高并发环境中将会导致性能大大降低。因此可以继续对懒汉式单例进行改进，我们发现无需对getInstance()方法进行锁定，仅需锁定代码段`instance = new LazySingleton()`即可。故可进行如下改进：

```java
public static LazySingleton getInstance() { 
    if (instance == null) {
        synchronized (LazySingleton.class) {
            instance = new LazySingleton(); 
        }
    }
    return instance; 
}
```

问题看似解决，但如果使用上述代码，实际上还是会存在单例对象不唯一的情况。因为线程A和线程B如果同时进入判断，由于锁的原因，一个会先创建，但是另一个并不知道对象已经创建，这样就会导致产生多个实例对象。违背了单例模式的设计思想。我们需要使用双重检查锁定，即在锁内再进行一次`instance == null`的判断。使用双重检查锁定实现的懒汉式单例类的完整代码如下：

```java
public class LazySingleton { 
    private volatile static LazySingleton instance = null; 

    private LazySingleton() { } 

    public static LazySingleton getInstance() { 
        //第一重判断
        if (instance == null) {
            //锁定代码块
            synchronized (LazySingleton.class) {
                //第二重判断
                if (instance == null) {
                    instance = new LazySingleton(); //创建单例实例
                }
            }
        }
    return instance; 
    }
}

```

&lt;font color=&#34;red&#34;&gt;需要注意的时，如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符volatile，被volatile修饰的成员变量可以确保多个线程都能够正确处理。&lt;/font&gt;

### 饿汉式单例类与懒汉式单例类的比较

* 饿汉式单例类：无须考虑多个线程同时访问的问题；调用速度和反应时间优于懒汉式单例；资源利用效率不及懒汉式单例；系统加载时间可能会比较长。
* 懒汉式单例类：实现了延迟加载；必须处理好多个线程同时访问的问题；需通过双重检查锁定等机制进行控制，将导致系统性能受到一定影响。

## 使用静态内部类实现单例模式

饿汉式单例类不能实现延迟加载，不管将来用不用始终占用内存；懒汉式单例类安全控制烦琐，而且性能受影响。可见它们都存在一些问题，为了克服这些问题，在Java语言中可以通过Initialization on Demand Holder（IoDH）技术来实现单例模式。

在IoDH中，需要在单例类中增加一个静态内部类，在该内部类中创建单例对象，再将该单例对象通过getInstance()方法返回给外部使用，实现代码如下：

```java
//Initialization on Demand Holder
public class Singleton {
    private Singleton() {
    }

    //静态内部类
    private static class HolderClass {
        private final static Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return HolderClass.instance;
    }

    public static void main(String args[]) {
        Singleton s1, s2; 
        s1 = Singleton.getInstance();
        s2 = Singleton.getInstance();
        System.out.println(s1==s2);
    }
}
```

通过使用IoDH既可以实现延迟加载，又可以保证线程安全，不影响系统性能，不失为一种最好的Java语言单例模式实现方式；其缺点是与编程语言本身的特性相关，很多面向对象语言并不支持IoDH。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/04.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


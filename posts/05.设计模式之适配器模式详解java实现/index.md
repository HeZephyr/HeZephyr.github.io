# 设计模式之适配器模式详解（Java实现）

## 适配器模式介绍

![image-20220422215001021](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/b833aba52d5a779a04c7488e5d964277-20231125211243243.png)

在现实生活中生活用电220V和笔记电脑20V不兼容，我们需要引入 AC Adapter（交流电适配器），在软件开发中我们也会存在不兼容的结构，这个时候就需要引入适配器模式。

适配器模式（Adapter Pattern）可以将一个类的接口和另一个类的接口匹配起来，而无需修改原来的适配者接口和抽象目标接口。

&lt;font color=&#34;red&#34;&gt;它将一个类的接口转换称客户希望的另一个接口，让那些接口不兼容的类可以一起工作。&lt;/font&gt;

适配器模式的别名为包装器模式（Wrapper Pattern），它既可以作为类结构模式，也可以作为对象结构型模式。在适配器模式的定义中所提及的接口是指广义的接口，它可以表示为方法或者方法的集合。

&gt; **主要解决：** 主要解决在软件系统中，常常要将一些&#34;现存的对象&#34;放到新的环境中，而新环境要求的接口是现对象不能满足的。
&gt;
&gt; **何时使用：** 1、系统需要使用现有的类，而此类的接口不符合系统的需要。 2、想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。 3、通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）
&gt;
&gt; **如何解决：** 继承或依赖（推荐）。
&gt;
&gt; **关键代码：** 适配器继承或依赖已有的对象，实现想要的目标接口。
&gt;
&gt; **应用实例：** 1、美国电器 110V，中国 220V，就要有一个适配器将 110V 转化为 220V。 2、JAVA JDK 1.1 提供了 Enumeration 接口，而在 1.2 中提供了 Iterator 接口，想要使用 1.2 的 JDK，则要将以前系统的 Enumeration 接口转化为 Iterator 接口，这时就需要适配器模式。 3、在 LINUX 上运行 WINDOWS 程序。 4、JAVA 中的 jdbc。
&gt;
&gt; **优点：** 1、可以让任何两个没有关联的类一起运行。 2、提高了类的复用。 3、增加了类的透明度。 4、灵活性好。
&gt;
&gt; **缺点：** 1、过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。 2.由于 JAVA 至多继承一个类，所以至多只能适配一个适配者类，而且目标类必须是抽象类。
&gt;
&gt; **使用场景：** 有动机地修改一个正常运行的系统的接口，这时应该考虑使用适配器模式。
&gt;
&gt; **注意事项：** 适配器不是在详细设计时添加的，而是解决正在服役的项目的问题。

## 适配器模式详解

### 适配器模式结构

类适配器模式的结构图如下：

![image-20220423235933375](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/ef6ae55bab0ce79b1d62d9aab38cd62f-20231125211249068.png)

对象适配器模式的结构图如下：

![image-20220424000012210](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/c4a91f039a8904ea70d69c2ac37fe206.png)

由上图可知，适配器模式包含以下3个角色。

1. Target（目标抽象类）：目标抽象类定义客户所需的接口，可以是一个抽象类或者接口，也可以是具体类。在类适配器中，由于Java语言不支持多重继承，它只能是接口。
2. Adapter（适配器类）：它可以调用另一个接口 ，作为一个转换器，对Adaptee和Target进行适配。适配器Adapter是适配器模式的核心，在类适配器中，它通过实现Target接口并继承Adaptee类来使二者产生联系，在对象适配器中，它通过继承Target并关联一个Adaptee对象来使二者产生联系。
3. Adaptee（适配者类）：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体的类，包含了客户希望使用的业务方法，在某些情况下甚至没有适配者类的代码。

### 适配器模式实现

#### 类适配器

根据上图，在类适配器中适配者类Adaptee没有request方法，而客户端期待这个方法，但在适配者类中实现了specificRequest()方法，该方法提供的实现正式客户端所需要的。&lt;font color=&#34;red&#34;&gt;为了使客户端能够使用适配者类，提供了一个中间类，即适配器类Adapter，适配器类实现了抽象目标类接口Target，并继承了适配者类，在适配器类的request()方法中调用所继承的适配者类的specificRequest()方法，达到了适配的目的。&lt;/font&gt;

因为适配器类与适配者类使继承关系，所以这种适配器模式称为类适配器模式。典型的类适配器代码如下：

```java
public class Adapter extends Adaptee implements Target {
    public void request() {
        super.specificRequest();
    }
}
```

#### 对象适配器

根据上图，在对象适配器中适配者类Adaptee没有request方法，而客户端期待这个方法，但在适配者类中实现了specificRequest()方法，该方法提供的实现正式客户端所需要的。为了使客户端能够使用适配者类，需要提供一个包装类Adapter，即适配器类。

这个包装类包装了一个适配者的实例，从而将客户端与适配者衔接起来，在适配器的request()方法中调用适配者的specificRequest()方法。

因为适配器类与适配者类是关联关系，所以这种适配器模式称为对象适配器模式。典型的对象适配器代码如下：

```java
public class Adapter extends Target {
    // 维持一个对适配者对象的引用
    private Adaptee adaptee;
    
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    public void request() {
        // 转发调用
        adaptee.specificRequest();
    }
}
```

### 适配器模式应用举例

* **题目描述**

	&gt;某公司欲开发一款儿童玩具汽车，为了更好地吸引小朋友的注意力，该玩具汽车在移动过程中伴随着灯光闪烁和声音提示。在该公司以往的产品中已经实现了控制灯光闪烁（例如警灯闪烁）和声音提示（例如警笛音效）的程序，为了重用先前的代码并且使得汽车控制软件具有更好的灵活性和扩展性，现使用适配器模式设计该玩具汽车控制软件。

* **UML类图**

	使用对象适配器模式来实现，其UML类图如下：

	![image-20220424183227232](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/fe3c7c70cabdff83879a158ba1f3d214-20231125211259811.png)

	其中，CarController类充当抽象目标，PoliceSound和PoliceLamp类充当适配者，PoliceCarAdapter充当适配器。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/adapter_pattern/object_adapter)

## 缺省适配器模式

缺省适配器模式(Default Adapter Pattern)：当不需要实现一个接口所提供的所有方法时，可先设计一个抽象类实现该接口，并为接口中每个方法提供一个默认实现（空方法），那么该抽象类的子类可以选择性地覆盖父类的某些方法来实现需求，它适用于不想使用一个接口中的所有方法的情况，又称为单接口适配器模式。其结构图如下：

![image-20220424094138258](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bcfad8998fb4a4f6edd2269ce17e14ad.png)

由上图可知，在缺省适配器模式中包含以下3个角色：

1. ServiceInterface（适配者接口）：它是一个接口，通常在接口中声明了大量的方法。
2. AbstractServiceClass（缺省适配器类）：它是缺省适配器模式的核心类，使用空方法的形式实现了在ServiceInterface接口中声明的方法。通常将它定义为抽象类，因为对它进行实例化没有任何意义。
3. ConcreteServiceClass（具体业务类）：它是缺省适配器类的子类，在没有引入适配器之前它需要实现适配者接口，因此需要实现在适配者接口中定义的所有方法，而对于一些无须使用的方法不得不提供空实现。在有了缺省适配器之后可以直接继承该适配者类，根据需要有选择性地覆盖在适配器类中定义的方法。

缺省适配器类的典型代码如下：

```java
public abstract class AbstractServiceClass implements ServiceInterface {
    public void serviceMethod1() {  }  //空方法
    public void serviceMethod2() {  }  //空方法
    public void serviceMethod3() {  }  //空方法
}
```

## 双向适配器

在对象适配器的使用过程中，如果在适配器中同时包含对目标类和适配者类的引用，适配者可以通过它调用目标类中的方法，目标类也可以通过它调用适配者类的方法，那么该适配器就是一个双向适配器。其结构图如下：

![image-20220424095546821](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/70a63549103f1c8e28e7ac3adb8cd8d8-20231125211308022.png)

其典型代码如下：

```java
public class Adapter implements Target,Adaptee {
    // 同时维持对抽象目标类和适配者类的引用
    private Target target;
    private Adaptee adaptee;
  
    public Adapter(Target target) {
        this.target = target;
    }
  
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
  
    public void request() {
        adaptee.specificRequest();
    }
  
    public void specificRequest() {
        target.request();
    }
}
```


---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/05.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


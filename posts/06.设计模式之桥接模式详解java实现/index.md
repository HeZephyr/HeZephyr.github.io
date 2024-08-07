# 设计模式之桥接模式详解（Java实现）

## 桥接模式介绍

毛笔和蜡笔是两种很常见的文具，它们都归属于画笔。假设我们需要大、中、小3种型号的画笔，能够绘制12种不同的颜色。那么我们的解决方案如下：

![image-20220424191437477](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/778fc90ed46e2c273ccb2ad4cafc080c-20231125211551147.png)

显然，在毛笔中，我们将颜色和型号进行了分离，增加新的颜色或者型号对另一方没有影响，即用软件工程中的术语，可以认为在蜡笔中颜色和型号之间存在较强的耦合性，而毛笔很好地将二者解耦，使用起来非常灵活，扩展也较为方便。

在软件开发中也有一种设计模式可以用来处理上述类似的具有多变化维度的情况，它就是桥接模式（Bridge Pattern）。它将抽象部分与它的实现部分解耦，使得两者都能独立变化。

桥接模式又被称为柄体(Handle and Body)模式或接口(Interface)模式，用抽象关联取代了传统的多层继承，将类之间的静态继承关系转换为动态的对象组合关系。

&gt; **主要解决：** 在有多种可能会变化的情况下，用继承会造成类爆炸问题，扩展起来不灵活。
&gt;
&gt; **何时使用：** 实现系统可能有多个角度分类，每一种角度都可能变化。
&gt;
&gt; **如何解决：**把这种多角度分类分离出来，让它们独立变化，减少它们之间耦合。
&gt;
&gt; **关键代码：** 抽象类依赖实现类。
&gt;
&gt; **应用实例：** 1、猪八戒从天蓬元帅转世投胎到猪，转世投胎的机制将尘世划分为两个等级，即：灵魂和肉体，前者相当于抽象化，后者相当于实现化。生灵通过功能的委派，调用肉体对象的功能，使得生灵可以动态地选择。 2、墙上的开关，可以看到的开关是抽象的，不用管里面具体怎么实现的。
&gt;
&gt; **优点：** 1、抽象和实现的分离。 2、优秀的扩展能力。 3、实现细节对客户透明。
&gt;
&gt; **缺点：**桥接模式的引入会增加系统的理解与设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计与编程。
&gt;
&gt; **使用场景：** 1、如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。 2、对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用。 3、一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。
&gt;
&gt; **注意事项：** 对于两个独立变化的维度，使用桥接模式再适合不过了。

## 桥接模式详解

### 桥接模式结构

桥接模式的结构图如下：

![image-20220424192119428](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e186c443621072881aa345cbbad3807d.png)

由上图可知，桥接模式包括以下4个角色。

1. Abstraction（抽象类）：它是用于定义抽象类的接口，通常是抽象类而不是接口，其中定义了一个Implementor（实现类接口）类型的对象并可以维护该对象，&lt;font color=&#34;red&#34;&gt;它与Implementor之间具有关联关系，它既可以包含抽象业务方法，也可以包含具体业务方法。&lt;/font&gt;
2. RefinedAbstraction（扩充抽象类）：它扩充由Abstraction定义的接口，通常情况下它不再是抽象类而是具体类，实现了在Abstraction中声明的抽象业务方法，在RefinedAbstraction中可以调用在Implementor中定义的业务方法。
3. Implementor（实现类接口）：它是定义实现类的接口，这个接口不一定要与Abstraction的接口完全一致，事实上这两个接口可以完全不同。一般而言，Implementor接口仅提供基本操作，而Abstraction定义的接口可能会做更多更复杂的操作。&lt;font color=&#34;red&#34;&gt;Implementor接口对这些基本操作进行了声明，而具体实现交给其子类。通过关联关系，在Abstraction中不仅拥有自己的方法，还可以调用到Implementor中定义的方法，使用关联关系代替继承关系。&lt;/font&gt;
4. ConcreteImplementor（具体实现类）：它具体实现了Implementor接口，在不同的ConcreteImplementor中提供基本操作的不同实现，在程序运行时ConcreteImplementor对象将替换其父类对象，提供给抽象类具体的业务操作方法。

### 桥接模式实现

在具体编码实现时，由于在桥接模式中存在两个独立变化的维度，为了降低两者之间的耦合度，首先需要针对两个不同的维度提取抽象类和实现类接口，并建立一个抽象关联关系。对于“实现部分维度”，典型的接口代码如下：

```java
public interface Implementor {
    public void operationImpl();
}
```

在实现Implementor接口的子类ConcreteImplementor中实现了在该接口中声明的方法，用于定义与该维度相对应的一些具体方法，代码如下：

```java
public class ConcreteImplementor implements Implementor {
    public void operationImpl() {
        //具体业务方法的实现
    }
}
```

对于另一“抽象部分”维度而言，其典型的抽象类代码如下：

```java
public abstract class Abstraction {
    protected Implementor impl; //定义实现类接口对象

    public void setImpl(Implementor impl) {
        this.impl = impl;
    }

    public abstract void operation(); //声明抽象业务方法
}
```

在抽象类Abstraction中定义了一个实现类接口类型的成员对象impl，再通过Setter方法或者构造方法以注入的方式给该对象赋值。一般将该对象的可见性定义为protected，以便在其子类中访问Implementor的方法，其子类一般称为扩充对象类或细化抽象类（RefinedAbstraction），典型的RefinedAbstraction类代码如下：

```java
public class RefinedAbstraction extends Abstraction {
    public void operation() {
        //业务代码
        impl.operationImpl(); //调用实现类的方法
        //业务代码
    }
}
```

对于客户端而言，可以针对两个维度的抽象层编程，在程序运行时再动态确定两个维度的子类，动态组合对象，将两个独立变化的维度完全解耦，以便能够灵活地扩充任一维度而对另一维度不造成任何影响。

### 桥接模式应用实例

* **题目描述**

	&gt; 某软件公司欲开发一个数据转换工具，可以将数据库中的数据转换成多种文件格式，例如TXT、XML、PDF等格式，同时该工具需要支持多种不同的数据库。试用桥接模式对其进行设计。

* **UML类图**

	![image-20220424202329537](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/2257ba4f82dc4651efc4d6991df7529c-20231125211559159.png)

	其中，FileConvertor 充当抽象类角色，TXTConvertor、XMLConvertor 和 PDFConvertor
	充当扩充抽象类角色，DataHandler 充当实现类接口角色，OracleHandler 和SQLServerHandler充当具体实现类角色。  

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/bridge_pattern)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/06.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


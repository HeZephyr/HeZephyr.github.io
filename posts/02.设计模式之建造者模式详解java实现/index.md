# 设计模式之建造者模式详解（Java实现）

## 建造者模式介绍

建造者模式（Builder Pattern）使用多个简单的对象一步一步构建成一个复杂的对象。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

一个 Builder 类会一步一步构造最终的对象。该 Builder 类是独立于其他对象的。

&gt; **意图：** 将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。
&gt;
&gt; **主要解决：** 主要解决在软件系统中，有时候面临着&#34;一个复杂对象&#34;的创建工作，其通常由各个部分的子对象用一定的算法构成；由于需求的变化，这个复杂对象的各个部分经常面临着剧烈的变化，但是将它们组合在一起的算法却相对稳定。
&gt;
&gt; **何时使用：** 一些基本部件不会变，而其组合经常变化的时候。
&gt;
&gt; **如何解决：** 将变与不变分离开。
&gt;
&gt; **关键代码：** 建造者：创建和提供实例，导演：管理建造出来的实例的依赖关系。
&gt;
&gt; **应用实例：** 1、去肯德基，汉堡、可乐、薯条、炸鸡翅等是不变的，而其组合是经常变化的，生成出所谓的&#34;套餐&#34;。 2、JAVA 中的 StringBuilder。
&gt;
&gt; **优点：** 1、建造者独立，易扩展。 2、便于控制细节风险。
&gt;
&gt; **缺点：** 1、产品必须有共同点，范围有限制。 2、如内部变化复杂，会有很多的建造类。
&gt;
&gt; **使用场景：** 1、需要生成的对象具有复杂的内部结构。 2、需要生成的对象内部属性本身相互依赖。
&gt;
&gt; **注意事项：** 与工厂模式的区别是：建造者模式更加关注与零件装配的顺序。

## 建造者模式详解

### 建造者模式结构

建造者模式的UML类图如下：

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/40c35512ccb37dfcc48e4c95537583df-20231125205916176.gif)

由上图可知，建造者模式包含以下4个角色。

1. Builder（抽象建造者）：它为创建一个产品对象的各个部件指定抽象接口，在该接口中一般声明两类方法，一类方法是buildPartX()（如图中的buildPartA()、buildPathB()等），它们用于创建复杂对象的各个部件；另一类方法是getResult（），它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。
2. ConcreteBuilder（具体建造者）：它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确所创建的复杂对象，还可以提供一个方法返回创建好的复杂产品对象（该方法也可以由抽象建造者实现）。
3. Product（产品）：它是被构建的复杂对象，包含多个组成部件，具体建造者创建该产品的内部表示并定义它的装配过程。
4. Director（指挥者）：指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。

### 建造者模式实现

典型的复杂对象类的代码如下：

```java
public class Product {
    private String partA; // 定义部件，部件可以是任意类型，包括值类型和引用类型
    private String partB;
    private String partC;
    // 属性的Getter和Setter方法省略
}
```

典型的抽象建造者类的代码如下：

```java
public abstract class Builder {
    // 创建产品对象
    protected Product product = new Product();
    
    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract void buildPartC();
    
    // 返回产品对象
    public Product getResult() {
        return product;
    }
}
```

典型的具体建造者类的代码如下：

```java
public class ConcreteBuilder1 extends Builder {
    public void buildPartA() {
        product.setPartA(&#34;A1&#34;);
    }
    public void buildPartB() {
        product.setPartA(&#34;B1&#34;);
    }
    public void buildPartC() {
        product.setPartA(&#34;C1&#34;);
    }
}
```

典型的指挥者类的代码如下：

```java
public class Director {
    private Builder builder;
    
    public Director(Builder builder) {
        this.builder = builder;
    }
    public void setBuilder(Builder builder) {
        this.builder = builder;
    }
    // 产品的构建与组装方法
    public Product construct() {
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getResult();
    }
}
```

### 建造者模式应用举例

* **题目描述**

	&gt; 计算机组装工厂可以将CPU、内存、硬盘、主机、显示器等硬件设备组装在一起构成一台完整的计算机，且构成的计算机可以是笔记本，也可以是台式机，还可以是不提供显示器的服务器主机。对于用户而言，无须关心计算机的组成设备和组装过程，工厂返回给用户的。是完整的计算机对象，使用建造者模式实现计算机组装过程。

* **UML类图**

	![image-20220422112353610](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/1fdeeeff495aee804c759ee81b8094b6.png)

	其中，Computer充当符合产品，ComputerBuilder充当抽象建造者，Notebook、Desktop和Server充当具体建造者，ComputerAssembleDirector充当指挥者，其assemble()方法用于定义产品的构造过程。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/builder_pattern)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/02.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%BB%BA%E9%80%A0%E8%80%85%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


# 设计模式之外观模式详解（Java实现）

## 外观模式概述

在软件开发中，有时候为了完成一项较为复杂的功能，一个客户类需要和多个业务类交互，而这些需要交互的业务类经常会作为一个整体出现，由于设计的类比较多，导致使用代码较为复杂，此时特别需要一个类似服务员的角色，由它来负责和多个业务类进行交互，而客户类只需要与该类进行交互。&lt;font color=&#34;red&#34;&gt;外观模式（Facade Pattern）通过引入一个新的外观类(Facade)来负责和多个业务类【子系统(Subsystem)，所指的子系统是一个广义的概念，它可以是一个类、一个功能模块、系统的一个组成部分或者一个完整的系统】进行交互，而客户类只需与外观类交互。&lt;/font&gt;

![image-20220428170959567](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/63f22a69af86393e476cec1550564a39-20231125211947955.png)

它为子系统中的一组接口提供了一个统一的入口。外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

&gt; **主要解决：** 降低访问复杂系统的内部子系统时的复杂度，简化客户端之间的接口。
&gt;
&gt; **何时使用：** 1、客户端不需要知道系统内部的复杂联系，整个系统只需提供一个&#34;接待员&#34;即可。 2、定义系统的入口。
&gt;
&gt; **如何解决：** 客户端不与系统耦合，外观类与系统耦合。
&gt;
&gt; **关键代码：** 在客户端和复杂系统之间再加一层，这一层将调用顺序、依赖关系等处理好。
&gt;
&gt; **应用实例：** 1、去医院看病，可能要去挂号、门诊、划价、取药，让患者或患者家属觉得很复杂，如果有提供接待人员，只让接待人员来处理，就很方便。 2、JAVA 的三层开发模式。
&gt;
&gt; **优点：** 1、减少系统相互依赖。 2、提高灵活性。 3、提高了安全性。
&gt;
&gt; **缺点：**不符合开闭原则，如果要改东西很麻烦，继承重写都不合适。
&gt;
&gt; **使用场景：** 1、为复杂的模块或子系统提供外界访问的模块。 2、子系统相对独立。 3、预防低水平人员带来的风险。
&gt;
&gt; **注意事项：** 在层次化结构中，可以使用外观模式定义系统中每一层的入口。

## 外观模式详解

### 外观模式结构

外观模式没有一个一般化的类图描述，下图可以用来描述外观模式的结构图。

![image-20220428170935558](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/e6d5eb18385d33361f1831b249bc2c7b.png)

由上图可知，外观模式包含以下两个角色。

1. Facade（外观角色）：在客户端可以调用它的方法，在外观角色中可以知道相关的（一个或者多个）子系统的功能和责任；在正常情况下，它将所有从客户端发来的请求委派到相应的子系统，传递给相应的子系统对象处理。
2. SubSystem（子系统角色）：在软件系统中可以有一个或者多个子系统角色，每一个子系统可以不是一个单独的类，而是一个类的集合，它实现子系统的功能；每一个子系统都可以被客户端直接调用，或者被外观角色调用，它处理由外观类传过来的请求；子系统并不知道外观的存在，对于子系统而言，外观角色仅仅是另外一个客户端而已。

### 外观模式实现

子系统类通常是一些业务类，实现了一些具体的、独立的业务功能，其典型代码如下：

```java
public class SubSystemA {
    public void methodA() {
        //业务实现代码
    }
}
public class SubSystemB {
    public void methodB() {
        //业务实现代码
    }
}
public class SubSystemC {
    public void methodC() {
        //业务实现代码
    }
}
```

外观类的典型代码如下：

```java
public class Facade {
    private SubSystemA obj1 = new SubSystemA();
    private SubSystemB obj2 = new SubSystemB();
    private SubSystemC obj3 = new SubSystemC();

    public void method() {
        obj1.method();
        obj2.method();
        obj3.method();
    }
}
```

由于在外观类中维持了对子系统对象的引用，客户端可以通过外观类来间接调用子系统对象的业务方法，而无须与子系统对象直接交互。在引入外观类后，客户端代码变得非常简单，其典型代码如下：

```java
public class Client {
    public static void main(String args[]) {
        Facade facade = new Facade();
        facade.method();
    }
}
```

### 外观模式应用举例

* **题目描述**

	&gt; 某软件公司要开发一个可应用于多个软件的文件加密模块，该模块可以对文件中的数据进行加密并将加密之后的数据存储在一个新文件中，具体的流程包括3个部分，分别是读取源文件、加密、保存加密之后的文件，其中，读取文件和保存文件使用流来实现，加密操作通过求模运算实现。这3个操作相对独立，为了实现代码的独立重用，让设计更符合单一职责原则，这3个操作的业务代码封装在3个不同的类中。
	&gt; 现使用外观模式设计该文件加密模块。

* **UML类图**

	![image-20220428192930568](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/9dc66aef9ddf76f27b95b5ed2171ef0c-20231125211956431.png)

	其中，EncryptFacade充当外观类，FileReader、CipherMachine和FileWriter充当子系统类。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/facade_pattern)

## 抽象外观类

在标准的外观模式的结构图中，如果需要增加、删除或更换与外观类交互的子系统类，必须修改外观类或客户端的源代码，这将违背开闭原则，因此可以通过引入抽象外观类对系统进行改进，在一定程度上解决该问题。

如2.3中的题目我们需要更换一个加密类，如果我们需要增加一个新的外观类NewEncryptFacade与FileReader类、FileWriter类以及新增加的NewClipherMachine类交互，虽然原有系统类库无须作修改，但是因为客户端代码中原来针对EncryptFacade类进行编程，现在需要修改为NewEncryptFacade类，所以需要修改客户端源代码。如何在不修改客户端代码的前提下使用新的外观类呢？那么我们可以引入一个抽象外观类，客户端针对抽象外观类编程即可。结构图如下：

![image-20220428215237165](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/d01355cacfd19c18222e6de1310393f1.png)

修改后的代码如下：

[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/abstract_facade_pattern)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/09.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%A4%96%E8%A7%82%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


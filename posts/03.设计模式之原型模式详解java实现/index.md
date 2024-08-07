# 设计模式之原型模式详解（Java实现）

## 原型模式介绍

原型模式（Prototype Pattern）是一种对象创建型模式，它是使用原型实例指定待创建对象的类型，并且通过复制这个原型来创建新的对象。

它的工作原理很简单：将一个原型对象传给要发动创建的对象（即客户端对象），这个要发动创建的对象通过请求原型对象复制自己来实现创建过程。

![image-20220422113402622](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7df0dd0d0b7cb94c6ad28059de701ca8-20231125210009217.png)

&gt; **意图：** 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
&gt;
&gt; **主要解决：** 在运行期建立和删除原型。
&gt;
&gt; **何时使用：** 1、当一个系统应该独立于它的产品创建，构成和表示时。 2、当要实例化的类是在运行时刻指定时，例如，通过动态装载。 3、为了避免创建一个与产品类层次平行的工厂类层次时。 4、当一个类的实例只能有几个不同状态组合中的一种时。建立相应数目的原型并克隆它们可能比每次用合适的状态手工实例化该类更方便一些。
&gt;
&gt; **如何解决：** 利用已有的一个原型对象，快速地生成和原型对象一样的实例。
&gt;
&gt; **关键代码：** 1、实现克隆操作，在 JAVA 继承 Cloneable，重写 clone()，在 .NET 中可以使用 Object 类的 MemberwiseClone() 方法来实现对象的浅拷贝或通过序列化的方式来实现深拷贝。 2、原型模式同样用于隔离类对象的使用者和具体类型（易变类）之间的耦合关系，它同样要求这些&#34;易变类&#34;拥有稳定的接口。
&gt;
&gt; **应用实例：** 1、细胞分裂。 2、JAVA 中的 Object clone() 方法。
&gt;
&gt; **优点：** 1、性能提高。 2、逃避构造函数的约束。
&gt;
&gt; **缺点：** 1、配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。 2、必须实现 Cloneable 接口。
&gt;
&gt; **使用场景：** 1、资源优化场景。 2、类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。 3、性能和安全要求的场景。 4、通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。 5、一个对象多个修改者的场景。 6、一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。 7、在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。
&gt;
&gt; **注意事项：** 与通过对一个类进行实例化来构造新对象不同的是，原型模式是通过拷贝一个现有对象生成新对象的。浅拷贝实现 Cloneable，重写，深拷贝是通过实现 Serializable 读取二进制流。



## 原型模式详解

### 原型模式结构

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205953513.png)
原型模式包含以下3个角色。

1. Prototype（抽象原型类）：它是声明克隆方法的接口，是所有具体原型类的公共父类，它可以是抽象类也可以是接口，甚至还可以是具体实现类。
2. ConcretePrototype（具体原型类）：它实现在抽象原型类中声明的克隆方法，在克隆方法中返回自己的一个克隆对象。
3. Client（客户类）：在客户类中，让一个原型对象克隆自身从而创建一个新的对象，只需要直接实例化或通过工厂方法等方式创建一个原型对象，再通过调用该对象的克隆方法即可得到多个相同的对象。

### 深克隆与浅克隆

根据在复制原型对象的同时是否复制包含在原型对象中引用类型的成员变量，原型模式的克隆机制可分为两种，即浅克隆（Shallow Clone）和深克隆（Deep Clone）。

#### 浅克隆

在浅克隆中，如果原型对象的成员变量是值类型，将复制一份给克隆对象；如果原型对象的成员变量是引用类型，则将引用对象的地址复制一份给克隆对象，也就是说原型对象和克隆对象的成员变量指向相同的内存地址。&lt;font color=&#34;red&#34;&gt;简单来说，在浅克隆中，当对象被复制时只复制它本身和其中包含的值类型的成员变量，而引用类型的成员对象并没有复制。&lt;/font&gt;

![image-20220422141726192](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/6455d2fc6436a691d1004ad93e9221d6-20231125210018047.png)

#### 深克隆

在深克隆中，无论原型对象的成员变量是值类型还是引用类型，都将复制一份给克隆对象，深克隆将原型对象的所有引用对象也复制一份给克隆对象。&lt;font color=&#34;red&#34;&gt;简单来说，在深克隆中，除了对象本身被复制外，对象所包含的所有成员变量也将复制。&lt;/font&gt;

![image-20220422141957029](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a42be2f7d6daecf9b71e2353a78d023e-20231125210026768.png)

### 原型模式实现

&lt;font color=&#34;red&#34;&gt;实现原型模式的关键在于如何实现克隆方法。&lt;/font&gt;这里介绍两种在Java语言中最常用的克隆实现方法。

#### 通用实现方法

通用的克隆实现方法是在具体原型类的克隆方法中实例化一个与自身类型相同的对象并将其返回，同时将相关的参数传入新创建的对象中，保证它们的成员变量相同。

典型的抽象原型类代码如下：

```java
public abstract class Prototype {
    public abstract Prototype clone();
}
```

典型的具体原型类代码如下：

```java
public class ConcretePrototype extends Prototype {
    private String name; // 成员变量
    public void setName(String name) {
        this.name = name;
    }
    public void getName() {
        return this.name;
    }
    // 克隆方法实现
    public Prototype clone() {
        Prototype prototype = new ConcretePrototype(); // 创建新对象
        prototype.setName(this.name);
        return prototype;
    }
}
```

这样我们就只需要在客户类中创建一个ConcretePrototype对象作为原型对象，然后调用其clone()方法即可得到对应的克隆对象。

&lt;font color=&#34;red&#34;&gt;此方法是原型模式的通用实现，它与编程语言本身的特性无关，其他面向对象编程语言也可以使用这种形式来实现对原型对象的克隆。&lt;/font&gt;

在这种通用实现方法中，可通过手工编写clone()方法来实现浅克隆和深克隆。对于引用类型的对象，可以在clone()方法中通过赋值的方式来实现复制，这是一种浅克隆实现方案；如果在clone()方法中通过创建一个全新的成员对象来实现复制，则是一种深克隆实现方案。

#### Java语言中的clone()方法和Cloneable接口

在Java语言中，所有的Java类均继承自java.lang.Object类，Object类提供了一个clone()方法，可以将一个Java对象复制一份。因此在Java中可以直接使用Object提供clone()方法来实现对象的浅克隆。

需要注意的是能够实现克隆的Java类都必须实现一个标识接口Cloneable，表示这个Java类支持被复制。如果一个类没有实现这个接口但是调用了clone()方法，Java编译器将会抛出一个CloneNotSupportedException异常。如下代码所示：

```java
public class ConcretePrototype implements Cloneable {
    public Prototype clone() {
        Object object = null;
        try {
            object = super.clone(); // 浅克隆
        } catch (CloneNotSupportedException exception) {
            System.err.println(&#34;Not support Cloneable&#34;);
        }
        return (Prototype)object;
    }
}
```

为了获取对象的一个克隆，可以直接利用Object类的clone()方法，其具体步骤如下：

1. 在派生类中覆盖基类的clone()方法，并声明为public。
2. 在派生类的clone()方法中调用super.clone()。
3. 派生类需实现Cloneable接口。

此时，Object类相当于抽象原型类，所有实现了Cloneable接口的类相当于具体原型类。

### 原型模式应用举例

* **题目描述**

	某数据处理软件需要增加一个图表复制功能。在图表对象（DataChart）中包含一个数据集对象(DataSet)。数据集对象用于封装要显示的数据，用户可以通过界面上的复制按钮将该图表复制一份，复制后，即可得到新的图表对象，然后可以修改新图表的编号、颜色、数据。试用原型模式设计软件实现深克隆。

* **UML类图**

	![image-20220422152228233](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/399b5aa92c8a241ec1e910275423794d-20231125210034731.png)在该设计方案中，DataChart 类包含一个 DataSet 对象，在复制 DataChart 对象的同时将
	复制 DataSet 对象，因此需要使用深克隆技术，可使用流来实现深克隆。其中Serializable是java.io包中定义的、用于实现Java类的序列化操作而提供的一个语义级别的接口。&lt;font color=&#34;red&#34;&gt;Serializable序列化接口没有任何方法或者字段，只是用于标识可序列化的语义。实现了Serializable接口的类可以被ObjectOutputStream转换为字节流，同时也可以通过ObjectInputStream再将其解析为对象。&lt;/font&gt;故我们实现这个接口即可使用流来实现深克隆

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/prototype_pattern/example_1)

## 原型管理器

### 原型管理器实现

&lt;font color=&#34;red&#34;&gt;原型管理器(Prototype Manager)是将多个原型对象存储在一个集合中供客户端使用，它是一个专门负责克隆对象的工厂&lt;/font&gt;，其中定义了一个集合用于存储原型对象， 如果需要某个原型对象的一个克隆，可以通过复制集合中对应的原型对象来获得。 在原型管理器中针对抽象原型类进行编程，以便扩展。  其结构如图所示：

![image-20220422145331518](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/d2524150ea07197b94004a6da74667e5.png)

其中典型的原型管理器PrototypeManager类的实现代码片段如下：

```java
package prototype_pattern;

import java.util.Hashtable;

/**
 * @author Cnc_hzf
 * @date 2022/4/22 15:00
 */
public class PrototypeManager {
    private Hashtable prototypeTable = new Hashtable(); // 使用Hashtable存储原型对象
    public PrototypeManager() {
        prototypeTable.put(&#34;A&#34;, new ConcretePrototypeA());
        prototypeTable.put(&#34;B&#34;, new ConcretePrototypeB());
    }
    public void add(String key, Prototype prototype) {
        prototypeTable.put(key, prototype);
    }
    public Prototype get(String key) {
        Prototype clone = ((Prototype) prototypeTable.get(key)).clone(); // 通过克隆方法创建新对象
        return clone;
    }
}
```

在实际开发中可以将PrototypeManger设计为单例类，确保系统中有且仅有一个PrototypeManager对象，这样既有利于节省系统资源，还可以更好地对原型管理器对象进行控制。

### 原型管理器应用举例

* **题目描述**

	&gt; 某公司需要创建一个公文管理器，公文管理器中需要提供一个集合对象来存储一些公文模板，用户可以通过复制这些模板快速的创建新的公文，试使用带有原型管理器的原型模式来设计该公文管理器并使用Java代码编程模拟。

* **UML类图**

	![image-20220422164033617](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f2054dfa969450564f705ed0380eec8a.png)

	其中，OfficialDocument （抽象公文类）充当抽象原型类，其子类 FAR（Feasibility Analysis
	Report，可行性分析报告）和 SRS（Software Requirements Specification，软件需求规格说明书）充当具体原型类，PrototypeManager 充当原型管理器。  

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/prototype_pattern/example_2)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/03.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%8E%9F%E5%9E%8B%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


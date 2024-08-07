# 设计模式之工厂模式详解（Java实现）

## 工厂模式介绍

工厂顾名思义就是创建产品，根据&lt;font color=&#34;red&#34;&gt;产品是具体产品还是具体工厂&lt;/font&gt;可分为简单工厂模式（Simple Factory Pattern）和工厂方法模式（Factory Method Pattern），根据&lt;font color=&#34;red&#34;&gt;工厂的抽象程度&lt;/font&gt;可分为工厂方法模式和抽象工厂模式（Abstract Factory Pattern）。该模式用于封装和管理对象的创建，是一种创建型模式。
![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205715587.png)


这样看我们工厂模式（Factory Pattern）可以分为三类，但其中简单工厂其实不是一个标准的的设计模式。GOF 23 种设计模式中只有「工厂方法模式」与「抽象工厂模式」。简单工厂模式可以看为工厂方法模式的一种特例。

在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

它的优点如下：

- **可以使代码结构清晰，有效地封装变化**。
- **对调用者屏蔽具体的产品类**
- **降低耦合度**。

&gt; 适用场景：作为一种创建类模式，在任何需要生成复杂对象的地方，都可以使用工厂方法模式。有一点需要注意的地方就是&lt;font color=&#34;red&#34;&gt;复杂对象适合使用工厂模式&lt;/font&gt;，而简单对象，特别是只需要通过 new 就可以完成创建的对象，无需使用工厂模式。如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度。
&gt;
&gt; 其次，工厂模式是一种典型的**解耦模式**，迪米特法则在工厂模式中表现的尤为明显。假如调用者自己组装产品需要增加依赖关系时，可以考虑使用工厂模式，将会大大降低对象之间的耦合度。再次，由于工厂模式是依靠抽象架构的，它把实例化产品的任务交由实现类完成，**扩展性比较好**。也就是说，当需要系统有比较好的扩展性时，可以考虑工厂模式，不同的产品用不同的实现工厂来组装。

## 工厂模式详解

### 简单工厂模式

#### 简单工厂模式结构

该模式对对象创建管理方式最为简单，因为其仅仅简单的对不同类对象的创建进行了一层薄薄的封装。该模式通过向工厂传递类型来指定要创建的对象，其UML类图如下：

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f3202d7f2ecb6b7a3dc236ac753a5429.png)

其中根据上图可知，简单工厂模式包含以下3个角色。

1. Factory（工厂角色）：即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实例的内部逻辑；其可以被外界直接调用，创建所需的产品对象。
2. AbstractProduct（抽象产品角色）：它是工厂类创建的所有对象的父类，封装了各种产品对象的共有方法。
3. Product1（具体产品角色）：它是简单工厂模式的创建目标，所有被创建的对象都充当这个角色的某个具体类的实例。

#### 简单工厂模式实现

典型的抽象产品类代码如下：

```java
public abstract class AbstractProduct {
    // 所有产品类的公共业务方法
    public void methodSame() {
        // 公有方法的实现
    }
    // 声明抽象业务方法
    public abstract void methodDiff() {
        
    }
}
```

典型的具体产品类的代码如下：

```java
public class Product1 extends AbstractProduct {
    // 实现业务方法
    public void methodDiff() {
        // 业务方法的实现
    }
}
```

典型的工厂类的代码如下：

```java
public class Factory {
    // 静态工厂方法
    public static AbstractProduct createProduct(String arg) {
        AbstractProduct product = null;
        if (arg.equalsIgnoreCase(&#34;1&#34;)) {
            product = new Product1();
        } else if (arg.equalsIgnoreCase(&#34;2&#34;)) {
            product = new Product2();
        }
        return product;
    }
}
```

客户端代码中，通过调用工厂类的工厂方法即可得到产品对象。

#### 简单工厂模式应用举例

* **题目描述**

	&gt;使用简单工厂模式设计一个可以创建不同几何图形（Shape），如Circle，Rectangle，Triangle等绘图工具类，每个几何图形均具有绘制draw()和擦除erase()两个方法；要求在绘制不支持的几何图形时，抛出一个UnsuppShapeException异常，绘制类图并使用Java语言实现。

* **UML类图**

	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205715677.png)


  其中，Shape接口充当抽象产品，其子类Circle、Rectangle、Triangle等充当具体产品，ShapeFactory充当工厂类。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/simple_factory_pattern)

### 工厂方法模式

#### 工厂方法模式结构

和简单工厂模式中工厂负责生产所有产品相比，工厂方法模式将生成具体产品的任务分发给具体的产品工厂，也就是定义一个抽象工厂，其定义了产品的生产接口，但不负责具体的产品，将生产任务交给不同的派生类工厂。这样不用通过指定类型来创建对象了。其UML类图如下：

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/3e3bdce81f61fec8cff2772bca642685-20231125205752903.png)

由上图可知，工厂方法模式包含以下四个角色。

1. AbstractProduct（抽象产品）：它是定义产品的接口，是工厂方法模式所创建对象的公共父类。
2. Product1（具体产品）：它实现了抽象产品接口，某种类型的具体产品由专门的具体工厂创建，具体工厂和具体产品之间一一对应。
3. AbstractFactory（抽象工厂）：在抽象工厂类中声明了工厂方法，用于返回一个产品。&lt;font color=&#34;red&#34;&gt;抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口。&lt;/font&gt;
4. ConcreteFactory1（具体工厂）：它是抽象工厂类的子类，实现了在抽象工厂中声明的工厂方法，并可由客户端调用，返回一个具体产品类的实例。

#### 工厂方法模式实现

典型的抽象产品类代码如下：

```java
public abstract class AbstractProduct {
    // 所有产品类的公共业务方法
    public void methodSame() {
        // 公有方法的实现
    }
    // 声明抽象业务方法
    public abstract void methodDiff() {
        
    }
}
```

典型的具体产品类的代码如下：

```java
public class Product1 extends AbstractProduct {
    // 实现业务方法
    public void methodDiff() {
        // 业务方法的实现
    }
}
```

典型的抽象工厂代码如下：

```java
public interface AbstractFactory {
    public Product createProduct();
}
```

典型的具体工厂代码如下：

```java
public class ConcreteFactory1 implements Factory {
    public Product createProduct() {
        return new Product1();
    }
}
```

#### 工厂方法模式应用举例

* **题目描述**

	&gt; 现需要设计一个程序来读取多种不同类型的图片格式，针对每一种图片格式都设计一个图片读取器（ImgReader），如gif图片读取器（GifReader）用于读取gif格式的图片,jpg图片读取器（JpgReader）用于读取jpg格式的图片。图片读取器对象通过图片读取器工厂ImgReaderFactory来创建。ImgReaderFactory是一个抽象类，用于定义创建图片读取器的工厂方法，其GifReaderFactory和JpgReaderFactory用于创建具体的图片读取器对象。

* **UML类图**

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/watermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAdW5pcXVlX3B1cnN1aXQ%3D%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16-20231125205715777.png)


  其中，接口ImageReaderFactory充当抽象工厂，GifReaderFactory和JpgReaderFactory充当具体工厂，ImageReader充当抽象产品，GifReader和JpgReader充当具体产品。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/factory_method_pattern)

### 抽象工厂模式

#### 抽象工厂模式结构

上面两种模式不管工厂怎么拆分抽象，都只是针对一类产品（AbstractProduct），应该怎么表示呢？

最简单的方式是把2中介绍的工厂方法模式完全复制一份，不过这次生产的是另一类产品。但同时也就意味着我们要完全复制和修改原来生产管理的所有代码，显然这是一个笨办法，并不利于扩展和维护。

抽象工厂模式通过在AbstarctFactory中增加创建产品的接口，并在具体子工厂中实现新加产品的创建，当然前提是子工厂支持生产该产品。否则继承的这个接口可以什么也不干。其UML图如下：

![img](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/d41cbac06dda47c2aa7bab88ccae27ed-20231125205801919.png)

由上图可知，抽象工厂模式包含以下四个角色。

1. AbstractProduct（抽象产品）：它为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法。
2. Product1（具体产品）：它定义具体工厂生产的具体产品的具体产品对象，实现抽象产品接口中声明的业务方法。
3. AbstractFactory（抽象工厂）：它声明了一组用于创建一族产品的方法，每一个方法对应一种产品。
4. ConcreteFactory1（具体工厂）：它实现了在抽象工厂中声明的创建产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中。

#### 抽象工厂模式实现

抽象工厂类典型代码如下：

```java
public interface AbstractFactory {
    public AbstractProduct1 createProduct1(); // 工厂方法一
    public AbstractProduct2 createProduct2(); // 工厂方法、二
}
```

具体工厂类典型代码如下：

```java
public class ConcreteFactory1 implements AbstractFactory {
    // 工厂方法一
    public AbstractProduct1 createProduct1() {
        return new Product1();
    }
    // 工厂方法二
    public AbstractProduct2 createProduct2() {
        return new Product2();
    }
}
```

#### 抽象工厂模式应用举例

* **题目描述**

	&gt; 抽象工厂模式最早的应用是用于创建分属于不同操作系统的视窗构建。比如:命令按键(Button)与文字框(Text)都是视窗构件，在Unix、Windows和Linux操作系统的视窗环境中，这两个构件有不同的本地实现，它们的细节也有所不同。试使用抽象工厂模式来设计并模拟实现该结构。

* **UML类图**

	![image-20220421213130863](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/d4667ae8a9b283a36eeb33c16d253dab.png)

	其中，接口AbstractFactory充当抽象工厂，其子类WindowsFactory、UnixFactory和LinuxFactory充当具体工厂；Text和Button充当抽象产品，其子类WindowsText、UnixText、LinuxText和WindowsButton、UnixButton、LinuxButton充当具体产品。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/abstract_factory_pattern)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/01.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


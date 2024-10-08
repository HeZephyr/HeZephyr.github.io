# 设计模式之装饰模式详解（Java实现）

## 装饰模式介绍

在生活中，我们往往会给图片增加一些边框等来装饰图片，可以让图片变得更漂亮，如下图，就是对小狗图片的装饰。

![image-20220428105146788](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/de649ce18e81ce23062d76054f2400fe.png)

在软件设计中，我们也有一种类似图片的技术可以对已有对象（图片）的功能进行扩展（装修），以获得更加符合用户需求的对象，使得对象具有更加强大的功能。这种技术对应于一种被称之为装饰模式（Decorator Pattern）的设计模式。装饰模式能动态地给一个对象增加一些额外的职责。就扩展功能而言，装饰模式提供了一种比使用子类更加灵活的替代方案。引入了装饰类，在装饰类中既可以调用待装饰的原有类的方法，还可以增加新的方法，以扩展原有类的功能

&gt; **主要解决：** 一般的，我们为了扩展一个类经常使用继承方式实现，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。
&gt;
&gt; **何时使用：** 在不想增加很多子类的情况下扩展类。
&gt;
&gt; **如何解决：** 将具体功能职责划分，同时继承装饰者模式。
&gt;
&gt; **关键代码：** 1、Component 类充当抽象角色，不应该具体实现。 2、修饰类引用和继承 Component 类，具体扩展类重写父类方法。
&gt;
&gt; **应用实例：** 1、孙悟空有 72 变，当他变成&#34;庙宇&#34;后，他的根本还是一只猴子，但是他又有了庙宇的功能。 2、不论一幅画有没有画框都可以挂在墙上，但是通常都是有画框的，并且实际上是画框被挂在墙上。在挂在墙上之前，画可以被蒙上玻璃，装到框子里；这时画、玻璃和画框形成了一个物体。
&gt;
&gt; **优点：** 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。
&gt;
&gt; **缺点：** 多层装饰比较复杂。
&gt;
&gt; **使用场景：** 1、扩展一个类的功能。 2、动态增加功能，动态撤销。
&gt;
&gt; **注意事项：** 可代替继承。

## 装饰模式详解

### 装饰模式结构

装饰模式的结构如图所示：

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f71d1074816042ab93a295eaa7c14f35-20231125211811875.png)


由结构图可知，装饰模式包含以下4个角色。

1. Component（抽象构件）：它是具体构件和抽象装饰类的共同父类，声明了在具体构件中的业务方法，&lt;font color=&#34;red&#34;&gt;它的引入可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象，实现客户端的透明操作。&lt;/font&gt;
2. ConcreteComponent（具体构件）：它是抽象构件的子类，用于定义具体的构件对象，实现了在抽象构件中声明的方法，装饰类可以给它增加额外的职责（方法）。
3. Decorator（抽象装饰类）：它也是抽象构件类的子类，用于给具体构件增加职责，但是具体职责在其子类中实现。&lt;font color=&#34;red&#34;&gt;它维护一个指向抽象构件对象的引用，通过该引用可以调用装饰之前构件对象的方法，并通过其子类扩展该方法，以达到装饰的目的。&lt;/font&gt;
4. ConcreteDecorator（具体装饰类）：它是抽象装饰类的子类，负责向构件添加新的职责。&lt;font color=&#34;red&#34;&gt;每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用于扩充该对象的行为。&lt;/font&gt;

### 装饰模式实现

抽象构件类典型代码：

```java
public abstract class Component {
    public abstract void operation();
}
```

具体构件类典型代码：

```java
public class ConcreteComponent extends Component {
    public void operation() {
        //实现基本功能    
    }
}
```

抽象装饰类典型代码：

```java
public class Decorator extends Component {
    private Component component; //维持一个对抽象构件对象的引用

    //注入一个抽象构件类型的对象
    public Decorator(Component component) {
        this.component=component;
    }

    public void operation() {
        component.operation();  //调用原有业务方法
    }
}
```

具体装饰类典型代码：

```java
public class ConcreteDecorator extends Decorator {
    public ConcreteDecorator(Component component) {
        super(component); 
    }

    public void operation() {
        super.operation(); //调用原有业务方法
        addedBehavior(); //调用新增业务方法
    }

    //新增业务方法
    public void addedBehavior() { 
        ……
    }
}
```

### 装饰模式应用举例

* **题目描述**

	&gt; 简单的手机（SimplePhone）再接收到来电的时候，会发出声音提醒主人；而现在我们需要为该手机添加一项功能，在接收来电的时候，除了声音还能产生振动（JarPhone）；还可以得到更高级的手机（ComplexPhone），来电的时候，它不仅能够发声，产生振动，而且还有灯光在闪烁提示。现在用装饰模式来模拟一下手机的升级过程，要求绘制类图并编程实现。

* **UML类图**

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/b3d0ac743ea44328a7dea6880b95c025-20231125211801620.png)


  其中，Cellphone 为抽象类，声明了来电方法 receiveCall()，SimplePhone 为简单手机类，
  提供了声音提示，JarPhone 和 ComplexPhone 分别提供了振动提示和灯光闪烁提示。
  PhoneDecorator 是抽象装饰者，它维持一个对父类对象的引用。  

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/decorator_pattern)

## 透明装饰模式和半透明装饰模式

1. **透明装饰模式**

	要求客户端完全针对抽象编程，装饰模式的透明性要求客户端程序不应该将对象声明为具体构件类型或具体装饰类型，而应该全部声明为抽象构件类型。对于客户端而言，具体构件对象和具体装饰对象没有任何区别。可以让客户端透明地使用装饰之前的对象和装饰之后的对象，无须关心它们的区别。可以对一个已装饰过的对象进行多次装实例：

	```java
	……
	Component component_o,component_d1,component_d2; //全部使用抽象构件定义
	component_o = new ConcreteComponent();
	component_d1 = new ConcreteDecorator1(component_o);
	component_d2 = new ConcreteDecorator2(component_d1);
	component_d2.operation();
	//无法单独调用component_d2的addedBehavior()方法
	……
	```

2. **半透明装饰模式**

	用具体装饰类型来定义装饰之后的对象，而具体构件使用抽象构件类型来定义。对于客户端而言，具体构件类型无须关心，是透明的；但是具体装饰类型必须指定，这是不透明的。可以给系统带来更多的灵活性，设计相对简单，使用起来也非常方便，客户端使用具体装饰类型来定义装饰后的对象，因此可以单独调用addedBehavior()方法。最大的缺点在于不能实现对同一个对象的多次装饰，而且客户端需要有区别地对待装饰之前的对象和装饰之后的对象。实例：

	```java
	……
	Component component_o; //使用抽象构件类型定义
	component_o = new ConcreteComponent();
	component_o.operation();
	ConcreteDecorator component_d; //使用具体装饰类型定义
	component_d = new ConcreteDecorator(component_o);
	component_d.operation();
	component_d.addedBehavior(); //单独调用新增业务方法
	……
	```
   

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/08.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E8%A3%85%E9%A5%B0%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


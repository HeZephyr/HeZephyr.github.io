# 设计模式之观察者模式详解（Java实现）

## 观察者模式介绍

“红灯停，绿灯行”。在这个过程中，交通信号灯是汽车的观察目标，而汽车则是观察者。随着交通信号灯的变化，汽车的行为也随之变化，一盏交通信号灯可以指挥多辆汽车。

![image-20220430155302091](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/7bbcdbf54bdfbe3b781f717872a93242-20231125212711452.png)

在软件系统中有些对象之间也存在类似交通信号灯和汽车之间的关系，一个对象的状态或行为的变化将导致其他对象的状态或行为也发生改变。观察者模式（Observer Pattern）则是用于建立一种对象与对象之间的依赖关系，使得每当一个对象状态发生改变时其相关依赖对象皆得到通知并被自动更新。发生改变的对象称为观察目标，被通知的对象称为观察者，一个观察目标可以对应多个观察者。它有如下别名：

* 发布-订阅(Publish/Subscribe)模式
* 模型-视图(Model/View)模式
* 源-监听器(Source/Listener)模式
* 从属者(Dependents)模式

&gt; **主要解决：** 一个对象状态改变给其他对象通知的问题，而且要考虑到易用和低耦合，保证高度的协作。
&gt;
&gt; **何时使用：** 一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知。
&gt;
&gt; **如何解决：** 使用面向对象技术，可以将这种依赖关系弱化。
&gt;
&gt; **关键代码：** 在抽象类里有一个 ArrayList 存放观察者们。
&gt;
&gt; **应用实例：** 1、拍卖的时候，拍卖师观察最高标价，然后通知给其他竞价者竞价。 2、西游记里面悟空请求菩萨降服红孩儿，菩萨洒了一地水招来一个老乌龟，这个乌龟就是观察者，他观察菩萨洒水这个动作。
&gt;
&gt; **优点：** 1、观察者和被观察者是抽象耦合的。 2、建立一套触发机制。
&gt;
&gt; **缺点：** 1、如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。 2、如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。 3、观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。
&gt;
&gt; **使用场景：**
&gt;
&gt; - 一个抽象模型有两个方面，其中一个方面依赖于另一个方面。将这些方面封装在独立的对象中使它们可以各自独立地改变和复用。
&gt; - 一个对象的改变将导致其他一个或多个对象也发生改变，而不知道具体有多少对象将发生改变，可以降低对象之间的耦合度。
&gt; - 一个对象必须通知其他对象，而并不知道这些对象是谁。
&gt; - 需要在系统中创建一个触发链，A对象的行为将影响B对象，B对象的行为将影响C对象……，可以使用观察者模式创建一种链式触发机制。
&gt;
&gt; **注意事项：** 1、JAVA 中已经有了对观察者模式的支持类。 2、避免循环引用。 3、如果顺序执行，某一观察者错误会导致系统卡壳，一般采用异步方式。

## 观察者模式详解

### 观察者模式结构

观察者模式结构中通常包括观察目标和观察者两个继承层次结构，其结构图如下：

![image-20220430160031478](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/f24da870325ba3cdac402a25213fd7df.png)

由图可知，观察者模式包含以下4个角色。

1. Subject（目标）：目标又称为主题，它是指被观察的对象。在目标中定义了一个观察者集合，一个观察目标可以接受任意数量的观察者来观察，它提供了一系列方法来增加和删除观察者对象，同时它定义了通知方法notify()。目标类可以是接口，也可以是抽象类或具体类。
2. ConcreteSubject（具体目标）：具体目标是目标类的子类，它通常包含有经常发生改变的数据，当它的状态发生改变时它向各个观察者发出通知；同时它还实现了在目标类中定义的抽象业务逻辑方法。&lt;font color=&#34;puple&#34;&gt;如果无须扩展目标类，具体目标类可以省略。&lt;/font&gt;
3. Observer（观察者）：观察者将对观察目标的改变做出反映，观察者一般定义为接口，该接口声明了更新数据的方法update()，因此又称为抽象观察者。
4. ConcreteObserver（具体观察者）：&lt;font color=&#34;red&#34;&gt;在具体观察者中维护一个指向具体目标对象的引用，它存储具体观察者的有关状态，这些状态需要和具体目标的状态保持一致；&lt;/font&gt;它实现了在抽象观察者Observer中定义的update()方法。通常在实现时可以调用具体目标类的attach()方法将自己添加到目标类的集合中或者通过detach()方法将自己从目标类的集合中删除。

### 观察者模式实现

抽象目标类典型代码如下：

```java
import java.util.List;
import java.util.ArrayList;

public abstract class Subject {
    //定义一个观察者集合用于存储所有观察者对象
    protected List&lt;Observer&gt; observers = new ArrayList();

    //注册方法，用于向观察者集合中增加一个观察者
    public void attach(Observer observer) {
        observers.add(observer);
    }

    //注销方法，用于在观察者集合中删除一个观察者
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    //声明抽象通知方法
    public abstract void notify();
}
```

具体目标类典型代码如下：

```java
public class ConcreteSubject extends Subject {
    //实现通知方法
    public void notify() {
        //遍历观察者集合，调用每一个观察者的响应方法
        for(Observer obs:observers) {
            obs.update();
        }
    } 
}
```

抽象观察者典型代码如下：

```java
public interface Observer {
    //声明响应方法
    public void update();
}
```

具体观察者典型代码如下：

```java
public class ConcreteObserver implements Observer {
    //实现响应方法
    public void update() {
        //具体响应代码
    }
}
```

* 有时候在&lt;font color=&#34;red&#34;&gt;具体观察者类ConcreteObserver中需要使用到具体目标类ConcreteSubject中的状态（属性），会存在关联或依赖关系。&lt;/font&gt;
* 如果在具体层之间具有关联关系，系统的扩展性将受到一定的影响，&lt;font color=&#34;red&#34;&gt;增加新的具体目标类有时候需要修改原有观察者的代码&lt;/font&gt;，在一定程度上违背了开闭原则，但是如果原有观察者类无须关联新增的具体目标，则系统扩展性不受影响。

### 观察者模式应用举例

* **题目描述**

	&gt; 在某多人联机对战游戏中，多个玩家可以加入同一战队组成联盟，当战队中的某一成员受到敌人攻击时将给所有其他盟友发送通知，盟友收到通知后将做出响应。现使用观察者模式设计并实现该过程，以实现战队成员之间的联动。

* **题目分析**

	战队成员之间的联动过程：联盟成员受到攻击——&gt;发送通知给盟友——&gt;盟友做出响应。
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/70b57d5d86854ded97a30929030f7ea8.png)


* **UML类图**
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/bc370875124047e695fbffd1e232fca5-20231125212702939.png)

	其中，AllyControlCenter充当抽象目标类，ConcreteAllyControlCenter充当具体目标类，Observer充当抽象观察者，Player充当具体观察者。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/observer_pattern)

## 观察者模式与MVC

在当前流行的MVC（Model-View-Controller）架构中也应用了观察者模式，MVC是一种架构模式，它包含了3个角色，即模型(Model)，视图(View)和控制器(Controller)。其中，模型可对应于观察者模式中的观察目标，而视图对应于观察者，控制器可充当两者之间的中介者。当模型层的数据发生改变时，视图层将自动改变其显示内容。MVC的结构图如下：

![image-20220430171138766](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/cf6f1e886d92e88a2d3e47e0efd2e12d.png)

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/12.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


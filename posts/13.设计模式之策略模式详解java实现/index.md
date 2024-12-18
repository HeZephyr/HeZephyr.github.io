# 设计模式之策略模式详解（Java实现）

## 策略模式介绍

在很多情况下，实现某个目标的途径不止一条，例如在外出旅游时游客可以选择多种不同的出行方式，可根据实际情况来确定最适合的出行方式。

![image-20220430182337891](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/293afe7e145a9bf9ef941bd9281efee7-20231125212746115.png)

在软件开发中，也常常会遇到类似的情况，实现某个功能有多种算法，一种常用的方法是通过硬编码将所有的算法集中在一个类中，在该类中提供多个方法，每个方法对应一个算法；当然也可以将这些算法封装在一个统一的方法中，通过if...else...等条件判断选择。这两种实现方法都可以称为硬编码。但这样的方式封装了大量的算法，代码非常复杂，维护也很困难。

策略模式（Strategy Pattern）定义一些独立的类来封装不同的算法，&lt;font color=&#34;red&#34;&gt;每一个类封装一种具体的算法，在这里每一个封装算法的类都可以称为一种策略（Strategy），为了保证这些策略在使用时具有一致性，一般会提供一个抽象的策略类来做算法的声明，而每种算法对应于一个具体策略类。&lt;/font&gt;

策略模式定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法可以独立于使用它们的客户而变化。它又称为政策（Policy）模式，是一种对象行为型模式。

## 策略模式详解

### 策略模式结构

其结构图如下：

![image-20220430184548630](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a0bafc2ebad2e45b4d50aa5c618086d1.png)

由图可知，策略模式包含以下3个角色。

1. Context（环境类）：环境类是使用算法的角色，它在解决某个问题时可以采用多种策略。&lt;font color=&#34;red&#34;&gt;在环境类中维持一个对抽象策略类的引用实例，用于定义所采用的策略。&lt;/font&gt;
2. Strategy（抽象策略类）：抽象策略类为所支持的算法声明了抽象方法，是所有策略类的父类，它可以是抽象类或具体类，也可以是接口。&lt;font color=&#34;red&#34;&gt;环境类通过抽象策略类中声明的方法在运行时调用具体策略类中实现的算法。&lt;/font&gt;
3. ConcreteStrategy（具体策略类）：具体策略类实现了在抽象策略类中声明的算法，在运行时具体策略类将覆盖在环境类中定义的抽象策略类对象，使用一种具体的算法实现某个业务功能。

### 策略模式实现

抽象策略类典型代码如下：

```java
public abstract class AbstractStrategy {
    // 声明抽象算法
    public abstract void algorithm();
}
```

具体策略类典型代码如下：

```java
public class ConcreteStrategyA extends AbstractStrategy {
    // 算法的具体实现
    public void algorithm() {
        // 算法A
    }
}
```

环境类典型代码如下：

```java
public class Context {
    private AbstractStrategy strategy; // 维持一个对抽象策略类的引用
    public void setStrategy(AbstractStrategy strategy) {
        this.strategy = strategy;
    }
    // 调用策略类中的算法
    public void algorithm() {
        strategy.algorithm();
    }
}
```

### 策略模式应用举例

* **题目描述**

	&gt; 某软件公司为某电影院开发了一套影院售票系统，在该系统中需要为不同类型的用户提供不同的电影票打折方式，具体打折方案如下：*  (1)  学生凭学生证可享受票价8折优惠。
	&gt; (2)  年龄在10周岁及以下的儿童可享受每张票减免10元的优惠（原始票价需大于等于20元）。
	&gt; (3)  影院VIP用户除享受票价半价优惠外还可进行积分，积分累计到一定额度可换取电影院赠送的奖品。
	&gt; 该系统在将来可能还要根据需要引入新的打折方式。现使用策略模式设计该影院售票系统的打折方案。

* **UML类图**
	![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/ced8cdbe32c449a8b2780bd9e9113a20.png)


  其中，MovieTicket充当环境类角色，Discount充当抽象策略角色，StudentDiscount、ChildrenDiscount和VIPDiscount充当具体策略角色。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/strategy_pattern)

	

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/13.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


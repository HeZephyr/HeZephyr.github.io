# 遵循最佳实践，各种语言实现设计模式的模版

23种设计模式分类如下：
| **模式类型** | **描述** | **包括的模式** |
| :----------: | :------: | :------------: |
| 创建型模式 | 这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 `new` 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。 | 工厂模式（Factory Pattern）&lt;br&gt;抽象工厂模式（Abstract Factory Pattern）&lt;br&gt;单例模式（Singleton Pattern）&lt;br&gt;建造者模式（Builder Pattern）&lt;br&gt;原型模式（Prototype Pattern） |
| 结构型模式 | 这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。 | 适配器模式（Adapter Pattern）&lt;br&gt;桥接模式（Bridge Pattern）&lt;br&gt;过滤器模式（Filter、Criteria Pattern）&lt;br&gt;组合模式（Composite Pattern）&lt;br&gt;装饰器模式（Decorator Pattern）&lt;br&gt;外观模式（Facade Pattern）&lt;br&gt;享元模式（Flyweight Pattern）&lt;br&gt;代理模式（Proxy Pattern） |
| 行为型模式 | 这些设计模式特别关注对象之间的通信，即对象之间的行为。 | 责任链模式（Chain of Responsibility Pattern）&lt;br&gt;命令模式（Command Pattern）&lt;br&gt;解释器模式（Interpreter Pattern）&lt;br&gt;迭代器模式（Iterator Pattern）&lt;br&gt;中介者模式（Mediator Pattern）&lt;br&gt;备忘录模式（Memento Pattern）&lt;br&gt;观察者模式（Observer Pattern）&lt;br&gt;状态模式（State Pattern）&lt;br&gt;空对象模式（Null Object Pattern）&lt;br&gt;策略模式（Strategy Pattern）&lt;br&gt;模板模式（Template Pattern）&lt;br&gt;访问者模式（Visitor Pattern） |

设计模式的六大原则是软件设计中的基石，它们为构建灵活、易于维护和升级的软件系统提供了指导。以下是对这六大原则的简要说明：

1. **开闭原则（Open Close Principle）**：
   - 意义：对扩展开放，对修改关闭。即在需要进行拓展时，不修改原有代码，实现热插拔的效果。
   - 实现方式：使用接口和抽象类，通过抽象化来保持程序的扩展性，易于维护和升级。

2. **里氏代换原则（Liskov Substitution Principle）**：
   - 意义：任何基类可以出现的地方，子类一定可以出现。基类能够被派生类替换，且不影响软件单位的功能，实现真正的复用。
   - 补充关系：是对开闭原则的补充，通过继承关系实现对抽象化的具体步骤规范。

3. **依赖倒转原则（Dependence Inversion Principle）**：
   - 基础：是开闭原则的基础。具体内容为针对接口编程，依赖于抽象而不依赖于具体。

4. **接口隔离原则（Interface Segregation Principle）**：
   - 意义：使用多个隔离的接口比使用单个接口更好，降低类之间的耦合度。
   - 设计思想：从大型软件架构出发，强调降低依赖，降低耦合。

5. **迪米特法则，又称最少知道原则（Demeter Principle）**：
   - 定义：一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。

6. **合成复用原则（Composite Reuse Principle）**：
   - 意义：尽量使用合成/聚合的方式，而不是使用继承。通过组合而非继承实现代码复用，增强系统的灵活性。

这些原则提供了一种指导思想，帮助开发人员设计出具有弹性、可维护性和可扩展性的软件系统。通过遵循这些原则，可以有效应对软件变化，提高代码质量和可维护性。

以下是部分语言实现设计模式的仓库，以供学习。
1. [Go](https://github.com/mohuishou/go-design-pattern)
2. [Java](https://github.com/youlookwhat/DesignPattern)
3. [C&#43;&#43;](https://github.com/jaredtao/DesignPattern/)
4. [Python](https://github.com/tim-chow/DesignPattern)
5. [JavaScript](https://github.com/nnupoor-zz/js_designpatterns)

---

> 作者: Pursuit  
> URL: https://hezephyr.github.io/posts/02.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E5%AE%9E%E7%8E%B0%E6%A8%A1%E7%89%88/  


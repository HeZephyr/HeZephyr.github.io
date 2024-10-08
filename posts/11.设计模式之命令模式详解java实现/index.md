# 设计模式之命令模式详解（Java实现）

## 命令模式介绍

在现实生活中人们通过使用开关来控制一些电器的打开和关闭，例如电灯或者排气扇，如下图。

![image-20220430133107777](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/0a2cc91d8f957073bf2d39beb425e9bd-20231125212109454.png)

我们可以将开关看成一个请求发送者，电灯或者排气扇则是请求的最终接收者和处理者。开关和电灯之间并不存在直接耦合关系，它们通过电线连接在一起，使用不同的电线可以连接不同的请求接收者，只需要更换一根电线，相同的发送者（开关）即可对应不同的接收者（电器）。

在软件开发中也存在很多与开关和电器类似的请求发送者和接收者对象，例如按钮和事件处理类。&lt;font color=&#34;red&#34;&gt;为了降低系统的耦合度，将请求的发送者和接收者解耦，我们可以使用命令模式（Command Pattern）来设计系统。&lt;/font&gt;

在命令模式中发送者与接收者之间引入了新的命令对象（类似电线），将发送者的请求封装在命令对象中，再通过命令对象来调用接收者的方法。它可以使请求发送者和接收者完全解耦，发送者和接收者之间没有直接引用的关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求。

&gt; **定义：** 将一个请求封装为一个对象，从而可用不同的请求对客户进行参数化，对请求排队或者记录请求日志，以及支持可撤销的操作。
&gt;
&gt; **主要解决：** 在软件系统中，行为请求者与行为实现者通常是一种紧耦合的关系，但某些场合，比如需要对行为进行记录、撤销或重做、事务等处理时，这种无法抵御变化的紧耦合的设计就不太合适。
&gt;
&gt; **何时使用：** 在某些场合，比如要对行为进行&#34;记录、撤销/重做、事务&#34;等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将&#34;行为请求者&#34;与&#34;行为实现者&#34;解耦？将一组行为抽象为对象，可以实现二者之间的松耦合。
&gt;
&gt; **如何解决：** 通过调用者调用接受者执行命令，顺序：调用者→命令→接受者。
&gt;
&gt; **关键代码：** 定义三个角色：1、received 真正的命令执行对象 2、Command 3、invoker 使用命令对象的入口
&gt;
&gt; **应用实例：** struts 1 中的 action 核心控制器 ActionServlet 只有一个，相当于 Invoker，而模型层的类会随着不同的应用有不同的模型类，相当于具体的 Command。
&gt;
&gt; **优点：** 1、降低了系统耦合度。 2、新的命令可以很容易添加到系统中去。
&gt;
&gt; **缺点：** 使用命令模式可能会导致某些系统有过多的具体命令类。
&gt;
&gt; **使用场景：** 认为是命令的地方都可以使用命令模式，比如： 1、GUI 中每一个按钮都是一条命令。 2、模拟 CMD。
&gt;
&gt; **注意事项：** 系统需要支持命令的撤销(Undo)操作和恢复(Redo)操作，也可以考虑使用命令模式，见命令模式的扩展。

## 命令模式详解

### 命令模式结构

命令模式的核心在于引入了抽象命令类和具体命令类，通过命令类来降低发送者和接收者的耦合度，请求发送者只需指定一个命令对象，再通过命令对象来调用请求接收者的处理方法，结构图如下：

![image-20220430135535250](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/804efd921798edcb4146c22d0e3e6749.png)

由图可知，命令模式包含以下4个角色。

1. Command（抽象命令类）：抽象命令类一般是一个抽象类或接口，在其中声明了用于执行请求的execute()等方法，通过这些方法可以调用请求接收者的相关操作。
2. ConcreteCommand（具体命令类）：具体命令类是抽象命令类的子类，实现了在抽象命令类中声明的方法，它对应具体的接收者对象，将接收者对象的动作绑定其中。&lt;font color=&#34;red&#34;&gt;具体命令类在实现execute()方法时将调用接收者对象的相关操作(Action)。&lt;/font&gt;
3. Invoker（调用者）：调用者即请求发送者，它通过命令对象来执行请求。一个调用者并不需要在设计时确定其接收者，因此它只与抽象命令类之间存在关联关系。在程序运行时可以将一个具体命令对象注入其中，再调用具体命令对象的execute()方法，从而实现间接调用请求接收者的相关操作。
4. Receiver（接收者）：接收者执行与请求相关的操作，具体实现对请求的业务处理。

### 命令模式实现

典型的抽象命令类代码如下：

```java
public abstract class Command {
    public abstract void execute();
}
```

对于请求发送者（即调用者）而言，将针对抽象命令类进行编程，可以通过构造函数或者Setter方法在运行时注入具体命令类对象，并在业务方法中调用命令对象的execute()方法，其典型代码如下：

```java
public class Invoker {
    private Command command;
    
    // 构造注入
    public Invoker(Command command) {
        this.command = command;
    }
    
    // 设值注入
    public setCommand(Command command) {
        this.command = command;
    }
    
    // 业务方法，用于调用命令类中的execute()方法
    public void call() {
        command.execute();
    }
}
```

具体命令类继承了抽象命令类，它与请求接收者关联，实现了在抽象命令类中声明的execute()方法，并在实现时调用接收者的请求响应方法。其典型代码如下：

```java
public class ConcreteCommand extends Command {
    private Receiver receiver; //维持一个对请求接收者对象的引用

    public void execute() {
        receiver.action(); //调用请求接收者的业务处理方法action()
    }
}
```

请求接收者Receiver具体实现对请求的业务处理，它拥有action()方法，用于执行与请求相关操作，其典型代码如下：

```java
public class Receiver {
    public void action() {
        //具体操作
    }
}
```

### 命令模式应用举例

* **题目描述**

	&gt; 为了用户使用方便，某系统提供了一系列功能键，用户可以自定义功能键的功能，例如功能键FunctionButton可以用于退出系统（由SystemExitClass类来实现），也可以用于显示帮助文档（由DisplayHelpClass类来实现）。
	&gt;
	&gt; 用户可以通过修改配置文件来改变功能键的用途，现使用命令模式来设计该系统，使得功能键类与功能类之间解耦，可为同一个功能键设置不同的功能。

* **UML类图**

![在这里插入图片描述](https://raw.githubusercontent.com/unique-pure/NewPicGoLibrary/main/img/a291b90eb8cd4690a650084d0a446427.png)



  其中，FunctionButton充当请求调用者，SystemExitClass和DisplayHelpClass充当请求接收者，Command是抽象命令类，ExitCommand和HelpCommand充当具体命令类。

* **代码**

	[代码地址](https://github.com/unique-pure/designpattern_code/tree/main/src/command_pattern)

## 实现命令队列

有时候，当一个请求发送者发送一个请求时有不止一个请求接收者产生响应，这些请求接收者将逐个执行业务方法，完成对请求的处理，此时可以通过命令队列来实现。

命令队列的实现方法有多种形式，其中最常用、灵活性最好的一种方式就是增加一个CommandQueue类，由该类负责存储多个命令对象，而不同的命令对象可以对应不同的请求接收者。CommandQueue类的典型代码如下：

```java
package homework;

import java.util.ArrayList;
import java.util.List;

public class CommandQueue {
    private List&lt;Command&gt; commandList = new ArrayList();
    public void addCommand(Command command) {
        commandList.add(command);
    }
    public void removeCommand(Command command) {
        commandList.remove(command);
    }

    /**
     * 循环调用每一个命令对象的execute()方法
     */
    public void execute() {
        for (Command command : commandList) {
            command.execute();
        }
    }
}
```

在增加命令队列类CommandQueue以后，请求发送者Invoker将针对CommandQueue编程。即将Command修改为CommandQueue即可。

---

> 作者: [HeZephyr](https://github.com/HeZephyr)  
> URL: https://hezephyr.github.io/posts/11.%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E4%B9%8B%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F%E8%AF%A6%E8%A7%A3java%E5%AE%9E%E7%8E%B0/  


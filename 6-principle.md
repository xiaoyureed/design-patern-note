# 程序设计6原则

ref: <<设计模式之禅>>

- 开放封闭原则（Open Close Principle,简称OCP） - 尽量通过扩展软件实体来解决需求变化，而不是通过修改已有的代码来完成变化

    总纲领

- 单一职责原则（Single Responsibility Principle，简称SRP ）- 针对类来说的; 

    所以一个类/接口的职责应该尽可能少

- 接口隔离原则（Interface Segregation Principle,简称ISP） - 针对接口来说的; 

    尽量细化接口，接口中的方法尽量少

- 里氏替换原则（Liskov Substitution Principle,简称LSP） - 在使用基类的的地方可以任意使用其子类，能保证子类完美替换基类。

    尽量不要破坏继承体系

- 依赖倒置原则（Dependence Inversion Principle,简称DIP） - 高层模块不应该依赖底层模块, 而是底层模块注入高层模块

    也就是 "面向接口编程"

- 迪米特法则（Law of Demeter,简称LoD） - 类间解耦(一个类对自己依赖的类知道的越少越好)。方法的权限修饰符尽可能小; 加入中间类

## 开-闭原则（总）

和依赖倒置原则有点类似, 但是有区别, 我的感觉是开闭原则更加偏向的是使用抽象来避免修改源代码，主张通过扩展去应对需求变更，而依赖倒置更加偏向的是层和层之间的解耦。

这儿有个例子: 假设需要一个发送节日祝福的功能, 通过短信

先看 bad demo：

```java
/**
 * 信息发送类, 通过短信发送
 *
 * @version: 0.1
 * @author: xy
 * @date: 2018年1月23日 下午10:27:28
 */
public class MessageSend {

    public void send(String msg) {
        System.out.println("Text Message send : " + msg);

    }
}

/**
 * 服务类, 供客户端调用
 *
 * @version: 0.1
 * @author: xy
 * @date: 2018年1月23日 下午10:29:09
 */
public class MessageService {

    private MessageSend sender;
    
    public MessageService() {
        this.sender = new MessageSend();
    }
    
    public void send(String msg) {
        sender.send(msg);
    }
}

/**
 * 客户端调用
 *
 * @version: 0.1
 * @author: xy
 * @date: 2018年1月23日 下午10:30:11
 */
public class MessageClient {

    public static void main(String[] args) {
        MessageService messageService = new MessageService();
        messageService.send("Merry Christmas !");
    }
}

/////////////////////////////////////////////////////

// 现在需要新支持另一种发送方式: 微信, 只好修改源码

/**
 * 新增的信息发送帮助类, 支持email
 *
 * @version: 0.1
 * @author: xy
 * @date: 2018年1月23日 下午10:37:55
 */
public class EmailMessageSend {

    public  void send(String msg) {
        System.out.println("Email Message send: " + msg);
    }
}

/**
 * 新增的信息发送类型
 *
 * @version: 0.1
 * @author: xy
 * @date: 2018年1月23日 下午10:42:36
 */
public enum SendType {

    /**
     * 短信
     */
    TEXT,
    /**
     * 邮件
     */
    EMAIL 
}

/**
 * 服务类, 供客户端调用
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月23日 下午10:53:18
 */
public class MessageService {

    private MessageSend sender;
    private SendType t;
    public MessageService(SendType t) {
        this.t= t;
    }
    
    public void send(String msg) {
        // 根据不同的类型进行不同的处理
    }
}

// 然后是client增加新的调用

// 后续如果又要支持新的发送方式, 又需要大改MessageService源码;
// 如果遵循开闭原则设计, 会怎么样呢?
```

看看 good demo:

```java

////////////////////////////////////////////////////////////////

/**
 * 信息发送接口
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月23日 下午11:02:30
 */
public interface ISendable {

    void send();
}

// 各个信息发送类实现接口
public class MessageSend implements ISendable{}
public class EmailMessageSend implements ISendable{}

// messageService以后可以一直不变

public class MessageService {

    private ISendable sender;// 这是典型的组合优于继承;
    
    public MessageService(ISendable sender) {
        this.sender = messageHelper;
    }
    
    public void send(String msg) {
        sender.send(msg);
    }
}

// 客户端调用
public class MessageClient {

    public static void main(String[] args) {
        MessageService messageService = new MessageService(new MessageSend());
        messageService.send("Merry Christmas !");
        
        MessageService messageService2 = new MessageService(new EmailMessageSend());
        messageService2.send("Merry Christmas !");
    }
}

// 对于MessageService服务类来说，不用做任何修改，只需要扩展新的推送消息的工具类即可, 这就是遵循开闭原则的好处

```

## 单一职责原则

也可理解为单一职责原则, 一个类或方法，只负责单一功

解决这种问题: 假如有类Class1完成职责T1，T2，当职责T1或T2有变更需要修改时，有可能影响到该类的另外一个职责正常工作。

错误示例：

```java
public class Calculator {
    public int add() throws NumberFormatException, IOException{
		//这里读取和加法耦合在一起了，如果需要增加业务方法【减法】，就要更改代码
        File file = new File("E:/data.txt");
        BufferedReader br = new BufferedReader(new FileReader(file));
        int a = Integer.valueOf(br.readLine());
        int b = Integer.valueOf(br.readLine());
        return a+b;
    }
    public static void main(String[] args) throws NumberFormatException, IOException {
        Calculator calculator = new Calculator();
        System.out.println("result:" + calculator.add());
    }
}

```

遵循单一职责原则后, 是这样: 分离出来一个类用来读取数据，将一个类拆成了两个类，这样以后我们如果有减法，乘法等等，就不用出现那么多重复代码了

```java
public class Reader {
    int a,b;
    public Reader(String path) throws NumberFormatException, IOException{
        BufferedReader br = new BufferedReader(new FileReader(new File(path)));
        a = Integer.valueOf(br.readLine());
        b = Integer.valueOf(br.readLine());
    }
    public int getA(){
        return a;
    }
    public int getB(){
        return b;
    }
}

//单独的计算类
public class Calculator {
    public int add(int a,int b){
        return a + b;
    }
    public static void main(String[] args) throws NumberFormatException, IOException {
        Reader reader = new Reader("E:/data.txt");
        Calculator calculator = new Calculator();
        System.out.println("result:" + calculator.add(reader.getA(),reader.getB()));
    }
}

```

## 里氏替换原则 (是否使用需要权衡)

里氏代换原则中说，所有引用基类的地方必须能透明地替换为使用其子类的对象, 是针对`继承`这一特性提出的(继承时最好遵循里氏替换)

更直接的理解是: 子类一般不会重写父类的方法

就是尽量不要从可实例化的父类中继承，而是要使用基于抽象类和接口的继承。

错误示例：

```java
/* 父类 */
public class Parent {
    public void method(){
        System.out.println("parent method");
    }
}
/* 子类 */
public class SubClass extends Parent{
    //结果某一个子类重写了父类的方法，说不支持该操作了
    public void method() {
        throw new UnsupportedOperationException();
    }
}

//某一个类
public class SomeoneClass {
    //有某一个方法，使用了一个父类类型
    public void someoneMethod(Parent parent){
        parent.method();
    }
}
/* 测试类 */
public class Client {
    public static void main(String[] args) {
        SomeoneClass someoneClass = new SomeoneClass();
        someoneClass.someoneMethod(new Parent());
        someoneClass.someoneMethod(new SubClass());//此处报错, 运行时才产生报错 // 破坏了继承体系
    }
}

```

## 接口隔离原则

接口最小化原则，保证接口内的方法尽可能的最少; 使用多个隔离的接口，比使用单个接口要好

解决这种问题: 类A通过接口interface依赖类B，类C通过接口interface依赖类D，如果接口interface对于类A和类B来说不是最小接口，则类B和类D必须去实现他们不需要的方法。

```java
//这是错误示例，当实现此接口 时，必须实现不必要的方法 other()
public interface Mobile {
    public void call();//手机可以打电话
    public void sendMessage();//手机可以发短信
    public void game();//手机可以玩愤怒的小鸟？这是不合适的方法
}
//正确的方式是：新建一个接口来拓展Mobile接口
public interface SmartPhone extends Mobile{
    public void game();//智能手机的接口就可以加入这个方法了
}

```

## 依赖倒置原则

即实现都是易变的，而只有接口是稳定的，所以当依赖于接口时，实现的变化并不会影响客户端的调用。

就像常说的`面向接口编程`, 底层注入高层而不是高层依赖底层;

解决这种问题: 类A直接依赖类B，假如要将类A改为依赖类C，则必须通过修改类A的代码来达成, 这明显不符合"开闭原则"; 将类A修改为依赖接口interface，类B和类C各自实现接口interface，类A通过接口interface间接与类B或者类C发生联系，则会大大降低修改类A的几率

```java
//实现加法，抽象出一个接口，具体的实现类分别实现从数据库中读取、从文件系统读取，从XML文件读取
public interface Reader {
    public int getA();
    public int getB();
}

```

spring ioc 就是 依赖倒置的思想 (https://www.zhihu.com/question/23277575/answer/169698662)


## 迪米特原则（常用）

也称最小知识原则, 即一个类中不要出现无关的类, 如出现连续的get方法 `getXXX().getXXX().getXXX()`就可能违反了迪米特法则

比如类A的方法需要借助类B完成, 类B又需要借助类C中的方法完成

错误示例：

```java
public class A {
	public String name;
	public A(String name) {
		this.name = name;
	}
	public B getB(String name) {
		return new B(name);
	}
	public void work() {
		B b = getB("李四");
		C c = b.getC("王五");// A 和C耦合了
		c.work();
	}
}
public class B {
	private String name;
	public B(String name) {
		this.name = name;
	}
	public C getC(String name) {
		return new C(name);
	}
}
public class C {
	public String name;
	public C(String name) {
		this.name = name;
	}
	public void work() {
		System.out.println(name + "把这件事做好了");
	}
}

public class Client {
	public static void main(String[] args) {
		A a = new A("张三");
		a.work();
	}
}

```

正确的改进:

```java
public class A {
	public String name;
	public A(String name) {
		this.name = name;
	}
	public B getB(String name) {
		return new B(name);
	}
	public void work() {
		B b = getB("李四");
		b.work();
	}
}
public class B {
	private String name;
	public B(String name) {
		this.name = name;
	}
	public C getC(String name) {
		return new C(name);
	}
	
	public void work(){
		C c = getC("王五");
		c.work();
	}
}

类 C 不变

```


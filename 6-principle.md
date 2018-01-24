# 程序设计6原则

## 开-闭原则（总）

对拓展开放，对修改关闭, 实现一个热插拔的效果

和依赖倒置原则有点类似, 但是有区别, 我的感觉是开闭原则更加偏向的是使用抽象来避免修改源代码，主张通过扩展去应对需求变更，而依赖倒置更加偏向的是层和层之间的解耦。

这儿有个例子: 假设需要一个发送节日祝福的功能, 通过短信

```java
/**
 * 信息发送帮助类, 通过短信发送
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

    private MessageSend messageHelper;
    
    public MessageService() {
        this.messageHelper = new MessageSend();
    }
    
    public void send(String msg) {
        messageHelper.send(msg);
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
``` 

现在需要新支持另一种发送方式: 微信, 只好修改源码

```java
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

    private MessageSend messageHelper;
    private SendType t;
    public MessageService(SendType t) {
        this.t= t;
    }
    
    public void send(String msg) {
        // 根据不同的类型进行不同的处理
    }
}

// 然后是client增加新的调用

```
后续如果又要支持新的发送方式, 又需要大改MessageService源码;
如果遵循开闭原则, 会怎么样呢?

```java
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

    private ISendable messageHelper;
    
    public MessageService(ISendable messageHelper) {
        this.messageHelper = messageHelper;
    }
    
    public void send(String msg) {
        messageHelper.send(msg);
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

```

对于MessageService服务类来说，不用做任何修改，只需要扩展新的推送消息的工具类即可, 这就是遵循开闭原则的好处

## 合成复用原则（常用）

也可理解为单一职责原则, 一个类或方法，只负责单一功能，尽量解耦

错误示例：这里有明显多职责问题
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

## 里氏替换原则 (有事会违背以得到更多)

里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现

更直接的理解是: 子类一般不会重写父类的方法

需要多注意的是：经常会违背这个原则，以得到更多

里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

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
        someoneClass.someoneMethod(new SubClass());//此处报错
		//这个异常是运行时才会产生的，也就是说，我的SomeoneClass并不知道会出现这种情况，结果就是我调用下面这段代码的时候，本来我们的思维是Parent都可以传给someoneMethod完成我的功能，我的SubClass继承了Parent，当然也可以了，但是最终这个调用会抛出异常。
    }
}

```


## 接口隔离原则（常用）

接口最小化原则，保证接口内的方法尽可能的最少; 使用多个隔离的接口，比使用单个接口要好

```java
//这是错误示例，当实现此接口 时，必须实现不必要的方法playbird()
public interface Mobile {
    public void call();//手机可以打电话
    public void sendMessage();//手机可以发短信
    public void playBird();//手机可以玩愤怒的小鸟？这是不合适的方法
}
//正确的方式是：新建一个接口来拓展Mobile接口
public interface SmartPhone extends Mobile{
    public void playBird();//智能手机的接口就可以加入这个方法了
}

```

## 依赖倒置原则（常用）

即实现都是易变的，而只有抽象是稳定的，所以当依赖于抽象时，实现的变化并不会影响客户端的调用。
就像常说的`面向接口编程`

```java
//实现加法，抽象出一个接口，具体的实现类分别实现从数据库中读取、从文件系统读取，从XML文件读取
public interface Reader {
    public int getA();
    public int getB();
}

```


## 迪米特原则（常用）

也称最小知道原则，即一个类应该尽量不要知道其他类太多的东西，不要和陌生的类有太多接触。
错误示例：
```java
//读取的工具类
public class Reader {
    int a,b;
    private String path;
    private BufferedReader br;
    public Reader(String path){
        this.path = path;
    }
	//此方法不属于主要功能方法，暴露出来耦合度就太高了
    public void setBufferedReader() throws FileNotFoundException{
        br = new BufferedReader(new FileReader(new File(path)));
    }
	//此方法同上
    public void readLine() throws NumberFormatException, IOException{
        a = Integer.valueOf(br.readLine());
        b = Integer.valueOf(br.readLine());
    }
	、//此方法是功能方法，应该暴露
    public int getA(){
        return a;
    }
	//主功能方法，该暴露
    public int getB(){
        return b;
    }
}
public class Client {
    public static void main(String[] args) throws Exception {
        Reader reader = new Reader("E:/test.txt");
        reader.setBufferedReader();//耦合度太高
        reader.readLine();
        int a = reader.getA();
        int b = reader.getB();
        //以下用于计算等等
    }
}

```

正确的改进:

```java
public class Reader {
    int a,b;
    private String path;
    private BufferedReader br;
    public Reader(String path) throws Exception{
        super();
        this.path = path;
        setBufferedReader();
        readLine();
    }
    //注意，我们变为私有的方法，给封装起来了
    private void setBufferedReader() throws FileNotFoundException{
        br = new BufferedReader(new FileReader(path));
    }
    //注意，我们变为私有的方法
    private void readLine() throws NumberFormatException, IOException{
        a = Integer.valueOf(br.readLine());
        b = Integer.valueOf(br.readLine());
    }
	//只暴露这两个方法
    public int getA(){
        return a;
    }
    public int getB(){
        return b;
    }
}

```


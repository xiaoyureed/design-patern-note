# 命令模式

别名"动作模式", "事务模式"

## 定义

将命令(请求)封装为一个对象, 从而可以用不同的命令对目标对象进行各种操作, 分离了命令发出者和执行者;

## 结构

![](../assets/pic9.png)
![](../assets/pic10.png)

## 代码分析

```java
// 先看看 tv 的api

/**
 * command receiver: tv
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:02:53
 */
public class Tv {

    /**
     * current channel
     */
    private int channel = 0;
    
    /**
     * turn on tv
     */
    public void turnOn() {
        System.out.println("turn on tv.");
    }
    
    /**
     * turn off tv
     */
    public void turnOff() {
        System.out.println("turn off tv.");
    }
    
    /**
     * change tv channel
     * @param channel
     * @return
     */
    public int changeChannel(int channel) {
        int pre = this.chanel;
        this.channel = channel;
        return pre;
    }
    /**
     * show current channel
     * @return
     */
    public int showChannel() {
        return this.channel;
    }
}

// 在看 对 命令 的封装

/**
 * command interface
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:02:11
 */
public interface Command<T> {

    void exec();
}

// 命令: 打开电视, 关闭电视, 切换频道

/**
 * command: turn on tv
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:09:14
 */
public class CommandOn implements Command<Tv> {
    
    private Tv tv;
    
    public CommandOn(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void exec() {
        tv.turnOn();
    }

}

/**
 * command: turn off tv
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:14:11
 */
public class CommandOff implements Command<Tv> {

    private Tv tv;
    
    public CommandOff(Tv tv) {
        this.tv = tv;
    }
    
    @Override
    public void exec() {
        tv.turnOff();
    }

}

/**
 * command: change tv channel
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:15:46
 */
public class CommandChangeChannel implements Command<Tv> {
    
    private int channel;
    private Tv tv;
    
    public CommandChangeChannel(Tv tv, int channel) {
        this.channel = channel;
        this.tv = tv;
    }

    @Override
    public void exec() {
        tv.changeChannel(channel);
    }

}

/**
 * invoker, 可以看作是遥控器, 命令发送者;
 * 它和tv, 即命令接受者完全解耦
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:22:35
 */
public class Controller {

    private Command<?> cmdOn;
    private Command<?> cmdOff;
    private Command<?> cmdChangeChannel;
    
    public Controller(Command<?> cmdOn, Command<?> cmdOff, Command<?> change) {
        this.cmdOn = cmdOn;
        this.cmdOff = cmdOff;
        this.cmdChangeChannel = change;
    }
    
    
    public void turnOn() {
        cmdOn.exec();
    }
    
    public void turnOff() {
        cmdOff.exec();
    }
    
    public void changeChannel() {
        cmdChangeChannel.exec();
    }
    
}

// 测试

/**
 * client
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午6:59:00
 */
public class Person {

    public static void main(String[] args) {
        Tv tv = new Tv();
        //----------------------------准备工作, 准备遥控器-----------------------------------
        Command<?> cmdOn = new CommandOn(tv);
        Command<?> cmdOff = new CommandOff(tv);
        Command<?> change = new CommandChangeChannel(tv, 2);
        Controller controller = new Controller(cmdOn, cmdOff, change);
        //-----------------------------------准备完毕------------------------------------------------
        // 此时controller和tv是完全分离的
        controller.turnOn();
        controller.changeChannel();
        System.out.println("current channel: " + tv.showChannel());
        controller.turnOff();
    }
}


```

## 模式分析

*   命令模式本质是对命令的封装, 将发出者和执行者分离
*   命令模式关键在于引入了抽象命令接口, 且发送者针对命令接口编程, 只有实现了抽象命令接口的具体命令才能与命令接收者关联
*   相关模式: 职责链模式, 容易将二者关联在一起的原因是，二者都是为了处理请求/命令而存在的，而且二者都是为了将请求者与响应者解耦，不同的是命令模式中，客户端需要知道一个命令的接受者，在创建命令的时候就把接受者与命令绑定在一起发送给调用者，而职责链模式中，客户端并不关心最终处理请求的对象是谁，客户端只是封装一个请求对象，随后交给职责链的头部而已，也正因为这样，二者的实现方式，有着很大的区别

优点:

*   解耦
*   新的命令加入系统很容易
*   可以很容易的设计一个命令队列or命令组合(宏命令)
*   可以方便的设计命令的撤销or重做

缺点:

*   具体命令类过多, 因为每一个命令都需要有一个对应的具体命令类

## 适用场景

*   系统需要将请求调用者和请求接受者解耦, 两者不直接交互
*   系统在不同时间发出请求, 需要将请求做成请求队列执行
*   系统需要支持命令撤销or重做
*   系统需要将一组命令组合在一起, 即支持宏命令

## 应用实例

java.lang.Runnable

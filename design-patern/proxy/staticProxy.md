# 静态代理

## 定义

给某一个对象提供一个代 理，并由代理对象控制对原对象的引用

代理对象和被代理对象一般实现相同的接口，调用者与代理对象进行交互。代理的存在对于调用者来说是透明的，调用者看到的只是接口



## 代码分析

```java
/**
 * 代理目标和代理都要实现的统一接口
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月26日 下午10:40:28
 */
public interface Moveable {

    void move();
}

/**
 * 代理目标
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月26日 下午10:43:01
 */
public class Car implements Moveable {

    @Override
    public void move() {
        System.out.println("Car 开始移动...");
        try {
            Thread.sleep(new Random().nextInt(10000));// 单位是ms
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

/**
 * Car 代理类, 添加了时间统计功能
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月26日 下午10:47:44
 */
public class CarTimingProxy implements Moveable {
    
    private Moveable car;
    public CarTimingProxy(Moveable car) {
        super();
        this.car = car;
    }

    @Override
    public void move() {
        long start = System.currentTimeMillis();
        car.move();
        long end = System.currentTimeMillis();
        System.out.println("move持续时间: " + (end-start));
    }

}

/**
 * car 代理类, 记录日志
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月26日 下午10:57:13
 */
public class CarLogProxy implements Moveable {
    
    private Moveable car;
    
    public CarLogProxy(Moveable car) {
        super();
        this.car = car;
    }

    @Override
    public void move() {
        System.out.println("log: Car开始移动");
        car.move();
        System.out.println("log: Car结束移动");
    }

}

/**
 * 测试
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月26日 下午11:00:13
 */
public class Client {

    public static void main(String[] args) {
        Moveable car = new CarLogProxy(new CarTimingProxy(new Car()));
        car.move();
    }
}

```

## 模式分析

* 代理目标&代理类需要实现统一接口
* 相关模式: 适配器模式, 对于适配器模式当中的定制适配器，它与静态代理有着相似的部分，二者都有复用功能的作用，不同的是，静态代理会修改一部分原有的功能，而适配器往往是全部复用，而且在复用的同时，适配器还会将复用的类适配一个接口

优点:

*   解耦
*   远程代理使得客户端可以访问在远程机器上的对象

缺点:

*   请求变慢
*   系统变得复杂

## 适用场景



*   远程代理: 为远程的对象提供一个本地的代理, 可以不在一台主机, 也可以在同一台主机
*   虚拟代理: 以一个消耗资源小的对象代理消耗资源大的对象
*   保护代理: 控制被代理对象的访问, 给不同用户提供不同级别访问权限
*   缓冲代理: 在代理对象中为某一个操作的结果提供临时的缓存存储空间，以便在后续使用中能够共享这些结果，从而可以避免某些方法的重复执行，优化系统性能

## 实例

EJB、Web Service等分布式技术都是代理模式的应用。在EJB中使用了RMI机制，远程服务器中的企业级Bean在本地有一个桩代理，客户端通过桩来调用远程对象中定义的方法，而无须直接与远程对象交互。在EJB的使用中需要提供一个公共的接口，客户端针对该接口进行编程，无须知道桩以及远程EJB的实现细节。

典型的jdk中的java.lang.reflect.Proxy

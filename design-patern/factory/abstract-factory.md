# 抽象工厂模式

## 引入

在工厂方法模式中每一个具体工厂对应一种具体产品。每一个具体工厂中只有一个工厂方法。或者一组重载的工厂方法。但是有时候我们需要一个工厂可以提供一组对象,(这组对象各不相同, 不属于一个继承体系), 而不是单一的产品对象(or对象组), 此时有必要引入抽象工厂模式.

## 定义

提供这样一个接口, 能够创建一系列相关or相互依赖的对象而不必指定这些对象具体的类

隔离了具体类, client无需知道是哪个具体类被生成, 只需知道是那个具体工厂即可

## 代码分析

```java
/**
 * factory interface
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:08:47
 */
public interface IFactory {

    AbsProductA getProdA();
    AbsProductB getProdB();
}

/**
 * abstract productA
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:12:00
 */
public abstract  class AbsProductA {

    public abstract void use();
}

/**
 * abstract productB
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:14:45
 */
public abstract class AbsProductB {

    public abstract void eat();
}

/**
 * concrete product A1
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:19:21
 */
public class ProductA1 extends AbsProductA {

    @Override
    public void use() {
        System.out.println("ProductA1: use()");
    }

}

/**
 * concrete product B1
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:21:00
 */
public class ProductB1 extends AbsProductB {

    @Override
    public void eat() {
        System.out.println("ProductB1: eat()");
    }

}

/**
 * concrete factory 1: 生产产品组1
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:16:03
 */
public class Factory1 implements IFactory {

    @Override
    public AbsProductA getProdA() {
        return new ProductA1();
    }

    @Override
    public AbsProductB getProdB() {
        return new ProductB1();
    }

}

/**
 * concrete product A2
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:23:41
 */
public class ProductA2 extends AbsProductA {

    @Override
    public void use() {
        System.out.println("ProductA2: use()");
    }

}

/**
 * concrete product B2
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:24:50
 */
public class ProductB2 extends AbsProductB {

    @Override
    public void eat() {
        System.out.println("ProductB2: eat()");
    }

}

/**
 * concrete factory2: 生产 产品组2
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午3:23:05
 */
public class Factory2 implements IFactory {

    @Override
    public AbsProductA getProdA() {
        return new ProductA2();
    }

    @Override
    public AbsProductB getProdB() {
        return new ProductB2();
    }

}

/**
    测试
*/
public class Client {

    public static void main(String[] args) {
        IFactory fac1 = new Factory1();// 隔离了具体类, client无需知道是哪个具体类被生成, 只需知道是那个具体工厂即可
        AbsProductA A1 = fac1.getProdA();
        AbsProductB B1 = fac1.getProdB();
        A1.use();
        B1.eat();
        
        IFactory fac2 = new Factory2();
        AbsProductA A2 = fac2.getProdA();
        AbsProductB B2 = fac2.getProdB();
        A2.use();
        B2.eat();
    }
}

```

## 模式分析

优点:

*   抽象工厂模式隔离了具体类的生成, 客户端并不知道哪个具体类被创建。此时更换具体工程变得很容易。可以很容易改变整个软件系统的行为
*   当不在一个继承体系中的多个对象被设计在一起工作时, 抽象工厂模式能够保证客户端始终只使用同一个产品组中的对象。
*   符合开闭原则

缺点:

*   开闭原则支持的不是很好, 增加新的工厂和产品族容易, 增加爱新的产品等级结构(继承体系)麻烦

## 使用场景

*   一个系统不应当依赖于产品类实例如何被创建组合等等细节.这对于所有类型的工程模式都适用。
*   系统中有多于一个的产品族，而且每次只使用其中一个产品族。属于同一产品族的产品将在一起使用

## 应用实例

例如, 一个系统需要更换界面主题, 要求界面中的按钮, 文本框, 背景色, 等等元素一起改变, 可以将这些元素放到同一个工厂.

# 分类

工厂模式可以分为三类： 

- 简单工厂模式（Simple Factory） ---简单工厂实际是工厂方法的一种特例, 也可以就把简单工厂归类为工厂方法
- 工厂方法模式（Factory Method） --- 在简单工厂上增加了工厂接口, 增加新的生产逻辑不必修改代码, 直接新增一个工厂实现
- 抽象工厂模式（Abstract Factory） -- 工厂可以生产不同的一组对象, 即工厂提供多个方法, 每个方法可以生产一类同一类型对象; 没有关系的一组对象可以放到一个工厂生成;

 这三种模式从上到下逐步抽象，并且更具一般性。

 三个工厂相关模式区别：

*   首先从简单工厂进化到工厂方法，是因为工厂方法弥补了简单工厂对修改开放的弊端，即简单工厂违背了开闭原则。

*   从工厂方法进化到抽象工厂，是因为抽象工厂弥补了工厂方法只能创造一个系列的产品的弊端(只能创建同一个继承体系产品)。


此外, 工厂模式还有一个巨大优点, 方便拓展外部组建, 如果有一个第三方jar包，我们自己要拓展他的功能：

```java
//抽象产品
interface Product{}
//具体产品
class ProductA implements Product{}
class ProductB implements Product{}
//工厂接口
interface Factory{
    Product getProduct();
}
//具体的工厂A，创造产品A
class FactoryA implements Factory{
    public Product getProduct() {
        return new ProductA();
    }
}
//具体的工厂B，创造产品B
class FactoryB implements Factory{
    public Product getProduct() {
        return new ProductB();
    }
}
/*   假设以上是一个第三方jar包中的工厂方法模式，我们无法改动源码   */
//我们自己特有的产品
interface MyProduct{}
//我们自己特有的产品实现
class MyProductA implements MyProduct{}
class MyProductB implements MyProduct{}
//我们自己的工厂接口
interface MyFactory{
    MyProduct getMyProduct();
}
//我们自己特有的工厂A，产生产品A
class MyFactoryA implements MyFactory{
    public MyProduct getMyProduct() {
        return new MyProductA();
    }
}
//我们自己特有的工厂B，产生产品B
class MyFactoryB implements MyFactory{
    public MyProduct getMyProduct() {
        return new MyProductB();
    }
}
/*  到这里是我们自己的一套工厂方法模式，去创造我们自己的产品，以下我们将以上二者组合   */
//我们使用组合的方式将我们的产品系列和jar包中的产品组合起来, 即新建一个组合工厂, 继承我们自己原有工厂接口and第三方组建接口
class AssortedFactory implements MyFactory,Factory{
    MyFactory myFactory;
    Factory factory;
    public AssortedFactory(MyFactory myFactory, Factory factory) {
        super();
        this.myFactory = myFactory;
        this.factory = factory;
    }
    public Product getProduct() {
        return factory.getProduct();
    }
    public MyProduct getMyProduct() {
        return myFactory.getMyProduct();
    }
}

``` 
# 装饰器模式(Decorator)

## 定义

代理模式（Proxy Pattern），为其它对象提供一种代理以控制对这个对象的访问。

装饰模式（Decorator Pattern），给一个对象添加一些额外的职责。

被代理对象由代理对象创建，客户端甚至不需要知道被代理类的存在；被装饰对象由客户端创建并传给装饰对象

装饰可以 装了又装，层层包裹；代理通常不会代了又代。

## 代码分析

```java
/* 
 * 创建一个对象的抽象也就是接口 
*/  
public interface Basket {  
    public void show();  
      
}
/** 
 *身份：被装饰的对象 
 *一个对接口的实现，这个对象表示要我们将来要修饰的篮子里装内容，如果想修饰篮子的造型，还可以创建其他类实现Basket的接口,比如Shape 
 * 不理解的话可以查看java语言的接口抽象机制 
*/  
public class Original implements Basket{  
    public void show(){  
      System.out.println("The original Basket contains");  
    }  
}

/** 
 *身份：装饰器 
 *为原来的类添加新的功能 
*/  
public class AppleDecorator implements Basket{  
    private Basket basket;  
    public AppleDecorator( Basket basket){  
      super();  
      this.basket = basket;  
    }  
      
    public void show(){  
      basket.show();  
      System.out.println("An Apple");  
    }  
      
}
/** 
 *身份：装饰器 
*/  
public class BananaDecorator implements Basket{  
    private Basket basket;  
    public BananaDecorator(Basket basket){  
        super();  
        this.basket = basket;  
    }  
      
    public void show(){  
        basket.show();  
        System.out.println("A Banana");  
    }  
      
}

/** 
 *身份：装饰器 
*/  
public class OrangeDecorator implements Basket{  
    private Basket basket;  
    public OrangeDecorator(Basket basket){  
        super();  
        this.basket = basket;  
    }  
      
    public void show(){  
        basket.show();  
        System.out.println("An Oranage");  
    }  
      
}

public class DecoratorPattern {  
  
    /** 
     * @param args the command line arguments 
*/  
    public static void main(String[] args) {  
        // TODO code application logic here  
        Basket basket = new Original();  
        //一个装饰的过程  
        Basket myBasket =new AppleDecorator(new BananaDecorator(new OrangeDecorator(basket)));
        myBasket.show();  
    }  
}

```

## 模式分析

*   使用装饰模式来实现扩展比继承更加灵活，它以对客户透明的方式动态地给一个对象附加更多的责任。装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。
*   尽量保持具体构件类Component作为一个“轻”类，也就是说不要把太多的逻辑和状态放在具体构件类中，可以通过装饰类

优点:

*   装饰模式可以提供比继承更多的灵活性。(继承是一种耦合度较大的静态关系，无法在程序运行时动态扩展)
*   可以通过一种动态的方式来扩展一个对象的功能，通过配置文件可以在运行时选择不同的装饰器，从而实现不同的行为。
*   通过使用不同的具体装饰类以及这些装饰类的排列组合，可以创造出很多不同行为的组合。可以使用多个具体装饰类来装饰同一对象，得到功能更为强大的对象。
*   体构件类与具体装饰类可以独立变化，用户可以根据需要增加新的具体构件类和具体装饰类，在使用时再对其进行组合，原有代码无须改变，符合“开闭原则”

缺点:

*   这种比继承更加灵活机动的特性, 难于debug

## 适用场景

选择关键点：添加的功能是否需要动态组装 

*   在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责。
*   需要动态地给一个对象增加功能，这些功能也可以动态地被撤销。
*   当不能采用继承的方式对系统进行扩充或者采用继承不利于系统扩展和维护时。不能采用继承的情况主要有两类：第一类是系统中存在大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长；第二类是因为类定义不能继承（如final类）.

## 实例

java.io.BufferedInputStream(InputStream) , 和适配器有类似

servlet 中的 HttpServletRequestWrapper

# 单例模式(Singleton Pattern)

## 引入

对于一个系统中的某些类来说, 只有一个实例很重要. 例如一个系统中可以有多个打印任务，但是只能有一个正在工作中的打印任务。只能有一个窗口管理器或文件系统, 只能有一个ID生成器。此时让这个类自身负责保存它的唯一实例, 并且无法在外部被实例化比较好。并且提供一个方便的访问该实例的方法

## 定义

单例模式，确保某一个类只有一个实例。而且自行实例化，并向整个系统提供这个实例


## 代码分析

结构太简单, 咱们直接看代码

```java
/**
 * 带有双重判断的单例模式
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午4:48:41
 */
public class SyncSingleton {

  //一个静态的实例
    private static SyncSingleton instance;
  //私有化构造函数
    private SyncSingleton() {}
  //给出一个公共的静态方法返回一个单一实例
    public static SyncSingleton getInstance() {
        if (instance == null) {
          //这里第二重判断：AB两个线程同时进入第一个判断内部，此
          //时A当先拿到锁进入第二判断，创建了对象，B拿到锁后会再次
          //进行判断，如果此处不判断，则会创建第二个对象；
            synchronized (SyncSingleton.class) {
                if (instance == null) {
                    instance = new SyncSingleton();
                }
            }
        }
        return instance;
    }
}

///////////////////////// 推荐这种写法: 借助 jvm 的特性帮助实现 singleton ////////////////////////////////////

/**
 * 更安全, 更简单的单例
 *
 *首先来说一下，这种方式为何会避免了上面莫名的错误，
 *主要是因为一个类的静态属性只会在第一次加载类时初始化，这是类加载机制决定的
 *另外由于静态变量只初始化一次，所以singleton仍然是单例的。
 *
 * @version 0.1
 * @author xy
 * @date 2018年1月25日 下午5:02:23
 */
public class Singleton {
    private Singleton(){}
    public static Singleton getInstance(){
        return SingletonInstance.instance;
    }
    //类的静态成员只加载一次，这保证了只有一个对象
    private static class SingletonInstance{
        private static Singleton instance = new Singleton();
    }
}



```

## 模式分析

*   构造函数私有
*   提供一个公有静态工厂方法
*   双重判断, 同步第二个判断, 解决多线程问题
*   原型模式：单例模式是只有一个实例，原型模式每拷贝一次都会创造一个新的实例。

优点:

* 节约系统资源
* 在单例模式基础上拓展, 获取指定个数的实例

缺点:

* 违背了单一职责原则, 单例类既有工厂方法, 又有业务方法, 即充当工厂, 又充当产品

## 适用场景

*   系统只需要一个实例对象
*   客户端调用只允许一个调用点
*   系统要求一个类只能有指定个数的实例, 可拓展单例模式

## 应用实例

主键编号生成器

spring中的bean默认都是单例生成的;

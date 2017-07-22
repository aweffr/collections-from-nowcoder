# Singleton设计模式

## Motivation
当一个系统/软件需要全局配置信息, 或者一个Factory, 或者一个*主控类*, 可以用Singleton的代码方式达成目的.

实现上, 基本思路在于:
1. 私有（private）的构造函数，阻止这个类的用户(Client)自己新建实例.
2. 即然这个类是不可能形成实例，那么，我们需要一个静态的方式让其形成实例：getInstance(). 注意这个方法是在new自己，因为其可以访问私有的构造函数，所以他是可以保证实例被创建出来的.
3. 在getInstance()中，先做判断是否已形成实例，如果已形成则直接返回，否则创建实例.
4. 所形成的实例保存在自己类中的私有成员中.
5. 我们取实例时，只需要使用Singleton.getInstance()就行了.

## Eager版本
*饥汉版本能少考虑好多问题*
```java
public class Singleton1 {
    private static Singleton1 singleton1 = new Singleton1();

    private Singleton1() {
    }

    public static Singleton1 getInstance() {
        return singleton1;
    }
}
```

饥汉版本的问题在于我们不能自己控制类被创建的时间(比如我们希望它能在我们第一次getInstance()时才被真正创建, 也许该类的创建还依赖于与别的类的资源或者某个运行时生成的配置文件), 饥汉版本类的创建其实委托给了类装载器.

## Lazy版本
```java
public class Singleton2 {
    private static Singleton2 instance = null;

    private Singleton2() {
    }

    public static Singleton2 getInstance() {
        if (instance == null) {
            instance = new Singleton2();
        }
        return instance;
    }
}
```

这个版本在单线程下`基本`没有问题. 但是在**多线程**下, 如果有一个线程运行到`if (instance == null)`后被中断切换到另一个线程,则可能会创建多个实例.
因此`if (instance == null)`的条件判断和`new Singleton2()`应该组成一个原子操作,如下所示

## Lazy版本(考虑多线程)
```java
public class Singleton3 {
    private String message = "This is ver3.";
    private static Singleton3 instance = null;

    private Singleton3() {
    }

    public static Singleton3 getInstance() {
        synchronized (Singleton3.class) {
            if (instance == null) {
                instance = new Singleton3();
            }
        }
        return instance;
    }
}
```

这个版本保证了基本的正确性了. 但是它有了一个新的问题: 现在所有的getInstance()操作都要线程同步, 严重影响后续性能.
那还要做调整: 在线程同步之前先判断一次`if (instance == null)`, 如果对象已创建, 那么就不需要线程同步了.

## "双重检查"版本
*这就是常用"双重检查(double-check)"版本.*
```java
public class Singleton4 {
    private String message = "This is ver4.";
    private static Singleton4 instance = null;

    private Singleton4() {
    }

    public static Singleton4 getInstance() {
        if(instance == null) {
            synchronized (Singleton4.class) {
                if (instance == null) {
                    instance = new Singleton4();
                }
            }
        }
        return instance;
    }
}
```

## 关键字volatile的作用

但是`double-check`版本还是可能存在问题的, 因为`instance = new Singleton4()`一句在JVM中大概做了三件事情:
1. 给Singleton分配内存
2. 调用Singleton的构造函数来初始化成员变量, 形成实例
3. 将singleton对象指向分配内存的空间


    而JVM的即时编译器(JIT)是可以对指令重排序优化的, 即2和3的顺序是不能保证的. 
    最后执行的顺序可能是1-2-3也可能是1-3-2. 在1-3-2情况下, 
    如果线程一进入`instance = new Singleton4()`而在真正构造函数执行前被挂起, 
    别的线程还是会看到instance非null, 然后报NullPointerExpection.

对此, 我们还需要做的事是申明类的实例是volatile的. 如下
```java
public class Singleton5 {
    private String message = "This is ver5.";
    private static volatile Singleton5 instance = null;

    private Singleton5() {
    }

    public static Singleton5 getInstance() {
        if (instance == null) {
            synchronized (Singleton5.class) {
                if (instance == null) {
                    instance = new Singleton5();
                }
            }
        }
        return instance;
    }
}
```
使用volatile有两个功用:
1. 这个变量不会再多个线程中存在副本, 直接从内存读取
2. 这个关键字会禁止指令重排序优化, 也就是`volatile`变量的赋值操作后边会有一个内存屏障(在汇编代码上), 保证读操作不会被重排序到内存屏障之前.
3. 本说明只对Java5及以后有用 (在java5.0修改了内存模型,使用volatile声明的变量可以强制屏蔽编译器和JIT的优化工作) =>老版本的Java内存模型是有缺陷的

以上版本不够优雅, 以下请观摩大师版本
## Effective Java 1st 版本 (内部类)
```java
public class SingletonMaster1 {
    private SingletonMaster1() {
    }

    private static class SingletonHolder {
        private static final SingletonMaster1 INSTANCE = new SingletonMaster1();
    }

    public static SingletonMaster1 getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

上面这种方式，仍然使用JVM本身机制保证了线程安全问题；由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它只有在getInstance()被调用时才会真正创建；同时读取实例的时候没有`synchronized`操作，没有性能缺陷；也不依赖 JDK 版本。
关键在于:
1. JSL规范定义，类的构造必须是原子性的，非并发的
2. 由JVM来保证线程安全

## Effective Java 2nd 版本 (枚举)
```java
public enum Singleton{
   INSTANCE;
}
```
当一个Java类第一次被真正使用到的时候静态资源被初始化、Java类的加载和初始化过程都是线程安全的。所以，创建一个enum类型是线程安全的。

## 还有一个问题(序列化)
如果我们的这个Singleton类是一个关于我们程序配置信息的类。我们需要它有序列化的功能，那么，当反序列化的时候，我们将无法控制别人不多次反序列化。不过，我们可以利用一下Serializable接口的readResolve()方法，比如：
```java
public class Singleton implements Serializable
{
    ......
    ......
    protected Object readResolve()
    {
        return getInstance();
    }
}
```

序列化是Java中一个很常用而且很强大的功能。将java对象保存到磁盘，以后再从磁盘中读出来，这是java最常用到的功能之一。在基本的情况下，序列化能够“简单的起作用(just work)”。然而，随着越来越复杂的对象格式以及设计模式的被采用，透明的对象(transparent object)序列化可以“简单的起作用(just work)”的可能性变得越来越不可能了。在处理一个可控制集合的实例，比如单例和enum，就是序列化需要一些而外帮助的一种场景。

**序列化操作提供了一个很特别的钩子(hook)-- 类中具有一个私有的被实例化的方法readresolve(),这个方法可以确保类的开发人员在序列化将会返回怎样的object上具有发言权。**

现在通过可序列化的工具，我们可以将一个单例的实例对象写到磁盘，然后再读回来，从而有效地获得一个实例。即使构造函数是私有的，可序列化工具依然可以通过特殊的途径去创建类的一个新的实例。

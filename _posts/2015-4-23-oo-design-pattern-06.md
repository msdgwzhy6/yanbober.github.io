---
layout: post
keywords: 设计模式, 单例模式, Singleton Pattern
description: 设计模式系列学习记录
title: "设计模式(创建型)之单例模式(Singleton Pattern)"
categories: [设计模式]
tags: [单例模式, 创建型模式]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**概述**

单例模式可能是23种设计模式中最简单的。应用也非常广泛，譬如Android中的数据库访问等操作都可以运用单例模式。

<hr>

##**核心**

**概念：** 确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。

**重点：** 单例模式结构重要核心模块：

私有的构造方法

指向自己实例的私有静态引用

以自己实例为返回值的静态的公有的方法

**模式：**根据实例化对象时机不同分为懒汉模式与饿汉模式

饿汉模式是单例类被加载时候就实例化一个对象交给自己的引用；懒汉模式时在调用取得实例方法的时候才会实例化对象。

**记忆小技巧：**饿汉模式（E）-加载类初始化实例化（先），懒汉模式（L）-调运实例初始化（后）；所以E先L后，依据字母表顺序记忆即可。

<hr>

##**使用场景**

- 系统只需要一个实例对象。
- 需要频繁实例化且销毁的对象。
- 创建对象时耗时过多或者耗资源过多，但又经常用到的对象。
- 有状态的对象。
- 频繁访问数据库或文件的对象。

<hr>

##**程序猿实例**

以下代码都以模拟Android App数据库操作封装管理类为例说明各种单例模式。

**饿汉模式**代码示例：

如下当类DBManager被加载时，静态变量mInstance被初始化，单例类的唯一实例被创建。

{% highlight ruby %}
package yanbober.github.io;
//饿汉模式
class DBManager {
    private static DBManager mInstance = new DBManager();

    private DBManager() {}

    public static DBManager getInstance() {
        return mInstance;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

**懒汉模式Bug版本：**调用getInstance()方法时实例化对象，在类加载时不自行实例化（延迟加载）。但是如果你足够钻牛角尖你会发现如下代码有很大一个Bug，那就是
多线程访问时，如果DBManager实例化比较耗时时会出现创建多个mInstance对象。也就是违反了单例模式的初衷，那么有啥办法可以解决吗？

{% highlight ruby %}
package yanbober.github.io;
//懒汉模式（线程对象不唯一不安全）
class DBManager {
    private static DBManager mInstance;

    private DBManager() {}

    public static DBManager getInstance() {
        if (null == mInstance) {
            mInstance = new DBManager();
        }
        return mInstance;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

**懒汉模式常规性能Ok版本：**升级如上懒汉模式代码为线程安全的模式如下，其实质就是通过添加synchronized进行锁定。但是苛刻的你一定又会挑刺吧，多线程安全了，但是我要是
高并发这玩意不是及其低性能么？每次都去判断锁，多蛋疼。是这样的，慢慢来嘛，下面会继续升级的。

{% highlight ruby %}
package yanbober.github.io;
//懒汉模式（线程安全高并发低性能）
class DBManager {
    private static DBManager mInstance;

    private DBManager() {}

    public static synchronized DBManager getInstance() {
        if (null == mInstance) {
            mInstance = new DBManager();
        }
        return mInstance;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

**懒汉模式Bug版本：**线程安全解决了，高并发没解决，下面就解决了高并发问题，但是一不小心带了一个新的Bug，妹的！
仔细分析下面的代码会发现，假设现在有A、B线程同时调运getInstance，然后A进入了DBManager实例化，此时假设B已经进入if (null == mInstance)然后在等待
A释放锁，但是问题来了，当A释放锁以后已经实例化了变量mInstance，B却傻逼的不知道，然后继续持有锁实例化一个新的对象。真蛋疼。。。继续等待升级版吧！

{% highlight ruby %}
package yanbober.github.io;
//懒汉模式（高并发OK线程不安全）
class DBManager {
    private static DBManager mInstance;

    private DBManager() {}

    public static DBManager getInstance() {
        if (null == mInstance) {
            synchronized(DBManager.class) {
                mInstance = new DBManager();
            }
        }
        return mInstance;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

**懒汉模式相对靠谱版：**上面代码可能有Bug，那就在Bug处在判断下是不是null就完事了，所以就有了如下代码。
如果使用双重检查锁定来实现懒汉式单例类，需要在静态成员变量instance之前增加修饰符volatile，
被volatile修饰的成员变量只能在JDK 1.5及以上版本中才能正确执行。
volatile关键字会屏蔽Java虚拟机所做的一些代码优化，会导致系统运行效率降低，所以这种写法的单例模式也不是一种牛叉的写法。 

{% highlight ruby %}
package yanbober.github.io;
//懒汉模式(相对比较靠谱模式)
class DBManager {
    private static volatile DBManager mInstance;

    private DBManager() {}

    public static DBManager getInstance() {
        if (null == mInstance) {
            synchronized(DBManager.class) {
                if (null == mInstance) {
                    mInstance = new DBManager();
                }
            }
        }
        return mInstance;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

**个人学到的相对最靠谱优化的懒汉模式：**上面把双重检查都用上了都不够优化，鱼和熊掌不可兼得，这里好了那里就不好，那里好了这里就不好，各种问题，
但是目前来看还是有二者兼得的靠谱写法的，这里就来一发。

如下写法的亮点就在于这种写法使用了JVM本身机制保证线程安全问题；由于InstanceHolder是私有的，
除了getInstance()之外没有办法访问它，因此它是懒汉式的；同时读取实例的时候不会进行同步，没有性能缺陷；也不依赖JDK版本。

{% highlight ruby %}
package yanbober.github.io;
//懒汉模式(靠谱模式)
class DBManager {
    private static class InstanceHolder {
        static final DBManager INSTANCE = new DBManager();
    }

    private DBManager() {}

    public static DBManager getInstance() {
        return InstanceHolder.INSTANCE;
    }

    public boolean update() {
        System.out.println("update dbase success!");
        return true;
    }
}

public class Main {
    public static void main(String[] args) {
        DBManager dbManager = DBManager.getInstance();
        dbManager.update();
    }
}
{% endhighlight %}

<hr>

##**总结一把**

上面懒汉模式会有一堆分析，所以说选择啥样的单例模式好呢？其实没标准答案，自己权衡吧，依据上面分析的利弊去权衡。

单例模式的主要优点如下：

- 单例模式提供访问对象实例的唯一性。
- 内存中只存在一个对象，对于频繁创建和销毁的对象可以提高性能。

单例模式的主要缺点如下：

- 单例类的扩展很困难。
- 单例类的职责过重，充当多角色，在一定程度上违背了“单一职责原则”。

{% highlight ruby %}

{% endhighlight %}


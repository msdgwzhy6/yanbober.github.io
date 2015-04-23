---
layout: post
keywords: 设计模式, 抽象工厂模式, Abstract  Factory Pattern
description: 设计模式系列学习记录
title: "设计模式(创建型)之抽象工厂模式(Abstract  Factory Pattern)"
categories: [设计模式]
tags: [抽象工厂模式, 创建型模式]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**概述**

前面介绍了工厂方法模式通过引入工厂等级结构（实现统一的工厂接口），
解决了简单工厂模式中工厂类职责太重（违背单一职责原则）的问题，
但是由于工厂方法模式中的每个工厂只生产一类产品（通过实现同一个工厂接口），
带来的问题就是系统中会增加大量的工厂类。

问题来了。。。

如何解决如上描述的工厂方法相对来说的缺点呢？

可以考虑将一些相关的产品组成一个“产品族”，由同一个工厂来统一生产，这就是抽象工厂模式。
也就是说抽象工厂模式是工厂方法模式的升级牛逼版本，他用来创建一组相关或者相互依赖的对象。
与工厂方法模式的区别在于工厂方法模式针对的是一个产品等级结构；抽象工厂模式则是针对的多个产品等级结构。
工厂方法模式提供的所有产品都是实现同一个接口，而抽象工厂模式所提供的产品是实现自不同的接口。
抽象工厂模式产品族概念：产品族是指位于不同产品等级结构中功能相关联的产品组成的家族。
抽象工厂模式所提供的一系列产品就组成一个产品族；而工厂方法提供的一系列产品称为一个等级结构。

<hr>

##**核心**

先上一张图：

<img src="http://yanbober.github.io/image/sjms/1.png" />

我们还是以苦逼的程序猿为例来说抽象工厂模式的一些核心概念。通过上图你可以发现，横纵二维坐标可以确定平面上一个唯一的点，这也就是抽象工厂的核心。

**产品等级结构：**就是继承结构。就像上面Android，IOS，PHP这些技能继承自一个抽象的技能类（譬如前面的ICode），这个抽象类与这些子类构成了产品等级结构。
同理的Android书，C语言书，脚本书继承自一个工具书类，这个工具书抽象类与这些子类构成了等级结构。

**产品族：**抽象工厂模式中的产品族官方定义是指由同一个工厂生产的，位于不同产品等级结构中的一组产品。
譬如上面的Android位于技能等级结构中，Android书位于工具书等级结构中，Android技能和Android书是位于不同产品结构的一组产品，但是任何一个程序猿都需要具备技能和工具书，
譬如一个Android程序猿需要有Android技能及Android书，所以这个Android程序猿就是一个产品族。

**概念：** 提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式。

**重点：** 抽象工厂模式结构重要核心模块：

抽象工厂

声明一组用于创建一族产品的方法，每一个方法对应一种产品。

具体工厂

实现了在抽象工厂中声明的创建产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中。

抽象产品

它为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法。

具体产品

定义具体工厂生产的具体产品对象，实现抽象产品接口中声明的业务方法。

<hr>

##**使用场景**

当需要创建的对象是一系列相互关联或相互依赖的产品族时，便可以使用抽象工厂模式。
大白话意思就是一个继承体系中，如果存在着多个等级结构（即存在着多个抽象类，像上面的技能与工具书），
并且分属各个等级结构中的实现类之间存在着一定的关联或者约束，就可以使用抽象工厂模式。当然了，
同样的道理就是如果各个等级结构中的实现类之间不存在关联或约束，则使用多个独立的工厂来对产品进行创建。

<hr>

##**程序猿实例**

如下实例就是上图何如上文字解释的实现代码，具体不再解释：

{% highlight ruby %}
package yanbober.github.io;
/*技能等级结构部分*/
interface ICode {
    void coding();
}

class CodeImplAndroid implements ICode {
    @Override
    public void coding() {
        System.out.println("Coding Android!");
    }
}

class CodeImplPHP implements ICode {
    @Override
    public void coding() {
        System.out.println("Coding PHP!");
    }
}
/*工具书等级结构*/
interface INeedBook {
    void lookBook();
}

class NeedBookImplAndroid implements INeedBook {
    @Override
    public void lookBook() {
        System.out.println("Look Android Book!");
    }
}

class NeedBookImplPHP implements INeedBook {
    @Override
    public void lookBook() {
        System.out.println("Look PHP Book!");
    }
}
/*产品族*/
interface IAbstractFactory {
    ICode getCodingSkill();
    INeedBook getNeedBook();
}

class FactoryImplAndroid implements IAbstractFactory {
    @Override
    public ICode getCodingSkill() {
        return new CodeImplAndroid();
    }

    @Override
    public INeedBook getNeedBook() {
        return new NeedBookImplAndroid();
    }
}

class FactoryImplPHP implements IAbstractFactory {
    @Override
    public ICode getCodingSkill() {
        return new CodeImplPHP();
    }

    @Override
    public INeedBook getNeedBook() {
        return new NeedBookImplPHP();
    }
}

public class Main {
    public static void main(String[] args) {
        IAbstractFactory factory = new FactoryImplAndroid();
        ICode code = factory.getCodingSkill();
        INeedBook book = factory.getNeedBook();
        code.coding();
        book.lookBook();

        factory = new FactoryImplPHP();
        code = factory.getCodingSkill();
        book = factory.getNeedBook();
        code.coding();
        book.lookBook();
    }
}
{% endhighlight %}

**技巧Tips：**依旧可以使用配置与反射实现自动适应。

<hr>

##**总结一把**

抽象工厂模式的优点：

- 和前面一样，隔离具体类的生成，使客户并不需要知道什么被创建。
- 增加新的产品族很方便，无须修改已有系统，符合“开闭原则”。
 
抽象工厂模式的缺点：
       
- 增加新的产品等级结构麻烦，需要对原有系统进行较大的修改，甚至需要修改抽象层代码，违背“开闭原则”。
	
<hr>
	
##**终极总结工厂三个模式**

总结一下回过头发现工厂模式包含三部分：简单工厂模式，工厂方法模式，还是抽象工厂模式。他们的实现都是为了解耦。
所以个人觉得到底用这三个的哪个没有定数，只要达到最好的解耦就是牛逼的选择。

{% highlight ruby %}

{% endhighlight %}


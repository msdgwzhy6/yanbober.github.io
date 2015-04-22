---
layout: post
keywords: 设计模式, 简单工厂模式, Simple Factory Pattern
description: 设计模式系列学习记录
title: "设计模式(创建型)之简单工厂模式(Simple Factory Pattern)"
categories: [设计模式]
tags: [简单工厂模式, 创建型模式]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**概述**

简单工厂模式（Simple Factory Pattern）又叫静态工厂方法模式（Static FactoryMethod Pattern），
是通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。
当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无须知道其创建细节。

通常脱口而出所说的工厂模式是指工厂方法模式，它是几种工厂模式里使用频率最高的模式。
而这里先不说工厂方法模式，先来说说工厂模式系列中不属于设计模式的一种模式--简单工厂模式。
它不属于设计模式，但在软件开发中应用也较为频繁，通常将它作为学习其他工厂模式的入门。
其实工厂模式分为了最弱的简单工厂模式，工厂方法模式，牛逼的抽象工厂模式。

切记，简单工厂模式不属于设计模式，是一种编码的习惯，但他又和工厂模式有些关系，所以先说。

<hr>

##**核心**

概念：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。
因为在简单工厂模式中用于创建实例的方法是静态(static)方法，
因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。


重点：简单工厂模式的结构包含如下几个核心模块：

Factory（工厂角色）

工厂角色即工厂类，它是简单工厂模式的核心，负责实现创建所有产品实例的内部逻辑；
工厂类可以被外界直接调用，创建所需的产品对象；在工厂类中提供了静态的工厂方法，
返回类型为抽象产品类型。

Product（抽象产品角色）

工厂类所创建的所有对象的父类，封装了各种产品对象的公有方法，它的引入将提高系统的灵活性，
使得在工厂类中只需定义一个通用的工厂方法，因为所有创建的具体产品对象都是其子类对象。

ConcreteProduct（具体产品角色）

简单工厂模式的创建目标，所有被创建的对象都充当这个角色的某个具体类的实例。
每一个具体产品角色都继承了抽象产品角色，需要实现在抽象产品中声明的抽象方法。

**特例：** 有时候，为了简化简单工厂模式，我们可以将抽象产品类和工厂类合并，将静态工厂方法移至抽象产品类中。
       
在简单工厂模式中，客户端通过工厂类来创建一个产品类的实例，而无须直接使用new关键字来创建对象。

<hr>

##**使用场景**

工厂类负责创建的对象比较少。

客户端只知道传入工厂类的参数，对于如何创建对象并不关心。

<hr>

##**程序猿实例**

如下以程序猿技能为例使用简单工厂模式写代码，ICode就是抽象产品角色，CodeImplAndroid、
CodeImplIOS、CodeImplPHP等将来添加的就是具体产品角色，FactoryCreater就是工厂角色。Main方法
是客户端模拟调运。

{% highlight ruby %}
package yanbober.github.io;

interface ICode {
    void coding();
}

class CodeImplAndroid implements ICode {
    @Override
    public void coding() {
        System.out.println("Coding Android!");
    }
}

class CodeImplIOS implements ICode {
    @Override
    public void coding() {
        System.out.println("Coding IOS!");
    }
}

class CodeImplPHP implements ICode {
    @Override
    public void coding() {
        System.out.println("Coding PHP!");
    }
}

class FactoryCreater {
    public static ICode getCodingSkill(String type) {
        ICode iCode = null;
        if ("android".equalsIgnoreCase(type)) {
            iCode = new CodeImplAndroid();
        }
        else if ("ios".equalsIgnoreCase(type)) {
            iCode = new CodeImplIOS();
        }
        else if ("php".equalsIgnoreCase(type)) {
            iCode = new CodeImplPHP();
        }
        return iCode;
    }
}

public class Main {
    public static void main(String[] args) {
        ICode code = FactoryCreater.getCodingSkill("php");
        code.coding();
        code = FactoryCreater.getCodingSkill("android");
        code.coding();
        code = FactoryCreater.getCodingSkill("ios");
        code.coding();
    }
}
{% endhighlight %}

**技巧Tips：** 其实在Android开发中有时候会设计客户化等操作，可能会用到简单工厂方法，在使用FactoryCreater的type时会发现，
在代码中通过FactoryCreater.getCodingSkill("php");修改php这种type时都需要重新编译代码；每次修改FactoryCreater.getCodingSkill("php");
的类都违背了开闭原则。所以有一个技巧解决这些问题，使用配置文件，在FactoryCreater中传入配置文件的值即可解决这种开闭原则。

<hr>

**总结一把**

简单工厂模式的主要优点如下：

- 客户端可以免除直接创建对象的职责，只关心使用对象，简单工厂模式实现了对象创建和使用的分离。
- 客户端不用知道创建的产品类具体类名，只要知道具体产品类所对应的参数即可。
- 通过配置文件，提高了系统灵活性。
 
简单工厂模式的主要缺点如下：

- 工厂类负责所有对象的创建逻辑，该类出问题整个系统挂掉。
- 系统扩展困难，一旦添加新产品就不得不修改工厂逻辑。
- 简单工厂模式由于使用了静态工厂方法，所以工厂角色无法形成基于继承的等级结构。

{% highlight ruby %}

{% endhighlight %}


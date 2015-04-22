---
layout: post
keywords: 设计模式, 工厂方法模式, Factory Method Pattern
description: 设计模式系列学习记录
title: "设计模式(创建型)之工厂方法模式(Factory Method Pattern)"
categories: [设计模式]
tags: [工厂方法模式, 创建型模式]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**概述**

前面介绍了简单工厂模式，在最后代码示例部分展示的程序猿技能工厂类时提到了一个严重的问题。
当FactoryCreater中需要引入新对象时需要修改源代码，这违背了“开放封闭原则”，
使得具体产品与工厂类之间的耦合度高，严重影响了系统的灵活性和扩展性。

问题来了。。。

如何实现增加新类别对象而不影响已有代码？

那必须是得看工厂方法模式啊。在工厂方法模式中，我们不再提供一个统一的工厂类来创建所有的产品对象，
而是针对不同的产品提供不同的工厂，系统提供一个与产品等级结构对应的工厂等级结构。

<hr>

##**核心**

**概念：** 定义一个用于创建对象的接口，让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。
工厂方法模式又简称为工厂模式(Factory Pattern)，又可称作虚拟构造器模式(Virtual Constructor Pattern)或多态工厂模式(Polymorphic Factory Pattern)。
工厂方法模式是一种类创建型模式。

**重点：** 工厂方法模式结构包含如下四大核心模块：

工厂接口

工厂接口（或者抽象类）是工厂方法模式的核心，与调用者直接交互用来提供产品。

工厂实现

需要有多少种产品，就需要有多少个具体的工厂实现。

产品接口

定义产品规范，所有的产品实现都必须遵循产品接口定义的规范。
产品接口（也可以是抽象类，但最好别违背里氏原则）是调用者最为关心的，产品接口定义的优劣直接决定了调用者代码的稳定性。

产品实现

实现产品接口的具体类，决定了产品在客户端中的具体行为。
        
**简单工厂模式 & 工厂方法模式区别：**简单工厂只有三个核心要素，没有工厂接口，并且得到产品的方法一般是静态的。

**特例：**有时为进一步简化客户端代码会隐藏掉工厂方法，在工厂类中将直接调用产品类的业务方法，
客户端无须调用工厂方法创建产品，直接通过工厂即可使用所创建的对象中的业务方法。

<hr>

##**使用场景**

不管是简单工厂模式，工厂方法模式还是抽象工厂模式，他们具有类似的特性，所以他们的适用场景也是类似的：

在任何需要生成复杂对象的地方，都可以使用工厂方法模式；复杂对象适合使用工厂模式，new就可以完成创建的对象无需使用工厂模式；
如果使用工厂模式，就需要引入一个工厂类，会增加系统的复杂度；工厂模式是一种典型的解耦模式；
当需要系统有比较好的扩展性时，可以考虑工厂模式，不同的产品用不同的实现工厂来组装；

<hr>

##**程序猿实例**

如下以程序猿技能为例使用工厂方法模式写代码，ICode就是产品接口，CodeImplAndroid、
CodeImplPHP等将来添加的就是产品实现，IFactory就是工厂接口，FactoryImplAndroid、
FactoryImplPHP等将来依据产品添加的就是工厂实现，Main是客户端模拟调运。

{% highlight ruby %}
package yanbober.github.io;
//具体编程技能模块
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
//创建技能工厂模块
interface IFactory {
    ICode getCodingSkill();
}

class FactoryImplAndroid implements IFactory {
    @Override
    public ICode getCodingSkill() {
        return new CodeImplAndroid();
    }
}

class FactoryImplPHP implements IFactory {
    @Override
    public ICode getCodingSkill() {
        return new CodeImplPHP();
    }
}

public class Main {
    public static void main(String[] args) {
        IFactory factory = new FactoryImplAndroid();
        ICode code = factory.getCodingSkill();
        code.coding();

        factory = new FactoryImplPHP();
        code = factory.getCodingSkill();
        code.coding();
    }
}
{% endhighlight %}

**技巧Tips：** 其实在Android开发中有时候会设计客户化等操作，可能会用到工厂方法模式，在使用IFactory factory = new FactoryImplAndroid();
时你会发现FactoryImplAndroid这个名字可能会变FactoryImplPHP等等。有一个类似简单工厂方法中的解决办法就是使用配置文件和Java的反射机制来动态
创建factory实例。

<hr>

**总结一把**

工厂方法模式的主要优点如下：

- 在工厂方法模式中，工厂方法用来创建客户化产品，向客户隐藏产品实例化细节，用户只需要关心所需产品对应的工厂，无须关心创建细节。
- 基于工厂角色和产品角色的多态性设计是工厂方法模式的关键，它能够让工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部。
- 在系统中加入新产品时，无须修改抽象工厂和抽象产品提供的接口，无须修改客户端，也无须修改其他的具体工厂和具体产品，完全符合“开闭原则”。
 
工厂方法模式的主要缺点如下：

- 在添加新产品时，需要编写新的具体产品类，而且还要提供与之对应的具体工厂类，系统中类的个数将成对增加，
在一定程度上增加了系统的复杂度，有更多的类需要编译和运行，会给系统带来一些额外的开销。
- 由于考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度。

{% highlight ruby %}

{% endhighlight %}


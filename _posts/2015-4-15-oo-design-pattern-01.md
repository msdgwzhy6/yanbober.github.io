---
layout: post
keywords: 设计模式, 软件架构, 软件设计原则
description: 设计模式系列学习记录
title: "设计模式之面向对象与类基础特征概念"
categories: [设计模式]
tags: [设计模式]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**背景知识**

设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。
使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。 
毫无疑问，设计模式于己于他人于系统都是多赢的；设计模式使代码编制真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。

这是官方的专业解释。大白话意思就是说设计模式就经验的总结，模板的运用。

<hr>

##**面向对象三大基本特性**

**封装**

封装是把过程和数据包围起来，对数据的访问只能通过已定义的界面。

**继承**

继承是一种类的层次模型，并且允许和鼓励类的重用，它提供了一种明确表述共性的方法。

**多态**

多态性是指允许不同类的对象对同一消息作出响应。多态性包括编译时多态和运行时多态。
主要作用就是用来将接口和实现分离开，改善代码的组织结构，增强代码的可读性。
在某些很简单的情况下，或许我们不使用多态也能开发出满足我们需要的程序，但大多数情况，如果没有多态，就会觉得代码极其难以维护。

面向对象通过类和对象来实现抽象，实现时诞生了三个重要的特性，也就是由于这三个特性才衍生出了各种各样的设计模式。

<hr>

##**面向对象类关系**

通过大量代码和经验可以得知，类与类之间主要有6种关系模式，这六种模板写法导致了平时书写代码的不同耦合度。具体如下所列（耦合度依次增强排列）：

1. 依赖关系
2. 关联关系
3. 聚合关系
4. 组合关系
5. 继承关系
6. 实现关系

为了记住类与类之间的关系，我自己总结了一句话方便记忆：《衣(依赖)冠(关联)剧(聚合)组(组合)纪(继承)实(实现)》。

**依赖关系**

一般而言，依赖关系在Java语言中体现为局域变量、方法的形参，或者对静态方法的调用。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

class Code {
    public void coding(String str) {
        System.out.println("Coding with "+ str +" !");
    }
}

class ProgramMonkey {
    public void programAndroid(Code code) {
        code.coding("Java/Android");
    }

    public void programIOS(Code code) {
        code.coding("OC/IOS");
    }

    public void programPHP(Code code) {
        code.coding("PHP");
    }
}

public class Main {
    public static void main(String[] args) {
	    ProgramMonkey monkey = new ProgramMonkey();
        Code code = new Code();
        monkey.programAndroid(code);
        monkey.programIOS(code);
        monkey.programPHP(code);
    }
}
{% endhighlight %}

**关联关系**

使一个类知道另一个类的属性和方法。关联可以是双向的，也可以是单向的。在Java语言中，关联关系一般使用成员变量来实现。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

class Code {
    public void coding(String str) {
        System.out.println("Coding with "+ str +" !");
    }
}

class ProgramMonkey {
    private Code mCode;

    public ProgramMonkey(Code mCode) {
        this.mCode = mCode;
    }
    //成员变量关联
    public void programAndroid() {
        mCode.coding("Java/Android");
    }
    //形参关联
    public void programIOS(Code code) {
        code.coding("OC/IOS");
    }
}

public class Main {
    public static void main(String[] args) {
	    ProgramMonkey monkey = new ProgramMonkey(new Code());
        Code code = new Code();
        monkey.programAndroid();
        monkey.programIOS(code);
    }
}
{% endhighlight %}

使用方法参数形式可以表示依赖关系，也可以表示关联关系。其实在本质上他们是由关系的。
在本例中，使用成员变量意思是代码是我自己写的，我具备这些语法技能，能写出来代码。
使用方法参数意思是IOS代码不是我自己写的，我只是个板砖码农，别人写好是啥样我抄啥样。

**聚合关系**

聚合是关联关系的一种，是强的关联关系，聚合是整体和个体之间的关系。与关联关系一样，聚合关系也是通过实例变量实现的，
但是关联关系所涉及的两个类是处在同一层次上的，而在聚合关系中，两个类是处在不平等层次上的，一个代表整体，另一个代表部分。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

class Code {
    public void coding(String str) {
        System.out.println("Coding with "+ str +" !");
    }
}

class ProgramMonkey {
    private Code mCode;

    public Code getmCode() {
        return mCode;
    }

    public void setmCode(Code mCode) {
        this.mCode = mCode;
    }

    public ProgramMonkey() {
    }
    //也是聚合关系
    public void programAndroid() {
        mCode.coding("Java/Android");
    }
}

public class Main {
    public static void main(String[] args) {
	    ProgramMonkey monkey = new ProgramMonkey(new Code());
        monkey.programAndroid();
    }
}
{% endhighlight %}

本例的意思就是说写程序是Monkey的一项技能。聚合关系一般使用setter方法给成员变量赋值。

**组合关系**

组合是关联关系的一种，是比聚合关系强的关系。它要求普通的聚合关系中代表整体的对象负责代表部分对象的生命周期，组合关系是不能共享的。
代表整体的对象需要负责保持部分对象和存活，在一些情况下将负责代表部分的对象湮灭掉。
代表整体的对象可以将代表部分的对象传递给另一个对象，由后者负责此对象的生命周期。
换言之，代表部分的对象在每一个时刻只能与一个对象发生组合关系，由后者排他地负责生命周期。部分和整体的生命周期一样。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

class Code {
    public void coding(String str) {
        System.out.println("Coding with "+ str +" !");
    }
}

class ProgramMonkey {
    private Code mCode;

    public ProgramMonkey(Code mCode) {
        this.mCode = mCode;
    }

    public void programAndroid() {
        mCode.coding("Java/Android");
    }
}

public class Main {
    public static void main(String[] args) {
	    ProgramMonkey monkey = new ProgramMonkey(new Code());
        monkey.programAndroid();
    }
}
{% endhighlight %}

意思就是说想要成为ProgramMonkey程序猿，你必须具备Code编程能力，没有编程能力，是毛程序员啊（饿死了）；我要是转行了，谁也拿不走我的Code技能；
我的技能跟随我的生命，我挂了技能也没了，技能没了，我就挂了！所以说为了表示组合关系，常常会使用构造方法来达到初始化的目的。

**继承关系**

继承表示类与类（或者接口与接口）之间的父子关系。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

class Monkey {
    public void run() {
        System.out.println("I can run!");
    }
}

class ProgramMonkey extends Monkey{
    public void program() {
        System.out.println("I can Program!");
    }
}

public class Main {
    public static void main(String[] args) {
        ProgramMonkey monkey = new ProgramMonkey();
        monkey.run();
        monkey.program();
    }
}
{% endhighlight %}

如上的意思就是说一开始Monkey猴子只会跑，后来进化成了ProgramMonkey程序猿，具备了新的program技能。也就是说程序猿具备了两项技能。

**实现关系**

接口定义好操作的集合，由实现类去完成接口的具体操作。

如下就是一个简单的例子：

{% highlight ruby %}
package yanbober.github.io;

interface CodeInterface {
    void code();
}

class ProgramMonkey implements CodeInterface {
    public void run() {
        System.out.println("I can run!");
    }

    @Override
    public void code() {
        System.out.println("I can coding!");
    }
}

public class Main {
    public static void main(String[] args) {
        ProgramMonkey monkey = new ProgramMonkey();
        monkey.run();
        monkey.code();
    }
}
{% endhighlight %}

意思就是说，有一个Code的技能，谁想学习拥有这个技能谁就实现它，我实现了他，我也就具备技能了！

<hr>

关联、聚合、组合只能配合语义，结合上下文才能够判断出来，而只给出一段代码让我们判断是关联，聚合，还是组合关系，则是无法判断的。

##**面向对象七大基本原则**

在运用面向对象的思想进行软件设计时，需要遵循的原则一共有7个，他们是：

1. 单一职责原则（Single Responsibility Principle）
2. 里氏替换原则（Liskov Substitution Principle）
3. 依赖倒置原则（Dependence Inversion Principle）
4. 接口隔离原则（Interface Segregation Principle）
5. 迪米特法则（Law Of Demeter）
6. 开闭原则（Open Close Principle）
7. 组合/聚合复用原则（Composite/Aggregate Reuse Principle CARP）

在软件设计的过程中，只要我们尽量遵循以上七条设计原则，设计出来的软件一定会是一个优秀的软件，
它必定足够健壮、足够稳定，并以极大的灵活性来迎接随时而来的需求变更等因素。

关于这七大基本原则是一个很深的话题，第二篇开始将分析学习七项基本原则。
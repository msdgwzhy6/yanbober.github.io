---
layout: post
keywords: ���ģʽ, ����ܹ�, ������ԭ��
description: ���ģʽϵ��ѧϰ��¼
title: "���ģʽ֮����"
categories: [���ģʽ]
tags: [���ģʽ]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**����֪ʶ**

���ģʽ��Design pattern����һ�ױ�����ʹ�á�������֪���ġ����������Ŀ�ġ�������ƾ�����ܽᡣ
ʹ�����ģʽ��Ϊ�˿����ô��롢�ô�������ױ�������⡢��֤����ɿ��ԡ� 
�������ʣ����ģʽ�ڼ���������ϵͳ���Ƕ�Ӯ�ģ����ģʽʹ��������������̻������ģʽ��������̵Ļ�ʯ���磬��ͬ���õĽṹһ����

���ǹٷ���רҵ���͡���׻���˼����˵���ģʽ�;�����ܽᣬģ������á�

<hr>

##**������������������**

**��װ**

��װ�ǰѹ��̺����ݰ�Χ�����������ݵķ���ֻ��ͨ���Ѷ���Ľ��档

**�̳�**

�̳���һ����Ĳ��ģ�ͣ���������͹���������ã����ṩ��һ����ȷ�������Եķ�����

**��̬**

��̬����ָ����ͬ��Ķ����ͬһ��Ϣ������Ӧ����̬�԰�������ʱ��̬������ʱ��̬��
��Ҫ���þ����������ӿں�ʵ�ַ��뿪�����ƴ������֯�ṹ����ǿ����Ŀɶ��ԡ�
��ĳЩ�ܼ򵥵�����£��������ǲ�ʹ�ö�̬Ҳ�ܿ���������������Ҫ�ĳ��򣬵��������������û�ж�̬���ͻ���ô��뼫������ά����

�������ͨ����Ͷ�����ʵ�ֳ���ʵ��ʱ������������Ҫ�����ԣ�Ҳ�����������������Բ��������˸��ָ��������ģʽ��

<hr>

##**����������ϵ**

ͨ����������;�����Ե�֪��������֮����Ҫ��6�ֹ�ϵģʽ��������ģ��д��������ƽʱ��д����Ĳ�ͬ��϶ȡ������������У���϶�������ǿ���У���

1. ������ϵ
2. ������ϵ
3. �ۺϹ�ϵ
4. ��Ϲ�ϵ
5. �̳й�ϵ
6. ʵ�ֹ�ϵ

Ϊ�˼�ס������֮��Ĺ�ϵ�����Լ��ܽ���һ�仰������䣺����(����)��(����)��(�ۺ�)��(���)��(�̳�)ʵ(ʵ��)����

**������ϵ**

һ����ԣ�������ϵ��Java����������Ϊ����������������βΣ����߶Ծ�̬�����ĵ��á�

���¾���һ���򵥵����ӣ�

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

**������ϵ**

ʹһ����֪����һ��������Ժͷ���������������˫��ģ�Ҳ�����ǵ���ġ���Java�����У�������ϵһ��ʹ�ó�Ա������ʵ�֡�

���¾���һ���򵥵����ӣ�

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
    //��Ա��������
    public void programAndroid() {
        mCode.coding("Java/Android");
    }
    //�βι���
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

ʹ�÷���������ʽ���Ա�ʾ������ϵ��Ҳ���Ա�ʾ������ϵ����ʵ�ڱ������������ɹ�ϵ�ġ�
�ڱ����У�ʹ�ó�Ա������˼�Ǵ��������Լ�д�ģ��Ҿ߱���Щ�﷨���ܣ���д�������롣
ʹ�÷���������˼��IOS���벻�����Լ�д�ģ���ֻ�Ǹ���ש��ũ������д����ɶ���ҳ�ɶ����

**�ۺϹ�ϵ**

�ۺ��ǹ�����ϵ��һ�֣���ǿ�Ĺ�����ϵ���ۺ�������͸���֮��Ĺ�ϵ���������ϵһ�����ۺϹ�ϵҲ��ͨ��ʵ������ʵ�ֵģ�
���ǹ�����ϵ���漰���������Ǵ���ͬһ����ϵģ����ھۺϹ�ϵ�У��������Ǵ��ڲ�ƽ�Ȳ���ϵģ�һ���������壬��һ�������֡�

���¾���һ���򵥵����ӣ�

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
    //Ҳ�ǾۺϹ�ϵ
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

��������˼����˵д������Monkey��һ��ܡ��ۺϹ�ϵһ��ʹ��setter��������Ա������ֵ��

**��Ϲ�ϵ**

����ǹ�����ϵ��һ�֣��ǱȾۺϹ�ϵǿ�Ĺ�ϵ����Ҫ����ͨ�ľۺϹ�ϵ�д�������Ķ���������ֶ�����������ڣ���Ϲ�ϵ�ǲ��ܹ���ġ�
��������Ķ�����Ҫ���𱣳ֲ��ֶ���ʹ���һЩ����½���������ֵĶ����������
��������Ķ�����Խ������ֵĶ��󴫵ݸ���һ�������ɺ��߸���˶�����������ڡ�
����֮�������ֵĶ�����ÿһ��ʱ��ֻ����һ����������Ϲ�ϵ���ɺ��������ظ����������ڡ����ֺ��������������һ����

���¾���һ���򵥵����ӣ�

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

��˼����˵��Ҫ��ΪProgramMonkey����Գ�������߱�Code���������û�б����������ë����Ա���������ˣ�����Ҫ��ת���ˣ�˭Ҳ�ò����ҵ�Code���ܣ�
�ҵļ��ܸ����ҵ��������ҹ��˼���Ҳû�ˣ�����û�ˣ��Ҿ͹��ˣ�����˵Ϊ�˱�ʾ��Ϲ�ϵ��������ʹ�ù��췽�����ﵽ��ʼ����Ŀ�ġ�

**�̳й�ϵ**

�̳б�ʾ�����ࣨ���߽ӿ���ӿڣ�֮��ĸ��ӹ�ϵ��

���¾���һ���򵥵����ӣ�

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

���ϵ���˼����˵һ��ʼMonkey����ֻ���ܣ�������������ProgramMonkey����Գ���߱����µ�program���ܡ�Ҳ����˵����Գ�߱�������ܡ�

**ʵ�ֹ�ϵ**

�ӿڶ���ò����ļ��ϣ���ʵ����ȥ��ɽӿڵľ��������

���¾���һ���򵥵����ӣ�

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

��˼����˵����һ��Code�ļ��ܣ�˭��ѧϰӵ���������˭��ʵ��������ʵ����������Ҳ�;߱������ˣ�

<hr>

�������ۺϡ����ֻ��������壬��������Ĳ��ܹ��жϳ�������ֻ����һ�δ����������ж��ǹ������ۺϣ�������Ϲ�ϵ�������޷��жϵġ�

##**��������������ԭ��**

��������������˼�����������ʱ����Ҫ��ѭ��ԭ��һ����6���������ǣ�

1. ��һְ��ԭ��Single Responsibility Principle��
2. �����滻ԭ��Liskov Substitution Principle��
3. ��������ԭ��Dependence Inversion Principle��
4. �ӿڸ���ԭ��Interface Segregation Principle��
5. �����ط���Law Of Demeter��
6. ����ԭ��Open Close Principle��

�������ƵĹ����У�ֻҪ���Ǿ�����ѭ�����������ԭ����Ƴ��������һ������һ������������
���ض��㹻��׳���㹻�ȶ������Լ�����������ӭ����ʱ�����������������ء�

�������������ԭ����һ������Ļ��⣬�ڶ�ƪ��ʼ������ѧϰ�������ԭ��

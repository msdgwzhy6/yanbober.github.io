---
layout: post
keywords: Android�������ʵ��
description: Android�������ʵ��
title: "(ת)Android�������ʵ��"
categories: [��Ŀ����]
tags: [Android��Ŀ����]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##������
	
	����û�¹�github����һƪAndroid�������ҹ����¡�ͬʱ��Ϊ�Լ���¼������ĥ����������ԭ���ߵ��Ͷ��ɹ���[���ҽ���GitHubԭ��](https://github.com/futurice/android-best-practices/blob/master/translations/Chinese/README.cn.md)

<hr>

# Android �������ʵ��
 
��[Futurice](http://www.futurice.com)��˾Android��������ѧ���ľ��顣
��ѭ����׼�򣬱����ظ��������ӡ������Կ���iOS��Windows Phone ����Ȥ��
�뿴[**iOS Good Practices**](https://github.com/futurice/ios-good-practices) �� [**Windows client Good Practices**](https://github.com/futurice/win-client-dev-good-practices) ����ƪ���¡�

## ժҪ

* ʹ�� Gradle �����Ƽ��Ĺ��̽ṹ
* ��������������ݷ���gradle.properties
* ��Ҫ�Լ�д HTTP �ͻ���,ʹ��Volley��OkHttp��
* ʹ��Jackson�����JSON����
* ����ʹ��Guavaͬʱʹ��һЩ���������*65k method limit*��һ��Android�����������ִ��65536��������
* ʹ�� Fragments������UI��ͼ
* ʹ�� Activities ֻ��Ϊ�˹��� Fragments
* Layout ������ XMLs���룬��֯������
* ��layoutout XMLs����ʱ��ʹ��styles�ļ�������ʹ���ظ�������
* ʹ�ö��style�ļ������ⵥһ��һ����style�ļ�
* �������colors.xml ���DRY(��Ҫ�ظ��Լ�)��ֻ�Ƕ����ɫ��
* ����ʹ��dimens.xml DRY(��Ҫ�ظ��Լ�)������ͨ�ó���
* ��Ҫ��һ�����ε�ViewGroup
* ��ʹ��WebViewsʱ�����ڿͻ��������������ڴ�й¶
* ʹ��Robolectric��Ԫ���ԣ�Robotium ��UI����
* ʹ��Genymotion ��Ϊ���ģ����
* ����ʹ��ProGuard �� DexGuard��������Ŀ

### Android SDK

�����[Android SDK](https://developer.android.com/sdk/installing/index.html?pkg=tools)�������homeĿ¼������Ӧ�ó����޹ص�λ�á�
����װ��Щ����SDK��IDE��ʱ�򣬿��ܻὫSDK����IDEͬһĿ¼�£�������Ҫ�����������°�װ��IDE�������IDEʱ����ǳ��鷳��
���⣬�������IDE������ͨ�û���������root�����У���Ҫ�����SDK�ŵ�һ����ҪsudoȨ�޵�ϵͳ����Ŀ¼�¡�

### ����ϵͳ

���Ĭ�ϱ��뻷��Ӧ����[Gradle](http://tools.android.com/tech-docs/new-build-system).
Ant �кܶ����ƣ�Ҳ�����ࡣʹ��Gradle��������¹����ܷ��㣺

- ����APP��ͬ�汾�ı���
- ���������ƽű�������
- �������������
- �Զ�����Կ
- ����

ͬʱ��Android Gradle�����Ϊ�±�׼�Ĺ���ϵͳ���ڱ�Google�����Ŀ�����

### ���̽ṹ

���������еĽṹ���ϵ�Ant & Eclipse ADT ���̽ṹ�����µ�Gradle & Android Studio ���̽ṹ��
��Ӧ��ѡ���µĹ��̽ṹ�������Ĺ��̻���ʹ���ϵĽṹ�����Ƿ����ɣ���������ֲ���µĽṹ��


�ϵĽṹ:

```
old-structure
���� assets
���� libs
���� res
���� src
��  ���� com/futurice/project
���� AndroidManifest.xml
���� build.gradle
���� project.properties
���� proguard-rules.pro
```

�µĽṹ

```
new-structure
���� library-foobar
���� app
��  ���� libs
��  ���� src
��  ��  ���� androidTest
��  ��  ��  ���� java
��  ��  ��     ���� com/futurice/project
��  ��  ���� main
��  ��     ���� java
��  ��     ��  ���� com/futurice/project
��  ��     ���� res
��  ��     ���� AndroidManifest.xml
��  ���� build.gradle
��  ���� proguard-rules.pro
���� build.gradle
���� settings.gradle
```

��Ҫ���������ڣ��µĽṹ��ȷ�ķֿ���'source sets' (`main`,`androidTest`)��Gradle��һ�����
��������������磬���Դ�顮paid���͡�free����src�У��⽫��Ϊ����Ӧ�ó���ĸ��Ѻ���ѵ�����ģʽ��Դ���롣

�����Ŀ���õ�������Ŀ��ʱ�����磬library-foobar����ӵ��һ����������`app`�ӵ���������Ŀ�������Ӧ�ó����Ƿǳ����õġ�
Ȼ��`settings.gradle`����������Щ����Ŀ������`app/build.gradle`�������á�

### Gradle ����

**���ýṹ** �ο�[Google's guide on Gradle for Android](http://tools.android.com/tech-docs/new-build-system/user-guide)


**С����** ����(shell, Python, Perl, etc)��Щ�ű����ԣ���Ҳ����ʹ��Gradle ��������
������Ϣ��ο�[Gradle's documentation](http://www.gradle.org/docs/current/userguide/userguide_single.html#N10CBF)��


**����** �����汾releaseʱ��app�� `build.gradle`����Ҫ���� `signingConfigs`.��ʱ��Ӧ�ñ����������ݣ�


_��Ҫ�����_ . �������ڰ汾�����С�

```groovy
signingConfigs {
	release {
		storeFile file("myapp.keystore")
		storePassword "password123"
		keyAlias "thekey"
		keyPassword "password789"
	}
}
```
	
���ǣ�����һ��������汾����ϵͳ��`gradle.properties`�ļ���

```
KEYSTORE_PASSWORD=password123
KEY_PASSWORD=password789
```


�Ǹ��ļ���gradle�Զ�����ģ��������`buld.gradle`�ļ���ʹ�ã����磺

```groovy
signingConfigs {
	release {
		try {
			storeFile file("myapp.keystore")
			storePassword KEYSTORE_PASSWORD
			keyAlias "thekey"
			keyPassword KEY_PASSWORD
		}
		catch (ex) {
			throw new InvalidUserDataException("You should define KEYSTORE_PASSWORD and KEY_PASSWORD in gradle.properties.")
		}
	}
}
```
	

**ʹ�� Maven ������������ʹ�õ���jar������** ����������Ŀ������ȷʹ����
jar�ļ�����ô���ǿ��ܳ�Ϊ���õİ汾����`2.1.1`.����jar�����������Ǻܷ����ģ�
�������Maven�ܺõĽ���ˣ�����Android Gradle������Ҳ���Ƽ��ķ��������
��ָ���汾��һ����Χ����`2.1.+`,Ȼ��Maven���Զ��������ƶ������°汾�����磺

```groovy
dependencies {
	compile 'com.netflix.rxjava:rxjava-core:0.19.+'
	compile 'com.netflix.rxjava:rxjava-android:0.19.+'
	compile 'com.fasterxml.jackson.core:jackson-databind:2.4.+'
	compile 'com.fasterxml.jackson.core:jackson-core:2.4.+'
	compile 'com.fasterxml.jackson.core:jackson-annotations:2.4.+'
	compile 'com.squareup.okhttp:okhttp:2.0.+'
	compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.+'
}
```

### IDEs and text editors

### IDE���ɿ����������ı��༭��


**����ʹ��ʲô�༭����һ��Ҫ����һ�����õĹ��̽ṹ** �༭��ÿ���˶����Լ���
ѡ������ı༭�����ݹ��̽ṹ�͹���ϵͳ�������������Լ������Ρ�

��������[Android Studio](https://developer.android.com/sdk/installing/studio.html),��Ϊ�����ɹȸ迪������ӽ�Gradle��Ĭ��ʹ�����µĹ��̽ṹ���Ѿ���beta�׶�
��Ŀǰ�Ѿ���release 1.0�ˣ���������ΪAndroid�������Ƶġ�

��Ҳ����ʹ��[Eclipse ADT](https://developer.android.com/sdk/installing/index.html?pkg=adt) ����������Ҫ�����������ã���Ϊ��ʹ���˾ɵĹ��̽ṹ
��Ant��Ϊ����ϵͳ������������ʹ�ô��İ�༭����Vim��Sublime Text������Emacs����������Ļ�������Ҫʹ��Gardle��`adb`�����С����ʹ��Eclipse����Gradle
���ʺ��㣬��ֻ��ʹ�������й������̣���Ǩ�Ƶ�Android Studio�����ɡ�


������ʹ�ú��ֿ������ߣ�ֻҪȷ��Gradle���µ���Ŀ�ṹ���ֹٷ��ķ�ʽ����Ӧ�ó��򣬱�����ı༭�������ļ����뵽�汾���ơ����磬�������Ant `build.xml`�ļ���
�ر������ı�Ant�����ã���Ҫ���Ǳ���`build.gradle`�����º������õġ�ͬʱ���ƴ����������ߣ���Ҫǿ�Ƹı����ǵĿ������ߺ�ƫ�á�

### ���


**[Jackson](http://wiki.fasterxml.com/JacksonHome)** ��һ����java����ת����JSON��JSONת��java�����⡣[Gson](https://code.google.com/p/google-gson/)
�ǽ�������������з�����Ȼ�����Ƿ���Jackson����Ч,��Ϊ��֧������ķ�������JSON:�����ڴ���ģ��,�ʹ�ͳJSON-POJO���ݰ󶨡����������ס��
Jsonkson�����GSON�������Ը���������ѡ�������ѡ��GSON������APP 65k���������ơ�����ѡ��: [Json-smart](https://code.google.com/p/json-smart/) and [Boon JSON](https://github.com/RichardHightower/boon/wiki/Boon-JSON-in-five-minutes)


**�������󣬻��棬ͼƬ** ִ�������˷��������м��ֽ����Ľ����������Ӧ�ÿ���ʵ�����Լ�������ͻ��ˡ�ʹ�� [Volley](https://android.googlesource.com/platform/frameworks/volley)
��[Retrofit](http://square.github.io/retrofit/)��Volley ͬʱ�ṩͼƬ�����ࡣ������ѡ��ʹ��Retrofit,��ô����ʹ��[Picasso](http://square.github.io/picasso/)
������ͼƬ�ͻ��棬ͬʱʹ��[OkHttp](http://square.github.io/okhttp/)��Ϊ��Ч����������Retrofit��Picasso��OkHttp������ͬһ�ҹ�˾������ע��
����[Square](https://github.com/square) ��˾�����������������ܺܺõ���һ�����С�[OkHttp ͬ�����Ժ�Volley��һ��ʹ�� Volley](http://stackoverflow.com/questions/24375043/how-to-implement-android-volley-with-okhttp-2-0/24951835#24951835).

**RxJava** �Ǻ���ʽ��Ӧ�Ե�һ����⣬���仰˵���ܴ����첽���¼���
����һ��ǿ��ĺ���ǰ;��ģʽ��ͬʱҲ���ܻ���ɻ�������Ϊ������˵Ĳ�ͬ��
���ǽ�����ʹ�������ܹ�����Ӧ�ó���֮ǰҪ�������ǡ�
��һЩ��Ŀ��ʹ��RxJava��ɵģ��������Ҫ�������Ը���Щ��ȡ����ϵ��
Timo Tuominen, Olli Salonen, Andre Medeiros, Mark Voit, Antti Lammi, Vera Izrailit, Juha Ristolainen.
����Ҳд��һЩ���ͣ�
[[1]](http://blog.futurice.com/tech-pick-of-the-week-rx-for-net-and-rxjava-for-android), [[2]](http://blog.futurice.com/top-7-tips-for-rxjava-on-android),
[[3]](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754),
[[4]](http://blog.futurice.com/android-development-has-its-own-swift).


������֮ǰ��ʹ�ù�Rx�ľ�������ʼ��API��ӦӦ������
���⣬�Ӽ򵥵�UI�¼�����ʼ���ã��絥���¼����������������¼���
�������Rx���������ģ�ͬʱ��Ҫ����Ӧ�õ��������ܹ��У���ô���ڸ��ӵĲ���д��Javadocs�ĵ���
���ס��������ϤRxJava�Ŀ�����Ա�����ܻ�ǳ������������Ŀ��
����ĵ�ȫ���������������Ĵ����Rx��

**[Retrolambda](https://github.com/evant/gradle-retrolambda)** ��һ����Android��ԤJDK8ƽ̨�ϵ�ʹ��Lambda���ʽ�﷨��Java��⡣
�������ڱ��������Ľ����ԺͿɶ��ԣ��ر���ʹ����RxJava���������ʱ��
ʹ����ʱ�Ȱ�װJDK8����Android Studio���̽ṹ�Ի����а������ó�ΪSDK·����ͬʱ����`JAVA8_HOME`��`JAVA7_HOME`����������
Ȼ���ڹ��̸�Ŀ¼������ build.gradle��

```groovy
dependencies {
	classpath 'me.tatarka:gradle-retrolambda:2.4.+'
}
```

ͬʱ��ÿ��module ��build.gradle�����

```groovy
apply plugin: 'retrolambda'

android {
	compileOptions {
	sourceCompatibility JavaVersion.VERSION_1_8
	targetCompatibility JavaVersion.VERSION_1_8
}

retrolambda {
	jdk System.getenv("JAVA8_HOME")
	oldJdk System.getenv("JAVA7_HOME")
	javaVersion JavaVersion.VERSION_1_7
}
```

Android Studio �ṩJava8 lambdas����Ǵ�����ʾ֧�֡�������lambdas����Ϥ��ֻ��������¿�ʼѧϰ�ɣ�

- �κ�ֻ����һ���ӿڵķ�������"lambda friendly"ͬʱ������Ա��۵��ɸ����յ��﷨
- ����Բ��������������ʣ���дһ����ͨ�������ڲ��࣬Ȼ����Android StatusΪ������һ��lambda��

**����dex���������ƣ�ͬʱ����ʹ�ù�������** Android apps���������һ��dex�ļ�ʱ����һ��65535��Ӧ�÷���ǿӲ����[[1]](https://medium.com/@rotxed/dex-skys-the-limit-no-65k-methods-is-28e6cb40cf71) [[2]](http://blog.persistent.info/2014/05/per-package-method-counts-for-androids.html) [[3]](http://jakewharton.com/play-services-is-a-monolith/)��
����ͻ��65k����֮����ῴ��һ������������ˣ�ʹ��һ��������Χ������ļ���ͬʱʹ��[dex-method-counts](https://github.com/mihaip/dex-method-counts)
������������Щ��������65k����֮��ʹ�ã��ر�ı���ʹ��Guava��⣬��Ϊ����������13k��������

### Activities and Fragments

[Fragments](http://developer.android.com/guide/components/fragments.html)Ӧ����Ϊ��ʵ��UI����Ĭ��ѡ��������ظ�ʹ��Fragments�û��ӿ���
��ϳ����Ӧ�á�����ǿ���Ƽ�ʹ��Fragments������activity������UI���棬�������£�

-  **�ṩ�ര�񲼾ֽ������** Fragments ��������Ҫ���ֻ�Ӧ�����쵽ƽ����ԣ�������ƽ��������������A��B�������񣬵������ֻ�Ӧ����A��B���ֱܷ����
  ������Ļ��������Ӧ���������ʹ����fragments����ô�Ժ����Ӧ�����䵽������ͬ�ߴ���Ļ�ͻ�ǳ��򵥡�

- **��Ļ������ͨ��** ��һ��Activity���͸�������(����Java����)������һ��Activity��Android��API��û���ṩ���ʵķ���������ʹ��Fragment�������ʹ��
һ��activityʵ����Ϊ���activity��fragments��ͨ��ͨ������ʹ������Activity��Activity���ͨ�źã���Ҳ�뿼��ʹ��Event Bus�ܹ���ʹ����
[Otto](https://square.github.io/otto/) ���� [greenrobot EventBus](https://github.com/greenrobot/EventBus)��Ϊ������ʵ�֡�
�����ϣ�������������һ����⣬RxJavaͬ������ʵ��һ��Event Bus��


- **Fragments һ��ͨ�õĲ�ֻ��UI** �������һ��û�н����fragment��ΪActivity�ṩ��̨������
��һ�������ʹ���������������һ��[fragment �����ı�����fragment���߼�](http://stackoverflow.com/questions/12363790/how-many-activities-vs-fragments/12528434#12528434)
�����ǰ�����߼�����activity�С�

- **����ActionBar ������ʹ���ڲ�fragment������** �����ѡ��ʹ��һ��û��UI�����fragment��ר�Ź���ActionBar,���������ѡ��ʹ����ÿ��Fragment��
������Լ���action ����Ϊ��Activity��ActionBar.[�ο�](http://www.grokkingandroid.com/adding-action-items-from-within-fragments/).

�ܲ��ң����ǲ�����㷺��ʹ��Ƕ�׵�[fragments](https://developer.android.com/about/versions/android-4.2.html#NestedFragments)����Ϊ
��ʱ������[matryoshka bugs](http://delyan.me/android-s-matryoshka-problem/)������ֻ�е���������(���磬��ˮƽ������ViewPager��
����Ļһ��fragment��)��������ȷ��һ�����ǵ�ѡ���ʱ��Ź㷺��ʹ��fragment��

��һ���ܹ��������APPӦ����һ��������activity���������󲿷�ҵ����ص�fragment����Ҳ���ܻ���һЩ������activity ����Щ������activity����activity
ͨ�źܼ������������ַ���
[`Intent.setData()`](http://developer.android.com/reference/android/content/Intent.html#setData(android.net.Uri)) �� [`Intent.setAction()`](http://developer.android.com/reference/android/content/Intent.html#setAction(java.lang.String))�����Ƶķ�����


### Java ���ṹ

Android Ӧ�ó����ڼܹ��ϴ�����Java�е�[Model-View-Controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)�ṹ��
��Android �� Fragment��Activityͨ�����ǿ�������(http://www.informit.com/articles/article.aspx?p=2126865).
���仰˵���������û��ӿڵĲ��֣�ͬ��Ҳ��Views��ͼ�Ĳ��֡�


������Ϊ��ˣ��ź����ϸ�Ľ�fragments (���� activities) �ϸ�Ļ��ֳ� ������controlloers������ͼ views��
��ǽ����Ƿ����Լ������� `fragments` ���С�ֻҪ����ѭ֮ǰ�ᵽ�Ľ��飬Activities ����Է��ڶ���Ŀ¼�¡�
������滮��2��3�����ϵ�activity����ô����ͬ���½�һ��`activities`���ɡ�

Ȼ�������ּܹ����Կ�������һ����ʽ��MVC��
����Ҫ������API��Ӧ��JSON���ݣ�������POJO��`models`���С�
��һ��`views`������������Զ�����ͼ��֪ͨ��������ͼ��widgets�ȵȡ�
������Adapter�������ݺ���ͼ֮�䡣Ȼ������ͨ����Ҫͨ��`getView()`����������һЩ��ͼ��
��������Խ�`adapters`������`views`�����档

һЩ��������ɫ������Ӧ�ó��򼶱�ģ�ͬʱ�ǽӽ�ϵͳ�ġ�
��Щ�����`managers`�����档
һЩ���ӵ����ݴ����࣬����˵"DateUtils",����`utils`�����档
���˽����������紦���࣬����`network`�����档


�ܶ���֮������ӽ��û���������ӽ����ȥ�������ǡ�

```
com.futurice.project
���� network
���� models
���� managers
���� utils
���� fragments
���� views
   ���� adapters
   ���� actionbar
   ���� widgets
   ���� notifications
```


### ��Դ�ļ� Resources


- **����** ��ѭǰ׺�������͵�ϰ�ߣ�����`type_foo_bar.xml`�����磺`fragment_contact_details.xml`,`view_primary_button.xml`,`activity_main.xml`.

**��֯�����ļ�** �����㲻ȷ������Ű�һ�������ļ�����ѭһ�¹�����ܻ��а�����

- ÿһ������һ�У�����4���ո�
- `android:id` ������Ϊ��һ������
- `android:layout_****` �������ϱ�
- `style` �����ڵײ�
- �رձ�ǩ`/>`������һ�У������ڵ���������µ�����
- ����ʹ��[Designtime attributes ���ʱ��������](http://tools.android.com/tips/layout-designtime-attributes)��Android Studio�Ѿ��ṩ֧�֣�������Ӳ����`android:text`
(����ע��ǽ��Ҳ���Բο�stormzhang����ƪ����[����](http://stormzhang.com/devtools/2015/01/11/android-studio-tips1/))��

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	>

	<TextView
		android:id="@+id/name"
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:layout_alignParentRight="true"
		android:text="@string/name"
		style="@style/FancyText"
		/>

	<include layout="@layout/reusable_part" />

</LinearLayout>
```

��Ϊһ�����鷨��,`android:layout_****`����Ӧ���� layout XML �ж���,ͬʱ��������`android:****` Ӧ���� styler XML�С��˹���Ҳ�����⣬�������幤��
�ĺܺá����˼�������Ǳ���layout����(positioning, margin, sizing) ��content�����ڲ����ļ��У�ͬʱ�����е����ϸ�����ԣ�colors, padding, font����
��style�ļ��С�


������������Щ:

- `android:id` ����Ӧ����layout�ļ���
- layout�ļ���`android:orientation`����һ��`LinearLayout`����ͨ����������
- `android:text` �����Ƕ������ݣ�Ӧ�÷���layout�ļ���
- ��ʱ��`android:layout_width` �� `android:layout_height`���Էŵ�һ��style����Ϊһ��ͨ�õķ���и������壬����Ĭ���������ЩӦ�÷ŵ�layout�ļ��С�

**ʹ��styles** ����ÿ����Ŀ����Ҫ�ʵ���ʹ��style�ļ�����Ϊ����һ����ͼ��˵��һ���ظ�������Ǻܳ����ġ�
��Ӧ���ж��ڴ�����ı����ݣ���������Ӧ����һ��ͨ�õ�style�ļ������磺

```xml
<style name="ContentText">
	<item name="android:textSize">@dimen/font_normal</item>
	<item name="android:textColor">@color/basic_black</item>
</style>
```

Ӧ�õ�TextView ��:

```xml
<TextView
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="@string/price"
	style="@style/ContentText"
	/>
```


�������ҪΪ��ť�ؼ���ͬ�������飬��Ҫֹͣ�������һ����صĺ��ظ�`android:****`�����Էŵ�һ��ͨ�õ�style�С�


**��һ�����style�ļ��ָ�ɶ���ļ�** ������ж��`styles.xml` �ļ���Android SDK֧�������ļ���`styles`����ļ����Ʋ�û�����ã������õ������ļ�
��xml��`<style>`��ǩ�����������ж��style�ļ�`styles.xml`,`style_home.xml`,`style_item_details.xml`,`styles_forms.xml`��
��������Դ�ļ�·����ҪΪϵͳ������������壬��`res/values`Ŀ¼�µ��ļ���������������



**`colors.xml`��һ����ɫ��** �����`colors.xml`�ļ���Ӧ��ֻ��ӳ����ɫ������һ��RGBAֵ����û�������ġ���Ҫʹ����Ϊ��ͬ�İ�ť������RGBAֵ��

*��Ҫ������*

```xml
<resources>
	<color name="button_foreground">#FFFFFF</color>
	<color name="button_background">#2A91BD</color>
	<color name="comment_background_inactive">#5F5F5F</color>
	<color name="comment_background_active">#939393</color>
	<color name="comment_foreground">#FFFFFF</color>
	<color name="comment_foreground_important">#FF9D2F</color>
	...
	<color name="comment_shadow">#323232</color>
```


ʹ�����ָ�ʽ�����ǳ����׵Ŀ�ʼ�ظ�����RGBAֵ����ʹ�����Ҫ�ı����ɫ��ĺܸ��ӡ�ͬʱ����Щ�����Ǹ�һЩ�������������ģ���`button`����`comment`,
Ӧ�÷ŵ�һ����ť����У���������`color.xml`�ļ��С�


�෴��������:

```xml
<resources>

	<!-- grayscale -->
	<color name="white"     >#FFFFFF</color>
	<color name="gray_light">#DBDBDB</color>
	<color name="gray"      >#939393</color>
	<color name="gray_dark" >#5F5F5F</color>
	<color name="black"     >#323232</color>

	<!-- basic colors -->
	<color name="green">#27D34D</color>
	<color name="blue">#2A91BD</color>
	<color name="orange">#FF9D2F</color>
	<color name="red">#FF432F</color>

</resources>
```

��Ӧ�����������Ҫ�����ɫ�壬���Ʋ���Ҫ��"green", "blue", �ȵ���ͬ��
"brand_primary", "brand_secondary", "brand_negative" ����������Ҳ����ȫ���Խ��ܵġ�
�������淶����ɫ�������޸Ļ��ع�����ʹӦ��һ��ʹ���˶����ֲ�ͬ����ɫ��÷ǳ�������
ͨ��һ������������ֵ��UI��˵������ʹ����ɫ�������Ƿǳ���Ҫ�ġ�


**��Դ�colors.xmlһ���Դ�dimens.xml�ļ�** �붨����ɫ��ɫ��һ������ͬʱҲӦ�ö���һ����϶����������С�ġ���ɫ�塱��
һ���õ����ӣ�������ʾ��

```xml
<resources>

	<!-- font sizes -->
	<dimen name="font_larger">22sp</dimen>
	<dimen name="font_large">18sp</dimen>
	<dimen name="font_normal">15sp</dimen>
	<dimen name="font_small">12sp</dimen>

	<!-- typical spacing between two views -->
	<dimen name="spacing_huge">40dp</dimen>
	<dimen name="spacing_large">24dp</dimen>
	<dimen name="spacing_normal">14dp</dimen>
	<dimen name="spacing_small">10dp</dimen>
	<dimen name="spacing_tiny">4dp</dimen>

	<!-- typical sizes of views -->
	<dimen name="button_height_tall">60dp</dimen>
	<dimen name="button_height_normal">40dp</dimen>
	<dimen name="button_height_short">32dp</dimen>

</resources>
```
	
����ʱ��д margins �� paddings ʱ����Ӧ��ʹ��`spacing_****`�ߴ��ʽ�����֣���������Դ�String�ַ���һ��ֱ��дֵ��
����д��ǳ��ио�����ʹ��֯�͸ı���򲼾��Ƿǳ����ס�

**�������ε���ͼ�ṹ** ��ʱ��Ϊ�˰ڷ�һ����ͼ������ܳ��������һ��LinearLayout�������ʹ�����ַ��������

```xml
<LinearLayout
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	>

	<RelativeLayout
		...
		>

		<LinearLayout
			...
			>

			<LinearLayout
				...
				>

				<LinearLayout
					...
					>
				</LinearLayout>

			</LinearLayout>

		</LinearLayout>

	</RelativeLayout>

</LinearLayout>
```


��ʹ��û�зǳ���ȷ����һ��layout�����ļ�������ʹ�ã��������Java�ļ��д�һ��view inflate�����inflate���벻��ȥ����������У� ������views���У�Ҳ�ǿ��ܻᷢ���ġ�


���ܻᵼ��һϵ�е����⡣����ܻ������������⣬��Ϊ��������Ҫ����һ�����ӵ�UI���ṹ��
�����ܻᵼ�����¸����ص�����[StackOverflowError](http://stackoverflow.com/questions/2762924/java-lang-stackoverflow-error-suspected-too-many-views).


��˾������������ͼtree��ѧϰ���ʹ��[RelativeLayout](https://developer.android.com/guide/topics/ui/layout/relative.html),
��� [optimize ��Ĳ���](http://developer.android.com/training/improving-layouts/optimizing-layout.html) �����ʹ��
[`<merge>` ��ǩ](http://stackoverflow.com/questions/8834898/what-is-the-purpose-of-androids-merge-tag-in-xml-layouts).


**С�Ĺ���WebViews������.** ����������ʾһ��web��ͼ��
����˵����һ���������£��������ͻ��˴���HTML�Ĺ�����
����ú�˹���ʦЭ����Ȼ������һ�� "*��*" HTML��
[WebViews Ҳ�ܵ����ڴ�й¶](http://stackoverflow.com/questions/3130654/memory-leak-in-webview)
�����������ǵ�Activity�������Ǳ��󶨵�ApplicationContext�е�ʱ��
��ʹ�ü򵥵����ֻ�ťʱ������ʹ��WebView����ʱʹ��TextView��Buttons���á�

### ���Կ��


Android SDK�Ĳ��Կ�ܻ����ڳ����׶Σ��ر��ǹ���UI���Է��档Android Gradle 
Ŀǰʵ����һ����[`connectedAndroidTest`](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Testing)�Ĳ��ԣ�
��[ʹ��һ��JUnit ΪAndroid�ṩ����չ��� extension of JUnit with helpers for Android](http://developer.android.com/reference/android/test/package-summary.html).�����������ɵ�JUnit���ԣ�


**ֻ������Ԫ����ʱʹ�� [Robolectric](http://robolectric.org/) ��views ����**
����һ�������ṩ"�������豸��"Ϊ�˼��ٿ����Ĳ��ԣ�
�ǳ�ʱ���� models �� view models �ĵ�Ԫ���ԡ�
Ȼ����ʹ��Robolectric����ʱ����ȷ�ģ�Ҳ����ȫ��UI���ԡ�
������йض�����UIԪ�ء��Ի���ȣ�����ʱ�������⣬
����Ҫ����Ϊ������ ���ںڰ��й���������û�пɿصĽ�������²��ԣ�


**[Robotium](https://code.google.com/p/robotium/) ʹдUI���Էǳ��򵥡�
** ����UI�����㲻�� Robotium �����豸���ӵĲ��ԡ�
�������ܻ�������棬����Ϊ���������������Ļ�úͷ�����ͼ��������Ļ��
���������������������򵥣�

```java
solo.sendKey(Solo.MENU);
solo.clickOnText("More"); // searches for the first occurence of "More" and clicks on it
solo.clickOnText("Preferences");
solo.clickOnText("Edit File Extensions");
Assert.assertTrue(solo.searchText("rtf"));
```


### ģ����

�����ȫְ����Android App,��ô��һ��[Genymotion emulator](http://www.genymotion.com/)license�ɡ�
Genymotion ģ�������и������֡���ٶȣ�������͵�AVDģ������������ʾ��APP�Ĺ��ߣ���������ģ���������ӣ�GPSλ�ã��ȵȡ���ͬʱ������������Ӳ��ԡ�
�����漰����ʹ�úܶ಻ͬ���豸����һ��Genymotion ��Ȩ�Ǳ�����ܶ����豸���˶�ġ�

ע�⣺Genymotionģ����û��װ�����е�Google������Google Play Store��Maps����Ҳ������
Ҫ����Samsungָ����API���������Ļ��㻹����Ҫ����һ����ʵ��Samsung�豸��


### ��������

[ProGuard](http://proguard.sourceforge.net/) ��һ����Android��Ŀ�й㷺ʹ�õ�ѹ���ͻ��������Դ��Ĺ��ߡ�

���Ƿ�ʹ��ProGuardȡ������Ŀ�����ã����㹹��һ��release�汾��apkʱ��ͨ����Ӧ������gradle�ļ���

```groovy
buildTypes {
	debug {
		minifyEnabled false
	}
	release {
		signingConfig signingConfigs.release
		minifyEnabled true
		proguardFiles 'proguard-rules.pro'
	}
}
```

Ϊ�˾�����Щ����Ӧ�ñ���������Щ����Ӧ�ñ��������㲻�ò�ָ��һ������ʵ��������Ĵ����С�
��Щʵ��Ӧ����ָ���������main������applets��midlets��activities���ȵȡ�
Android framework ʹ��һ��Ĭ�ϵ������ļ���������`SDK_HOME/tools/proguard/proguard-android.txt`
Ŀ¼���ҵ����Զ���Ĺ���ָ���� project-specific ������������`my-project/app/proguard-rules.pro`�ж��壬
�ᱻ��ӵ�Ĭ�ϵ������С�


���� ProGuard һ���ձ�����⣬�ǿ�Ӧ�ó����Ƿ��������`ClassNotFoundException` ���� `NoSuchFieldException` �����Ƶ��쳣��
��ʹ������û�о��沢���гɹ���
����ζ���������ֿ��ܣ�

1. ProGuard �Ѿ��Ƴ����࣬ö�٣���������Ա������ע�⣬�����Ƿ��Ǳ�Ҫ�ġ�
2. ProGuard �������࣬ö�٣���Ա���������ƣ�������Щ�����ֱ���ԭʼ����ʹ���ˣ�����ͨ��Java�ķ��䡣

���`app/build/outputs/proguard/release/usage.txt`�ļ���������Ķ����Ƿ��Ƴ��ˡ�
��� `app/build/outputs/proguard/release/mapping.txt` �ļ���������Ķ����Ƿ񱻻����ˡ�

In order to prevent ProGuard from *stripping away* needed classes or class members, add a `keep` options to your proguard config:
�Է� ProGuard *����* ��Ҫ��������Ա�����һ�� `keep`ѡ������� proguard �����ļ��У�
```
-keep class com.futurice.project.MyClass { *; }
```

��ֹ ProGuard *����* һЩ��ͳ�Ա����� `keepnames`:
```
-keepnames class com.futurice.project.MyClass { *; }
```

�鿴[this template's ProGuard config](https://github.com/futurice/android-best-practices/blob/master/templates/rx-architecture/app/proguard-rules.pro) �е�һЩ���ӡ�
����������ο�[Proguard](http://proguard.sourceforge.net/#manual/examples.html)��

**�ڹ�����Ŀ֮��������һ���汾** �����ProGuard�����Ƿ���ȷ�ı�������Ҫ�Ĳ��֡�
ͬʱ���ۺ�ʱ��������µ���⣬��һ�������汾��ͬʱapk���豸������������һ�¡�
��Ҫ�ȵ����appҪ���� "1.0"�汾�˲����汾��������ʱ������ܻ������ö����벻�����쳣����ҪһЩʱ��ȥ�޸����ǡ�

**Tips**ÿ�η����°汾��Ҫд `mapping.txt`��ÿ����һ���汾������û�����һ��bug��ͬʱ�ύ��һ���������Ķ�ջ���١�
ͨ������`mapping.txt`�ļ�����ȷ������Ե��Ե����⡣

**DexGuard** ��������Ҫ���Ĺ������Ż�����ר�Ż����ķ������룬����ʹ��[DexGuard](http://www.saikoa.com/dexguard),
һ����ҵ�����ProGuard Ҳ���������Ŷӿ����ġ�
��������׽�Dex�ļ��ָ�����65K�������������⡣


### ��л

��лAntti Lammi, Joni Karppinen, Peter Tackage, Timo Tuominen, Vera Izrailit, Vihtori M?ntyl?, Mark Voit, Andre Medeiros, Paul Houghton ��Щ�˺�Futurice �����߷������ǵ�Android�������顣

### License

[Futurice Oy](www.futurice.com)
Creative Commons Attribution 4.0 International (CC BY 4.0)

### Translation

Translated to Chinese by [andyiac](https://github.com/andyiac)

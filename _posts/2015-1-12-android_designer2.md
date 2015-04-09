---
layout: post
keywords: REST SOAP
description: Web 服务编程，REST 与 SOAP
title: "服务端REST与SOAP的探讨"
categories: [开发设计杂谈]
tags: [REST,SOAP]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**声明：**
	
闲来逛论坛看到一篇不错的文章，阅读后受益匪浅。

本文从一个简单的应用场景出发，使用REST和SOAP两种不同的架构风格实现，通过对REST与SOAP Web服务具体对比,旨在帮助读者更深刻理解REST架构风格。

<hr>

##**REST简介**
 
在开始我们的正式讨论之前，让我们简单看一下REST的定义。

REST（Representational State Transfer）是Roy Fielding提出的一个描述互联系统架构风格的名词。
为什么称为REST？Web本质上由各种各样的资源组成，资源由URI唯一标识。
浏览器（或者任何其它类似于浏览器的应用程序）将展示出该资源的一种表现方式，或者一种表现状态。
如果用户在该页面中定向到指向其它资源的链接，则将访问该资源，并表现出它的状态。
这意味着客户端应用程序随着每个资源表现状态的不同而发生状态转移，也即所谓REST。
关于REST本身，本文就不再这里过多地讨论，读者可以参考developerWorks上其它介绍REST的文章。
本文的重点在于通过REST与SOAP Web服务的对比，帮助读者更深刻理解REST架构风格的特点，优势。

<hr>

##**应用场景介绍（在线用户管理）**

本文将借助于一个应用场景，通过基于REST和SOAP Web服务的不同实现，来对两者进行对比。
该应用场景的业务逻辑会尽量保持简单且易于理解，以有助于把我们的重心放在REST和SOAP Web服务技术特质对比上。

####**需求描述**

这是一个在线的用户管理模块，负责用户信息的创建，修改，删除，查询。用户的信息主要包括：

- 用户名（唯一标志在系统中的用户）
- 头衔
- 公司
- EMAIL
- 描述

需求用例图如下图一：

<img src="http://yanbober.github.io/image/designer/1.png" />

如图所示，客户端1（Client1）与客户端2（Client2）对于信息的存取具有不同的权限，
客户端1可以执行所有的操作，而客户端2只被允许执行用户查询（Query User）与用户列表查询（Query User List）。
关于这一点，我们在对REST Web服务与SOAP Web服务安全控制对比时会具体谈到。
下面我们将分别向您介绍如何使用REST和SOAP架构实现Web服务。

<hr>

##**使用REST实现Web服务**

本部分将基于Restlet框架来实现该应用。Restlet为那些要采用REST结构体系来构建应用程序的Java开发者提供了一个具体的解决方案。
关于更多的Restlet相关内容，本文不做深入讨论，请见参考资源列表。

####**设计**

我们将采用遵循REST设计原则的ROA（Resource-Oriented Architecture，面向资源的体系架构）进行设计。
ROA是什么？简单点说，ROA是一种把实际问题转换成REST式Web服务的方法，它使得URI、HTTP和XML具有跟其他Web应用一样的工作方式。
在使用ROA进行设计时，我们需要把真实的应用需求转化成ROA中的资源，基本上遵循以下的步骤：

- 分析应用需求中的数据集。
- 映射数据集到ROA中的资源。
- 对于每一资源，命名它的URI。
- 为每一资源设计其Representations。
- 用hypermedia links表述资源间的联系。

接下来我们按照以上的步骤来设计本文的应用案例。

在线用户管理所涉及的数据集就是用户信息，如果映射到ROA资源，主要包括两类资源：用户及用户列表。
用户资源的URI用 http://localhost:8182/v1/users/{username} 表示，用户列表资源的URI用 http://localhost:8182/v1/users表示。
它们的Representation如下，它们都采用了如清单1和清单2所示的XML表述方式。

清单1. 用户列表资源Representation

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<users>
    <user>
        <name>tester</name>
        <link>http://localhost:8182/v1/users/tester</link>
    </user>
    <user>
        <name>tester1</name>
        <link>http://localhost:8182/v1/users/tester1</link>
    </user>
</users>
{% endhighlight %}

清单2. 用户资源 Representation

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<user>
    <name>tester</name>
    <title>software engineer</title>
    <company>IBM</company>
    <email>tester@cn.ibm.com</email>
    <description>testing!</description>
</user>
{% endhighlight %}

客户端通过User List Resource提供的LINK信息(如:<link>http://localhost:8182/v1/users/tester</link>)获得具体的某个USER Resource。

####**Restful Web服务架构**

首先给出Web服务使用REST风格实现的整体架构图，如下图所示：

<img src="http://yanbober.github.io/image/designer/2.png" />

接下来，我们将基于该架构，使用Restlet给出应用的RESTful Web服务实现。
下面的章节中，我们将给出REST Web服务实现的核心代码片段。
关于完整的代码清单，读者可以通过资源列表下载。

####**客户端实现**

清单给出的是客户端的核心实现部分，其主要由四部分组成：使用HTTP PUT增加、修改用户资源，
使用HTTP GET得到某一具体用户资源，使用HTTP DELETE删除用户资源，使用HTTP GET得到用户列表资源。
而这四部分也正对应了上面架构图中关于架构描述的四对HTTP消息来回。
关于UserRestHelper类的完整实现，请读者参见本文所附的代码示例。

{% highlight ruby %}
public class UserRestHelper {
    //The root URI of our ROA implementation.
    public static final String APPLICATION_URI = "http://localhost:8182/v1";

    //Get the URI of user resource by user name.
    private static String getUserUri(String name) {
        return APPLICATION_URI + "/users/" + name;
    }

    //Get the URI of user list resource.
    private static String getUsersUri() {
        return APPLICATION_URI + "/users";
    }
    //Delete user resource from server by user name.
    //使用 HTTP DELETE 方法经由 URI 删除用户资源
    public static void deleteFromServer(String name) {
        Response response = new Client(Protocol.HTTP).delete(getUserUri(name));
        ……
    }
    //Put user resource to server.
    //使用 HTTP PUT 方法经由 URI 增加或者修改用户资源
    public static void putToServer(User user) {
        //Fill FORM using user data.
        Form form = new Form();
        form.add("user[title]", user.getTitle());
        form.add("user[company]", user.getCompany());
        form.add("user[email]", user.getEmail());
        form.add("user[description]", user.getDescription());
        Response putResponse = new Client(Protocol.HTTP).put(
                getUserUri(user.getName()), form.getWebRepresentation());
        ……
    }
    //Output user resource to console.
    public static void printUser(String name) {
        printUserByURI(getUserUri(name));
    }

    //Output user list resource to console.
    //使用 HTTP GET 方法经由 URI 显示用户列表资源
    public static void printUserList() {
        Response getResponse = new Client(Protocol.HTTP).get(getUsersUri());
        if (getResponse.getStatus().isSuccess()) {
            DomRepresentation result = getResponse.getEntityAsDom();
            //The following code line will explore this XML document and output
            //each user resource to console.
            ……
        } else {
            System.out.println("Unexpected status:"+ getResponse.getStatus());
        }
    }

    //Output user resource to console.
    //使用 HTTP GET 方法经由 URI 显示用户资源
    private static void printUserByURI(String uri) {
        Response getResponse = new Client(Protocol.HTTP).get(uri);
        if (getResponse.getStatus().isSuccess()) {
            DomRepresentation result = getResponse.getEntityAsDom();
            //The following code line will explore this XML document and output
            //current user resource to console.
            ……
        } else {
            System.out.println("unexpected status:"+ getResponse.getStatus());
        }
    }
}
{% endhighlight %}

####**服务器端实现**

清单给出的是服务器端对于用户资源类（UserResourc）的实现，其核心的功能是响应有关用户资源的HTTP GET/PUT/DELETE请求，
而这些请求响应逻辑正对应了UserRestHelper类中关于用户资源类的HTTP请求。

{% highlight ruby %}
public class UserResource extends Resource {
    private User _user;
    private String _userName;

    public UserResource(Context context, Request request, Response response) {
        //Constructor is here.
        ……
    }

    //响应 HTTP DELETE 请求逻辑
    public void delete() {
        // Remove the user from container.
        getContainer().remove(_userName);
        getResponse().setStatus(Status.SUCCESS_OK);
    }

    //This method will be called by handleGet.
    public Representation getRepresentation(Variant variant) {
        Representation result = null;
        if (variant.getMediaType().equals(MediaType.TEXT_XML)) {
            Document doc = createDocument(this._user);
            result = new DomRepresentation(MediaType.TEXT_XML, doc);
        }
        return result;
    }

    //响应 HTTP PUT 请求逻辑。
    public void put(Representation entity) {
        if (getUser() == null) {
            //The user doesn't exist, create it
            setUser(new User());
            getUser().setName(this._userName);
            getResponse().setStatus(Status.SUCCESS_CREATED);
        } else {
            getResponse().setStatus(Status.SUCCESS_NO_CONTENT);
        }
        //Parse the entity as a Web form.
        Form form = new Form(entity);
        getUser().setTitle(form.getFirstValue("user[title]"));
        getUser().setCompany(form.getFirstValue("user[company]"));
        getUser().setEmail(form.getFirstValue("user[email]"));
        getUser().setDescription(form.getFirstValue("user[description]"));
        //Put the user to the container.
        getApplication().getContainer().put(_userName, getUser());
    }

    //响应 HTTP GET 请求逻辑。
    public void handleGet() {
        super.handleGet();
        if(this._user != null ) {
            getResponse().setEntity(getRepresentation(
                    new Variant(MediaType.TEXT_XML)));
            getResponse().setStatus(Status.SUCCESS_OK);
        } else {
            getResponse().setStatus(Status.CLIENT_ERROR_NOT_FOUND);
        }
    }

    //build XML document for user resource.
    private Document createDocument(User user) {
        //The following code line will create XML document according to user info.
        ……
    }

    //The remaining methods here
    ……
}
{% endhighlight %}

UserResource类是对用户资源类的抽象，包括了对该资源的创建修改（put方法），读取（handleGet方法 ）和删除（delete方法），
被创建出来的UserResource类实例被Restlet框架所托管，所有操纵资源的方法会在相应的HTTP请求到达后被自动回调。

另外，在服务端，还需要实现代表用户列表资源的资源类UserListResource，它的实现与UserResource类似，响应HTTP GET请求，
读取当前系统内的所有用户信息，形成如上用户列表资源Representation所示，然后返回该结果给客户端。
具体的实现请读者参见本文所附的代码示例。

<hr>

##**使用SOAP实现Web服务**

本文对于SOAP实现，就不再像REST那样，具体到代码级别的实现。
本节将主要通过URI,HTTP和XML来宏观上表述SOAP Web服务实现的技术本质，
为下一节REST Web服务与SOAP Web服务的对比做铺垫。

####**SOAP Web服务架构**

同样，首先给出SOAP实现的整体架构图，如下图所示：

<img src="http://yanbober.github.io/image/designer/3.png" />

可以看到，与REST架构相比，SOAP架构图明显不同的是：所有的SOAP消息发送都使用HTTP POST方法，
并且所有SOAP消息的URI都是一样的，这是基于SOAP的Web服务的基本实践特征。

####**获得用户信息列表**

基于SOAP的客户端创建如清单所示的SOAP XML文档，它通过类RPC方式来获得用户列表信息。

getUserList SOAP消息

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <p:getUserList xmlns:p="http://www.exmaple.com"/>
    </soap:Body>
</soap:Envelope>
{% endhighlight %}

客户端将使用HTTP的POST方法，将上述的SOAP消息发送至 http://localhost:8182/v1/soap/servlet/messagerouter URI，
SOAP SERVER收到该HTTP POST请求，通过解码SOAP消息确定需要调用getUserList方法完成该WEB服务调用，返回如下的响应：

getUserListResponse消息

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <p:get
            UserListResponse xmlns:p="http://www.exmaple.com">
        <Users>
            <username>tester<username>
            <username>tester1<username>
            ......
        </Users>
        <p: getUserListResponse >
    </soap:Body>
</soap:Envelope>
{% endhighlight %}

####**获得某一具体用户信息**

getUserByName SOAP消息

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <p:getUserByName xmlns:p="http://www.exmaple.com">
            <username>tester</username>
        </p:getUserByName >
    </soap:Body>
</soap:Envelope>
{% endhighlight %}

同样地，客户端将使用HTTP的POST方法，将上述的SOAP消息发送至http://localhost:8182/v1/soap/servlet/messagerouter URI，
SOAP SERVER处理后返回的Response如下：

getUserByNameResponse SOAP消息

{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <p:getUserByNameResponse xmlns:p="http://www.exmaple.com">
            <name>tester</name>
            <title>software engineer</title>
            <company>IBM</company>
            <email>tester@cn.ibm.com</email>
            <description>testing!</description>
        </p:getUserByNameResponse>
    </soap:Body>
</soap:Envelope>
{% endhighlight %}

实际上，创建新的用户，过程也比较类似，在这里，就不一一列出，因为这两个例子对于本文在选定的点上对比REST与SOAP已经足够了。

<hr>

##**REST与SOAP比较**

本节从以下几个方面来对比上面两节给出REST实现与SOAP实现。

####**接口抽象**

RESTful Web服务使用标准的HTTP方法(GET/PUT/POST/DELETE)来抽象所有Web系统的服务能力，
而不同的是，SOAP应用都通过定义自己个性化的接口方法来抽象Web服务，这更像我们经常谈到的RPC。
例如本例中的getUserList与getUserByName方法。

RESTful Web服务使用标准的HTTP方法优势，从大的方面来讲：标准化的HTTP操作方法，
结合其他的标准化技术，如URI，HTML，XML等，将会极大提高系统与系统之间整合的互操作能力。
尤其在Web应用领域，RESTful Web服务所表达的这种抽象能力更加贴近Web本身的工作方式，也更加自然。
同时，使用标准HTTP方法实现的RRESTful Web服务也带来了HTTP方法本身的一些优势：

**无状态性（Stateless）**

HTTP协议从本质上说是一种无状态的协议，客户端发出的HTTP请求之间可以相互隔离，
不存在相互的状态依赖。基于HTTP的ROA，以非常自然的方式来实现无状态服务请求处理逻辑。
对于分布式的应用而言，任意给定的两个服务请求Request 1与Request 2, 
由于它们之间并没有相互之间的状态依赖，就不需要对它们进行相互协作处理，
其结果是：Request 1与Request 2可以在任何的服务器上执行，
这样的应用很容易在服务器端支持负载平衡(load-balance)。

**安全操作与幂指相等特性（Safety /Idempotence）**

HTTP的GET、HEAD请求本质上应该是安全的调用，即：GET、HEAD 调用不会有任何的副作用，不会造成服务器端状态的改变。
对于服务器来说，客户端对某一URI做n次的GET、HAED调用，其状态与没有做调用是一样的，不会发生任何的改变。

HTTP的PUT、DELTE调用，具有幂指相等特性 , 即：客户端对某一URI做n次的PUT、DELTE调用，其效果与做一次的调用是一样的。
HTTP的GET、HEAD方法也具有幂指相等特性。

HTTP这些标准方法在原则上保证你的分布式系统具有这些特性，以帮助构建更加健壮的分布式系统。

####**安全控制**

为了说明问题，基于上面的在线用户管理系统，我们给定以下场景：

参考一开始我们给出的用例图，对于客户端Client2，我们只希望它能以只读的方式访问User和User List资源，而Client1具有访问所有资源的所有权限。

如何做这样的安全控制？

通行的做法是：所有从客户端Client2发出的HTTP请求都经过代理服务器(Proxy Server)。
代理服务器制定安全策略：所有经过该代理的访问User和User List资源的请求只具有读取权限，
即：允许 GET/HEAD 操作，而像具有写权限的PUT/DELTE是不被允许的。

如果对于REST，我们看看这样的安全策略是如何部署的。如下图所示：

REST与代理服务器 (Proxy Servers)

<img src="http://yanbober.github.io/image/designer/4.png" />

一般代理服务器的实现根据(URI, HTTP Method)两元组来决定HTTP请求的安全合法性。
当发现类似于（http://localhost:8182/v1/users/{username}，DELETE）这样的请求时，予以拒绝。
对于SOAP，如果我们想借助于既有的代理服务器进行安全控制，会比较尴尬，如下图：

SOAP与代理服务器 (Proxy Servers)

<img src="http://yanbober.github.io/image/designer/5.png" />

所有的SOAP消息经过代理服务器，只能看到（http://localhost:8182/v1/soap/servlet/messagerouter, HTTP POST）这样的信息，
如果代理服务器想知道当前的HTTP请求具体做的是什么，必须对SOAP的消息体解码，这样的话，
意味着要求第三方的代理服务器需要理解当前的SOAP消息语义，而这种SOAP应用与代理服务器之间的紧耦合关系是不合理的。

####**关于缓存**

众所周知，对于基于网络的分布式应用，网络传输是一个影响应用性能的重要因素。
如何使用缓存来节省网络传输带来的开销，这是每一个构建分布式网络应用的开发人员必须考虑的问题。

HTTP协议带条件的HTTP GET请求 (Conditional GET) 被设计用来节省客户端与服务器之间网络传输带来的开销，
这也给客户端实现Cache机制 ( 包括在客户端与服务器之间的任何代理 ) 提供了可能。
HTTP 协议通过 HTTP HEADER 域：If-Modified-Since/Last- Modified，If-None-Match/ETag 实现带条件的GET请求。

REST的应用可以充分地挖掘HTTP协议对缓存支持的能力。当客户端第一次发送HTTP GET请求给服务器获得内容后，
该内容可能被缓存服务器(Cache Server) 缓存。当下一次客户端请求同样的资源时，
缓存可以直接给出响应，而不需要请求远程的服务器获得。而这一切对客户端来说都是透明的。

REST与缓存服务器(Cache Server)

<img src="http://yanbober.github.io/image/designer/6.png" />

而对于SOAP，情况又是怎样的呢？

使用HTTP协议的SOAP，由于其设计原则上并不像REST那样强调与Web的工作方式相一致，
所以，基于SOAP应用很难充分发挥HTTP本身的缓存能力。

SOAP与缓存服务器 (Cache Server)

<img src="http://yanbober.github.io/image/designer/7.png" />

两个因素决定了基于SOAP应用的缓存机制要远比REST复杂：

其一、所有经过缓存服务器的SOAP消息总是HTTP POST，缓存服务器如果不解码SOAP消息体，
没法知道该HTTP请求是否是想从服务器获得数据。

其二、SOAP消息所使用的URI总是指向SOAP的服务器，如本文例子中的 http://localhost:8182/v1/soap/servlet/messagerouter，
这并没有表达真实的资源URI，其结果是缓存服务器根本不知道那个资源正在被请求，更不用谈进行缓存处理。

####**关于连接性**

在一个纯的SOAP应用中，URI本质上除了用来指示SOAP服务器外，本身没有任何意义。
与REST的不同的是，无法通过URI驱动SOAP方法调用。例如在我们的例子中，当我们通过
getUserList SOAP消息获得所有的用户列表后，仍然无法通过既有的信息得到某个具体的用户信息。
唯一的方法只有通过WSDL的指示，通过调用getUserByName获得，getUserList与getUserByName是彼此孤立的。
而对于REST，情况是完全不同的：通过http://localhost:8182/v1/users URI获得用户列表，
然后再通过用户列表中所提供的LINK属性，例如<link>http://localhost:8182/v1/users/tester</link>获得tester用户的用户信息。
这样的工作方式，非常类似于你在浏览器的某个页面上点击某个hyperlink, 浏览器帮你自动定向到你想访问的页面，并不依赖任何第三方的信息。

<hr>

##**总结**

典型的基于SOAP的Web服务以操作为中心，每个操作接受XML文档作为输入，提供XML文档作为输出。
在本质上讲，它们是RPC风格的。而在遵循REST原则的ROA应用中，服务是以资源为中心的，对每个资源的操作都是标准化的HTTP方法。

本文主要集中在以上的几个方面，对SOAP与REST进行了对比，可以看到，基于REST构建的系统其系统的扩展能力要强于SOAP，
这可以体现在它的统一接口抽象、代理服务器支持、缓存服务器支持等诸多方面。
并且，伴随着Web Site as Web Services演进的趋势，基于REST设计和实现的简单性和强扩展性，
有理由相信，REST将会成为 Web 服务的一个重要架构实践领域。

####**终极总结**
 
SOAP(Simple Object Access Protocol)是一个严格定义的信息交换协议，
用于在Web Service中把远程调用和返回封装成机器可读的格式化数据。
事实上SOAP数据使用XML数据格式，定义了一整套复杂的标签，
以描述调用的远程过程、参数、返回值和出错信息等等。
而且随着需要的增长，又不得增加协议以支持安全性，这使SOAP变得异常庞大，背离了简单的初衷。
另一方面，各个服务器都可以基于这个协议推出自己的API，即使它们提供的服务及其相似，定义的API也不尽相同，
这又导致了WSDL的诞生。WSDL (Web Service Description Language) 也遵循XML格式，用来描述哪个服务器提供什么服务，
怎样找到它，以及该服务使用怎样的接口规范，简言之，服务发现。
现在，使用Web Service的过程变成，获得该服务的WSDL描述，根据WSDL构造一条格式化的SOAP请求发送给服务器，
然后接收一条同样SOAP格式的应答，最后根据先前的WSDL解码数据。绝大多数情况下，请求和应答使用HTTP协议传输，
那么发送请求就使用HTTP的POST方法。
 
REST(REpresentational State Transfort)形式上应该表述为客户端通过申请资源来实现状态的转换，
在这个角度系统可以看成一台虚拟的状态机。抛开R. T. Fielding博士论文里晦涩的理论不说，
REST应该满足这样的特点：
 
1. 客户端和服务器结构；
2. 连接协议具有无状态性；
3. 能够利用Cache机制增进性能；
4. 层次化的系统；
5. 按需代码；
 
说到底，REST只是一种架构风格，而不是协议或标准。
但这种新的风格（也许已经历史悠久？）对现有的以SOAP为代表的Web Service造成的冲击也是革命性的，
因为它面向资源，甚至连服务也抽象成资源，因为它和HTTP紧密结合，因为它服务器无状态。
 
因为SOAP并不假定传输数据的下层协议，因此必须设计为能在各种协议上运行。即使绝大多数SOAP是运行在HTTP上，
使用URI标识服务，SOAP也仅仅使用POST方法发送请求，用一个唯一的URI标识服务的入口。
 
REST简单而直观，把HTTP协议利用到了极限，在这种思想指导下，
它甚至用HTTP请求的头信息来指明资源的表示形式（如果一个资源有多种形式的话，例如人类友善的页面还是机器可读的数据？），
用HTTP的错误机制来返回访问资源的错误。由此带来的直接好处是构建的成本减少了，例如用URI定位每一个资源可以利用通用成熟的技术，
而不用再在服务器端开发一套资源访问机制。又如只需简单配置服务器就能规定资源的访问权限，例如通过禁止非GET访问把资源设成只读。
 
服务器无状态带来了更多额外好处，因为每次请求都包含响应需要的所有信息，
所有状态信息都存储在客户端，服务器的内存从庞大的状态信息中解放出来。
而且现在即使一台服务器突然死机对客户的影响也微乎其微，因为另一台服务器可以马上代替它的位置，
而不需要考虑恢复状态信息。更多的缓存也变成可能，而之前由于服务器有状态，对同一个URI的请求可能导致完全不同的响应。
 
总体结果是，网络的容错性和延展性都增强了，这些本来是WEB设计的初衷，
日趋复杂和定制的WEB把它们破坏了，现在REST又返璞归真，试图把Web Service带回简单的原则中来。
 
但是REST就是万能的吗？无状态带来了巨大的优势，同时也带来了难以解决的问题，
例如，怎样授权特定用户才能使用的服务？怎样验证用户身份？如果坚持服务器无状态，
也就是不记录用户登录状态，势必要求每一次服务请求都包含完整的用户身份和验证信息。
在这种情况下，怎样避免冒认？怎样避免用户信息泄漏？事实上，构建REST附属的安全机制已经在讨论中，
其结果无非导致另一个SOAP：复杂的需求摧残了易用性。
 
REST的支持者声称REST的请求和应答数据简单可读，而SOAP则需要一系列繁琐的封装；
即使如此，SOAP仍然不能达到接口的一致性，不同的厂商有各自的接口，
而REST只使用HTTP定义的方法，因此是通用的。事实确实如此吗？
试想用REST实现两数求和的服务，如果按照建议的做法，把服务（此处是加法）作为一个资源，
参数（此处是两个加数）作为请求的参数，结果以XML或JSON语法返回，是否比SOAP更简单易用？
通用接口仍然没法达到，因为资源的名称、参数的名称、结果的格式仍然是服务提供者定义的。
 
为了解决这个问题，提出了WASL(Web Application Description Language)来描述REST接口。
WADL就像是WSDL的REST版，随着REST被应用到复杂的领域，SOAP的影子无处不在。
 
面向资源和面向事务（非常明显的说明了Rest的试用范围，请求地图数据就可以认为主要是请求一种特殊的资源）
REST在面向资源的应用中左右逢源，但在面向事务的应用中却未如人意。面向资源的应用操作简单，
无非创建、读取、改变、删除几项，但是面向事务的应用不允许用户直接操作资源，用户只需向系统提交一个事务说明要求，
然后等待事务的完成，就如一个网上银行的用户不直接修改账户和存款，而是提交一个事务告诉银行自己要转账。
如果把这样的服务看成一种资源，通过向资源发送POST请求完成事务，那不过是SOAP的翻版而已，
无论是这样，还是通过PUT来创建事务，都改变了系统的状态（资源本身未改变，此处是改变了用户的余额），
显然违背了REST直观的初衷。
 
事实上，一些Web Service提供者提供的REST API只有REST的外壳，传输的请求和应答全然是简化了的SOAP，
这种新瓶装旧酒的做法只是加深了标准的分歧而已。归根结底REST无法简单地解决一些应用，
因此我们只能看到SOAP在REST外壳下的借尸还魂。没有一项技术能一劳永逸地解决所有问题，
只需要在预定的约束下优美地解决所在领域的问题就足够了。一项新技术推出的时候总是引来无数的跟风和吹捧，
只有当尘埃落定之后才能得到中肯的评价。

<hr>

##**PS：**

[点我进入原文](https://www.ibm.com/developerworks/cn/webservices/0907_rest_soap/)


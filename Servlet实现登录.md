# Servlet 实现登录

- 项目要求

  1.系统中的信息分为公用和私有，共有资源不用验证用户信息，私有资源只有验证的用户可以查看
  2.需要访问私有资源，跳转到登陆页面
  3.未登录只能查看公有资源
  4.登录之后可以查看公有资源和私有资源
  5.是否登录有不同的主页（前台输入url是一样的）

  使用技术：servlet和jsp，不使用任何框架
  具体一点
  `public` 为共用资源
  `private` 为私有资源

  输入url（/index）
  如果是未登录，返回`public-index.jsp`
  如果已登陆，返回`private-index.jsp`

  登录操作（/login）,成功后返回`private-index.jsp`

  任何一个`url`访问，都要判断该资源的类型是否可以访问（即权限控制，当前的规则是`public`都可以访问，`private`需要登录后才可以访问）

  - 知识点：

  类设计
  数据存储设计（不要局限于数据库，有很多更简单的方式） 
  弄清楚认证和授权概念
  知道过滤器的使用，能完成哪些常用操作
  Servlet中如何实现MVC

- **关于servlet ** 

  - 概述

    Servlet（Server Applet），全称Java Servlet，未有中文译文。是用Java编写的服务器端程序。其主要功能在于交互式地浏览和修改数据，生成动态Web内容。狭义的Servlet是指Java语言实现的一个接口，广义的Servlet是指任何实现了这个Servlet接口的类，一般情况下，人们将Servlet理解为后者。
    Servlet运行于支持Java的应用服务器中。从原理上讲，Servlet可以响应任何类型的请求，但绝大多数情况下Servlet只用来扩展基于HTTP协议的Web服务器。
    最早支持Servlet标准的是JavaSoft的Java Web Server。此后，一些其它的基于Java的Web服务器开始支持标准的Servlet。

  - 实现过程

    最早支持 Servlet 技术的是 JavaSoft 的 Java Web Server。此后，一些其它的基于 Java 的 Web Server 开始支持标准的 Servlet API。Servlet 的主要功能在于交互式地浏览和修改数据，生成动态 Web 内容。这个过程为：
       1.客户端发送请求至服务器端；

    2. 服务器将请求信息发送至 Servlet；
    3. Servlet 生成响应内容并将其传给服务器。响应内容动态生成，通常取决于客户端的请求；

       4.服务器将响应返回给客户端。
    Servlet 看起来像是通常的 Java 程序。Servlet 导入特定的属于 Java Servlet API 的包。因为是对象字节码，可动态地从网络加载，可以说 Servlet 对 Server 就如同 Applet对 Client 一样，但是，由于 Servlet 运行于 Server 中，它们并不需要一个图形用户界面。从这个角度讲，Servlet 也被称为 FacelessObject。
    一个 Servlet 就是 Java 编程语言中的一个类，它被用来扩展服务器的性能，服务器上驻留着可以通过“请求-响应”编程模型来访问的应用程序。虽然 Servlet 可以对任何类型的请求产生响应，但通常只用来扩展 Web 服务器的应用程序。
    目前最新版本为 3.1。

    运行结果如下：

    张三对象被创建了，有18岁，是男的
    李四对象被创建了，有19岁，是女的  

- **系统架构**

  - 登录用例图

  ![HashMap数据结构](http://img.blog.csdn.net/20160523233107444)

  - 页面流程图

    ![HashMap数据结构](http://img.blog.csdn.net/20160523233144023)

  - 系统架构图

    ![HashMap数据结构](http://img.blog.csdn.net/20160523233254665)

  - 数据库设计

    ````sql
    CREATE TABLE "db_test"."user" (
    "c_id" char(32) COLLATE "default" NOT NULL,
    "c_username" varchar(50) COLLATE "default",
    "c_password" varchar(50) COLLATE "default",
    "c_email" varchar(50) COLLATE "default"
    )
    WITH (OIDS=FALSE)
    ;
    ````

    ​


---
title: 有关于 Android 隐式启动 Activity
date: 2017-07-01 14:35:08
tags: Android
---
忽然发现，半年没有更新过 Blog 了。。。。。
好尴尬，本来这个用来记录自己技术的平台是打算每月至少更新一两次，记录一下这段时间的收获的，可是由于懒散和换了电脑后环境的问题，一直没有继续写。
今天谈一下 Intent 的隐式启动。
这是一个很入门级的问题，但由于之前没用过所以也没管过，既然上周做了今天就写一下自己的体验。
Intent 是平时最常用的组件，我们平时用于页面跳转的就是显示 Intent。
例如：
```java
  val intent = Intent(this, SecondActivity::class.java)
  intent.putExtra(".....", ......)
  startActivity(intent)
```
而隐式启动则是通过 url 的跳转来实现的。
首先是 url 的结构。
虽然我们不是做网页的但是我们要明白，一个完整的 url 是由哪几部分构成的。
url一般主要由Schema、Host、Path以及QueryParameter等构成。
```
Schema 访问协议

Host 域名

port 端口

pathName 路径名

QueryParameter 查询参数
```
我们使用 intent.data 可以接收到这个uri。
这里要提一下 uri 和 url 的区别。
```
首先，URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。
而URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。
而URN，uniform resource name，统一资源命名，是通过名字来标识资源，比如mailto:java-net@java.sun.com。
也就是说，URI是以一种抽象的，高层次概念定义统一资源标识，而URL和URN则是具体的资源标识的方式。URL和URN都是一种URI。

在Java的URI中，一个URI实例可以代表绝对的，也可以是相对的，只要它符合URI的语法规则。而URL类则不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的，schema必须被指定。
```
我们使用 intent.data 接收到的是一个 Uri 的对象，我们可以通过该对象获取到我们需要的信息。
我的做法是建立一个 UrlDispatcher，在 application 中初始化，用来接受和处理 url。
要实现隐式跳转的第一步就是在清单文件中注册 Intent 过滤器。
```XML
  <activity
     android:name=".ui.common.account.UserNameActivity"
     android:launchMode="singleTop">
     <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.BROWSABLE"/>
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="zhuyl" />
      </intent-filter>
  </activity>
```
这样我们就可以接收到域名协议为 zhuyl 的 url 了。
再来看看我们的 UrlDispatcher。
```java
       var actions: MutableMap<String, (MutableMap<String, String>) -> Unit> = HashMap()
       val url = uri.host + uri.path
       val action = if (url.endsWith("/")) url.dropLast(1) else url

       val params: MutableMap<String, String> = HashMap()
       uri.queryParameterNames.forEach {
           val parameter = uri.getQueryParameter(it)
           params[it] = parameter
       }

       actions[action]?.invoke(params)
```
而我们根据 action 来判断调用的方法。
举个例子。
```
zhuyl://open_link/?url=https://baidu.com
```
通过我们的代码解析后，可以得到 action 为 "open_link"
那么
```java
actions["open_link"] = { params ->
   val intent = Intent(application, SecondActivity::class.java)
   val url = params["url"]
   intent.putExtra("url", url)
   intentToActivity(intent)
}
```
这样做，我们在调用 url 启动 activity 的时候就可以拥有一种比较优雅的跳转方式了。
当然，从网页端发出的跳转也就可以处理了。

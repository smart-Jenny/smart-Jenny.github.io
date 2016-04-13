---
layout: post
title: 远程调试H5：weinre+ngrok
date: 2016-03-22
author: "Jenny"
tags: [远程调试]
---

最近做H5开发时，经常遇到在chrome模拟器上样式显示正常，在手机上却出现样式错误的情况，这时仅仅使用chrome模拟器来调试很难解决问题，于是开始琢磨怎么在PC端远程调试手机页面。

weinre使用比较简单，首先用npm安装weinre:

    npm install -g weinre

启动服务监听端口：

    weinre --boundHost 10.75.6.51 --httpPort 9999

浏览器打开服务地址：

    10.75.6.51:9999

在待调试页面中引入下图中的js文件：

![](/img/remote-debug/1.png)

找到debug地址，并在浏览器中打开：

![](/img/remote-debug/2.png)

手机上访问待调试页面，会看到weinre能监听到：

![](/img/remote-debug/3.png)

还可以对页面元素审查：

![](/img/remote-debug/4.png)

weinre兼容性挺强，而且能支持微信端页面的调试，到此为止，如果页面请求使用的是http，那weinre已经可以解决调试问题了。

但是如果要调试https请求的页面，仅仅使用weinre无法解决，因为在页面中需要引入调试的js文件，weinre启动的是http服务，于是使用反向代理软件ngrok，它可以做地址映射，并支持http/https/tcp等，使用也比较简单：

可以上[官网](https://ngrok.com/download)下载，或者可以直接使用npm下载：

    npm install -g ngrok

然后启动ngrok：

    ngrok http 10.75.6.51:9999

![](/img/remote-debug/5.jpg)

其中地址：http://1af5499c.ngrok.io 中子域名由ngrok生成，每次启动可能不一样，可以使用下面命令自定义子域名，但需要付费用户才能使用- -

    ngrok http -subdomain=test 10.75.6.51:9999

启动后可以打开 http://127.0.0.1:4040 查看连接信息：

![](/img/remote-debug/7.png)

oh，回到weinre调试这个话题~现在可以在html中这样引入js了：

![](/img/remote-debug/8.png)

***参考资料：***

[于江水-移动端前端开发调试](http://yujiangshui.com/multidevice-frontend-debug)

[移动端调试工具-Weinre](http://www.cnblogs.com/chaojidan/p/4430213.html)

[Remote Debugging Localhost With Weinre(and HTTPS Too)](http://www.undefinednull.com/2015/03/17/remote-debugging-localhost-with-weinre)

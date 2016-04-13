---
layout: post
title: 《移动Web手册》读书笔记
date: 2016-02-17
author: "Jenny"
tags: [移动web开发,viewport]
---

<h3>关于浏览器</h3>
<h4>代理浏览器</h4>

显示静态文件，生成简单的UI，高度压缩资源文件，只需要一个请求和响应，节省流量，对于HTML/CSS处理很好，但没有客户端交互（每触发一次js，都可能抓取新的内容，返回新的页面，这个我在Opera Mini上尝试过，点击“展开阅读全文”，需要等待较长时间，且刷新了整个页面）。

<h4>混合浏览器</h4>

既可以做完备浏览器，又可以做代理浏览器，如Opera Mobile，Chrome等。

<h4>Android浏览器</h4>

+ 内置浏览器：基于WebKit渲染引擎;
+ Chrome：基于Blink渲染引擎，通常与V8 javascript引擎配合使用。
+ 开源浏览器Chromium：基于Blink渲染引擎;
+ 三星Chrome：基于Chromium开发的三星内置浏览器;
+ 等等。

<h3>视口</h3>
<h4>三种视口</h4>

+ 布局视口：远宽于屏幕宽度，缩放不会影响布局视口;
+ 视觉视口：用户正在看到的网站的区域，与设备屏幕一样宽，但可通过缩放来操作视觉视口;
+ 理想视口：拥有最理想的浏览与阅读的宽度，用户刚进页面时不需要缩放。

<h4>一些术语</h4>

+ 物理分辨率（DPI）：设备每英寸的点数;
+ 设备像素比（DPR）：设备像素个数与理想视口的比，或者说物理设备像素：CSS逻辑像素。例如在iPhone 4设备屏幕中的物理像素数为640*960px,而CSS逻辑像素数为320*480像素。因此，使用大约4个物理像素来显示一个CSS像素。
+ dppx和dpi：dppx是1像素单位中的点数，dpi是1英寸单位中的点数，由于在CSS标准中，1英寸=96像素，所以1dppx=96dpi。

<h4>meta视口</h4>

+ width：设置布局视口的宽度为特定的值，设为"device-width"将布局视口设置成跟理想视口一致，设备旋转时Safari不支持自动将布局视口调整成理想视口（init-scale=1能解决这个问题）;
+ init-scale：设置页面的初始缩放程度，缩放程度跟视觉视口尺寸是逆相关的，一般设置成1时，视觉视口尺寸和理想视口尺寸一样（IE10有着跟Safari完全相反的问题，使用width=device-width可解决）;

得出完美的meta视口：

	<meta name="viewport" content="width=device-width, initial-scale=1">

+ minimum-scale和maximum-scale：设置最小/最大的缩放程度;
+ user-scalable：是否阻止用户进行缩放。这很邪恶，应该无视它。

<h4>媒体查询</h4>

其实就是CSS的if语句。

<h4>媒体查询与js属性相对应：</h4>

+ 布局视口尺寸：document.documentElement.clientWidth/Height，被普遍支持;
+ 视觉视口尺寸：window.innerWidth/Height，接近被普遍支持;
+ 理想视口尺寸：screen.width/height，可能得到的是屏幕的设备像素尺寸，有严重的浏览器兼容问题;
+ 设备屏幕比：window.devicePixelRatio;
+ 屏幕方向：window.orientation。

<h4>参考资料</h4>

+ <a href="http://weizhifeng.net/viewports.html">两个viewport的故事（第一部分）</a>
+ <a href="http://weizhifeng.net/viewports2.html">两个viewport的故事（第二部分）</a>
+ <a href="http://html5online.com.cn/articles/2013041701.html">在各种高分辨率中使用像素</a>
+ <a href="http://www.kancloud.cn/kancloud/the-mobile-web-handbook/64055">《移动Web手册》读书笔记</a>

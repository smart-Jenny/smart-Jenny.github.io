---
layout: post
title: jquery源码之整体架构分析
date: 2015-11-24
author: "Jenny"
tags: [jquery源码]
---

jquery的整体代码架构简化如下：

    (function( window, undefined ) {
        //...定义各种变量
        jQuery = function( selector, context ) {
    		return new jQuery.fn.init( selector, context, rootjQuery );
    	},
        jQuery.fn = jQuery.prototype = {
            constructor: jQuery,
    	    init: function( selector, context, rootjQuery ) {},
    	    ...
    	};
        jQuery.fn.init.prototype = jQuery.fn;
        if ( typeof window === "object" && typeof window.document === "object" ) {
    	    window.jQuery = window.$ = jQuery;
        }
    })( window );

这里有几点需要整明白的：

1、使用一个闭包定义jquery，创建一个私有空间，这样内外部的命名空间就不会相互干扰了。为了在外部能够使用jquery，将jquery($)提升到window作用域下，即：

    window.jQuery = window.$ = jQuery;

2、为什么要将window作为参数传入：

有两个好处，一是便于代码压缩，jquery每个版本都有压缩版，如将传入的window参数命名为w，则可减少大量的代码量；二是将window全局变量变成局部变量，可加快访问速度。

3、为什么要增加undefined参数：

undefined在某些浏览器（如chrome、ie等）是可以被修改的，如：

    undefined = "hello";

为了保证在jQuery内部，使用到的undefined真的是undefined而没有被修改过，将它作为参数传入。

ok，上面几点都比较好理解，下面开始分析jquery的构建方式。

jQuery相当于一个类，在jquery中，我们使用$()或jQuery()来得到jquery实例，按此思路定义jQuery类如下：

    var jQuery = function() {
        return new jQuery();
    }

啊噢，死循环了~

想拿到实例，即this，原方法拿不到，就去原型那里拿咯：

    var jQuery = function() {
        return jQuery.prototype.init();
    }
    jQuery.prototype = {
        init:function() {
            return this;
        }
    }

但这样还是会有问题，因为这样相当于单例了，创建出来的对象相互共享作用域，必然导致很多问题，如：

    var jQuery = function() {
        return jQuery.prototype.init();
    }
    jQuery.prototype = {
        init:function() {
            return this;
        },
        age:18
    }
    var jq1 = new jQuery();
    var jq2 = new jQuery();
    jq1.age = 20;
    console.log(jq1.age); //20
    console.log(jq2.age); //20

所以需要让他们的作用域分离，互不干扰，故每次创建新对象即可解决：

    var jQuery = function() {
        return new jQuery.prototype.init();
    }
    jQuery.prototype = {
        init:function() {
            return this;
        }
    }

但是这样仍然是有问题的，如：

    var jQuery = function() {
        return new jQuery.prototype.init();
    }
    jQuery.prototype = {
        init:function() {
            return this;
        },
        age:18
    }
    var jq = new jQuery();
    console.log(jq.age); //undefined

原因是这里init和外部的jquery.prototype分离了，init对象中并没有age属性。

改改改：

    var jQuery = function() {
        return new jQuery.prototype.init();
    }
    jQuery.prototype = {
        init:function() {
            return this;
        },
        age:18
    }
    jQuery.prototype.init.prototype = jQuery.prototype;

好了，到此为止基本没啥问题了，jQuery也是这样实现的，只是多了个jQuery.fn作为jQuery.prototype的引用而已，没多少特别意义。

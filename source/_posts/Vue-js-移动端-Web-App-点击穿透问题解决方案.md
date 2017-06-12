---
title: Vue.js 移动端 Web App 点击穿透问题解决方案
date: 2017-06-12 22:00:41
tags: Vue.js, 移动端
---

## 描述

在近期的一个移动端项目中，有一个页面需要有弹框提示，并且这个弹框通过关闭按钮关闭。页面当中使用了 iScroll 来实现页面局部滚动，在 iScroll 的配置当中把 `tap` 和 `click` 事件都开启了。
代码如下：
```
this.myScroll = new IScroll(this.$refs.wrapper, {
  mouseWheel: true,
  click: true,
  tap: true
})
```
在实现过程中，遇到了一个奇怪的问题，由于按钮的位置与弹框右上角的关闭按钮位置一致，当我点击按钮时，弹框一闪而过。

<!-- more -->

效果如下：
![效果图](https://wx3.sinaimg.cn/mw690/85f4065cgy1fgiqepbjycg20df0lkwip.gif)


## 原因

### 什么是点击穿透？

假如页面上有两个元素A和B。B元素在A元素之上。我们在B元素的touchstart事件上注册了一个回调函数，该回调函数的作用是隐藏B元素。我们发现，当我们点击B元素，B元素被隐藏了，随后，A元素触发了click事件。

通过上网查找有关资料，翻阅了移动端的书籍，发现在手机端中，事件的触发顺序为：`touchstart` -> `touchmove` -> `touchend`，而 `click` 事件有 300ms 的延迟，当 `touchstart` 事件把B元素隐藏之后，隔了300ms，浏览器触发了 `click` 事件，但是此时B元素不见了，所以该事件被派发到了A元素身上。如果A元素是一个链接，那此时页面就会意外地跳转。


## 解决方案

### 1. 改用 `touch` 事件

由于项目使用的是 `Vue.js`，这里就提供一下 `Vue.js` 的解决方法。使用了 [vue-tap](https://github.com/MeCKodo/vue-tap)  的一个插件，具体使用方法参看官方文档，在需要点击事件的时候，通过 `v-tap` 指令来绑定。
```javascript
// main.js
import vueTap from 'v-tap' // 引入插件
Vue.use(vueTap) // 全局注册
```
```javascript
v-tap="{methods:showReceiveModel}" // 在元素上绑定事件
```

### 2. 使用 fastclick 插件

这个也是在网上看到的，也可以解决点透问题，使用方法可以看 [fastclick](https://github.com/ftlabs/fastclick) 的文档，在这里提供一下 `Vue.js` 的引入及使用
```
import FastClick from 'fastclick'; // 引入插件
FastClick.attach(document.body, options); // 使用 fastclick
```

最终没有使用这个方案是因为有一些小 bug ，如 [Fastclick 导致click事件触发两次的问题](http://blog.csdn.net/forevercjl/article/details/46730157)。

## 其他

###  `tap` 一词
对于 `tap` 这个词，用过 `Zepto` 或 `KISSY` 等移动端js库的人肯定对tap事件不陌生，做PC页面时绑定 `click`，相应地手机页面就绑定 `tap`。但原生的 `touch` 事件本身是没有 `tap` 的，js库里提供的tap事件都是模拟出来的。

手机上响应 `click` 事件会有300ms的延迟，那么这300ms到底是干嘛了？浏览器在 `touchend` 后会等待约300ms，原因是判断用户是否有双击`（double tap）`行为。如果没有 `tap` 行为，则触发 `click` 事件，而双击过程中就不适合触发 `click` 事件了。由此可以看出 `click` 事件触发代表一轮触摸事件的结束。

既然说tap事件是模拟出来的，我们可以看下 `Zepto` 对 singleTap 事件的处理。见 [源码 136-143 行](https://github.com/madrobby/zepto/blob/master/src/touch.js#L136-L143)，可以看出在 `touchend`响应 250ms 无操作后，则触发 `singleTap`。


## 参考

### 博文
- [也来说说touch事件与点击穿透问题](https://segmentfault.com/a/1190000003848737)
- [移动页面点击穿透问题解决方案](http://www.ayqy.net/blog/%E7%A7%BB%E5%8A%A8%E9%A1%B5%E9%9D%A2%E7%82%B9%E5%87%BB%E7%A9%BF%E9%80%8F%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)
- [点击穿透原理及解决](http://www.cnblogs.com/shytong/p/5463673.html)

### 书籍
- 《移动 Web 手册》

> 以上部分资料搜集整理自网络，如有不对的地方希望及时告知，欢迎大家批评指正，谢谢！



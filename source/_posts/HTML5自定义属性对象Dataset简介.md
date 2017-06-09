---
title: HTML5自定义属性对象Dataset简介
date: 2017-06-09 20:46:16
tags: 前端, HTML5, Dataset
---

## 介绍

在HTML5中我们可以使用data-前缀设置我们需要的自定义属性，来进行一些数据的存放，例如我们要在一个文字按钮上存放相对应的id：
`<a href="javascript:" data-id="2312">测试</a>`
这里的data-前缀就被称为data属性，其可以通过脚本进行定义，也可以应用CSS属性选择器进行样式设置。数量不受限制，在控制和渲染数据的时候提供了非常强大的控制。

<!-- more -->

## 例子

``` html
<div id="day2-meal-expense" 
  data-drink="coffee" 
  data-food="sushi" 
  data-meal="lunch">¥20.12</div>
```

以上代码要想获取某个属性的值
除了通过`getAttribute("属性名")`以外，我们还可以用`dataset.属性名`来获取

``` javascript
var expenseday2 = document.querySelector('#day2-meal-expense');  
var typeOfDrink = expenseday2.dataset.drink;
```

## 要求

需要注意的是带连字符连接的名称在使用的时候需要命名驼峰化，即大小写组合书写，这与应用元素的style对象类似，`dom.style.borderColor`。例如，假设上面的例子中现在有如下data属性，`data-meal-time`，则我们要获取相应的值可以使用：
`expenseday2.dataset.mealTime`

> data属性基本上所有的浏览器都是支持的，但是dataset对象就属于新贵，目前仅在Opera 11.1+， Chrome 9+下可以通过JavaScript，使用dataset访问你自定义的data属性。至于其他浏览器，FireFox 6+（未出）以及Safari 6+（未出）会支持dataset对象，至于IE浏览器，目前看来还是遥遥无期的趋势。具体的些兼容性数据，您可以[点击这里](http://caniuse.com/#feat=dataset)访问。

## 优势

在速度上有明显提升。
如果使用传统的方法获取属性值应该会类似下面：

`var typeOfDrink = document.querySelector('#day2-meal-expense').getAttribute('data-drink');`
现在，假设我们要获得多个自定义的属性值，得，折腾的事情就来了，我们可能要采用类似下面的N行代码实现了：

``` javascript
var attrs = expenseday2.attributes,
expense = {}, i, j;  
for (i = 0, j = attrs.length; i < j; i++) {
  if(attrs[i].name.substring(0, 5) == 'data-') {
    expense[attrs[i].name.substring(5)] = attrs[i].value;
  }
}
```

而使用dataset属性，我们根本不需要任何循环去获取你想要的那个值，直接秒杀：
`expense = document.querySelector('#day2-meal-expense').dataset;`

> dataset并不是典型意义上的JavaScript对象，而是个DOMStringMap对象，DOMStringMap是HTML5一种新的含有多个名-值对的交互变量。

## 兼容

每次你使用自定义data属性的时候，使用dataset去获取名-值对就是个不错的选择。考虑到现在很多浏览器还是把dataset当作不认识的外星生物看待，所以，在实际使用的时候，有必要进行下特征检测，看看是否支持dataset，类似下面的使用：

``` javascript
if(expenseday2.dataset) {
  expenseday2.dataset.dessert = 'icecream';
} else {
  expenseday2.setAttribute('data-dessert', 'icecream');
}
```

注意：如果你的应用程序会频繁更新data属性值的话，建议使用JavaScript对象进行数据管理，而不是每次都经由data属性进行更新。

> 参考资料：https://dev.opera.com/articles/introduction-to-datasets/


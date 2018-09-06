---
title: Vue.js 移动端适配之 vw 解决方案
date: 2018-09-03 10:55:08
tags: 
- vw
- Vue.js
---

## 1. 安装并配置PostCss插件

```bash
npm i postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano --S
```
还需要安装 cssnano-preset-advanced
```bash
npm i cssnano-preset-advanced --save-dev
```
<!-- more -->

## 2. 对 PostCss 进行配置

找到在根目录中的`.postcssrc.js`，对PostCSS插件进行配置

```
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    // to edit target browsers: use "browserslist" field in package.json
    "postcss-write-svg": {
      uft8: false
    },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
      viewportWidth: 750, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
      viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
      unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
      viewportUnit: 'vw', // 指定需要转换成的视窗单位，建议使用vw
      selectorBlackList: ['.ignore', '.hairlines'], // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
      minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
      mediaQuery: false // 允许在媒体查询中转换`px`
    },
    "postcss-viewport-units": {},
    "cssnano": {
      preset: "advanced",
      autoprefixer: false, // 和cssnext同样具有autoprefixer，保留一个
      "postcss-zindex": false
    }
  }
}
```

## 3. 引入viewport-units-buggyfill解决兼容问题

在 `index.html` 中引入js

```
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>
<script>
  window.onload = function () { 
    window.viewportUnitsBuggyfill.init({ hacks: window.viewportUnitsBuggyfillHacks });
  }
</script>
```

## 4. 遇到的问题及解决方案

1. img图片不显示

全局引入CSS样式

```css
img { content: normal !important; }
```

2. 与第三方UI库兼容问题

我这里使用了 Element 的 Mint-UI，在编译的过程中会出现这个错误：

```bash
 warning  in ./node_modules/mint-ui/lib/style.css

(Emitted value instead of an instance of Error) postcss-viewport-units: /Users/Wyj/Workspace/imglive/wx/node_modules/mint-ui/lib/style.css:267:1: '.mint-cell-allow-right::after' already has a 'content' property, give up to overwrite it.

 @ ./node_modules/mint-ui/lib/style.css 4:14-118 13:3-17:5 14:22-126
 @ ./src/plugins/mint-ui/index.js
 @ ./src/plugins/index.js
 @ ./src/main.js
 @ multi (webpack)-dev-server/client?http://0.0.0.0:8080 webpack/hot/dev-server ./src/main.js
```

可通过自行修改 `postcss-px-to-viewport`

在node_modules中找到 `postcss-px-to-viewport` ，打开`index.js`
新增对exclude选项的处理

```
module.exports = postcss.plugin('postcss-px-to-viewport', function (options) {

  var opts = objectAssign({}, defaults, options);
  var pxReplace = createPxReplace(opts.viewportWidth, opts.minPixelValue, opts.unitPrecision, opts.viewportUnit);

  return function (css) {

    css.walkDecls(function (decl, i) {
      if (options.exclude) {  // 添加对exclude选项的处理
        if (Object.prototype.toString.call(options.exclude) !== '[object RegExp]') {
          throw new Error('options.exclude should be RegExp!')
        }
        if (decl.source.input.file.match(options.exclude) !== null) return;
      }
      // This should be the fastest test and will remove most declarations
      if (decl.value.indexOf('px') === -1) return;

      if (blacklistedSelector(opts.selectorBlackList, decl.parent.selector)) return;

      decl.value = decl.value.replace(pxRegex, pxReplace);
    });

    if (opts.mediaQuery) {
      css.walkAtRules('media', function (rule) {
        if (rule.params.indexOf('px') === -1) return;
        rule.params = rule.params.replace(pxRegex, pxReplace);
      });
    }

  };
});
```
然后在 `.postcssrc.js` 添加 `postcss-px-to-viewport` 的 `exclude` 选项

```
"postcss-px-to-viewport": {
  viewportWidth: 750,
  viewportHeight: 1334,
  unitPrecision: 3,
  viewportUnit: 'vw',
  selectorBlackList: ['.ignore', '.hairlines'],
  minPixelValue: 1,
  mediaQuery: false,
  exclude: /(\/|\\)(node_modules)(\/|\\)/
},
```

或者使用改良版的  `postcss-px-to-viewport-opt`

```
npm install postcss-px-to-viewport-opt -S
```

然后在 `.postcssrc.js` 配置 `postcss-px-to-viewport-opt` 

```
'postcss-px-to-viewport-opt': {
  viewportWidth: 375, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
  viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
  unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
  viewportUnit: 'vw', // 指定需要转换成的视窗单位，建议使用vw
  selectorBlackList: ['.ignore', '.hairlines'], // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
  minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
  mediaQuery: false, // 允许在媒体查询中转换`px`
  exclude: /(\/|\\)(node_modules)(\/|\\)/
},
```


## 5. 参考资料

[Vue+ts下的移动端vw适配（第三方库css问题）](https://zhuanlan.zhihu.com/p/36913200)
[再聊移动端页面的适配](https://www.w3cplus.com/css/vw-for-layout.html)
[如何在Vue项目中使用vw实现移动端适配](https://www.w3cplus.com/mobile/vw-layout-in-vue.html)

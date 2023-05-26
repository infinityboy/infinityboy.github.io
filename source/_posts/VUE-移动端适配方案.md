---
title: VUE 移动端适配方案
date: 2020-02-05 15:08:52
tags: vue
categories:
  - 前端
---

> 本篇博客仅以个人学习记录使用
> 参照开源项目地址：https://juejin.im/entry/5aa09c3351882555602077ca

### 搭建步骤

1.可直接使用 cli 搭建项目工程，然后执行以下命令安装需要用的依赖包

```
yarn add postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-preset-env postcss-viewport-units cssnano
```

安装成功后 package.json 文件中就会有这些依赖包

2.接下来在.postcssrc.js 文件对新安装的 PostCSS 插件进行配置

```
module.exports = {
    "plugins": {
        "postcss-import": {},
        "postcss-url": {},
        "postcss-aspect-ratio-mini": {},
        "postcss-write-svg": {
            utf8: false
        },
        "postcss-cssnext": {},
        "postcss-px-to-viewport": {
            viewportWidth: 750,     // (Number) The width of the viewport.
            viewportHeight: 1334,    // (Number) The height of the viewport.
            unitPrecision: 3,       // (Number) The decimal numbers to allow the REM units to grow to.
            viewportUnit: 'vw',     // (String) Expected units.
            selectorBlackList: ['.ignore', '.hairlines'],  // (Array) The selectors to ignore and leave as px.
            minPixelValue: 1,       // (Number) Set the minimum pixel value to replace.
            mediaQuery: false       // (Boolean) Allow px to be converted in media queries.
        },
        "postcss-viewport-units":{},
        "cssnano": {
            preset: "advanced",
            autoprefixer: false,
            "postcss-zindex": false
        }
    }
}
```

```
postcss-px-to-viewport 插件对应配置

"postcss-px-to-viewport": {
    viewportWidth: 750,      // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
    viewportHeight: 1334,    // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
    unitPrecision: 3,        // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
    viewportUnit: 'vw',      // 指定需要转换成的视窗单位，建议使用vw
    selectorBlackList: ['.ignore', '.hairlines'],  // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
    minPixelValue: 1,       // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
    mediaQuery: false       // 允许在媒体查询中转换`px`
}
```

3.到了这一步已经完成了大部分，但是还是会有一些兼容性的问题，但是可借助 viewport 的 polyfill 解决 viewport 的兼容问题，将以下代码引入到 vue 中的 index.html 中

```
引入polyfill
<script src="//g.alicdn.com/fdilab/lib3rd/viewport-units-buggyfill/0.6.2/??viewport-units-buggyfill.hacks.min.js,viewport-units-buggyfill.min.js"></script>

调用polyfill
<script>
    window.onload = function () {
        window.viewportUnitsBuggyfill.init({
            hacks: window.viewportUnitsBuggyfillHacks
        });
    }


    var winDPI = window.devicePixelRatio;
    var uAgent = window.navigator.userAgent;
    var screenHeight = window.screen.height;
    var screenWidth = window.screen.width;
    var winWidth = window.innerWidth;
    var winHeight = window.innerHeight;

    //测试设备基本属性
    alert(
        "Windows DPI:" + winDPI +
        ";\ruAgent:" + uAgent +
        ";\rScreen Width:" + screenWidth +
        ";\rScreen Height:" + screenHeight +
        ";\rWindow Width:" + winWidth +
        ";\rWindow Height:" + winHeight
    )

</script>

```

具体的使用。在你的 CSS 中，只要使用到了 viewport 的单位（vw、vh、vmin 或 vmax ）地方，需要在样式中添加 content：

```
.my-viewport-units-using-thingie {
    width: 50vmin;
    height: 50vmax;
    top: calc(50vh - 100px);
    left: calc(50vw - 100px);

    /* hack to engage viewport-units-buggyfill */
    content: 'viewport-units-buggyfill; width: 50vmin; height: 50vmax; top: calc(50vh - 100px); left: calc(50vw - 100px);';
}
```

这可能会令你感到恶心，而且我们不可能每次写 vw 都去人肉的计算。特别是在我们的这个场景中，咱们使用了 postcss-px-to-viewport 这个插件来转换 vw，更无法让我们人肉的去添加 content 内容。
这个时候就需要前面提到的 postcss-viewport-units 插件。这个插件将让你无需关注 content 的内容，插件会自动帮你处理。转换后的代码可以在浏览器中查看到类似 content: 'viewport-units-buggyfill;这样的标识

4.Viewport Units Buggyfill 还提供了其他的功能。详细的这里不阐述了。但是 content 也会引起一定的副作用。比如 img 和伪元素::before(:before)或::after（:after）。在 img 中 content 会引起部分浏览器下，图片不会显示。这个时候需要全局添加：

```
img {
    content: normal !important;
}
```

而对于::after 之类的，就算是里面使用了 vw 单位，Viewport Units Buggyfill 对其并不会起作用。比如：

```
// 编译前
.after {
    content: 'after content';
    display: block;
    width: 100px;
    height: 20px;
    background: green;
}

// 编译后
.after[data-v-469af010] {
    content: "after content";
    display: block;
    width: 13.333vw;
    height: 2.667vw;
    background: green;
}
```

### 相关插件介绍

#### postcss-import

postcss-import 主要功有是解决@import 引入路径问题。使用这个插件，可以让你很轻易的使用本地文件、node_modules 或者 web_modules 的文件。这个插件配合 postcss-url 让你引入文件变得更轻松

#### postcss-url

该插件主要用来处理文件，比如图片文件、字体文件等引用路径的处理。

在 Vue 项目中，vue-loader 已具有类似的功能，只需要配置中将 vue-loader 配置进去

#### autoprefixer

autoprefixer 插件是用来自动处理浏览器前缀的一个插件。如果你配置了 postcss-cssnext，其中就已具备了 autoprefixer 的功能。在配置的时候，未显示的配置相关参数的话，表示使用的是 Browserslist 指定的列表参数，你也可以像这样来指定 last 2 versions 或者 > 5%。

如此一来，你在编码时不再需要考虑任何浏览器前缀的问题，可以专心撸码。这也是 PostCSS 最常用的一个插件之一。

#### 其他插件

Vue-cli 默认配置了上述三个 PostCSS 插件，但我们要完成 vw 的布局兼容方案，或者说让我们能更专心的撸码，还需要配置下面的几个 PostCSS 插件：

```
postcss-aspect-ratio-mini
postcss-px-to-viewport
postcss-write-svg
postcss-cssnext  (这个插件已经被postcss-preset-env所替代)
cssnano
postcss-viewport-units
```

#### postcss-preset-env/postcss-cssnext(废弃)

该插件可以让我们使用 CSS 未来的特性，其会对这些特性做相关的兼容性处理

#### cssnano

cssnano 主要用来压缩和清理 CSS 代码。在 Webpack 中，cssnano 和 css-loader 捆绑在一起，所以不需要自己加载它。不过你也可以使用 postcss-loader 显式的使用 cssnano

在 cssnano 的配置中，使用了 preset: "advanced"，所以我们需要另外安装：

```
yarn add cssnano-preset-advanced
```

cssnano 集成了一些其他的 PostCSS 插件，如果你想禁用 cssnano 中的某个插件的时候，可以像下面这样操作：

```
"cssnano": {
    autoprefixer: false,
    "postcss-zindex": false
}
```

上面的代码把 autoprefixer 和 postcss-zindex 禁掉了。前者是有重复调用，后者是一个讨厌的东东。只要启用了这个插件，z-index 的值就会重置为 1。这是一个天坑，千万记得将 postcss-zindex 设置为 false。

#### postcss-px-to-viewport

postcss-px-to-viewport 插件主要用来把 px 单位转换为 vw、vh、vmin 或者 vmax 这样的视窗单位，也是 vw 适配方案的核心插件之一。

在配置中需要配置相关的几个关键参数：

```
"postcss-px-to-viewport": {
    viewportWidth: 750,      // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
    viewportHeight: 1334,    // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
    unitPrecision: 3,        // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
    viewportUnit: 'vw',      // 指定需要转换成的视窗单位，建议使用vw
    selectorBlackList: ['.ignore', '.hairlines'],  // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类名
    minPixelValue: 1,       // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
    mediaQuery: false       // 允许在媒体查询中转换`px`
}
```

目前出视觉设计稿，我们都是使用 750px 宽度的，那么 100vw = 750px，即 1vw = 7.5px。那么我们可以根据设计图上的 px 值直接转换成对应的 vw 值。在实际撸码过程，不需要进行任何的计算，直接在代码中写 px，比如:

```
.test {
    border: .5px solid black;
    border-bottom-width: 4px;
    font-size: 14px;
    line-height: 20px;
    position: relative;
}
[w-188-246] {
    width: 188px;
}
```

编译出来的

```
.test {
    border: .5px solid #000;
    border-bottom-width: .533vw;
    font-size: 1.867vw;
    line-height: 2.667vw;
    position: relative;
}
[w-188-246] {
    width: 25.067vw;
}
```

在不想要把 px 转换为 vw 的时候，首先在对应的元素（html）中添加配置中指定的类名.ignore 或.hairlines(.hairlines 一般用于设置 border-width:0.5px 的元素中)：

```
<div class="box ignore"></div>
```

写 CSS 的时候：

```
.ignore {
    margin: 10px;
    background-color: red;
}
.box {
    width: 180px;
    height: 300px;
}
.hairlines {
    border-bottom: 0.5px solid red;
}
```

编译出来的 CSS:

```
.box {
    width: 24vw;
    height: 40vw;
}
.ignore {
    margin: 10px; /*.box元素中带有.ignore类名，在这个类名写的`px`不会被转换*/
    background-color: red;
}
.hairlines {
    border-bottom: 0.5px solid red;
}
```

上面解决了 px 到 vw 的转换计算。那么在哪些地方可以使用 vw 来适配我们的页面。根据相关的测试：

容器适配，可以使用 vw
文本的适配，可以使用 vw
大于 1px 的边框、圆角、阴影都可以使用 vw
内距和外距，可以使用 vw

#### postcss-aspect-ratio-mini

postcss-aspect-ratio-mini 主要用来处理元素容器宽高比。在实际使用的时候，具有一个默认的结构

```
<div aspectratio>
    <div aspectratio-content></div>
</div>
```

在实际使用的时候，你可以把自定义属性 aspectratio 和 aspectratio-content 换成相应的类名，比如：

```
<div class="aspectratio">
    <div class="aspectratio-content"></div>
</div>
```

也可以用自定义属性，它和类名所起的作用是同等的。结构定义之后，需要在你的样式文件中添加一个统一的宽度比默认属性：

```
[aspectratio] {
    position: relative;
}
[aspectratio]::before {
    content: '';
    display: block;
    width: 1px;
    margin-left: -1px;
    height: 0;
}

[aspectratio-content] {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
}
```

如果我们想要做一个 188:246（188 是容器宽度，246 是容器高度）这样的比例容器，只需要这样使用：

```
[w-188-246] {
    aspect-ratio: '188:246';
}
```

有一点需要特别注意：aspect-ratio 属性不能和其他属性写在一起，否则编译出来的属性只会留下 aspect-ratio 的值，比如：

```
<div aspectratio w-188-246 class="color"></div>
```

编译前的 CSS 如下：

```
[w-188-246] {
    width: 188px;
    background-color: red;
    aspect-ratio: '188:246';
}
```

编译之后：

```
[w-188-246]:before {
    padding-top: 130.85106382978725%;
}
```

主要是因为在插件中做了相应的处理，不在每次调用 aspect-ratio 时，生成前面指定的默认样式代码，这样代码没那么冗余。所以在使用的时候，需要把 width 和 background-color 分开来写：

```
[w-188-246] {
    width: 188px;
    background-color: red;
}
[w-188-246] {
    aspect-ratio: '188:246';
}
```

这个时候，编译出来的 CSS 就正常了：

```
[w-188-246] {
    width: 25.067vw;
    background-color: red;
}
[w-188-246]:before {
    padding-top: 130.85106382978725%;
}
```

#### postcss-write-svg

postcss-write-svg 插件主要用来处理移动端 1px 的解决方案。该插件主要使用的是 border-image 和 background 来做 1px 的相关处理。比如:

```
@svg 1px-border {
    height: 2px;
    @rect {
        fill: var(--color, black);
        width: 100%;
        height: 50%;
    }
}
.example {
    border: 1px solid transparent;
    border-image: svg(1px-border param(--color #00b1ff)) 2 2 stretch;
}
```

编译出来的 CSS:

```
.example {
    border: 1px solid transparent;
    border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E") 2 2 stretch;
}
```

上面演示的是使用 border-image 方式，除此之外还可以使用 background-image 来实现。比如：

```
@svg square {
    @rect {
        fill: var(--color, black);
        width: 100%;
        height: 100%;
    }
}

#example {
    background: white svg(square param(--color #00b1ff));
}
```

编译出来就是：

```
#example {
    background: white url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg'%3E%3Crect fill='%2300b1ff' width='100%25' height='100%25'/%3E%3C/svg%3E");
}
```

```
特别声明：由于有一些低端机对border-image支持度不够友好，个人建议你使用background-image的这个方案。
```

#### postcss-viewport-units

postcss-viewport-units 插件主要是给 CSS 的属性添加 content 的属性，配合 viewport-units-buggyfill 库给 vw、vh、vmin 和 vmax 做适配的操作。

这是实现 vw 布局必不可少的一个插件，因为少了这个插件，这将是一件痛苦的事情。后面你就清楚。

### 适用情况

上面解决了 px 到 vw 的转换计算。那么在哪些地方可以使用 vw 来适配我们的页面。根据相关的测试：

容器适配，可以使用 vw
文本的适配，可以使用 vw
大于 1px 的边框、圆角、阴影都可以使用 vw
内距和外距，可以使用 vw

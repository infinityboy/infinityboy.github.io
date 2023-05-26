---
title: css 水平垂直居中
date: 2019-08-03 15:47:35
tags: css
categories: 
- 前端
---

>本篇博客仅以个人学习记录使用
>参考链接：https://yanhaijing.com/css/2018/01/17/horizontal-vertical-center/


### 介绍

使用css实现水平垂直居中是一个最常见不过的需求了，这里做一下总结，特别区分定宽高和不定宽高的水平垂直居中

#### 仅居中元素定宽高适用

1.absolute + 负margin 

```
<div class="wp">
    <div class="box size">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```



```
绝对定位的百分比是相对于最近已定位父元素的宽高，通过这个特性可以让子元素的居中显示，但绝对定位是基于子元素的左上角，期望的效果是子元素的中心居中显示

为了修正这个问题，可以借助外边距的负值，负的外边距可以让元素向相反方向定位，通过指定子元素的外边距为子元素宽度一半的负值，就可以让子元素居中了，css代码如下

/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 50%;
    left: 50%;
    margin-left: -50px;
    margin-top: -50px;
}


这是比较常用的方式，这种方式比较好理解，兼容性也很好，缺点是需要知道子元素的宽

```

2.absolute + margin auto 

```
<div class="wp">
    <div class="box size">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
这种方式也要求居中元素的宽高必须固定
这种方式通过设置各个方向的距离都是0，此时再讲margin设为auto，就可以在各个方向上居中了

/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}

这种方法兼容性也很好，缺点是需要知道子元素的宽高

```


3.absolute + calc

```
<div class="wp">
    <div class="box size">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
这种方式也要求居中元素的宽高必须固定

感谢css3带来了计算属性，既然top的百分比是基于元素的左上角，那么在减去宽度的一半就好了，代码如下

/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: calc(50% - 50px);
    left: calc(50% - 50px);
}

这种方法兼容性依赖calc的兼容性，缺点是需要知道子元素的宽高

```

#### 居中元素不定宽高

1.absolute + transform 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```


```
还是绝对定位，但这个方法不需要子元素固定宽高，所以不再需要size类了

修复绝对定位的问题，还可以使用css3新增的transform，transform的translate属性也可以设置百分比，其是相对于自身的宽和高，所以可以讲translate设置为-50%，就可以做到居中了，代码如下

/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

这种方法兼容性依赖translate2d的兼容性

```

2.lineheight 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
利用行内元素居中属性也可以做到水平垂直居中,不需要size

把box设置为行内元素，通过text-align就可以做到水平居中，但很多同学可能不知道通过通过vertical-align也可以在垂直方向做到居中，代码如下

/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    line-height: 300px;
    text-align: center;
    font-size: 0px;
}
.box {
    font-size: 16px;
    display: inline-block;
    vertical-align: middle;
    line-height: initial;
    text-align: left; /* 修正文字 */
}

这种方法需要在子元素中将文字显示重置为想要的效果

```

3.writing-mode 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
很多人一定和我一样不知道writing-mode属性，感谢@张鑫旭老师的反馈，简单来说writing-mode可以改变文字的显示方向，比如可以通过writing-mode让文字的显示变为垂直方向

<div class="div1">水平方向</div>
<div class="div2">垂直方向</div>

.div2 {
    writing-mode: vertical-lr;
}

显示效果如下：

水平方向
垂
直
方
向

```

```

更神奇的是所有水平方向上的css属性，都会变为垂直方向上的属性，比如text-align，通过writing-mode和text-align就可以做到水平和垂直方向的居中了，只不过要稍微麻烦一点

<div class="wp">
    <div class="wp-inner">
        <div class="box">123123</div>
    </div>
</div>


/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    writing-mode: vertical-lr;
    text-align: center;
}
.wp-inner {
    writing-mode: horizontal-tb;
    display: inline-block;
    text-align: center;
    width: 100%;
}
.box {
    display: inline-block;
    margin: auto;
    text-align: left;
}

```
4.table 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
曾经table被用来做页面布局，现在没人这么做了，但table也能够实现水平垂直居中，但是会增加很多冗余代码

<table>
    <tbody>
        <tr>
            <td class="wp">
                <div class="box">123123</div>
            </td>
        </tr>
    </tbody>
</table>

tabel单元格中的内容天然就是垂直居中的，只要添加一个水平居中属性就好了

.wp {
    text-align: center;
}
.box {
    display: inline-block;
}

这种方法就是代码太冗余，而且也不是table的正确用法

```

5.css-table 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
.wp {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
.box {
    display: inline-block;
}

这种方法和table一样的原理，但却没有那么多冗余代码，兼容性也还不错
```
6.flex 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
flex作为现代的布局方案，颠覆了过去的经验，只需几行代码就可以优雅的做到水平垂直居中

.wp {
    display: flex;
    justify-content: center;
    align-items: center;
}

目前在移动端已经完全可以使用flex了，PC端需要看自己业务的兼容性情况
```


7.grid 

```
<div class="wp">
    <div class="box">123123</div>
</div>

wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高

/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```

```
.wp {
    display: grid;
}
.box {
    align-self: center;
    justify-self: center;
}

代码量也很少，但兼容性不如flex，不是特别推荐使用，不过如果是在现代浏览器当中是完全可以使用的
```

最后看下来发现当定宽高的时候都是用定位去实现的，不定宽的时候在现代浏览器中使用flex就比较简单了

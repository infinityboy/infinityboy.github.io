---
title: ES6技巧
date: 2019-12-14 14:39:05
tags: JS
categories:
    - 前端
---

> 本篇博客仅以个人学习记录使用
>原文链接: https://www.h5jun.com/post/six-nifty-es6-tricks.html

### 通过参数默认值强制要求传参
ES6 指定默认参数在它们被实际使用的时候才会被执行，这个特性让我们可以强制要求传参：
```
/**
* Called if a parameter is missing and
* the default value is evaluated.
*/
function mandatory() {
    throw new Error("Missing parameter");
}
function foo(mustBeProvided = mandatory()) {
    return mustBeProvided;
}
```
函数调用 mandatory() 只有在参数 mustBeProvided 缺失的时候才会被执行。
在控制台测试:
```
> foo()
Error: Missing parameter
> foo(123)
123
```
更多内容:
段落： ["Required parameters"](https://exploringjs.com/es6/ch_parameter-handling.html#_required-parameters) 。
### 通过 for-of 循环来遍历数组元素和索引
方法 forEach() 允许你遍历一个数组的元素和索引：
```
var arr = ["a", "b", "c"];
arr.forEach(function (elem, index) {
    console.log("index = "+index+", elem = "+elem);
});
// Output:
// index = 0, elem = a
// index = 1, elem = b
// index = 2, elem = c
```
ES6 的 for-of 循环支持 ES6 迭代（通过 iterables 和 iterators）和解构。如果你通过数组的新方法 enteries() 再结合解构，可以达到上面 forEach 同样的效果：
```
const arr = ["a", "b", "c"];
for (const [index, elem] of arr.entries()) {
    console.log(`index = ${index}, elem = ${elem}`);
}
```
arr.enteries() 通过索引-元素配对返回一个可迭代对象。然后通过解构数组 [index, elem] 直接得到每一对元素和索引。console.log() 的参数是 ES6 中的模板字面量特性，这个特性带给字符串解析模板变量的能力。
更多内容:
章节： ["Destructuring"](https://exploringjs.com/es6/ch_destructuring.html) 。
章节： ["Iterables and iterators"](https://exploringjs.com/es6/ch_iteration.html)。
段落： ["Iterating with a destructuring pattern"](https://exploringjs.com/es6/ch_iteration.html) 。
章节： ["Template literals"](https://exploringjs.com/es6/ch_template-literals.html) 。

### 遍历 Unicode 表示的字符串
一些 Unicode 编码的字由两个 JavaScript 字符组成，例如，emoji 表情：
![](data:image/png;base64,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)
字符串实现了 ES6 迭代，如果你通过迭代来访问字符串，你可以获得编码过的单个字（每个字用 1 或 2 个 JavaScript 字符表示）。例如
```
for (const ch of "x\uD83D\uDE80y") {
    console.log(ch.length);
}
// Output:
// 1
// 2
// 1
```
这让你能够很方便地得到一个字符串中实际的字数：
```
> [..."x\uD83D\uDE80y"].length
3
```
展开操作符 (...) 将它的操作对象展开并插入数组。
更多内容：
章节： ["Unicode in ES6"](https://exploringjs.com/es6/ch_unicode.html) 。
段落： ["The spread operator (...)"](https://exploringjs.com/es6/ch_parameter-handling.html#sec_spread-operator) 。

### 通过变量解构交换两个变量的值
如果你将一对变量放入一个数组，然后将数组解构赋值相同的变量（顺序不同），你就可以不依赖中间变量交换两个变量的值：
```
[a, b] = [b, a];
```
可以想象，JavaScript 引擎在未来将会针对这个模式进行特别优化，去掉构造数组的开销。
更多内容:
章节: ["Destructuring"](https://exploringjs.com/es6/ch_destructuring.html)

### 通过模板字面量（template literals）进行简单的模板解析
ES6 的模板字面量与文字模板相比，更接近于字符串字面量。但是，如果你将它们通过函数返回，你可以使用他们来做简单的模板渲染：
```
const tmpl = addrs => `
    <table>
    ${addrs.map(addr => `
        <tr><td>${addr.first}</td></tr>
        <tr><td>${addr.last}</td></tr>
    `).join("")}
    </table>
`;
```
tmpl 函数将数组 addrs 用 map（通过箭头函数） join 拼成字符串。tmpl() 可以批量插入数据到表格中：
```
const data = [
    { first: "<Jane>", last: "Bond" },
    { first: "Lars", last: "<Croft>" },
];
console.log(tmpl(data));
// Output:
// <table>
//
//     <tr><td><Jane></td></tr>
//     <tr><td>Bond</td></tr>
//
//     <tr><td>Lars</td></tr>
//     <tr><td><Croft></td></tr>
//
// </table>
```
更多内容:
博客文章: ["Handling whitespace in ES6 template literals"](http://www.2ality.com/2016/05/template-literal-whitespace.html)
段落: ["Text templating via untagged template literals"](https://exploringjs.com/es6/ch_template-literals.html#_text-templating-via-untagged-template-literals)
章节: ["Arrow functions"](http://exploringjs.com/es6/ch_arrow-functions.html)

### 通过子类工厂实现简单的合成器
当 ES6 类继承另一个类，被继承的类可以是通过任意表达式创建的动态类：
```
// Function id() simply returns its parameter
const id = x => x;

class Foo extends id(Object) {}
```
这个特性可以允许你实现一种合成器模式，用一个函数来将一个类 C 映射到一个新的继承了C的类。例如，下面的两个函数 Storage 和 Validation 是合成器：
```
const Storage = Sup => class extends Sup {
    save(database) { ··· }
};
const Validation = Sup => class extends Sup {
    validate(schema) { ··· }
};
```
你可以使用它们去组合生成一个如下的 Employee 类：
```
class Person { ··· }
class Employee extends Storage(Validation(Person)) { ··· }
```
更多内容:
段落: ["Simple mixins"](https://exploringjs.com/es6/ch_classes.html#_simple-mixins)

### 进一步阅读
下面的两个章节提供了很好地概括了 ECMAScript 6 的特性：
["An overview of what’s new in ES6"](https://exploringjs.com/es6/ch_overviews.html)
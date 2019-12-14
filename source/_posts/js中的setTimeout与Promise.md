---
title: js中的setTimeout与Promise
date: 2019-12-14 08:34:48
tags: JS
categories:
    - 前端
---

> 本篇博客仅以个人学习记录使用

### 介绍

Promise 是在 ES5 之后出现的，这是属于 JavaScript 引擎本身的能力，由 JavaScript 引擎发起的任务被称作微观任务。Promise 是 JavaScript 语言提供的一种标准化的异步管理方式，当需要 IO、等待或者其他异步操作的函数，不直接返回真实结果，而返回一个 Promise,
函数的调用方可以在合适的时机，选择等待这个 Promise 兑现（通过 Promise 的 then 方法的回调）

Promise 的基本用法示例如下：

```
    function sleep(duration) {
        return new Promise(function(resolve, reject)) {
            setTimeout(resolve,duration);
        }
    }

    sleep(1000).then( () => console.log('完成'))
```

Promise 的 then 回调是一个异步的执行过程，下面来研究一下 Promise 函数中的执行顺序：

```
let r = new Promise(function(resolve.reject) {
    console.log("a");
    resolve();
});
r.then( () => console.log("c"));
console.log("b")
```

执行这段代码之后，输出的顺序是 a,b,c。在进入 cpnsole.log(b)之前，毫无疑问 r 已经得到了 resolve,但是 Promise 的 resolve 始终是异步操作，所以 c 无法出现在 b 之前。

setTimeout 是浏览器的 api，是在 window 下的，是宿主环境中的，他被称作宏观任务。

那么，如果将 setTimeout 混用 Promise 会怎样呢？？

在下面的代码中，设置了两段互不相干的异步操作：通过 setTimeout 执行 console.log("d"),通过 Promise 执行 console.log("c").

```
let r = new Promise(function(resolve,reject) {
    console.log("a");
    resolve()
})

setTimeout( () => console.log("d"))

r.then( () => console.log("c"))
console.log("b")
```

不论代码顺序如何，d 必定发生在 c 之后，因为 Promise 产生的是 JavaScript 引擎内部的微任务，而 setTimeout 是浏览器 api,它产生宏任务，而微任务始终优先于宏任务。举例代码：

```
setTimeout( () => console.log("d"), 0)
let r = new Promise(function(resolve,reject){
    resolve()
});
r.then(() => {
    let begin = Date.now();
    while(Date.now() - begin < 1000);
    console.log("c1")
    new Promise(function(resolve,reject) {
        resolve();
    }).then( () => console.log("c2"));
});
```

以上代码强制了 1 秒的执行耗时，这样，可以确保任务 c2 是在 d 之后被添加到任务队列。但是即便是这样，先执行了 c1,由执行了 c2,d 仍然是最后执行的，这就很好的解释了微任务优先的原理。

通过以上的分析，总结一下如何分析异步执行的顺序： 1.首先分析有多少个宏观任务； 2.在每个宏观任务中，分析有多少个微观任务； 3.根据调用次序，确定宏任务中的微任务执行次序； 4.根据宏任务的触发规则和调用次序，确定宏任务的执行次序； 5.确定整个顺序。

这是一个稍微复杂的例子;

```
    function sleep(duration) {
        return new Promise(function(resolve,reject) {
            console.log("b");
            setTimeout(resolve,duration);
        })
    }
    console.log("a");
    sleep(5000).then( () => console.log("C"));
```

这是一段分非常常用的封装方法，利用 Promise 把 setTimeout 封装成可以用于异步的函数。我们首先来看，setTimeout 把整个代码分割成了 2 个宏观任务，这里不论是 5 秒还是 0 秒，都是一样的。第一个宏观任务中，包含了先后同步执行的 console.log("a");和 console.log("b")
setTimeout 后，第二个宏观任务执行调用了 resolve，然后 then 中的代码异步得到执行，所以调用了 console.log("c"),最终输出的顺序才是 a,b,c.

```
setTimeout(function() {
	console.log(1);
})

new Promise(function(resolve, reject) {
    console.log(2);
    resolve(3)
}).then((res) => {
    console.log(res)
})
console.log(4);

2
4
3
1
```

原因：
首先 setTimeout 会在下一轮时间循环执行，所以不会当时就打印。
Promise 对象在实例的时候其实就已经执行了内部的代码，所以 2 首先打印了。
Promise.then() 在本轮事件循环结束之后执行，所以 3 会在 4 之后打印。
本轮事件循环结束，开始下一循环打印 setTimeout 中的 1


```
 async function async1() {
            console.log("async1 start");
            await  async2();
            console.log("async1 end");
 
        }
        async  function async2() {
           console.log( 'async2');
        }
        console.log("script start");
        setTimeout(function () {
            console.log("settimeout");
        },0);
        async1();
        new Promise(function (resolve) {
            console.log("promise1");
            resolve();
        }).then(function () {
            console.log("promise2");
        });
        console.log('script end');


script start
async1 start
async2
promise1
script end
promise2
async1 end
settimeout
```

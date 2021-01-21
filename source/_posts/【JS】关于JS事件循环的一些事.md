---
title: 【JS】关于JS事件循环的一些事
date: 2020-10-25 20:36
---

事情是这样的，看到一个关于事件循环的题,这道题的主要考察点是事件循环中函数执行顺序的问题，其中包括`async`、`await`、`setTimeout`、`promise`函数。(答案请见：[解题步骤](#回到本题))
<!-- more -->
```javascript
//请写出输出内容
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
	console.log('async2');
}

console.log('script start');

setTimeout(function() {
    console.log('setTimeout');
}, 0)

async1();

new Promise(function(resolve) {
    console.log('promise1');
    resolve();
}).then(function() {
    console.log('promise2');
});
console.log('script end');
```
下面来剖析一下这道题涉及到的知识点：

---

### 任务队列
首先我们需要明白以下几件事：

> - JS分为同步任务和异步任务
> - 同步任务都在主线程上执行，形成一个执行栈
> - 主线程之外，事件触发线程管理着一个任务队列，只要异步任务有了执行结果，就会在任务队列里加入一个事件
> - 一旦执行栈中的同步任务都执行完毕（此时JS引擎空闲），系统就会读取任务队列，将可执行的异步任务添加到可执行栈中，开始执行

根据规范，事件循环是通过任务队列来协调的，一个Event Loop中，可以有一个或者多个任务队列（Task Queue），一个任务队列就是一系列有序任务的集合；**每个任务都有一个任务源（task source），属于同一个任务源的任务必须放到同一个任务队列中去。**setTimeout/Promise等API便是任务源，**进入任务队列的，是他们指定的具体执行任务。**

---

### 宏任务
(macro)task，又称之为宏任务，可以理解为**每次执行栈执行的代码就是一个宏任务**，包括从事件队列中取出的一个事件回调并放到执行栈中执行。
**浏览器为了让宏任务和DOM任务有序执行，在一个宏任务结束之后，下一个宏任务开始之前，会重新渲染一次页面**
> (macro)task主要包括：script(整体代码)、setTimeout、setInteval、I/O、UI交互事件、postMessage、MessageChannel、setImmidiate(Node.js环境)。


---

### 微任务
microtask，又称为微任务，可以理解为**在当前task执行结束之后立刻执行的任务**，也就是说，_在当前task结束后，页面渲染开始前。_
所以他的响应速度比setTimeout（setTimeout是task）会更快，因为无需等待渲染。
> microtask主要包含：Promise.then、MutaionObserver、process.nextTick(Node.js环境)。


---

### 运行机制
在事件循环中，每一次循环操作称为tick，每一次tick的任务处理模型关键步骤如下：

1. 执行一个宏任务（执行栈中没有就从事件队列中调取）
1. 执行过程中如果碰到微任务(microtask)，就把他加入到**微任务**的事件队列中
1. 宏任务执行完毕之后，立即执行**微任务事件队列**中的所有微任务，依次执行
1. 当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染
1. 渲染完毕后，JS开始执行下一个宏任务（从事件队列中获取）

---

### Promise和async中的立即执行
Promise中的异步体现在`.then`和`.catch`中，所以写在Promise中的代码是被当作同步任务立即执行的。而在async/await中，在await出现之前，其中的代码也是被立即执行的。那么**出现await之前发生了什么？**
#### await做了什么？
从字面意思上看await就是等待，await等待的是一个表达式，这个表达式的返回值可以是一个Promise对象也可以是其他值。
**实际上await是一个让出线程的标志。await后面的表达式会先执行一遍，将await后面的代码加入到macrotask中，然后就跳出整个async函数去执行后面的代码。**
_**await后面的代码是microtask**_

---

### 回到本题
详细步骤请看原文链接：[详细步骤](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/7)
下面给出我的解题步骤：
> 输出：
> 1、script start
> 3、async1 start
> 4、async2
> 6、promise1
> 8、script end
> 宏任务列表：
> 2、setTimeout
> microtask：
> Promise microtask列表：
> 5、async1 end
> 7、promise2

1，2，3，···，8是程序执行的步骤，其中2是将setTimeout加入到宏任务列表，5和7是将两个微任务加到对应的微任务列表。
第8步的时候宏任务script已经执行结束，按照上面说的，一个宏任务结束之后会先按顺序执行微任务列表中的所有微任务，也就是输出5和7，再然后，就是检查渲染页面，渲染之后就是开始执行下一个宏任务，也就是从宏任务列表中将2执行，所以最终结果是：
```javascript
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

---

### 最后来个练习题：
```javascript
async function a1 () {
    console.log('a1 start')
    await a2()
    console.log('a1 end')
}
async function a2 () {
    console.log('a2')
}

console.log('script start')

setTimeout(() => {
    console.log('setTimeout')
}, 0)

Promise.resolve().then(() => {
    console.log('promise1')
})

a1()

let promise2 = new Promise((resolve) => {
    resolve('promise2.then')
    console.log('promise2')
})

promise2.then((res) => {
    console.log(res)
    Promise.resolve().then(() => {
        console.log('promise3')
    })
})
console.log('script end')

// 答案是：
script start
a1 start
a2
promise2
script end
promise1
a1 end
promise2.then
promise3
setTimeout
```
对其中的Promise可以查看这一篇文章：[关于JS中的Promise](https://www.jianshu.com/p/b16e7c9e1f9f).

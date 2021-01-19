---
title: 【JS】聊一聊定时器setTimeout和setInterval
date: 2020-07-19 18:13
update: 2020-07-19 22:10
---
写JS的时候经常会用到定时器，写这篇文章的目的是详细说明一下定时器的使用方法以及使用场景，以及一些注意事项。
<!-- more -->

### 一、定时器的介绍
Windows对象提供了两种定时器方法，分别是：

```javascript
window.setTimeout(code, millisec) # 第一种
window.setInterval(code, millisec) # 第二种
```
其中，
第一种的作用是：**让code等待millisec时间后运行**，
第二种的作用是：**每隔millisec时间就运行一次code**。

里边的`code`可以是用引号括起来的一段代码，也可以是一个函数名。这里需要注意的是：
> 如果使用函数名作为code的部分，则里边一定不能带参数，如果想要带参数的话，只能是使用字符串形式。
下面用代码说明：

```javascript
// test是一个带参数的函数
function test(param) {
	console.log(param)
}

// 如果想让test等待3秒后执行，应该这么写
setTimeout('test('正确的写法')', 3000)
// 可以看到 setTimeout 的第一个参数是一个字符串的形式，而这个字符串其实是一个带参数的函数

// 如果像下面这么写的话，test 则会立即执行
setTimeout(test('错误的写法'), 3000)
```
可以这么来记忆：
> 函数名，不带参；
> 字符串，放 可执行；
> 匿名函数直接写

### 二、清除定时器
定时器在调用的时候会返回一个整数，代表了这个定时器的序号。方便我们清除这个定时器⏲️。

调用是 set， 那么清除的时候很显然就是 clear 了。
至于为什么要清除定时器，原因挺多的，有时候是因为如果不清楚，那么线程空闲的时候他还会再执行一次。像setInterval这种还会不停的去执行，所以根据项目实际的需求来决定怎么去删除定时器。

```javascript
window.clearTimeout(obj) # 清除第一种定时器🆑
window.clearInterval(obj) # 清除第二种定时器🆑
```

*有时候实际写的时候还喜欢给存储定时器的变量置空，这样可以释放内存，也方便后面代码的判断*
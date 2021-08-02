---
title: 【JS】Async-await和Promise的区别
date: 2020.10.21 18:30
update: 
---
> Async/await 是建立在Promise上的，不能被使用在普通回调以及节点回调
Async/await 和 Promise 很像，不阻塞
Async/await 代码看起来很像同步代码。

## 语法
假设`getJSON`返回值是`Promise`，并且`Promise resoles`有一些JSON对象，我们只想调用他并且记录JSON并返回完成。
**使用Promise**
```(javascript)
const makeRequest = () =>
  getJSON()
    .then(data => {
      console.log(data)
      return "done"
    })

makeRequest()
```
**使用Async**
```(javascript)
const makeRequest = async() => {
        console.log(await getJSON)
        return "done"
}

makeRequest()
```

**区别**
1. 在函数前有一个关键字`async`，`await`关键字只能在使用`async`定义的函数中使用。任何一个`async`函数都会隐式返回一个`promise`，并且`promise resolve`的值就是 `return` 返回的值 (例子中是`done`)。
2. 不能在函数开头使用`await`

## 有哪些好处
1. 简洁的代码，使用`Async`可以让代码变简洁很多，不需要像`Promise`一样需要使用`then`
比如条件语句，对比一下两种方式的写法：
**使用Promise**
```(javascript)
const makeRequest = () => {
  return getJSON()
    .then(data => {
      if (data.needsAnotherRequest) {
        return makeAnotherRequest(data)
          .then(moreData => {
            console.log(moreData)
            return moreData
          })
      } else {
        console.log(data)
        return data
      }
    })
}
```
**使用Async**
```(javascript)
const makeRequest = async () => {
  const data = await getJSON()
  if (data.needsAnotherRequest) {
    const moreData = await makeAnotherRequest(data);
    console.log(moreData)
    return moreData
  } else {
    console.log(data)
    return data    
  }
}
```
2. 中间值
在一些场景中，也许需要 promise1 去触发 promise2 再去触发 promise3，这个时候代码应该是这样的：
```(javascript)
const makeRequest = () => {
  return promise1()
    .then(value1 => {
      // do something
      return promise2(value1)
        .then(value2 => {
          // do something          
          return promise3(value1, value2)
        })
    })
}
```
使用`Async`就会变得很简单：
```(javascript)
const makeRequest = async () => {
  const value1 = await promise1()
  const value2 = await promise2(value1)
  return promise3(value1, value2)
}
```
3. 错误栈
如果`Promise`连续调用，对于错误的处理是很麻烦的。因为无法知道错误出在哪里，像这样：
```(javascript)
const makeRequest = () => {
  return callAPromise()
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => callAPromise())
    .then(() => {
      throw new Error("oops");
    })
}

makeRequest()
  .catch(err => {
    console.log(err);
    // output
    // Error: oops at callAPromise.then.then.then.then.then (index.js:8:13)
  })
```
但是对于`Async`就简单许多：
```(javascript)
const makeRequest = async () => {
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  await callAPromise()
  throw new Error("oops");
}

makeRequest()
  .catch(err => {
    console.log(err);
    // output
    // Error: oops at makeRequest (index.js:7:9)
  })
```

原文链接：[Async/await 和 Promises 区别](https://segmentfault.com/a/1190000013612116)
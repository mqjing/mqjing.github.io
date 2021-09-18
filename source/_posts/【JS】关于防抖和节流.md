---
title: 【JS】关于防抖和节流
date: 2021.08.03 18:55
---

本篇文章主要是参考的这位老哥的文章：[浅谈 JS 防抖和节流](https://segmentfault.com/a/1190000018428170/)。<br />
<br />防抖和节流严格算起来应该属于性能优化的知识，但实际上遇到的频率相当高，处理不当或者放任不管就容易引起浏览器卡死。<br />

<a name="OzzNu"></a>
## 场景定义
先说一个常见的功能，很多网站会提供这么一个按钮：用于返回顶部。

这个按钮只会在滚动到距离顶部一定位置之后才出现，那么我们现在抽象出这个功能需求 -- **监听浏览器滚动事件，返回当前滚条与顶部的距离。**
```javascript
function showTop() {
	let scollTop = document.body.scrollTop || document.documentElement.scrollTop;
  console.log('滚动条位置：'+scollTop)
}
window.onscroll = showTop
```
】但是这段代码一旦运行你就会发现，这个函数执行的频率太高了，点击一次键盘上的向下按钮甚至就会执行8-9次。<br />

<a name="WeIGh"></a>
## 防抖（debounce）
针对上面的场景，首先提出第一种思路：**在第一次触发函数的时候，不立即执行，而是给出一个延时，比如200ms，如果200ms内函数又被调用，则不执行函数且重新开始计时。**
```javascript
function debounce(fn, delay){
	let timer = null
  return function() {
  	if (timer) {
      clearTimeout(timer)
    }
    timer = setTimeout(fn, delay)
  }
}
```
最后给出防抖的定义：**对于短时间内连续触发的事件，防抖的作用就是限制其在给定的时间内只执行一次。**
<a name="XddJJ"></a>
## 节流（throttle）
继续思考，使用上面防抖的解决方案可能会导致一个问题：<br />

> 如果有个用户闲着无聊，不停的滚动页面，导致一直在重新计时，但是产品又有个需求是即使用户不断滚动页面，也会在某个时间间隔之后给出反馈。（虽然这个需求我看起来相当的不合理，仅用来讨论问题）


<br />实现方案可以是这样：**借助**`setTimeout`，加入一个`valid`位来表示是否是工作状态，如果不是工作状态，直接返回；反之，就设置一个`setTimeout`并且将`valid`设置为非工作状态。<br />
<br />代码实现可以如下：
```javascript
function throttle(fn, delay){
	let valid = true
  return function() {
    // 休息时间，直接返回
  	if (!valid) return
    // 工作时间，执行函数，并在间隔期内把状态位设为无效。**注意在执行完之后要将状态改为工作**
    valid = false
    setTimeout(()=>{
    	fn()
      valid = true
    }, delay)
  }
}
```
<a name="z0HBc"></a>
## 其他应用场景举例
平时开发中常遇到的情形

1. 搜索框input事件，例如要支持实时搜索，这时候可以使用节流方案（间隔一段时间就必须查询相关内容），或者实现输入间隔大于某个值（如500ms），就当作用户输入完成，然后开始搜索。
1. 页面resize事件，常见于需要做页面适配的时候。需要根据最终呈现的页面进行dom渲染。（这个情形一般是使用防抖，因为只需要判断最后一次变化的情况。）

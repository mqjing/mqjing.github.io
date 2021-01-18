---
title: 【JS】简单聊一下prototype
date: 2020-06-10 19:37
update: 2020-06-11 19:37
description: 通俗来讲：我们创建的每一个函数都有一个prototype属性，这个属性指向了一个对象，这个对象包含了一些共享的属性和方法，通过调用该构造函数所创建的对象会“继承”这些属性和方法。
---
## 什么是prototype
通俗来讲：我们创建的每一个函数都有一个prototype属性，这个属性指向了一个对象，这个对象包含了一些共享的属性和方法，通过调用该构造函数所创建的对象会“继承”这些属性和方法。
## prototype是如何工作的
JavaScript 是基于原型的语言。当我们调用一个对象的属性时，如果对象没有该属性，JavaScript 解释器就会从对象的原型对象上去找该属性，如果原型上也没有该属性，那就去找原型的原型，直到最后返回null为止，null没有原型。这种属性查找的方式被称为原型链（[prototype chain](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)）。

原型对象的优点是：所有的对象实例都可以共享它包含的属性和方法。这一点可以在构造函数里就可以看出来，因为构造函数在函数里面就定义了对象的实例信息，而原型对象可以在任何地方定义属性和方法。
## 一个🌰
**创建自定义类型最常见的方式就是组合使用 构造函数模式 和 原型模式。构造函数模式用于定义实例属性；**
**原型模式用于定义方法和共享的属性。**这样的好处是每个实例都会有一份属于自己的属性副本，但同时又共享着对方法的引用，最大限度的**节省了内存**。
```javascript
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.love = ['qiqi', 'lili'];
}
Person.prototype = {
  constructor: Person,
  dream: function () {
    console.log(this.love[0]);
  }
}
var person1 = new Person('bangbang', 18, 'programmer');
var person2 = new Person('xiaoya', 18, 'teacher');
console.log(person1.love) // ["qiqi", "lili"]
console.log(person2.love) // ["qiqi", "lili"]
console.log(person1.love === person2.love);  //false
//给person1的love属性添加元素
person1.love.push('niuniu');
console.log(person1.love)  // ["qiqi", "lili", "niuniu"]
console.log(person2.love)  // ["qiqi", "lili"]
console.log(person1.love === person2.love);  //false
console.log(person1.dream === person2.dream);//true
```
## 有那么·意思
```javascript
// 这行代码的输出结果是true，是不是很神奇😜
Array.prototype.__proto__.toString === Object.prototype.toString 
// 那么再猜猜这行呢😼
Number.prototype.__proto__.toString === Object.prototype.toString 
```
参考文章：
[深入理解prototype](https://blog.csdn.net/flyingpig2016/article/details/53048394)
[理解JS的prototype](https://zhuanlan.zhihu.com/p/35458229)

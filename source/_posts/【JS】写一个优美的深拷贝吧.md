---
title: 【JS】写一个优美的深拷贝吧
date: 2020-08-18 23:11
update: 2020-08-18 23:21
description: 一个闪瞎眼的深拷贝写法，不看后悔一辈子🏷️
---

先说明一下为什么会有这个问题，数据类型分为**原始类型**（number、string等）和**引用类型**（比如数组）。这就意味着：当你把一个数组赋值给一个变量的时候，你是将**数组的地址而非数组本身**赋给了变量。
先给出深拷贝和浅拷贝的定义：
> **浅拷贝**：创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型；如果属性是引用类型，拷贝的就是引用类型，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。
> **深拷贝**：将一个对象从内存中完整的拷贝出来一份，从堆内存中开辟一个新的区域存放新对象，且修改新对象不会影响原对象。

这就导致了很多问题，我们先看个最简单的问题，直接看下面的代码：
```javascript
a = [1, 2, 3, 4]
b = a
b[2] = 8
```
这样执行完之后，a 的值也变成了`[1, 2, 8, 4]`。
所以如果想解决这个问题，就用到了数组的深拷贝。怎么去实现深拷贝呢？有几种方法，我们先看第一种：
**方法一：**用JSON的`parse`方法和`stringify`方法
```javascript
a = [1, 2, 3, 4]
b = JSON.parse(JSON.stringify(a))
```
这个方式就是用`JSON.stringify()`将js对象序列化（JSON字符串），再使用`JSON.parse()`来还原js对象。这样做的弊端或者说坏处有以下几个：

1. 如果obj里有时间对象，那么经过这个方式拷贝后，时间将只是字符串形式，而不是时间对象。
1. 如果obj里有**RegExp**，**Error**对象，则序列化之后只能得到空对象。
1. 如果obj里有函数、undefined，则序列化的结果就会把函数或undefined丢失。
1. 如果obj里有NaN、Infinity、和-Infinity，则序列化的结果会变成null。
1. 如果obj里有构造函数生成的对象，那么使用这种方式深拷贝之后会求其对象的constructor。
1. 如果obj里存在循环引用的情况也无法正确实现深拷贝。

**_但是话说回来，这种方法在实际使用中还是比较常用的，只是要注意一下，要深拷贝的对象不包括以上的6种情况。_**

---

接下来，看一个闪瞎眼的深拷贝写法，参考的原文链接是这个：[如何写出一个惊艳面试官的深拷贝](https://blog.csdn.net/weixin_44363885/article/details/100933786)。
源代码也引用过来了，并且给他加上了注释，来享用吧：
```javascript
// 可继续遍历的数据类型
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';
const argsTag = '[object Arguments]';
// 不可继续遍历的数据类型
const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const numberTag = '[object Number]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
const errorTag = '[object Error]';
const regexpTag = '[object RegExp]';
const funcTag = '[object Function]';

const deepTag = [mapTag, setTag, arrayTag, objectTag, argsTag];

// 工具函数1  通用while循环
function forEach(array, iteratee) {
    let index = -1;
    const length = array.length;
    while (++index < length) {
        iteratee(array[index], index);
    }
    return array;
}

// 工具函数2  判断是否为引用类型
function isObject(target) {
    const type = typeof target;
    return target !== null && (type === 'object' || type === 'function');
}

// 工具函数3  获取实际类型
function getType(target) {
    return Object.prototype.toString.call(target);
}

// 工具函数4  初始化被克隆的对象
function getInit(target) {
    const Ctor = target.constructor;
    return new Ctor();
}

// 工具函数5  克隆symbol
function cloneSymbol(targe) {
    return Object(Symbol.prototype.valueOf.call(targe));
}

// 工具函数6  克隆正则
function cloneReg(targe) {
    const reFlags = /\w*$/;
    const result = new targe.constructor(targe.source, reFlags.exec(targe));
    result.lastIndex = targe.lastIndex;
    return result;
}

// 工具函数7  克隆函数
function cloneFunction(func) {
    const bodyReg = /(?<={)(.|\n)+(?=})/m;
    const paramReg = /(?<=\().+(?=\)\s+{)/;
    const funcString = func.toString();
    if (func.prototype) {
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            if (param) {
                const paramArr = param[0].split(',');
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        return eval(funcString);
    }
}

// 工具函数8  克隆不可遍历类型
function cloneOtherType(targe, type) {
    const Ctor = targe.constructor;
    switch (type) {
        case boolTag:
        case numberTag:
        case stringTag:
        case errorTag:
        case dateTag:
            return new Ctor(targe);
        case regexpTag:
            return cloneReg(targe);
        case symbolTag:
            return cloneSymbol(targe);
        case funcTag:
            return cloneFunction(targe);
        default:
            return null;
    }
}
// 原始类型直接返回
function clone(target, map = new WeakMap()) {

    // 克隆原始类型
    if (!isObject(target)) {
        return target;
    }

    // 初始化
    const type = getType(target);
    let cloneTarget;
    if (deepTag.includes(type)) {
        cloneTarget = getInit(target, type);
    } else {
        return cloneOtherType(target, type);
    }

    // 防止循环引用
    if (map.get(target)) {
        return map.get(target);
    }
    map.set(target, cloneTarget);

    // 克隆set
    if (type === setTag) {
        target.forEach(value => {
            cloneTarget.add(clone(value, map));
        });
        return cloneTarget;
    }

    // 克隆map
    if (type === mapTag) {
        target.forEach((value, key) => {
            cloneTarget.set(key, clone(value, map));
        });
        return cloneTarget;
    }

    // 克隆对象和数组
    const keys = type === arrayTag ? undefined : Object.keys(target);
    forEach(keys || target, (value, key) => {
        if (keys) {
            key = value;
        }
        cloneTarget[key] = clone(target[key], map);
    });

    return cloneTarget;
}

module.exports = {
    clone
};
```
一份Dior炸天的深拷贝代码就写好了。



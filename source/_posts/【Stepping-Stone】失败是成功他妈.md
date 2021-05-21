---
title: 【Stepping-Stone】失败是成功他妈
date: 2021.05.17 16:27
update: 
---
> 这篇文章主要是记录工作过程中遇到的一些问题，以及解决方案，本来想单独开一个仓库用来记录，后来想想好像也没必要，就用一篇长文章的方式来记录吧，后面会根据时间来作为索引。

## 2021-05-17 
### `JSON.stringify` 和 `qs.stringify` 的区别

qs可以通过`npm install qs`命令进行安装，是一个npm仓库管理的包。
`qs.stringify()`主要功能是将对象序列化成URL的形式，以`&`进行拼接。例如：
```(javascript)
var a = {name: 'jing', age: 18}
qs.stringify(a)
// 'name=jing&age=18'
```
而JSON是正常类型的JSON，
```(javascript)
JSON.stringify(a)
// '{"name":"jing","age":18}'
```

## 2021-05-18
### JS中查看数据类型的方法

#### JS中的数据类型
JS是一种弱类型语言，一般说来，JS有7种数据类型，包括三种基本类型：`Number、String、Boolean`，两种引用数据类型：`Object、Array`，以及两种特殊数据类型：`null、undefined`，es6中又新增了一种叫`Symbol`的数据类型，他的最大特点是**唯一性**，具体不展开说了，详情参考这个链接：[关于es6的Symbol数据类型](https://segmentfault.com/a/1190000018033214)。

#### 如何判断JS中的数据类型
不同类型的值所代表的布尔值情况：
· 空字符串： false
· null： false
· undefined： false
· 0: false
· NaN： false

除了上面的情况，其他情况都是true

##### 方法一：使用`typeOf`
```(javascript)
typeof('string') // string
typeof(123) // number
typeof(()=>{}) // function
typeof([1,2,3]) // object
typeof({a:1,b:2}) // object
typeof(null) // object
typeof(undefined) // undefined
typeof(true) // boolean
```
其中，`数组类型`和`对象类型`统一被划分成了`对象`，所以如果像区分数组和对象的话，需要用方法二和三。
其中，比较意外的是`null`，要记住他是一个空对象.

##### 方法二：使用`instanceof`
**instanceof只能用来判断数组或对象，其中数组类型的也是对象**，具体用法：
```(javascript)
let arr = [1,2,3,4]
let obj = {a:1, b:2}
arr instanceof Array // true
arr instanceof Array // true
obj instanceof Object // true
obj instanceof Array // false
```

##### 方法三：使用`constructor`
```(javascript)
let arr = [1,2,3,4]
let obj = {a:1, b:2}
arr.constructor == Object // false
arr.constructor == Array // true
obj.constructor == Object // true
```
##### 方法四：使用`toString`方法，此方法较佳
```(javascript)
let arr = [1,2,3,4]
let obj = {a:1, b:2}
Object.prototype.toString.call(123) // "[Object Number]"
Object.prototype.toString.call(arr) // "[Object Array]"
Object.prototype.toString.call(obj) // "[Object Object]"
```
利用上面的方法就可以封装一个函数，判断结果是是那个从而返回不同的结果。

## 2021-05-21
### VUE+element 组件中回调函数传递额外参数

问题是这样的，我想在组件中自带的回调函数函数中传递我自己的参数，但是发现用直接绑定的方式没办法传递，经过网上查找，发现可以用箭头函数的方式，即箭头函数里传原生参数，在箭头函数内调用的函数加入自己的参数。
示例代码：
```(javascript)
<!-- 这里的value是回调函数自带的参数，代表了输入框的值，后面的两个参数是自己加的 -->
<el-input @change="(value) => {handleChange(value, stepIndex, propIndex)}"> 
```


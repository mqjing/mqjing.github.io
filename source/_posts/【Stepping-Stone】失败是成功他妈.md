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

## 2021-05-23
### textarea 换行输入的问题

项目中又一个需求，是这样的：
> 1. 将数组中的字符串分行放到textarea中；2. 将textarea中的数据按行保存到一个数组中，即每一行都是一个数组元素

通过将textarea中的数据打印出来查看，发现数据是保留了换行信息的，所以解决这个问题的关键是用正则表达式，`/[(\r\n)\r\n]+/`这个正则表达式会按照换行或者回车进行识别，那么就好办了。详情看下面代码：

```(javaScript)
// 假设从textarea中获取回来的数据是这个
let value = `
aa
bb
cc
dd
`
// 接下来用split函数将上面的数据进行分割
let res = value.split(/[(\r\n)\r\n]+/)

// res = ['aa','bb','cc','dd']
```
这样就完成了，同理，要将数组换行显示到textarea组件中也只需要`res.join(/[(\r\n)\r\n]+/)`即可。
之前对正则表达式没什么好感，虽然也会经常用到，但就是提不起兴趣，解决完了这个问题之后，感觉要去研究一下了，最好结合自己的理解写一篇文章出来。

## 2021-06-04
### 判断对象是否是空对象的问题

最近真的是忙到爆炸，攒了好几个问题在笔记本上，没时间来整理，这会抽点空赶紧写一下。

问题背景是在实际应用中经常会用到对象和数组判空，如果不进行判空的话可能会导致报错，但是对象不像数组可以直接`.length===0`就很麻烦，有几种方法这里比较一下吧

· 方法一 将json对象转化为json字符串，再判断该字符串是否为"{}" -- 此方法写的代码较长，难看
```(javascript)

let data = {};

let b = (JSON.stringify(data) != "{}");

alert(b); //false

```

· 方法二 写个函数用遍历的方法，如果有元素返回true，没有返回false -- 此方法还可以，写在util-tools.js里直接引用过去用就行了
```(javascript)

let data = {};

let b = function(data) {
  for (let key in data) {
    return true
  }
  return false
};

alert(b(data));//false

```

· 方法三 使用`Object.getOwnPropertyNames()`方法

此方法会获取到对象中的属性名，并将其存储到一个数组中，然后再去判断这个数组的长度是否为零就行了，代码也太长，放弃
```(javascript)

let data = {};

let arr = Object.getOwnPropertyNames(data)

alert(arr.length !== 0)); //false

```

· 方法四 使用ES6的`Object.keys()`方法 

跟方法三类似，也是返回一个数组，好处是函数名短了不少，
```(javascript)

let data = {};

let arr = Object.keys(data)

alert(arr.length !== 0)); //false

```

## 2021-06-05
### vue项目中换行会被解析成空格

这个也是实际项目中用到的，背景是后端会返回给我一个字符串，这个字符串里有各种转义符号，如果想让这些转义符号按照html的格式显示出来只需要在标签上加一个`v-html`就好了，但是同时又发现只加`v-html`，他会将换行符比如`\n`等解析为空格，后来通过查询，发现解决这个问题也很简单，只需要借助css中的一个属性，那就是:

```(css)
white-space: pre-wrap
```

同理，如果不想看到换行的话，就可以用
```(css)
white-space: no-wrap
```
---
title: 【Stepping-Stone】失败是成功他妈
date: 2021.05.17 16:27
update: 
---
> 这篇文章主要是记录工作过程中遇到的一些问题，以及解决方案，本来想单独开一个仓库用来记录，后来想想好像也没必要，就用一篇长文章的方式来记录吧，后面会根据时间来作为索引。

## 2021-05-17 
### JSON.stringify 和 qs.stringify 的区别

qs可以通过`npm install qs`命令进行安装，是一个npm仓库管理的包。
`qs.stringify()`主要功能是将对象序列化成URL的形式，以**&**进行拼接。例如：
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
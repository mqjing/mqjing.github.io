---
title: 【JS】关于Date对象
date: 2021.07.17 16:27
update: 
---

关于Date对象的一些用法整理
<!-- more -->


这个问题的来源还是项目中的一个需求：在时间选择器的快捷选项中加入本周、本月、本年等功能。Element UI中提供的例子都是“最近一周”、“最近一个月”这种类型的。代码如下：
```(javascript)
{
  text: '最近一个月',
  onClick(picker) {
    const end = new Date()
    const start = new Date()
    start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
    picker.$emit('pick', [start, end])
  }
}
```
核心代码是`start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)`，也就是用setTime()函数将开始时间设置为现在的时间减去30天的毫秒数。

那么，我如果想要获取“本月”的日期范围该怎么做呢？“本月”跟“最近一个月”的区别就是本月是从当月的第一天到目前的日期，问题的关键就是获取本月的第一天。通过查资料，发现了这个方法：
```(javascript)
// 获取本月的开始日期
const now = new Date() // 当前日期
const nowYear = now.getFullYear() // 当前年份
const nowMonth = now.getMonth() // 当前月份
const nowDayOfWeek = now.getDay() // 今天是本周的第几天

const monthStartDay = new Date(nowYear, nowMonth, 1) // 本月的开始日期
const weekStartDate = new Date(nowYear, nowMonth, nowDay - nowDayOfWeek + 1) // 本周的开始日期
```


解决问题的核心就是Date构造函数可以接收一个日期序列，年、月、日，例如：`new Date(year, month, day, hours, minutes, seconds, milliseconds)`

关于`Date()`对象更详细的用法可以参考：[JavaScript 内置对象（二）：Date 对象（构造函数、属性和方法）](https://blog.csdn.net/jinshi_cn/article/details/2434439)
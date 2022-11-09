---
title: undefined与null的区别，以及数据类型检测的方法
---

## 1.`undefined`与`null`的区别

由来，为什么只有`JS`才有undefined这种数据类型呢

`JS`作者在设计时也是先设计了`null`,用来表示无的含义，根据高级编程语言的传统，`NULL`在进行转化时，会自动转化成对象类型，转换成数字就会变0，作者认为，这并不符合用于表示“无”的含义，作者认为这个“无”的类型不应该是一个对象，也不应该存在值，也就诞生了`undefined`作为一个单独的类型，并且转换成数字类型也是`NAN`

这也就了解`undefined`与`null`的区别

- 字面含义: `undefined`表示未定义，`NULL`表示空对象
- 转换成数字类型:`undefined`转化成NAN，`NULL`转化成0
- 转化字符串也就是字面字符串，注意的是 undefined==null； 因为这里存在强制转换也就自然相等

## 2.类型检测的一些方法

1. `typeof`对基本类型检测都没啥大问题，但是对对象类型检测就只会返回 `Object`
2. `instanceof`,此方法对数据检测非常严格，适用于精准检测引用数据类型，检测是否出现原型链中
3. **`Object.prototype.toString.call()`万能方法**：从使用中我们也能看出，将原型上的this指向要被检测的数据类型，然后转化成字符串，也就能检测所有的字符串类型

这里在体积以下数据类型

基本数据类：字符串，数字，布尔类型，undefined，null，symbol

引用数据类：对象Object，数组Array，函数Function，日期Date(),Math()，正则`RegExp()`,以及Set(),Map()
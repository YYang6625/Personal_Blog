---
title: Vue路由组件的三种模式
---

在`Vue`组件路由中存在三种模式，在这对其介绍加深理解，三种模式分别为: hash模式，history模式，abstract路由模式

hash模式:浏览器服务器兼容性好，安全性高，使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，hash值未改变时页面不会重新加载，其显示的网路路径中会有 “#” 号

hsitory模式:对兼容性有要求，页面主动刷新会重新请求服务器，需要进行一些配置，否则可能报错，没#更加美观

## 1.`hash`模式

为什么叫做`hash`模式呢，因为是通过改变`hash`值来执行对应的`js`来改变URL地址，在地址后面追加了一个#号，通过改变hash值的方式实现一个页面跳转的方式

1. hash指的是地址中#号以及后面的字符，也称为散列值。hash也称作锚点，本身是用来做页面跳转定位的。如`http://localhost/index.html#abc`，这里的#abc就是hash；
2. xxxxxxxxxx import { getCurrentInstance } from "vue";export default function useGlobalProperties() {  //appContext字面含义app实例的所有内容  const { appContext } = getCurrentInstance();  const globalProperties = appContext.config.globalProperties;  return {    globalProperties,  };}//需要注意的是应用了TS话会报错 需要将getCurrentInstance()进行断言处理 引入ComponentInternalInstance进行类型处理//getCurrentInstance() as ComponentInternalInstancejavascript
3. 监听 window 的 hashchange 事件，当散列值改变时，可以通过 location.hash 来获取和设置hash值；
4. location.hash值的变化会直接反应到浏览器地址栏；

**触发hashchange事件的几种情况：**

- 浏览器地址栏散列值的变化（包括浏览器的前进、后退）会触发window.location.hash值的变化，从而触发onhashchange事件；
- 当浏览器地址栏中URL包含哈希如 `http://www.baidu.com/#home`，这时按下输入，浏览器发送`http://www.baidu.com/`请求至服务器，请求完毕之后设置散列值为#home，进而触发onhashchange事件；
- 当只改变浏览器地址栏URL的哈希部分，这时按下回车，浏览器不会发送任何请求至服务器，这时发生的只是设置散列值新修改的哈希值，并触发onhashchange事件；
- html中`<a>`标签的属性 href 可以设置为页面的元素ID如 #top，当点击该链接时页面跳转至该id元素所在区域，同时浏览器自动设置 window.location.hash 属性，地址栏中的哈希值也会发生改变，并触发onhashchange事件；

```javascript
//设置 url 的 hash，会在当前url后加上'#abc'
window.location.hash='abc';
let hash = window.location.hash //'#abc'

window.addEventListener('hashchange',function(){
	//监听hash变化，点击浏览器的前进后退会触发
})

```

**优点**：在安全性方面更强大，是最安全的模式，兼容所有的浏览器和服务器

## 2.`history`模式

`history模式`：利用history API实现url地址改变，网页内容改变

主要有两个属性：

History.length：当前窗口访问过的网址数量（包括当前网页）。

History.state：History 堆栈最上层的状态值,可以理解成历史记录最上层的值，可以使用改变state的方法实现一个跳转，但是并不会刷新页面

页面进行跳转的三种方法:History.back()、History.forward()、History.go()

## 3.abstract 路由模式(了解即可)

abstract 是vue路由中的第三种模式，本身是用来在不支持浏览器API的环境中，充当fallback，而不论是hash还是history模式都会对浏览器上的url产生作用，本文要实现的功能就是在已存在的路由页面中内嵌其他的路由页面，而保持在浏览器当中依旧显示当前页面的路由path，这就利用到了abstract这种与浏览器分离的路由模式。
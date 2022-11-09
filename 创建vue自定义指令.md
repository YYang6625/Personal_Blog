---
title: 创建自定义指令
---

在组件标签的使用中，我们经常使用一些指令，例如v-for/v-model，在这里我们学习一下如何实现一个自定义指令，如何写一个自己想要实现的自定义指令功能，最好进行好封装，保证代码的可复用性

自定义指令的实现类似于组件的实现，文件内内写好指令代码进行引用即可

在创建自定指令前，先了解指令内的几个钩子函数

- `bind()`：当指令第一次绑定到元素时被调用，并只会被调用一次，用于初始化配置；
- `inserted()`：当绑定的元素被插入到父元素时被调用；
- `update()`：当绑定的组件被更新时调用，但会在其子组件更新之前被调用；
- `componentUpdated()`：当绑定的组件及其子组件全部更新之后被调用；
- `unbind()`：当指令和绑定的元素解绑时被调用，并且只会被调用一次

钩子函数有四个参数，分别是**el(绑定元素,可以获得子元素)，binding(一个对象)，vNode(虚拟节点)，oldNode(上一个虚拟节点)**

**binding对象内参数**：

- value 传递给指令的值
- oldValue 上一个值
- arg 传递给指令的参数
- modifiers 指令修饰符的对象
- instance 指令组件实例
- dir 指令对象

## 基本使用

```javascript
//在元素中使用绑定指令,注册指令
<div>
   //传入多个值用{}，传入单个值时需要添加''，说明这是值不是字符串
<button v-rloes="'admin'">编辑</button>
</div>
<script>
//类似于注册组件一样注册指令，引入指令，注册指令
import roles from './directives/(xxx)自定义指令.js';
export default {
   directives:{
      roles(自定义指令名);
   }
}
</script>
```

当自定指令被注册了，接下来就是对指令的内容进行编辑

```javascript
//这里做一个操作，假设登录者没有权限，就隐藏编辑按钮
import store from '../store/index'  //获取当前使用者权限
//默认导出
export default {
   //绑定元素时进行的操作
   bind(el,binding,vnode,oldnode){
      //假设获取当前使用者信息来确认权限,我们在上面传入值admin
      const roles = store.userInfo.rloes
      //如果传入的值不是store中的值，或者不包含这个值就隐藏button
      if(roles !== binding.value || !roles.includes(binding.value)){
         el.style.display = none;
      }
   }
   //更新时进行的操作
   update(){}
}
```

假设，这个div内部有多个按钮，部分需要权限设置，部分不需要，这如何满足呢，这时候需要利用到另一个**钩子函数inserted**，因为**bind只是初始化的时候调用，无法获取到父元素**，实现如下

```javascript
<div>
<button v-roles="'admin'">编辑</button>
<button v-roles="'admin'">删除</button>
<button v-roles="'user'">新增</button>
</div>
//这里做一个操作，假设登录者没有权限，就隐藏编辑按钮
import store from '../store/index'  //获取当前使用者权限
//默认导出
export default {
   //参数时一样的
	inserted(el,binding,vnode,oldnode){
      const roles = store.userInfo.roles；
      //如果不包含用户权限就删除
      if(roles !== binding.value || !roles.includes(binding.value)){
         el.parentNode.removeChild(el)
      }
   }
}
//以上就是一些简单的实现一个自定义指令
```



## 封装使用(复用)

接下来这里再**封装一些自定义指令的使用**,比如点击某个子元素产生某种效果

```javascript
//html部分
<div 
	class="xxx"
	v-xxx(自定义指令)="{
	curIdx,				 //可以通过子元素事件改变当前值，来获取索引
	className:'tag'     //传入子元素class获取子元素(复用性)、
	activeClass: ''
}"
>
	<div 
	class='tag'
	v-for="(item,index) of items"
	:key = "index"
	class='xxx'
	@click=changeIndex(index)
	></div>
</div>
```

接下来就是指令的封装

```javascript
//默认导出
//假设需要点击某个，某个变红，变红的效果就添加在activeClass:'xxx'
export default{
   //初始化绑定
   bind(el,binding,vnode,oldnode){
      const _ops = binging.value; //获取传递的所有值
      const _c = el.getElementsByClassName(_ops.calssName)  //绑定在父元素中，可以获取父元素，也可以通过类获取子元素
   }，
   update(el,binding,vnode,oldnode){
      const _ops = binging.value;
      const _c = el.getElementsByClassName(_ops.calssName);
      const _oOps = binding.oldValue,     //获取标签前一个值，没有前一个默认是空
      //点击某个某个产生效果(先清除上一个)
		_c[_oOps.curIdx].calssName = `${_oOps.className}`;   //点击之后，将上一个元素className还原即可
		_c[_ops.curIdx].calssName += `${_ops.activeClass}`;  //将事件触发的新的index元素添加 效果类
   }
}

//以上代码进行部分修改可以达成多种效果，具体看实际需要，实际要达成的效果(意思就是封装之后可复用)
```


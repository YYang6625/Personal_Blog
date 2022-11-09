---
title: Vue2,Vue3的区别
---

在个人学习过程中都是比较浅显的去了解Vue2，Vue3的一些已知的区别，主要区别有以下几点

1. 组合式api(composition   API)与选项api(options API)
2. proxy与Object.defineProperty(响应式原理)
3. 虚拟DOM新增字段pathFlag(不了解)
4. diff算法的优化
5. 多根节点(简单来说就是template下不需要div也可以直接放多个子结点)
6. 异步组件
7. Teleport组件
8. 事件缓存
9. 打包优化
10. 支持TypeScripte

vue3在组件标签中支持多个v-model

```javascript
<template>
  <child v-model="name" v-model:email="email" />
  <p>姓名：{{ name }}</p>
  <p>邮箱：{{ email }}</p>
</template>
```

vue3可以同时对多个源进行watche监听，并且增加`watchEffect API`

```javascript
const name = ref<string>('张三')
const userInfo = reactive({
  age: 18
})

// 同时监听name和userInfo的age属性
watch([name, () => userInfo.age], ([newName, newAge], [oldName, oldAge]) => {
  // 
})
```

个人能力水平有限，现阶段仅对一些可理解的进行解析记录

## 组合式`composition api`与选项式`options api`的区别

### `options api`优缺点

优点：

- 条例清晰，methods，computed，watch，data中等等定义属性和方法，共同处理页面逻辑，这也是为什么被称作 `Options API`

缺点：

- 修改困难，相同的放在相同的地方，在初期的编程阶段的确方便，但随着组件功能的增大，代码的增多，关联性会大大降低，组件的阅读和理解难度会增加；很容易出现反复横跳，修改代码上下来回翻转

- this:调用使用 this，但逻辑过多时 this 会出现问题，比如指向不明等

  ```javascript
  //这里举例一个防抖函数在vue2中的this问题
  //内容来源https://www.bilibili.com/video/BV1V14y1Y71c/?share_source=copy_web&vd_source=7efb188fb14ffde61b8af17387e620c6
  <script>
     //防抖
     function debounce(fn,delay){
     let timer = null;
     return funtion(){
        clearTimerot(timer)
        timer = setTimeout(()=>{
           fn()
           //修改this？为什么这个this就指向vm实例了呢？(vue内部机制，将methods中函数指向vm，采用的bind(),调用就修改指向)
           fn.call(this)
        },delay)
     }
  }
  export default{
     //假设有一个input使用了debouonceInput方法
     methods(){
        inputChange(){
           console.log(123);
        }
        //this无效问题
        //立刻报错，会发现这个this并不是指向实例对象vm，而是指向一个undefined
        //这是因为上面的箭头函数并没有this，vue的机制无法对箭头函数做处理，使它指向的是window，所以就是undefined
        debouonceInput:debounce(this.inputChange,500)
        
        //修改，将this.inputChange放入函数funcion中，上面也就是执行的function函数，然后将函数的this进行修改，也就成功修改成指向当前vm实例(debounce函数在methods中，vue机制会将methods中的function的this修改成指向当前实例，也就查找到了inputChange而不是undefined)
        //说白了还是箭头函数的问题，所以防抖函数内尽量不要使用箭头函数，造成this指向问题
        debouonceInput:debounce(function(){
           this.inputChange()
        },500)      
     }
  }
  </script>
  ```

  

### `composition api`的优化

通过将零散的的代码组合起来，方便代码的维护修改，这也是为什么叫做组合式api的原因

- 对this进行了修改，向上面的防抖函数在vue3中使用不会出现报错情况，正常使用即可

- 读取修改代码更加方便，不需要翻来覆去，不受模板和组件限制，可更好的进行封装以及模块化处理

  ```javascript
  //组合式api使用，为什么要进行return，因为一些函数被抽离出来了，并不局限与实例或者组件，这也决定了这更有利于封装以及模块化
  <script>
  import {ref} from 'vue';
  //在外面函数将需要处理的数据单独分离，方便后期维护，方便封装方法，方便复用(应该可以替代mixin混用，mixin会出现命名冲突的问题)
  const upDate = ()=>{
     const count = ref(0);
     const plus = ()=>{
        //需要加value，因为const地址不允许修改
        count.value++;
     }
     const minus = ()=>{
        count.value++;
     }
     return {
        count,
        minus,
     }
  }
  export default{
  	setup(){
        //解构获取内部方法(也避免了mixin命名冲突)
        { plus,minus } = upDate();
        return {
           count,
           plus,
           minus,
        }
     }
  }
  </script>
  ```

  

以上我们可以看到`composition api`的优点

后续有机会会对剩下的一些区别进行理解，对双向模拟，由于已经有一个blog进行了描述，这里不做赘述
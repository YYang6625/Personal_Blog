---
title: Vue使用路由的三种方式
---

# Vue3 获取路由$router/route 的三种方式

在 vue2 中，可以直接引用路由使用 this.$router/route 使用全局路由或者当前路由进行跳转,在 Vue3 则是进行了一些改变，setup(),没有 this 指向也就存在另外的三种方法

## 方法一.在当前 vue 组件中引入 vue-router 中的 useRouter，useRoute 进行使用

```javascript
<script setup>
  import {(useRoute, useRouter)} from 'vue-router' let $route=useRoute() let
  $router = useRouter() $router.push('/home') console.log($route.query,
  $router);
</script>
```

## 方法二：通过 const {proxy} = getCurrentInstance() 解构，获取当前实例起到一个 this 作用

```javascript
<script setup>
  import {getCurrentInstance} from 'vue' const {proxy} = getCurrentInstance()
  proxy.$router.push('/home') console.log(proxy.$route)
</script>
```

## 方法三：通过 appContext 从当前组件访问全局变量，const { appContext } = getCurrentInstance()

```javascript
<script setup>
  import {getCurrentInstance} from 'vue' const {appContext} =
  getCurrentInstance() appContext.config.globalProperties.$router.push('/home')
</script>
```

### 在 getCurrentInstance()基础上封装一个指向全局使用的方法 globalProperties

```javascript
import { getCurrentInstance } from "vue";
export default function useGlobalProperties() {
  //appContext字面含义app实例的所有内容
  const { appContext } = getCurrentInstance();
  const globalProperties = appContext.config.globalProperties;
  return {
    globalProperties,
  };
}
//需要注意的是应用了TS话会报错 需要将getCurrentInstance()进行断言处理 引入ComponentInternalInstance进行类型处理
//getCurrentInstance() as ComponentInternalInstance
```

---
title: elemengtUI中Form表单resetFields()无效问题,vue刷新页面
---

## 在elementUI的el-form组件库中，通过ref绑定el-form组件可以获取元素的一些数据以及对数据进行一些操作，以及编辑功能的实现

```javascript
      <el-form
        ref="form"
        :rules="rules"
        :inline="true"
        :model="formUser"
        label-width="120px"
      >
```

### 在编程的过程中，发现element组库提供的resetFields()方法不生效，如何解决呢？

resetFields()**官方解释**：对该表单项进行重置，将其值重置为初始值并移除校验结果

- 在编辑时通过 proxy.$refs.form.resetFields()进行清空数据失效这是为什么呢

- xxxxxxxxxx import { getCurrentInstance } from "vue";export default function useGlobalProperties() {  //appContext字面含义app实例的所有内容  const { appContext } = getCurrentInstance();  const globalProperties = appContext.config.globalProperties;  return {    globalProperties,  };}//需要注意的是应用了TS话会报错 需要将getCurrentInstance()进行断言处理 引入ComponentInternalInstance进行类型处理//getCurrentInstance() as ComponentInternalInstancejavascript

- ```javascript
      const EditUser = (userData) => {
        proxy.modelControl = 1;
        //显示元素  
        proxy.dialogVisible = true;
        // 将点击的数据赋给表单(直接赋值，form初始值就变成此赋值，因为dialog和赋值时同步的)
        proxy.$nextTick(() => {
          // 直接赋值并不会显示数据，因为是在下一次DOM更新之后才会显示
         proxy.formUser = userData;
        });
      };
  ```

  1. 通过上述操作的确可以使值的初始值就是空值，但是这里又有一个新的问题，什么问题呢？就是违背了我们的初衷，**清空数据之后，编辑表单中没有数据显示，只能将清空操作放置别处，我们这里只能对其进行赋值操作，否则不会显示数据，并且还必须是异步赋值，保持初始值为空**

  2. 但是异步赋值又会发现，这样页面就不显示我们需要编辑的数据了啊，这怎么办呢？

  4. 也很简单，$nextTick是在DOM更新之后渲染页面，我们就手动操作使其进行更新，**vue对所有数据都会进行监听，对于这样直接赋值的并不会引起虚拟DOM更新，因为两个对象并未发生改变，只是属性值的替换，此时我们就可以生成一个新的对象来迫使vue进行DOM更新使其显示数据**，

  5. 针对这里我们可以使用**深拷贝**的方法来使DOM进行更新，因为这里对对象本身进行了操作(引用数据类型)

  6. ```javascript
         const EditUser = (userData) => {
           proxy.modelControl = 1;
           proxy.dialogVisible = true;
           // 将点击的数据赋给表单(直接赋值，form初始值就变成此赋值，因为dialog和赋值时同步的)
           // 将赋值操作之后，这样的话userData也就不是赋值为初始值，而是修改后的值
           proxy.$nextTick(() => {
             // 这里对对象进行了修改所以会触发es6的set函数从而导致DOM更新而显示数据，上面写法并不会
             Object.assign(proxy.formUser, userData);
           });
         };
    ```
  
     

通过上述方法呢我们就可以做到在编辑form表单数据使即显示数据，并使DOM进行更新并显示数据，对于上述清空操作是在新增中进行的，由于初始就是为空，所以编辑之后新增还是能清空表单，关闭和取消也能清空表单，也就完成了编辑功能
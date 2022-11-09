---
title: Vue双向绑定理解与模拟
---

## 1.Vue的双向绑定理解

先从单向绑定进行理解，单向绑定就是将`Model`(data)绑定到视图层`View`，当我们使用`JS`更新`Model`数据层时，视图层就是进行自动更新，所以双向绑定也就是当`View`主动更新，也会去带动数据`Model`数据层进行更新。

那么他的原理是什么呢？

- 数据变化更新视图——>监听所有数据`Observer`（监听器）
- 视图变化更新数据——>保存或者说编译所有元素节点，改变时通知`Vue`实例进行数据更新，也就是需要一些更新函数`Complier`(解析器)

在`Vue`实例化过程中回首先对数据进行处理，监听每一个数据，当数据发生改变时通知变化，由于能力有限，先在`Vue2`的基础上对初步监听进行理解加深,以下时实现监听的代码

```javascript
class Vue {
   //obj_instace.data传入的data属性赋值给实例data属性
   constructor(obj_instace){
      this.$data = obj_instace.data
      //利用Observer监听属性
      Observer(this.$data)
      //解析模板(生成虚拟DOM，diff算法比对)
      Compiler(obj_instace.el，this)
   }
}
//定义监听函数--监听实例中的数据
//整体思路就是通过递归结合defineProperty中的setter，getter函数实现访问时进行监听与更新
function Observer(data_instace) {
   //监听前判断是否需要再次遍历，递归退出条件(没有子属性或者没有检测到对象也退出)
   if(!data_instance || typrof data_instance !=='object')
   //Object.keys()获取对象中的属性名keys并生成数组
   //对每一个属性进行遍历，也就是对每一个属性都进行设置，进行监听
   Object.keys(data_instance).forEaach(key=>{
      //首先保存每一个属性值，为了访问get与设置时能进行使用
      let value = data_instace[key];
      //假如属性值是一个对象或者数组，那么进行递归，确保遍历监听每一个属性值
      Observer(value)；
      //使用Object.defineProperty(obj,prop,descriptor)
      //分别传入操作对象，操作属性，定义属性{}实现数据监听
      Object.defineProperty(data_instace,key,{
         //属性可枚举
         enumerable:true;
         //属性描述符可改变,控制属性原型的一些特性是否可以修改
         configurable:true;
         //访问属性时触发getter函数
         get(){
         //访问时我们需要返回返回未被修改的属性值，不然访问显示为undefined的，
         return value；
      	}
         //设置属性时触发setter函数
      	set(newValue){
            //设置值后赋新值，修改后再访问就是访问新值
            value = newValue；
         }
      })
   })
}
//上述并不能监听新添加的对象，只是在已设置的数据中进行监听，新添加需要使用$set
```

通过上述代码已实现了监听data中的所有属性，接下来就对元素进行一些操作，解析编译HTML模板，利用fragment文档碎片存储文本节点，在文档碎片中进行数据处理，再结合发布者订阅者模式当数据发生变动时触发响应的监听回调，也就是`MVVM`模式

### `mvvm`模式

**M（model--data）**:需要observe的数据对象进行递归遍历，包括子属性对象的属性，都加上setter和getter这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化

**V（view）**:compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图

**VM(vue)**`：Watcher订阅者是Observer和Compile之间通信的桥梁，主要做的事情是: ①在自身实例化时往属性订阅器(dep)里面添加自己 ②自身必须有一个update()方法 ③待属性变动dep.notice()通知时，能调用自身的update()方法，并触发Compile中绑定的回调，则功成身退

以下是模拟模式实现的全部代码(由于正则使用较少还没全面理解，水平可能尚不能更好理解使用，为以后深入理解预留样板)

```javascript
class Vue {
  // 参数为对象实例 这个对象用于告知vue需要挂载到哪个元素并挂载数据
  constructor(obj_instance) {
    // 给实例赋值对象的data属性
    this.$data = obj_instance.data;
    // 进行数据劫持 监听对象里属性的变化
    Observer(this.$data);
    Complie(obj_instance.el, this);
  }
}

//数据劫持 —— 监听实例里的数据
function Observer(data_instance) {
  // 递归出口
  if (!data_instance || typeof data_instance !== "object") return;
  // 每次数据劫持一个对象时都创建Dependency实例 用于区分哪个对象对应哪个依赖实例和收集依赖
  const dependency = new Dependency();
  Object.keys(data_instance).forEach((key) => {
    // 使用defineProperty后属性里的值会被修改 需要提前保存属性的值
    let value = data_instance[key];
    // 递归劫持data里的子属性
    Observer(value);
    Object.defineProperty(data_instance, key, {
      enumerable: true,
      configurable: true,
      // 收集数据依赖
      get() {
        console.log(`获取了属性值 ${value}`);
        Dependency.temp && dependency.addSub(Dependency.temp);
        return value;
      },
      // 触发视图更新
      set(newVal) {
        console.log(`修改了属性值`);
        value = newVal;
        // 处理赋值是对象时的情况
        Observer(newVal);
        dependency.notify();
      },
    });
  });
}

//模板解析 —— 替换DOM内容 把vue实例上的数据解析到页面上
// 接收两个参数 1.vue实例挂载的元素<div id="app"> 2.vue实例
function Complie(element, vm) {
  vm.$el = document.querySelector(element);
  // 使用文档碎片来临时存放DOM元素 减少DOM更新
  const fragment = document.createDocumentFragment();
  let child;
  // 将页面里的子节点循环放入文档碎片
  while ((child = vm.$el.firstChild)) {
    fragment.appendChild(child);
  }
  fragment_compile(fragment);
  // 替换fragment里文本节点的内容
  function fragment_compile(node) {
    // 使用正则表达式去匹配并替换节点里的{{}}
    const pattern = /\{\{\s*(\S+)\s*\}\}/;
    if (node.nodeType === 3) {
      // 提前保存文本内容 否则文本在被替换一次后 后续的操作都会不生效
      // 打工人: {{name}}  => 打工人：西维 如果不保存后续修改name会匹配不到{{name}} 因为已经被替换
      const texts = node.nodeValue;
      // 获取正则表达式匹配文本字符串获得的所有结果
      const result_regex = pattern.exec(node.nodeValue);
      if (result_regex) {
        const arr = result_regex[1].split("."); // more.salary => ['more', 'salary']
        // 使用reduce归并获取属性对应的值 = vm.$data['more'] => vm.$data['more']['salary']
        const value = arr.reduce((total, current) => total[current], vm.$data);
        node.nodeValue = texts.replace(pattern, value);
        // 在节点值替换内容时 即模板解析的时候 添加订阅者
        // 在替换文档碎片内容时告诉订阅者如何更新 即告诉Watcher如何更新自己
        new Watcher(vm, result_regex[1], (newVal) => {
          node.nodeValue = texts.replace(pattern, newVal);
        });
      }
    }
    // 替换绑定了v-model属性的input节点的内容
    if (node.nodeType === 1 && node.nodeName === "INPUT") {
      const attr = Array.from(node.attributes);
      attr.forEach((item) => {
        if (item.nodeName === "v-model") {
          const value = item.nodeValue
            .split(".")
            .reduce((total, current) => total[current], vm.$data);
          node.value = value;
          new Watcher(vm, item.nodeValue, (newVal) => {
            node.value = newVal;
          });
          node.addEventListener("input", (e) => {
            // ['more', 'salary']
            const arr1 = item.nodeValue.split(".");
            // ['more']
            const arr2 = arr1.slice(0, arr1.length - 1);
            // vm.$data.more
            const final = arr2.reduce(
              (total, current) => total[current],
              vm.$data
            );
            // vm.$data.more['salary'] = e.target.value
            final[arr1[arr1.length - 1]] = e.target.value;
          });
        }
      });
    }
    // 对子节点的所有子节点也进行替换内容操作
    node.childNodes.forEach((child) => fragment_compile(child));
  }
  // 操作完成后将文档碎片添加到页面
  // 此时已经能将vm的数据渲染到页面上 但还未实现数据变动的及时更新
  vm.$el.appendChild(fragment);
}

//依赖 —— 实现发布-订阅模式 用于存放订阅者和通知订阅者更新
class Dependency {
  constructor() {
    this.subscribers = []; // 用于收集依赖data的订阅者信息
  }
  addSub(sub) {
    this.subscribers.push(sub);
  }
  notify() {
    this.subscribers.forEach((sub) => sub.update());
  }
}

// 订阅者
class Watcher {
  // 需要vue实例上的属性 以获取更新什么数据
  constructor(vm, key, callback) {
    this.vm = vm;
    this.key = key;
    this.callback = callback;
    //临时属性 —— 触发getter 把订阅者实例存储到Dependency实例的subscribers里面
    Dependency.temp = this;
    key.split(".").reduce((total, current) => total[current], vm.$data);
    Dependency.temp = null; // 防止订阅者多次加入到依赖实例数组里
  }
  update() {
    const value = this.key
      .split(".")
      .reduce((total, current) => total[current], this.vm.$data);
    this.callback(value);
  }
}
```


---
title: 利用mock简单的二次封装axios，Mock拦截
---



## axios 与 mock 结合同时配置本地与线上

1. 一般针对项目都会有自己的一些开发，测试，生产环境，由于没有经历过实际的项目经验，还确定具体环境配置文件，这里学习过程中用于学习的环境配置文件，以及一些相关知识

   ```JavaScript
   // 环境配置文件
   // 开发环境
   // 测试环境
   // 线上环境

   // import.meta.env.MODE: {string} 应用运行的模式。(vite官网)
   // import.meta.env.BASE_URL: {string} 部署应用时的基本 URL。他由base 配置项决定。
   // import.meta.env.PROD: {boolean} 应用是否运行在生产环境。
   // import.meta.env.DEV: {boolean} 应用是否运行在开发环境 (永远与 import.meta.env.PROD相反)。
   // import.meta.env.SSR: {boolean} 应用是否运行在 server 上。

   // 当前环境(取不到默认为线上环境)
   const env = import.meta.env.MODE || "prod";

   const EnvConfig = {
     // 开发环境
     development: {
      //默认地址
       baseApi: "/api",
       // 拦截请求地址(开启线上拦截之后将mockApi设置为默认路径，再去线上mock官网中获取设置好的接口数据)
       mockApi:
         "https://www.fastmock.site/mock/9f511f427bc80b2ea1b1ae5388de1da9/api",
     },
     // 测试环境(随便写的)
     test: {
       baseApi: "//test.futurecom/api",
       mockApi:
         "https://www.fastmock.site/mock/9f511f427bc80b2ea1b1ae5388de1da9/api",
     },
     // 生产环境
     pro: {
       baseApi: "//future.com/api",
       mockApi:
         "https://www.fastmock.site/mock/9f511f427bc80b2ea1b1ae5388de1da9/api",
     },
   };

   export default {
     // 导出当前环境
     env,
     // mock总开关，控制mock本地拦截或者在线拦截
     mock: true,
     ...EnvConfig[env],
   };
   ```

2. 结合 mock 二次封装 axios 请求进行拦截等操作

   ```javascript
   // 二次封装axios请求(对接口请求前后进行一些处理)
   import axios from "axios";
   // 设置环境
   import config from "../../config";
   // 引入错误提示
   import { ElMessage } from "element-plus";

   // 定义网络请求异常常量模拟错误
   const NETWORK_ERROR = "网络请求异常.请稍后尝试";

   // 创建axios实例  设置基础路径
   const service = axios.create({
     // 将基础路径设置为环境路径
     baseURL: config.baseApi,
   });

   // 利用axios拦截器对请求之前做一些处理
   service.interceptors.request.use((request) => {
     // 自定义一些header(请求头)
     // jwt-token认知时
     return request;
   });

   // 请求之后处理
   service.interceptors.response.use((response) => {
     // 对数据进行一些处理,解构直接获取请求后的数据
     const { code, data, msg } = response.data;
     if (code == 200) {
       return data;
     } else {
       // 网络请求错误,有返回错误信息就返回没有就报网络异常
       ElMessage.error(data.msg || NETWORK_ERROR);
       //  返回错误状态的Promise回调
       return Promise.reject(data.msg || NETWORK_ERROR);
     }
   });

   // 封装的axios实例请求核心函数
   function request(options) {
     // options就是请求地址的一些参数
     // console.log('request', options);
     // 设置options参数,有方法就使用方法,没有就使用get方法
     options.method = options.method || "get";
     // 将请求的字符串变成小写toLowerCase()
     if (options.method.toLowerCase() == "get") {
       //  将参数params的数据设置给data
       options.params = options.data;
     }

     // 单个接口对mock的处理,定义一个mock布尔值
     let isMock = config.mock;
     // 如果接口自带mock,则将原本总接口的mock覆盖做某个单独接口的mock(决定是否进行拦截处理,后端接口数据完成那也就没必要进行拦截了，直接使用后端接口数据)
     if (typeof options.mock !== "undefined") {
       isMock = options.mock;
     }
     // 对线上环境做处理
     if (config.env == "prod") {
       // 不给使用mock的机会,如果是生产环境则转接到开发环境防止产生严重bug(还在使用mock也就不太可能已经完善了项目)
       service.defaults.baseURL = config.baseApi;
     } else {
       // 不是生产环境就判断mock布尔值来确定是够需要进行拦截
       service.defaults.baseURL = isMock ? config.mockApi : config.baseApi;
     }
     // 当调用request时携带参数,根据参数发起相对应的请求返回数据
     // request就是返回了一个带有mock等数据的axios请求示例对象
     return service(options);
   }

   export default request;
   ```

3. axiso 请求数据(专门 api.js 进行请求与管理)

   ```javascript
   import request from "./index.js";
   export default {
     // 获取echart表格数据
     getEchartData(params) {
       return request({
         url: "/home/getEchartData",
         method: "get",
         // 单个接口mock的控制
         mock: true,
       });
     },
     // 获取userData数据
     getUserData(params) {
       return request({
         // 这里需要和拦截请求的地址一致,拦截之后返回自定义的一些数据
         url: "/user/getUser",
         method: "get",
         // 这里不需要先上就关闭mock
         mock: false,
         data: params,
       });
     },
   };
   ```

   4.创建 mock.js 进行拦截请求

   ```javascript
   // 一：本地mock的使用
   import Mock from "mockjs";
   import homeApi from "./mockData/home.js";
   import userApi from "./mockData/user.js";
   import permissionApi from "./mockData/permission.js";
   // 拦截请求(将对服务器的请求拦截转向本地请求，模拟异步请求操作)
   // 将请求路径进行拦截，并返回第二个参数的数据
   Mock.mock("/home/getData", homeApi.getHomeData);
   
   // 本地获取user的数据(正则) 注意：params参数传递需要以对象形式{params：data}
   Mock.mock(/user\/getUser/, "get", userApi.getUserList);
   // 拦截用户登录(使用字符串写的地址会被写死携带参数无法识别拦截，利用正则即可拦截此字段发出的请求)
   Mock.mock(/permission\/getMenu/, "post", permissionApi.getMenu);
   ```

## mockjs 的简单使用

如果不需要进行封装只是简单的拦截直接在组件使用即可

```javascript
        //1.本地mock请求
      await axios.get("/home/getData").then((res) => {
        console.log(res);
        if (res.data.code == 200) {
          tableData.value = res.data.data.tableData;
        }

      //2.在线mock官网请求
      await axios
        .get(
          "https://www.fastmock.site/mock/9f511f427bc80b2ea1b1ae5388de1da9/api/home/getTableData"
        )
        .then((res) => {
          console.log(res);
          if (res.data.code == 200) {
            tableData.value = res.data.data;
          }
```

设置好 mock.js 文件以及 mockData 文件返回对应数据即可

```javascript
//例如
//对应上面的 Mock.mock("/home/getData", homeApi.getHomeData);
export default {
  getHomeData: () => {
    return {
      code: 200,
      data: {
        tableData: [
          {
            name: "oppo",
            todayBuy: 500,
            mothBuy: 3500,
            totalBuy: 22000,
          },
          {
            name: "vivo",
            todayBuy: 500,
            mothBuy: 3500,
            totalBuy: 22000,
          },
          {
            name: "苹果",
            todayBuy: 500,
            mothBuy: 3500,
            totalBuy: 22000,
          },
          {
            name: "小米",
            todayBuy: 1200,
            mothBuy: 6500,
            totalBuy: 45000,
          },
          {
            name: "三星",
            todayBuy: 500,
            mothBuy: 2000,
            totalBuy: 34000,
          },
          {
            name: "魅族",
            todayBuy: 350,
            mothBuy: 3500,
            totalBuy: 22000,
          },
        ],
      },
    };
  },
};
```

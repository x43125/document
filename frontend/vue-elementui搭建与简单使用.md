# vue-elementui搭建与使用

## 结构搭建

> elementUI 2

首先使用vue脚手架搭建一个vue基础结构

```sh
#################### 1.初始化项目
vue init webpack project_name
# 根据实际情况选择即可
#################### 2.进入项目
cd project_name
#################### 3.运行项目，待项目运行成功后，点击下侧的项目地址
# 默认为：http://localhost:8080,查看页面
npm run dev
# 页面正常运行即可，停止项目

#################### 4.引入axios
npm install axios


#################### 5.引入element-ui
npm i element-ui -S
# 待引入成功后，需在项目中手动引入依赖

#################### 6.在project_name/src/main.js 中引入element-ui

import Vue from 'vue'
import App from './App'
import router from './router'

#################### 在此处添加如下三行代码：
##################################################################################
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);
##################################################################################
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>'
})



# 手动添加成功后，再次启动项目：运行如下指令
npm start
# 将再次看到网址：http://localhost:8080
```

## 使用

### 项目架构

assets 放资源包括img, css

业务组件写在 `project_name/src/views` 下

公共组件写在 `project_name/src/components` 下

### 添加路由

```js
// 饿汉式
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      redirect: '/index'
    },
    {
      // 注意写法
      path: '/index',
      name: 'Index',
      component: () => import('@/views/Index'),
      // 子路由
      children:[
        {
          path: '/login',
          name: 'Login',
          // 懒加载
          component: () => import('@/views/account/Login')
        },
        {
          path: '/register',
          name: 'Register',
          component: () => import('@/views/account/Register')
        }
      ]
    },
// 404页面
    {
      path: '*',
      component: () => import('../views/Invalid')
    }
  ]
})

```

### 请求封装
#### request.js
基础操作，拦截所有请求，返回等
```Javascript
import axios from "axios";  
  
// request是基本请求，设置  
// 拦截器等  
export function request(config) {  
  const instance = axios.create({  
    baseURL: 'http://localhost:8081',  
 timeout: 20000  
 })  
  
  // 发送的拦截器，发送前做的操作  
 instance.interceptors.request.use(config => {  
    // console.log('interceptors.request:');  
 // console.log(config); console.log("发送拦截器");  
 return config;  
 },error => {  
    console.log(error);  
 })  
  
  // 接受拦截器，接收到返回后做的操作  
 instance.interceptors.response.use(res => {  
    // console.log('interceptors.response');  
 // console.log(res.data); console.log("回收拦截器");  
 return res.data;  
 }, error => {  
    console.log(error);  
 })  
  
  return instance(config)  
}
```

#### 实际接口
对`request.js`进行二次包装以实现和第三方插件的解耦

```javascript
import {request} from "../../src/network/request";


export function sayHello() {
  return request({
    url: '/test/sayHello'
  })
}

export function testGet() {
  return request({
    url: '/test/testGet'
  })
}

export function testParam(key1, key2) {
  return request({
    url: '/test/testParam',
    method: 'post',
	// 要使用 `params` 而不是 `param`
    params: {
      key1, key2
    }
  })
}

// data中的requestBody的每一个参数要和后端的参数名一一对应，否则会报错
export function testBody(requestBody) {
  return request({
    url: '/test/testBody',
	method: 'post',
    data: requestBody
  })
}

export function testBody(requestBody) {
  return request({
    url: '/test/testBody',
	method: 'post',
    data: requestBody
  })
}
```

复杂一点的，包括错误处理的情况，其他js引入的时候使用，`import request from '@/network/request'` 的方式引入
```javascript
import axios from "axios";
import errorCode from '../../utils/errorCode'
import {Message, MessageBox} from "element-ui";

axios.defaults.headers['Content-Type'] = 'application/json;charset=utf-8'

// request是基本请求，设置
// 拦截器等
const instance = axios.create({
  baseURL: 'http://localhost:8081',
  timeout: 10000
})

instance.interceptors.request.use(config => {
  console.log("发送拦截器");
  return config;
}, error => {
  console.log(error)
  Promise.reject(error)
})

instance.interceptors.response.use(res => {
  const code = res.data.code || 200
  const msg = errorCode[code] || res.data.message || errorCode['default'];
  if (res.request.responseType === 'blob' || res.request.responseType === 'arraybuffer') {
    return res.data
  }
  if (code === 401) {
    MessageBox.confirm('登录状态已过期，您可以继续留在该页面，或者重新登录', '系统提示', {
        confirmButtonText: '重新登录',
        cancelButtonText: '取消',
        type: 'warning'
      }
    ).then(() => {
      store.dispatch('LogOut').then(() => {
        location.href = '/index';
      })
    }).catch(() => {
    });
    return Promise.reject('无效的会话，或者会话已过期，请重新登录。')
  } else if (code === 500) {
    Message({
      message: msg,
      type: 'error'
    })
    return Promise.reject(new Error(msg))
  } else if (code !== 200) {
    Message({
      message: msg,
      type: 'error'
    })
    return Promise.reject(msg)
  } else {
    Message({
      message: msg,
      type: 'success'
    })
    return res.data
  }
})

export default instance

```


## 问题解决

跨域：在controller的类名上加注解：`@CrossOrigin`

router-link 无法点击情况：

```vue
    <el-container>
    <!-- 不能放在 el-header 中 -->
      <h1>Index Page</h1>
      <el-header>
        <!-- 左上 -->
        <div>
          <router-link to="/index">主页</router-link>
          <router-link to="/blog">博客</router-link>
          <router-link to="/share">分享</router-link>
          <router-link to="/news">新闻</router-link>
          <router-link to="/game">游戏</router-link>
          <router-link to="/shopping">购物</router-link>
          <router-link to="/tools">工具箱</router-link>
        </div>
        <!-- 单独一角 右上 -->
        <div>
          <router-link to="/login">登录</router-link>
          <router-link to="/register">注册</router-link>
        </div>
      </el-header>
      <el-main>
        <router-view />
      </el-main>
    </el-container>
```


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

### 项目架构：

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


# vue

```sh
vue init webpack `Your Project Name`
cd `youProject`
npm run dev
npm install --save vuex
npm install --save axios

```

在src目录下，新建一个store并创建一个index.js文件

其内添加如下代码，以引入vuex

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

const store = new Vuex.Store({

});

export default store;

```

在main.js中添加如下代码

```js
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
// 新加
import axios from 'axios' 
import store from './store'

Vue.config.productionTip = false

Vue.prototype.$http = axios;

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  components: { App },
  template: '<App/>',
  store,
})

```
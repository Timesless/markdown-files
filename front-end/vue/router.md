## vue-router

``` js
index.js

import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from ../components/Home.vue
import Home from ../components/About.vue
// 安装插件
Vue.use(VueRouter);
const routes = [
    {
        path: '/home',
        component: Home
    }, {
        path: '/about',
        component: About
    }
]
// 创建router对象
new VueRouter({
    routes
})
// 将router挂载到vue实例
```


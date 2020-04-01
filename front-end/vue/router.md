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
// 创建router对象，并导出
new VueRouter({
    routes
})
// 将router挂载到vue实例
```

#### router-link 属性

``` js
// 默认渲染为a标签
// tag
// replace 使用 replaceState
活跃的路由会有该类属性 router-link-active
// 通过active-class | linkActiveClass指定类属性


不要跳过router直接使用history
// this.$router.push('/home')
// this.$router.repalce('/about')
```

#### 路由懒加载

``` js
// 把不同路由对应的组件分割成不同的代码块打包
const Home = () => import('./views/Home')
```

#### 路由参数传递

``` js
// params
// query对象
提供计算属性 / 事件方法
{
    path: '/home', query: {
        id: 11,
        name: 'zs'
    }
}
```

#### 全局导航守卫

``` js
// 监听路由跳转
// 前置钩子
router.beforeEach((to, from, next) => {
    next()
})
```

#### keep-alive

``` js
<keep-alive exclude='Home,About'>
    <router-link />
</keep-alive>
```


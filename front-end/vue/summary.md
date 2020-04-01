### Vue

+ mvvm
+ options
+ vue生命周期函数
+ template
+ mustache语法

### Vue基础语法

+ v-bind （:语法糖；对象，数组语法）
    + 动态绑定class
    + 动态绑定style



+ computed
    + getter
    + setter

``` js
computed: {
    
}
```

+ methods

``` js
methods: {
    
}
```

+ filters

``` js
filters: {
    // 自动传递参数
    filterPrice(price) {
        return '￥' + price.toFixed(2);
    }
}
// 使用
{{price | priceFilter}}
```

+ v-on（@语法糖）
    + 参数传递
    + 修饰符
+ v-show

``` js
// display: none
```

+ v-if

``` js

```

+ v-for

``` js

```

+ v-model（双向绑定）

``` js

```



+ 数组响应式
+ 对象响应式
+ JavaScript高阶函数

### 组件化开发

+ 全局组件，局部组件

+ 父子组件

    + 父子组件传值

        父传子props

        ``` js
        
        ```

        子传父，自定义事件

        ``` js
        
        ```

    + 父子组件访问值

    ``` js
    
    ```

+ 注册组件语法糖
+ 模板抽离
+ 组件data必须是函数
+ slot

``` js

```

+ 编译作用域
+ 规范
    + AMD
    + CMD
    + CommonJS
    + ES6
+ 模块引入和导出

``` js
// CommonJS
const http = require('http')
// ES6
import Vue from 'vue'

// 导出
export * as a
export default const Vue 
```

#### webpack

JavaScript应用的**静态模块打包工具**

1. npm install webpack -g
2. npm install webpack --save-dev

+ webpack.config.js

``` js

// loader（转换器配置）
modules {
    
}
// 插件
plugins {
    
}
// 本地服务器

```

+ package.json

``` js
// 可以定义许多脚本
```

+ webpack-css
+ webpack-less
+ webpack-图片文件处理
+ webpack-plugins
    + uglifyjs
    + dev-server

+ 配置文件分离

#### babel

**ES6转ES5**

+ webpack-vue配置过程
+ template 和 el

#### vue终极使用方案



### Vue CLI详解

#### runtime-only

``` mermaid
graph LR
render --> virtual-dom --> ui
```

> ast: 抽象语法树
>
> runtime-only 会使用render中的函数调用替换掉el中的内容
>
> **template由vue-template-complier转换为render，且是开发时依赖，所以不存在template了**

#### runtime + complier

``` mermaid
graph LR
template --> ast --> render --> vitural-dom --> ui
```

> runtime + compiler 会使用 template中的元素，替换掉el中的元素内容

``` js
// template， 以template开发选择这个
new Vue({
    el: '#app',
    template: '<App />'
})

// render 以.vue开发，template由vue-template-complier转换为render
import App from 'app'
new Vue({
    // el: '#app',
    // 传入标签 | 组件对象
    render: h => h(App)
}).$mount('#app');
```

+ CLI2
+ CLI3

> @vue/cli-service 很多开发时依赖由该脚手架统一管理
>
> runtime-only => vue-template-compiler将template渲染为render函数

``` js
// cl3修改配置
1. vue ui
2. vue.config.js vue会将此配置合并
```

+ ESLint

### vue-router

+ 路由
+ 后端渲染，前后端分离，前端路由
+ url的hash和html5的history模式

+ 路由映射配置
+ router-link
+ router-view
+ 动态路由
+ 路由懒加载
+ 路由嵌套
+ 参数传递
+ 全局导航守卫 navigation guard
+ keep-alive
+ tabbar

``` js
// 后端渲染
jsp，请求的页面是服务器已经渲染的
// 前后端分离
（ajax请求）后端只负责提供数据， html+css+js在静态服务器（nginx）获取
// SPA 前端路由（管理url与页面的映射关系）
只有一个html页面，路由控制显示与跳转
```

#### history模式

``` js
// h5的history模式
history.go(1) == history.forward();
history.go(-1) == history.back();
history.pushState({}, 'title', 'url');
history.replaceState({}, 'title', 'url');
```



#### Promise



### Vuex



+ devtools
+ mutations
+ actions
+ 单一状态树
+ getters
+ modules



### 网络封装（axios）



### 项目实战
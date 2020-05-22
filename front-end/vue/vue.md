### Vue

+ mvvm
+ options
+ vue生命周期函数
+ template
+ mustache语法

``` js
// 模板语法
v-once
v-html
v-text
v-pre
<span v-html='variable'> </span>

// 动态绑定属性
v-bind
<a :href='hrefA'>点击一下</a>
v-for
<li v-for="(m, index) in movies"></li>
```



### Vue基础语法

+ v-bind （:语法糖；对象，数组语法）
    + 动态绑定class
    + 动态绑定style



+ computed（可以只实现getter，看起来像函数）
    + getter
    + setter

``` js
computed: {
    totalPrice() {
        return this.itemList.reduce((prev, curr) => prev + curr, 0)
    }
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
// 使用，格式化价格
{{price | priceFilter}}
```



+ 事件监听（点击，键盘...）v-on（@语法糖）
    + 参数传递
    + 修饰符

``` javascript
// 点击时，浏览器会生成一个事件对象，vue默认传递它

1 不带参数
<button @click="doClick">按钮</button>
methods: {
    doClick(event) {
        console.info(event);
    }
}
2 带参数
<button @click="doClick(123, $event)">按钮</button>
doClick(data, event) {
    console.info(data);
    console.info(event);
}

// 修饰符
// 阻止事件冒泡
@click.stop="doClick"
// 阻止默认事件
@click.prevent="doClick"
// 键盘点击
.enter 回车
@keyup / @keydown.enter="keyUp"
// 只触发一次
@click.once="doClick"
```



+ v-show

``` js
// display: none
<span v-show="false">dispaly</span>
```

+ v-if

``` js
<span v-if="false">text</span>
<span v-else>text</span>
```

+ v-for

> // v-for推荐绑定唯一key
> v-for="(item, index) in list" :key="item"

``` js
const list = [1, 2, 3, 4];

// splice 删除，修改，添加
list.splice(2, 0, 5);

(item, index) 类似元组数据结构
// 遍历数组
<li v-for="(item, index) in itemList">{{ item.name }}</li>

// 遍历对象
v-for="(value, key) in obj"
```



#### 高阶函数

> 命令式编程 / 声明式编程
>
> 面向对象编程 / 函数式编程（函数是第一公民）

+ 数组高阶函数

``` js
// filter, map, reduce
```



+ v-model（双向绑定）

``` js
// 基本使用
<input type="text" v-model="message"></input>
// 类似于 :value & @input 事件的结合
<input type="text" :value="message" @input="message = $event.target.value" />
    
@input="changeText"
changeText(event) {
    this.message = event.target.value
}

// v-model 结合 checkbox
<input type="checkbox" value="game" v-model="hobbies">游戏
<input type="checkbox" value="swim" v-model="hobbies">游泳
hobbies: []

// 修饰符
v-model.lazy
v-model.number	// v-model默认为string
v-model.trim // 首位空格
```



+ 数组响应式

``` js
push
pop
shift
unshift	// 在最前面添加元素
splice
sort
reverse

// 修改
1 splice
2 Vue.set(this.list, 1, 'modifyValues')
```



+ 对象响应式



### **组件化开发

> 创建组件构造器：Vue.extend({template: ``})
>
> 注册组件：Vue.component()（全局组件）
>
> 使用组件：



***基本使用**

``` html
// 原始方法，Vue2.0开始使用语法糖
<div>
    // 使用3次组件
    <testCpn />
    <testCpn />
    <testCpn />
</div>
<script>
	const cpnConstructor = Vue.extend({
        template: `<div><h2>标题</h2><p>内容</p></div>`
    });
    
    Vue.component('testCpn', cpnConstructor);
</script>
```



+ 全局组件，局部组件

``` html
// 全局组件Vue.component({template: ``})

// 局部组件
const app = new Vue({
	...,
	components: {
		cpn: cpnConstructor
	}
})
```



+ 父子组件

``` html
<div id="app">
    <cpn2 />
</div>

<script>
// 组件一构造器
const cpnC1 = Vue.extend({
	template: ``
})

/*
 * 组件二构造器
 * 在编译时，会把<cpn1 />替换为模板，渲染为render函数
 */
const cpnC2 = Vue.extend({
	template: `<div><cpn1 /></div>`,
	// 注册组件一
	components: {
		cpn1: cpnC1
	}
})

// Vue中注册cpnC2
new Vue({
    ...,
    components: {
    	cpn2: cpnC2
	}
})
</script>
```



+ *注册组件语法糖（省去Vue.extend()调用，底层会调用）

``` html

// 全局组件语法糖
1. Vue.component('cpn1', {
	template: `
		<div>
            <h2>标题</h2>
		</div>`
})

// 局部组件语法糖
components: {
	cpn: {
		template: ``
	}
}

2. Vue.component(() => import('./Test.vue'))
```



+ 组件模板分离写法

``` html
1. <template id="tmpt">
	<div>
        <h2>标题</h2>
    </div>
</template>

2. <script type="text/x-template" id="tmpt">
	<div><h2>标题</h2></div>
</script>

<script>
	Vue.component('cpn', {
        template: 'tmpt'
    })
</script>
```



+ 子组件data必须是function

``` html
// 多个组件时，每个组件应该有一份自己的值
```



+ 父子组件通信

    父传子props，子传父自定义事件

    ``` html
    // 挂载vue
    <div id="app">
        // v-bind注意驼峰命名错误，可以用烤肉串格式
        // v-on监听子组件发射的事件
      <cpn :cmessage="message" @itemclick="fclick"></cpn>  
    </div>
    
    // 子组件
    <template id="cpn">
    	<div>
            {{cmessage }}
            <button @click="btnClick">子组件点击</button>
        </div>
    </template>
    
    <script>
        
        const cpn = {
            template: "#cpn",
            // 数组， 对象
            // props:['cmessage']
            props: {
              cmessage: string,
              cmessage: {
                  type: string,
                  default: '空消息'
              }
            },
            data() {
                
            },
            methods:{
                // 子传父，发射自定义事件
                btnClick(item) {
                    this.$emit('itemclick', item);
                }
            }
        }
        // vue
    	new Vue({
            el: '#app',
            data: {
                message: '父组件消息'
            },
            methods: {
                // 处理子组件发射的事件
                fclick(item) {
                    console.info(item)
                }
            },
            components: {
                cpn
            }
        })
    </script>
    ```




+ watch

``` html
data() {
	name: ''
},
watch: {
	// 这个名称与data中需要监听的数据名称相同
	name(newVal, oldVal) {
		console.info(oldVal);
	}
}

```



+ 父子组件对象的访问方式

``` html
// 父访问子
$children: 数组类型（一般不使用这个方式）

$refs：在组件上加属性 ref
<div id="app">
    <cpn ref="aaa"></cpn>
</div>
在父组件中：console.info(this.$refs.aaa)
    

// 子访问父
$parent
console.info(this.$parent)

this.$root 访问Vue实例
```



+ slot（插槽）

``` html
// 让组件有扩展性，抽取共性，预留插槽
<div id="app">
    // 在使用组件时，传入具体元素代替插槽
    <cpn>
    	<button>插槽预留</button>
        // 具名插槽的使用，指定名称
        <button slot="hhh">插槽预留</button>
    </cpn>
</div>

<template id="cpn">
    <div>
        <h2>标题</h2>
        // 预留两个插槽，设置默认元素
        <slot><span>默认元素</span></slot>
        // 具名插槽
        <slot name="hhh"><span>默认元素</span></slot>
    </div>
</template>

```



+ 作用域插槽

``` html
// 在父组件改变显示方式

<div id="app">
    // 这里可以通过数据修改展示方式，所以需要子组件传数据过来
    <cpn>
    	// 2.5以下使用<template></template>
        // 2.5以上可以使用div元素
        <div slot-scope="sslot">
            // 通过slot.data获取数据
            // <span v-for="item in sslot.data">{{item }}, </span>
            // a - b - c
            <span>{{sslot.data.join(' - ') }}</span>
        </div>
    </cpn>
</div>

<template id="cpn">
	<div>
        // 这是子组件的展示方式，将数据传出去
        <slot :data="itemList">
        	<ul>
                <li v-for="item in itemList">{{item }}</li>
            </ul>
        </slot>
    </div>
</template>
```



+ 编译作用域

``` html
在自己作用域里面查找变量
```



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
// 定义脚本
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
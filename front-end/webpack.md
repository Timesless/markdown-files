### 一个JS的静态模块打包工具

``` js
// 生成package.json
npm init -y
// 开发时依赖
npm install webpack --save-dev
// 运行时依赖
npm install lodash --save
// 1 由于浏览器不支持ES6，所以需要babel将ES6语法转换为ES5语法
npm install --save-dev @babel/core @babel/cli @babel/preset-env
// 2 babel只转换ES6基础语法，高级特性如Promise 等需要使用polyfill 完善浏览器对新方法的定义
npm install --save @babel/polyfill
// 3 core-js

// 打包
webpack ./app.js -o ../dist/bundle.js

// webpack4 需要安装cli
npm install webpack -g
npm install webpack-cli -g
```

#### loader

> oneOf 匹配到一个loader之后结束

+ npm安装相应的loader
+ 在webpack.config.js的modules下配置

``` js
// CommonJS规范
module.exports = {}
require('http');

// ES6规范
import Vue from 'vue';
export const name = 'hhh';
```

#### webpack.config.js

> + 文件资源缓存
> + tree shaking

```js
// 使用CommonJS模块规范

const { resolve } = require('path')

module.exports = {
    entry: './main.js',
    output: {
      filename: 'bundle.js',
      path: resolve(__dirname, 'build')
    },
    module: {
        // loader配置 下载，使用
      rules: [
        {
            test: /\.css$/,
           	loader: ['style-loader', 'css-loader']
        }
      ]
    },
    plugins: [
        // 下载 引入 使用
    ],
    // 1. 打包优化，将node_modules中打包为单独文件
    // 方式2. 动态引入module。
    // 1 2组合使用
    optimization: {
        splitChunks: {
            chunks: 'all'
        }
    }
    mode: 'development',
    // 通过cdn引入的js（模块）不需要打包，在入口html中手动引入
    externals: {
    	// 拒绝jquery打包
    	jquery: 'jQuery'
	},
    devServer: {
        
    },
    devtool: 'eval-source-map'
      
}
```

#### source-map构建后代码映射到源码

> dvetool: 'eval-source-map'
>
> [inline- | hidden- | eval- ]，[nosources- ]，[cheap - [module- ]]  source-map

``` js
// source-map
// inline-source-map（内部）
// hidden-source-map（隐藏源码）
// eval-source-map （内部）
// nosources-source-map（隐藏构建后代码和源码）
// cheap-source-map
// cheap-module-source-map
```

#### 缓存

+ babel缓存（编译时）
+ 模块缓存（运行时）

> + hash
> + chunkhash
> + contenthash

#### 代码分割

+ 打包优化
+ 动态module引入

```js
// 1 将node_modules中打包为单独文件
optimization: {
    splitChunks: {
        chunks: 'all'
    }
}

// 2 动态引入module
import()
```

#### externals

> 在index.html中通过cdn引入的模块，不需要打包

#### PWA 渐进式网络App（离线可访问）

``` js
// npm i workbox-webpack-plugin -D

// 1
new WorkboxWebpackPlugin.GenerateSW({
    clientClaim: true,
    skipWaiting: true
});
// 2 注册serviceWorker。必须运行在服务器
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
        navigator.serviceWorker
            .register('/service-worker.js')
            .then(() => console.info('注册成功'))
            .catch(() => console.error('注册失败'))
    });
}
```

#### dll（对某些第三方库单独打包）
### 一个JS的静态模块打包工具

``` js
// 生成package.json
npm init -y
// 打包工具 webpack
npm install webpack --save-dev
// 开发工具
npm install lodash --save
// 由于浏览器不支持ES6，所以需要babel将ES6语法转换为ES5语法
npm install --save-dev @babel/core @babel/cli @babel/preset-env
// ES6新特性，需要使用polyfill 完善浏览器对新方法的定义
npm install --save @babel/polyfill

// 打包
webpack ./app.js -o ../dist/bundle.js

// webpack4 需要安装cli
npm install webpack -g
npm install webpack-cli -g
```

#### loader

+ npm安装相应的loader
+ 在webpack.config.js的modules下配置

``` js
// CommonJS规范
module.exports = {
    
}
require('http');

// ES6规范
import Vue from 'vue';
export const name = 'hhh';
```


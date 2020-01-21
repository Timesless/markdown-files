# JavaScript

### 匿名函数

``` js
// 匿名函数
(function() {})
// 匿名函数复制给变量，相当于function f() {}
var f = (function() {});
// 匿名函数，可“自执行”
(function() { console.info(1) })();

// 函数作为参数传递
function f() {console.info('1')}
(function(f) { f(); })(f);

// 函数作为返回值
var f = (function() { return (function() { console.info(1) }) })();
// 执行f
f();

// 箭头函数
var f = (x => x+x);
console.info(f(3));

```

### Object

```js
Object.keys(p); 
Object.values(p);
Object.assign({a:1}, {b:2});  // {a:1, b:2}
```

### _Map & Set

``` js
var map = new Map();
var m = new Map([['a', 1], [2, 'b']]);
map.set('a', 1);
map.set(2, 'b');
map.has(2);
map.get(2);
map.delete(2);
console.log(map.keys() + map.values() + map.entries());

var set = new Set();
var s = new Set([1, 2, 3, 3]);
set.add(1);
set.add('2');
set.delete(1);
console.info(set.size);
// 遍历
map | set .forEach(function(v, k, s) {
    // s 自身对象
});
```

### 闭包

``` js
// 当一个嵌套的内部函数引用了外部函数的变量 | 函数，产生闭包
function f() {
    var a = 1;
    function f2() { console.info(a); }
}
```



### Prototype

```js
function Person(name, age) {
	this.name = name;
	this.age = age;
}
var p = new Person("hello", 20);
// 为构造器添加方法
Person.prototype.sex = function(sex) { this.sex = sex; }
p.sex = '男';
console.log(JSON.stringify(p));
```

### Iterator

+ 数组

``` js
var arr = [1, 2, 3];
1. for (let t of arr) { console.info(t); }
2. arr.forEach(function(e, idx, ary) {
    console.info(e);
    // ary 代表数组本身
})
```

+ 对象

``` js
var obj = {a: 1, b: 2};
for (let k in obj) { console.info(obj[k]); }
```

### 时间函数

``` js
// 1小时后的时间对象
new Date(new Date().getTime() + (60 * 60 * 60 * 1000));
new Date().toLocaleString(); // 2020-1-5 20:21:44
toLocaleDateString();
```

### Window

``` js
// 窗口信息
window.inner | outer Width | Height
// 分辨率
screen.width | height
// 浏览器信息
navigator.appName | .language | .platform
// URL
location.href = URL == location.assign(URL)
location.reload();
// history
history.back() | forward()
```

### FileReader

``` js
<form action='?' method='post' enctype='multipart/form-data'>
<input type='file' name='theFile' />
// H5针对表单文件上传提供File和FileReader
var reader = new FileReader();
// 回调readAsDataURL
reader.onload = function(e) {
    $('#div').backgroundImage = 'url(' + e.target.result +')';
}
reader.readAsDataURL($('#theFile').files[0]);
```

# JQuery

### load

``` js
// 页面元素加载完成，触发
window.onload = function() { ... }
$(function() { ... });
```

### Dom

``` js
// 二级查找
$('div p') == $('div').find('p')
// 筛选
$('div').parent();
		.first();
		.eq(idx); // 获取元素集合第idx个元素
		.last();
		.next();
// 元素遍历
$('ul li').each(function(idx, e) {
    console.info(idx + e);
});

// 操作元素
$('#div').html();	// 无参是获取内容，带参是修改内容
// 属性
$().attr('class') | .removeAttr()
$().attr('class', 'red');	// 修改属性
// 样式
$().css('color', '#ccc');
$().css('background', '#ccc');
```

### 表单

``` javascript
$('[type=checkbox]').attr('checked', true)
$('[type=checkbox]:checked')
```

### toggle

``` js
// 切换
toggle();
slideToggle();
fadeToggle();
```




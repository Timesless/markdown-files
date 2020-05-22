#### 获取github所有用户

``` js
// 回调地狱
$.get('https://api.github.com/users', function(data) {
    $.get('', function(datas) { console.log(datas); });
});
// axios库Promise
const userPromise = axios.get('https://api.github.com/users');
userPromise.then(resp => {
    let name = resp.data[0].login;
    return axios.get('https://api.github.com/users/{name }/repos')
})
.then(resp => console.log(resp.data))
.catch(e => console.error(e));
```



### Promise

``` js
// 调用Promise构造函数创建
const promise = new Promise((resolve, reject) => {
    // do logic
    let rs = doLogic();
    setTimeout(() => {
        if (rs) {
            resolve('awesome success!');
        } else {
            reject(Error('failed!'));
        }
    }, 1000);
});
promise.then(data => console.info(data)).catch(e => console.error(e));

/**
 * Promise.all([]) 所有promise成功则resolve，其中一个失败则reject
 * Promise.race([])以第一个promise返回的结果决定调用resolve | reject
 */
Promise.all([userPromise, moviePromise])
.then(response => {
    const [user: users, movie: movies] = response;
    console.log(users);
    console.info(movies);
})
.catch(e => console.error(e));

/**
 * 立即resolved的Promise是在本轮事件循环末尾执行，总是晚于本轮循环的同步任务
 * 一般来说（良好习惯），调用resolve | reject之后，Promise任务就算完成，后继的任务应放在then里面
 * 比较好的做法是加上return语句
 */
// 2 1
new Promise((resolve, reject) => {
  resolve(1);
  console.log(2);
}).then(r => console.log(r));
// GOOD
new Promise((resovle, reject) => {
    return resovle(1);
    console.info(2);
})
```



### Symbol

``` js
// 唯一标识
const peter = Symbol();
const lili = Symbol();

// ESLint
```



### modules

``` js
// config.js 默认导出
export default const k1 = 'default export';
export const k2 = 'export2';
export const k3 = 'zzz';
export function func() { console.info('hh') }

// app.js 入口文件
import defaultExport {k2 as ktwo, k3, func} from './cofig.js'
```

### js面向对象

``` js
/**
 * class不存在函数提升
 * 继承时必须指定调用父类构造即super();
 * class myCollection extends Array {
 * constructor(name, ...args) { super(...args); this.name = name;  }
 * add(movie) { this.push(movie) }
 * }
 */
class User {
    constructor(name, email) {
        this.name = name;
        this.email = email;
    }
    info() { console.log(`I'm ${this.name }, my email is ${this.email }`) }
    // User.desc();
    static desc() { console.log(`I'm a github user`); }
    set github(value) { this.githubName = value; }
    get github() { return this.githubName; }
}

typeof User // function
const laravist = new User('laravist', 'i@lara.com');
User.desc();
```



### 遍历器

``` js
/**
 * Array，Map，Set
 */
const arr = ['apple', 'banana', 'cherries'];
const ae = arr.entries();
const ak = arr.keys();
const av = arr.values();
console.log(ae.next()); ak.next(); av.next();

// 自定义遍历器
Array.prototype.values = function() {
    let i = 0, items = this;
    return {
        next() {
            const done = i >= items.length;
            const value = done ? undefined : items[i++];
            return {
                value,
                done
            }
        }
    }
}
```



### Generator生成器

``` js
function req() {
    axios.get(url).then(resp => userGenerator.next(resp.data)).catch(e => console.error(e));
}
// generator function*，从上到下依次执行
function* generatorSteps() {
    const users = yield req('https://api.github.com/users');
    const firstUser = yield req(`https://api.github.com/users/${users[0].login }`);
    const followers = yield req(`https://api.github.com/${firstUser.follower_url }`);
    console.log(followers);
}
const userGenerator = generatorSteps();
userGenerator.next();
```



### Proxy

``` js
// 重写对象的默认方法，属性等，定义自己的逻辑
const person = {name: ' hh ', email = 'i@hh.com'};
// new Proxy(obj, handler);
const proxy = new Proxy(person, {
    set(target, k, v) { if (typeof v === 'string') { target[k] = v.trim(); } }
    get(target, k) {
    	return target[k].toUpperCase();
	}
});
proxy.name; //HH

/**
 * 格式化电话号码
 */
const phoneHandler = {
    set(target, k, v) {
        target[k] = v.match(/[0-9]/g).join('');
    },
    get(target, k) {
        return target[k].replace(/\d{3})(\d{4})(\d{4})/, '$1-$2=$3');
    }
};
const person =  { phone: 131 1232 3434 };
const _p_proxy = new Proxy([phone1, phone2], phoneHandler);
```



### 数组

``` js
/**
 * Array.from()将类数组，可迭代结构转换为数组
 * Array.of() 生成数组
 */
Array.from(arguments).reduce((prev, cur) => prev + cur, 0);
console.info(1, 2, 3);
Array.of(1, 2);

/**
 * find()
 * findIndex()
 * some()
 * every()
 */
const inventory = [{name: 'apples', quantity: 2},
    {name: 'bananas', quantity: 0},
    {name: 'chrries', quantity: 5}];
inventory.find(fruit => console.info(fruit.name));
```



### Set & WeakSet

``` js
const s = new Set(['apple', 'banana', 'cherry']);
const number = [1, 1, 2, 3, 4, 4, 5, 5];
const numSet = new Set(number);
const uniqueNum = [...numSet];
// weakset元素只能是对象，方便GC
```



### 扩展运算符

``` js
const young = [22, 21, 19];
const old = [47, 54, 49];
const all = [...young, 30, ...old]
const newAll = [...all]

const fruit = ['apple', 'banana', 'pear'];
const newFruit = ['orange', 'mongo'];
fruit.push(...newFruit);  // push传入多个参数
const dateFields = [2020, 01, 20];
new Date(...dateFields)
```

## Rest arguments

``` js
const result = [100010, 'hhh', 89, 68, 92, 106, 136, 124, 70];
const [id, name, ...scores] = result;
console.log(`编号：${id }，姓名：${name }的分数为： ${scores }`);
```



### 解构

+ 数组解构

``` javascript
let a = 10, b = 20;
// 交换ab的值 a = a ^ b; b = a ^ b; a = a ^ b;
[a, b] = [b, a]
```

+ 对象解构

``` js
const name = 'Laravist', age = 2, birthday = '2015-09';
const laravist = {
    name: name,
    age,
    birthday
}
console.log(laravist);
```



### 遍历

+ Iterable 结构使用for of

+ 对象使用 for in




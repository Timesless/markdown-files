> 基础API调用记录

``` js
document.querySelector('video').playbackRate = 2;
```

#### axios

``` js
// 基本使用
axios(config)
axios.request(config)
axios.get(url, config)
axios.post(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])

axios({
    url: 'http://...',
    method: 'get / post'
    params: {
    	param1: 'v1',
    	param2: 'v2'
	}
}).then(resp => {
    logic();
    console.info(resp);
}).catch(err => console.error(err))

// 2 都成功执行then
const a1 = axios({ ... })
const a2 = axios({ ... })                  
axios.all(a1, a2)
	.then(res => console.info(res))
	// .then(axios.spread((res1, res2) => { console.info(res1); console.info(res2); })
	.catch(err => console.error(err))

// 设置默认信息
axios.defaults.baseURL = 'http://localhost:8080'
axios.defaults.timeout = 4000
```


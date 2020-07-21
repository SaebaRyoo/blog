### axios中统一的封装了如下的问题
- 支持浏览器请求 [XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) 请求
- 支持node中的http模块的请求 [http](http://nodejs.org/api/http.html) 请求
- 支持 [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) API
- 可以拦截请求和响应
- 转换request和response数据
- 取消请求
- 自动转换JSON数据
- 默认增加解决XSRF（CSRF）攻击的参数 [XSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)


1. axios中是如何实现在浏览器和node中发起http请求的？且支持Promise调用

> 在axios中，针对浏览器和node环境中，分别通过Promise配合XMLHttpRequest和http模块，来实现了两个不同环境下的adapter，具体代码如下:
```js
function getDefaultAdapter() {
  // 判断当前是在浏览器环境还是node环境
  var adapter;
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('./adapters/xhr');
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    adapter = require('./adapters/http');
  }
  return adapter;
}
```

2. 它是如何拦截请求的？

当你通过axios发起请求时，且注册了`request interceptors` 或者 `response interceptors`后，axios会在初始化阶段创建一个`var chain = [dispatchRequest, undefined]`数组。然后会将注册的`request interceptors` 和 `response interceptors` 分别插入到`chain`的头部和尾部。
最后再通过promise顺序执行链式调用。完成请求和响应的拦截功能。

```js
  // Hook up interceptors middleware
  // 初始化的chain中的dispatchRequest就是执行的http请求
  // 添加拦截器中间件
  var chain = [dispatchRequest, undefined];
  // 初始化一个promise实例
  var promise = Promise.resolve(config);

  // 将request拦截器添加到chain的头
  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  // response拦截器添加到chain的尾部
  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  // 在向chain的头尾添加拦截器后，按照promise的then，顺序执行
  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }

  return promise;
```


3. 它是如何取消请求的呢？

如果熟悉`XMLHttpRequest`的方法，那我们应该知道一个`abort`。对，在axios中同样是通过abort来实现的取消请求。在`xhr.js`中有这么一段代码

```js
  // 首先需要在request config中配置cancelToken，
  if (config.cancelToken) {
    // Handle cancellation
    config.cancelToken.promise.then(function onCanceled(cancel) {
      if (!request) {
        return;
      }

      // 通过abort终止当前请求
      request.abort();
      reject(cancel);
      // Clean up request
      request = null;
    });
  }

```

实际应用中，我们可以通过cancelToken来记录请求url，以避免重复请求。并且可创建一个白名单，以防止一些需要多次请求的接口

```js
import axios from 'axios';


const whiteList = ['/youapi/1', '/youapi/2']; //这些接口不进行 防重复提交

const requestMap = new Map();
const CancelToken = axios.CancelToken;
// 防止重复请求
const removeRequest = (mapkey) => {
	const cancel = requestMap.get(mapkey);
	cancel('请勿重复操作');
	requestMap.delete(mapkey);
};

// 过滤请求
axios.interceptors.request.use(
  config => {
    /** 其他的配置 **/

    // 使用的是请求方法和请求url组成的key
		const mapkey = `${config.method}-${config.url}`;
		if (requestMap.has(mapkey)) {
      // 上删除上一次未完成的请求
			removeRequest(mapkey);
		}
		if (!requestMap.has(mapkey)) {
			config.cancelToken = new CancelToken(function executor(c) {
        // An executor function receives a cancel function as a parameter
				if (config.url && whiteList.indexOf(config.url) === -1) {
					requestMap.set(mapkey, c);
				}
			});
		}
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

// 添加响应拦截器
axios.interceptors.response.use(
  response => {
		const mapkey = `${response.config.method}-${response.config.url}`;
    // 在一个ajax响应后再执行一下取消操作
		if (requestMap.has(mapkey)) {
			removeRequest(mapkey);
		}
  },
  error => {
    
    // 在一个ajax响应后再执行一下取消操作
    const mapkey = `${error.config.method}-${error.config.url}`;
    if (requestMap.has(mapkey)) {
      removeRequest(mapkey);
    }
    return Promise.reject(error);
  }
);

// 添加响应拦截器
axios.interceptors.response.use(
  response => {
    /* 一些你自己的配置 */
    removePending(response.config, 'response'); // 在一个ajax响应后再执行一下取消操作，把已经完成的请求从pending中移除  下次请求同样的url就不会执行
  },
  error => {
    /* 一些你自己的配置 */
    return Promise.reject(error);
  }
);
```

4. axios中如何解决的XSRF（CSRF）安全问题？
在`defaults.js`中，有一些默认的config配置, 这需要后端来协同。具体文章可[查看](http://www.qiutianaimeili.com/html/page/2019/03/lhx9z5xt45.html)
```js
{
  xsrfCookieName: 'XSRF-TOKEN', // 默认值， 表示如果需要防止XSRF攻击，则需要后端生成一个key为XSRF-TOKEN的session（当然，也可以和后端协商，然后自己写config来覆盖）
  xsrfHeaderName: 'X-XSRF-TOKEN', // 然后在axios中自动的读取cookie中的值并在header中写入一个自定义的头X-XSRF-TOKEN
}
```

读取的操作在`xhr.js`中
```js

    if (utils.isStandardBrowserEnv()) {
      var cookies = require('./../helpers/cookies');

      // Add xsrf header
      // (当配置了允许访问cookie设置 || 访问该请求在同一个源 ) && config.xsrfCookieName
      var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
        cookies.read(config.xsrfCookieName) :
        undefined;

      if (xsrfValue) {
        requestHeaders[config.xsrfHeaderName] = xsrfValue;
      }
    }
```
# uni_cloud_router

欢迎使用 uni_cloud_router。此工具是本人在短暂从事“全栈”开发过程中，对云函数路由相关功能实现总结的一个工具。之前做 node 用过 express/koa ，所以这个小工具的语法风格和 koa-router/express 路由风格类似，但是只实现了基础功能，但至少在我使用期间，这些简陋的功能足以够用。

uni_cloud_router 主要解决的问题是在单个云函数内运行多个业务接口。处于各种原因，我们很少会采用一个接口对应一个云函数的做法。这时，就考虑到要在一个云函数内运行多个接口。简单的说，就是一个云函数，跑一个项目。我们可以根据云函数 event 中的信息判断 path 和 method。uni_cloud_router 就是对这个判读的封装。

koa/express 的路由风格基本上都是 app/router[method]('/xxx', cb...)。uni_cloud_router 借鉴了这种风格。不过略有不同，针对云函数的场景做了些许适应。

ps:本项目完全基础 uniCloud 。在此鸣谢 uniapp 官方。

## 安装

下载 uni_cloud_router 源码。在 common 目录下新建公共模块 router/index.js 并将 uni_cloud_router 代码粘贴保存即可。

![](https://uniapp.yinchengnuo.com/img/router1.png)

## 初始化

```javascript
exports.main = async (event, context) => { // 新建云函数
	const router = require('router')(event, context) // 引入 router 并实例化
	
	router.interceptors.request.use(async route => { // 实例化后可以在这里使用请求拦截器
		route.params.requestAppend = { message: '请求拦截器追加数据' }
		route.interceptor.interceptorAppend = { message: '请求拦截器传递给路由业务处理的数据' }
		// router.response = { code: 233, message: '请求拦截器响应', route } // 如果要在请求拦截器中终止请求可以直接在拦截器中将 router.response 赋值
	})

	await router.get('/', async ({ params, interceptor }) => {
		router.response = { code: 200, message: 'GET success' }
	})
	
	await router.post('/', async ({ params, data, interceptor }) => {
		router.response = { code: 200, message: 'POST success' }
	})
	
	;(async () => { // 响应拦截器相关处理
		router.response.data.responseAppend = { message: '响应拦截器追加数据' }
	})()
	
	return router.response || new Error('404 not found')
}
```

## url化请求

上传云函数，并在详情页配置url化和url路径

![](https://uniapp.yinchengnuo.com/img/router2.png)

然后你就可以在 [https://21d91afa-8266-426f-ada2-b23e9f16be9d.bspapp.com/http/index](https://21d91afa-8266-426f-ada2-b23e9f16be9d.bspapp.com/http/index) 看到接口响应的结果了

## 模块化

通过 await router[method]('/', ...) 的方式使得接口路径和方式很清晰。但是再小的项目，也不可能把所有的接口全部写在一个文件里，除了极个别。因此基本上所有的路由工具都可以设置 prefix 来进行模块化。uni_cloud_router 也不例外。使用：

```javascript
exports.main = async (event, context) => { // 新建云函数
	const router = require('router')(event, context) // 引入 router 并实例化
	
	router.interceptors.request.use(async route => { // 实例化后可以在这里使用请求拦截器
		route.params.requestAppend = { message: '请求拦截器追加数据' }
		route.interceptor.interceptorAppend = { message: '请求拦截器传递给路由业务处理的数据' }
		// router.response = { code: 233, message: '请求拦截器响应', route } // 如果要在请求拦截器中终止请求可以直接在拦截器中将 router.response 赋值
  })
  
  ///////////////////////////////////////////////////////////////////////
  router.response = await require('./modules/module1.js')(event, context) // module1 模块
  router.response = await require('./modules/module2.js')(event, context) // module2 模块
  ///////////////////////////////////////////////////////////////////////


	await router.get('/', async ({ params, interceptor }) => {
		router.response = { code: 200, message: 'GET success' }
	})
	
	await router.post('/', async ({ params, data, interceptor }) => {
		router.response = { code: 200, message: 'POST success' }
	})
	
	;(async () => { // 响应拦截器相关处理
		router.response.data.responseAppend = { message: '响应拦截器追加数据' }
	})()
	
	return router.response || new Error('404 not found')
}
```

模块内部：

```javascript
// module1
module.exports = async (event, context) => {
	const router = require('router')(event, context, { prefix: '/module1'})
	
	await router.get('/', async ({ params, interceptor }) => {
		router.response = { code: 200, data, message: 'https://21d91afa-8266-426f-ada2-b23e9f16be9d.bspapp.com/http/index/module1' }
	})
	
	return router.response
}

```

请求地址：[https://21d91afa-8266-426f-ada2-b23e9f16be9d.bspapp.com/http/index/module1](https://21d91afa-8266-426f-ada2-b23e9f16be9d.bspapp.com/http/index/module1)

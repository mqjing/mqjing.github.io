---
title: 【JS】Axios的封装
date: 2021-08-27 21:33
---

一般项目中我们会在项目的 `src` 目录中新建一个 `request` 文件夹，里边放一个 `http.js` 和 `api.js` 文件，`http.js` 用来封装 `axios` ，`api.js` 用来统一管理 api 接口。
## 一 安装和引入
```javascript
// 首先通过安装命令 npm install axios 安装axios
// 引入
import axios from 'axios';
// 引入 qs 模块，此模块用来序列化 post 类型的数据
import qs from 'qs';
// 引入 toast 提示组件
import { Toast } from 'Element-UI'
```
## 二 配置 baseURL
由于项目会有不同的环境比如：开发环境、测试环境、生产环境。所以我们需要通过 node 的环境变量来匹配我们默认的 baseURL。可以通过设置 `axios.defaults.baseURL` 来设置 axios 的默认请求地址。
```javascript
if (process.env.NODE_ENV === 'development') {
  axios.defaults.baseURL = 'http://www.kaifa.com';
} else if (process.env.NODE_ENV === 'debug') {
  axios.defaults.baseURL = 'http://www.debug.com';
} else if (process.env.NODE_ENV === 'production') {
  axios.defaults.baseURL = 'http://www.production.com';
}
```
## 三 设置请求超时
通过设置 `axios.defaults.timeout` 来设置默认的请求超时时间，假如超过了这个时间就会提醒用户当前请求超时等信息。
## 四 设置 post 请求头
post 请求的时候，需要设置一个请求头，可以用 `axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded;charset=UTF-8';` 来设置。
## 五 设置请求拦截器
我们在发送请求之前要对请求进行一个拦截，比如有的接口是需要用户登陆之后才能访问的，或者 post 请求的时候，我们需要序列化我们提交的数据，这时候，我们可以在请求发送之前设置一个拦截器，从而进行我们需要的操作。
```javascript
// 先导入 vue-x，因为要用到里面的状态对象
import store from '@/store/index'
// 设置请求拦截器
axios.interceptors.request.use(
	config => {
    // 每次发送请求之前判断 vuex 中是否存在 token
    // 如果存在，则统一在 http 请求的 header 中加上 token
    // 即使存在，token 也可能是过期的，所以要对返回状态进行判断
    const token = store.state.token;
    token && (config.headers.Authorization = token);
    return config;
  },
  error => {
    return Promise.error(error);
  }
)
```
## 六 设置相应拦截器
与请求拦截器同理，我们可以对服务器返回来的数据提前做一些处理，如果后台返回的状态码是 200 ，则正常返回数据，如果返回的是其他错误码，那我们可以做一个统一的错误处理。
```javascript
axios.interceptors.response.use(
	response => {
    // 如果状态码是 200 ，说明接口请求成功，可以正常拿到数据
    // 否则跑出错误
    if (response.state === 200) {
      return Promise.resolve(response)
    } else {
      return Promise.reject(response)
    }
    // 状态码不是 200 的时候
    // 可以跟后端人员协商好统一的错误状态码
    // 然后根据返回的状态码进行一些操作，例如登陆过期提示，错误提示等等。
    error => {
      if (error.response.status) {
        switch(error.response.status){
          // 401: 未登录
          // 未登录则需要跳转到登陆页面，并携带当前页面的路径
          // 携带当前路径是为了登陆成功后返回当前页面，这一步需要在登陆页操作
          case 401:
            router.replace({
            	path: '/login',
              query: {
              	redirect: router.currentRoute.fullPath
              }
            });
            break;
            
          // 403: token 过期
          // 提示用户登陆过期
          // 清除本地 token 和清空 vuex 中的 token 对象
          case 403: 
            Toast({
            	message: '登陆过期，请重新登陆',
              duration: 1000,
              forbidClick: true
            })
            // 清除 token
            localStorage.removeItem('token');
            store.commit('loginSuccess', null);
            // 跳转登陆页面，并将要浏览的页面 fullPath 传过去，登陆成功后跳转要访问的页面
            setTimeout(()=>{
            	router.replace({
              	path: '/login',
                query: {
                	redirect: router.currentRoute.fullPath
                }
              });
            }, 1000);
            break;
          
          // 404: 请求不存在
          case 404:
            Toast({
            	message: '网络请求不存在',
              duration: 1500,
              forbidClick: true,
            });
            break;
            
          // 其他错误，直接抛出错误提示
          default: 
            Toast({
            	message: error.response.data.message,
              duration: 1500,
              forbidClick: true
            });
        }
        return Promise.reject(error.response)
      }
    }
  }
)
```
## 七 封装 get 和 post 方法
**get 方法：**定义一个 get 函数，该函数有两个参数，第一个表示我们要请求的 url 地址，第二个表示要携带的参数。get 函数返回一个 promise 对象，当 axios 请求成功时 resolve 服务器返回的内容，请求失败时 reject 错误值。最后通过 export 抛出 get 函数
```javascript
export function get(url, params) {
	return new Promise((resolve, reject) => {
  	axios.get(url, {
    	params: params
    }).then(res => {
    	resolve(res.data);
    }).catch(err => {
    	reject(err.data);
    })
  })
}
```
**post 方法：**原理跟 get 方法一样，但是 post 方法必须要对提交的参数对象进行序列化操作，所以这里使用 qs 模块来序列化我们的参数。
```javascript
export default post(url, params) {
	return new Promise((resolve, reject) => {
  	axios.post(url, qs.stringify(params))
    .then(res => {
    	resolve(res.data);
    })
    .catch(err => {
    	reject(err.data);
    })
  })
}
```
​

原文链接：[axios请求封装](https://www.cnblogs.com/chaoyuehedy/p/9931146.html)
​


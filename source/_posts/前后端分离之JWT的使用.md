---
title: 前后端分离之JWT的使用
date: 2017-06-07 15:23:05
tags: JWT
---


## 前端 Vue.js

### vue-router 
登录时，将后端返回的 token 存入 LocalStorage

使用 Vue-Router 判断是否存在 token，不存在跳转至登录

```javascript
// JWT 用户权限校验，判断 TOKEN 是否在 localStorage 当中
router.beforeEach(({name}, from, next) => {
  // 获取 JWT Token
  if (localStorage.getItem('JWT_TOKEN')) {
    // 如果用户在login页面
    if (name === 'login') {
      next('/');
    } else {
      next();
    }
  } else {
    if (name === 'login') {
      next();
    } else {
      next({name: 'login'});
    }
  }
});
```
<!-- more -->
### axios
axios 全局配置 inter
每次向后端请求携带 头信息

```javascript
// http request 拦截器
axios.interceptors.request.use(
  config => {
    if (localStorage.JWT_TOKEN) {  // 判断是否存在token，如果存在的话，则每个http header都加上token
      config.headers.Authorization = `token ${localStorage.JWT_TOKEN}`;
    }
    return config;
  },
  err => {
    return Promise.reject(err);
  });

// http response 拦截器
axios.interceptors.response.use(
  response => {
    return response;
  },
  error => {
    if (error.response) {
      console.log('axios:' + error.response.status);
      switch (error.response.status) {
        case 401:
          // 返回 401 清除token信息并跳转到登录页面
          store.commit('LOG_OUT');
          router.replace({
            path: 'login',
            query: {redirect: router.currentRoute.fullPath}
          });
      }
    }
    return Promise.reject(error.response.data);   // 返回接口返回的错误信息
  });

Vue.prototype.$http = axios;
```


## 后端 CI

后端 Controller 接收请求头信息，验证 token 有效性，无效返回 401 状态码

```php
$header = $this->input->get_request_header('Authorization', TRUE);
        if ($header != '' && jwt_helper::validate($header)) {
                $result = $this->user_model->add_user();
                echo json_encode($result);
        } else {
            show_error("Permission denied", 401, "Please check your token.");
        }
```

封装好的 Helper 类

```
<?php

class jwt_helper extends CI_Controller
{
    const CONSUMER_KEY = 'dididasan';
    const CONSUMER_SECRET = 'dididasan';
    const CONSUMER_TTL = 86400;

    // 生成 token
    public static function create($userid)
    {
        $CI =& get_instance();
        $CI->load->library('JWT');
        $token = $CI->jwt->encode(array(
            'consumerKey' => self::CONSUMER_KEY,
            'userId' => $userid,
            'issuedAt' => date(DATE_ISO8601, strtotime("now")),
            'ttl' => self::CONSUMER_TTL
        ), self::CONSUMER_SECRET);
        return $token;
    }

    // 验证 token 有效性
    public static function validate($header)
    {
        $CI =& get_instance();
        $CI->load->library('JWT');
        list($token) = sscanf($header, 'token %s');
        try {
            $CI->jwt->decode($token, self::CONSUMER_SECRET);
            return true;
        } catch (Exception $e) {
            return false;
        }

    }

    // 解码 token
    public static function decode()
    {
        $CI =& get_instance();
        $CI->load->library('JWT');
        list($token) = sscanf($header, 'token %s');
        try {
            $decodeToken = $CI->jwt->decode($token, self::CONSUMER_SECRET);
            return $decodeToken;
        } catch (Exception $e) {
            return false;
        }
    }
}
```



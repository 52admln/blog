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
axios 全局配置 interceptors
每次向后端请求携带 头信息

在 `src/main.js` 当中加上以下代码：

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

封装好的 JWT Helper 类
将代码复制，保存为 `jwt_helper.php` 放入 `application/helpers` 目录下

```php
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

JWT 类
将代码复制，保存为 `JWT.php` 放入 `application/libraries` 目录下

```php
<?php
/**
 * JSON Web Token implementation
 *
 * Minimum implementation used by Realtime auth, based on this spec:
 * http://self-issued.info/docs/draft-jones-json-web-token-01.html.
 *
 * @author Neuman Vong <neuman@twilio.com>
 * @minor changes for codeigniter <b3457m0d3@interr0bang.net>
 */
class JWT
{
    /**
     * @param string      $jwt    The JWT
     * @param string|null $key    The secret key
     * @param bool        $verify Don't skip verification process 
     *
     * @return object The JWT's payload as a PHP object
     */
    public function decode($jwt, $key = null, $verify = true)
    {
        $tks = explode('.', $jwt);
        if (count($tks) != 3) {
            throw new UnexpectedValueException('Wrong number of segments');
        }
        list($headb64, $payloadb64, $cryptob64) = $tks;
        if (null === ($header = $this->jsonDecode($this->urlsafeB64Decode($headb64)))
        ) {
            throw new UnexpectedValueException('Invalid segment encoding');
        }
        if (null === $payload = $this->jsonDecode($this->urlsafeB64Decode($payloadb64))
        ) {
            throw new UnexpectedValueException('Invalid segment encoding');
        }
        $sig = $this->urlsafeB64Decode($cryptob64);
        if ($verify) {
            if (empty($header->alg)) {
                throw new DomainException('Empty algorithm');
            }
            if ($sig != $this->sign("$headb64.$payloadb64", $key, $header->alg)) {
                throw new UnexpectedValueException('Signature verification failed');
            }
        }
        return $payload;
    }
    /**
     * @param object|array $payload PHP object or array
     * @param string       $key     The secret key
     * @param string       $algo    The signing algorithm
     *
     * @return string A JWT
     */
    public function encode($payload, $key, $algo = 'HS256')
    {
        $header = array('typ' => 'jwt', 'alg' => $algo);
        $segments = array();
        $segments[] = $this->urlsafeB64Encode($this->jsonEncode($header));
        $segments[] = $this->urlsafeB64Encode($this->jsonEncode($payload));
        $signing_input = implode('.', $segments);
        $signature = $this->sign($signing_input, $key, $algo);
        $segments[] = $this->urlsafeB64Encode($signature);
        return implode('.', $segments);
    }
    /**
     * @param string $msg    The message to sign
     * @param string $key    The secret key
     * @param string $method The signing algorithm
     *
     * @return string An encrypted message
     */
    public function sign($msg, $key, $method = 'HS256')
    {
        $methods = array(
            'HS256' => 'sha256',
            'HS384' => 'sha384',
            'HS512' => 'sha512',
        );
        if (empty($methods[$method])) {
            throw new DomainException('Algorithm not supported');
        }
        return hash_hmac($methods[$method], $msg, $key, true);
    }
    /**
     * @param string $input JSON string
     *
     * @return object Object representation of JSON string
     */
    public function jsonDecode($input)
    {
        $obj = json_decode($input);
        if (function_exists('json_last_error') && $errno = json_last_error()) {
            $this->handleJsonError($errno);
        }
        else if ($obj === null && $input !== 'null') {
            throw new DomainException('Null result with non-null input');
        }
        return $obj;
    }
    /**
     * @param object|array $input A PHP object or array
     *
     * @return string JSON representation of the PHP object or array
     */
    public function jsonEncode($input)
    {
        $json = json_encode($input);
        if (function_exists('json_last_error') && $errno = json_last_error()) {
            $this->handleJsonError($errno);
        }
        else if ($json === 'null' && $input !== null) {
            throw new DomainException('Null result with non-null input');
        }
        return $json;
    }
    /**
     * @param string $input A base64 encoded string
     *
     * @return string A decoded string
     */
    public function urlsafeB64Decode($input)
    {
        $remainder = strlen($input) % 4;
        if ($remainder) {
            $padlen = 4 - $remainder;
            $input .= str_repeat('=', $padlen);
        }
        return base64_decode(strtr($input, '-_', '+/'));
    }
    /**
     * @param string $input Anything really
     *
     * @return string The base64 encode of what you passed in
     */
    public function urlsafeB64Encode($input)
    {
        return str_replace('=', '', strtr(base64_encode($input), '+/', '-_'));
    }
    /**
     * @param int $errno An error number from json_last_error()
     *
     * @return void
     */
    private function handleJsonError($errno)
    {
        $messages = array(
            JSON_ERROR_DEPTH => 'Maximum stack depth exceeded',
            JSON_ERROR_CTRL_CHAR => 'Unexpected control character found',
            JSON_ERROR_SYNTAX => 'Syntax error, malformed JSON'
        );
        throw new DomainException(isset($messages[$errno])
            ? $messages[$errno]
            : 'Unknown JSON error: ' . $errno
        );
    }
}
```


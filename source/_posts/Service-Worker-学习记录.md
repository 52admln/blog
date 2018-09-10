---
title: Service Worker 学习记录
date: 2018-09-10 13:43:55
tags:
- pwa
---
## Service Worker 学习记录

### 能干什么？

1. 后台消息传递
2. 网络代理，转发请求，伪造响应
3. 离线缓存
4. 消息推送

对于并发高的活动页面，缓存图片文件和js文件，达到减轻服务器负担的作用。


### 怎么用？

#### 页面中注册 Service Worker


```javascript
if (navigator.serviceWorker) {
    navigator.serviceWorker.register('service-worker.js').then(function(registration) {
        console.log('service worker 注册成功');
    }).catch(function (err) {
        console.log('servcie worker 注册失败')
    });
}
```

#### 缓存文件

```javascript
var cacheFiles = [
  '5BB072CAB1104C1F2E254F7F0EC0EB70.jpg',
  'main.js'
]
var VERSION = 'v1';

// 缓存
self.addEventListener('install', function(event) {
  event.waitUntil(
    caches.open(VERSION).then(function(cache) {
      return cache.addAll(cacheFiles);
    })
  );
});
```

#### 更新版本

```javascript
// 缓存更新
self.addEventListener('activate', function(event) {
  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          // 如果当前版本和缓存版本不一致
          if (cacheName !== VERSION) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
```

#### 缓存文件请求拦截

```javascript
// 缓存图片
self.addEventListener('fetch', function (evt) {
  evt.respondWith(
    caches.match(evt.request).then(function(response) {
      if (response) {
        return response;
      }
      var request = evt.request.clone();
      return fetch(request).then(function (response) {
        if (!response && response.status !== 200 && !response.headers.get('Content-type').match(/image/)) {
          return response;
        }
        var responseClone = response.clone();
        caches.open('my-test-cache-v1').then(function (cache) {
          cache.put(evt.request, responseClone);
        });
        return response;
      });
    })
  )
});

```

### 注意事项

1. 需要 `https` 才可以使用，但是 `localhost 或 127.0.0.1` 除外。
2. 生命周期为： installing、installed、activating、activated
3. 浏览器支持情况桌面端Chrome和Firefox可用，IE不可用。移动端Chrome可用
4. 在微信上的支持情况，iOS设备不支持，安卓支持
4. 只需要更新 `service-worker`的注册文件，更换版本，即可更新资源


### 参考地址：

- 借助Service Worker和cacheStorage缓存及离线开发：https://www.zhangxinxu.com/wordpress/2017/07/service-worker-cachestorage-offline-develop/
- Service Worker初体验：http://www.alloyteam.com/2016/01/9274/
- Service Worker那些事：https://zhuanlan.zhihu.com/p/20040372
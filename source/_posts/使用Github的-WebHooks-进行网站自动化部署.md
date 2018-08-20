---
title: 使用Github的 WebHooks 进行网站自动化部署
date: 2018-08-20 09:50:10
tags:
- Webhooks
- Github
---


## 原理
利用Github在仓库进行操作时，可以通过配置webhook向服务器发送请求，在服务器端接到请求后，使用脚本来自动进行git pull操作。

![image](http://wx3.sinaimg.cn/large/85f4065cgy1ftv5b4a0coj20uy0ly43u.jpg)

<!-- more -->

[图片来源：Github的webhook触发vps上的脚本](https://www.xiajunyi.com/pages/p41.html)

## 构建 Webhook 服务


通过执行
```bash
npm i -g github-webhook-handler
```
来安装  github-webhook-handler 中间件

新建文件 `webhook.js`

```js
var http = require('http')
var createHandler = require('github-webhook-handler')
var handler = createHandler({ path: '/', secret: 'root' })
// 上面的 secret 保持和 GitHub 后台设置的一致
function run_cmd(cmd, args, callback) {
  var spawn = require('child_process').spawn;
  var child = spawn(cmd, args);
  var resp = "";
  child.stdout.on('data', function(buffer) { resp += buffer.toString(); });
  child.stdout.on('end', function() { callback (resp) });
}
http.createServer(function (req, res) {
  handler(req, res, function (err) {
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(7777)
handler.on('error', function (err) {
  console.error('Error:', err.message)
})
handler.on('push', function (event) {
  console.log('Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref);
    run_cmd('sh', ['./deploy.sh',event.payload.repository.name], function(text){ console.log(text) });
})
```

其中 
```
var handler = createHandler({ path: '/', secret: 'root' })
```

`secret` 字段为 Github 中设置的，需要与这里相对应


> 注意，在运行的时候如果提示 `github-webhook-handler is not defined` 未找到 ，可以在目录中执行 `npm link github-webhook-handler`

## 同一服务多个 webhook

当你有多个仓库需要自动部署时，可以在一个服务上开启多个 webhook。

```js
var http = require('http')
var createHandler = require('node-github-webhook')
var handler = createHandler([ // 多个仓库
  {
    path: '/app1',
    secret: 'CUSTOM'
  },
  {
    path: '/app2',
    secret: 'CUSTOM'
  }
])
// var handler = createHandler({ path: '/webhook1', secret: 'secret1' }) // 单个仓库

http.createServer(function (req, res) {
  handler(req, res, function (err) { 
    res.statusCode = 404
    res.end('no such location')
  })
}).listen(7777)

handler.on('error', function (err) {
  console.error('Error:', err.message)
})

handler.on('push', function (event) {
  console.log(
    'Received a push event for %s to %s',
    event.payload.repository.name,
    event.payload.ref
  )
  switch (event.path) {
    case '/app1':
      runCmd('sh', ['./app1_deploy.sh', event.payload.repository.name], function (text) { console.log(text) })
      break
    case '/app2':
      runCmd('sh', ['./app2_deploy.sh', event.payload.repository.name], function (text) { console.log(text) })
      break
    default:
      // 处理其他
      break
  }
})

function runCmd (cmd, args, callback) {
  var spawn = require('child_process').spawn
  var child = spawn(cmd, args)
  var resp = ''
  child.stdout.on('data', function (buffer) {
    resp += buffer.toString()
  })
  child.stdout.on('end', function () {
    callback(resp)
  })
}
```


> 同一服务多个webhook时，最终你的payload URL 则为：`http:/yourdomain:7777/app1` 或者 `http:/yourdomain:7777/app2` ，注意我在实践过程中发现，不能使用 / 目录，会无法监听到 webhook。

参考地址：https://github.com/rvagg/github-webhook-handler/pull/22#issuecomment-274301907

## 完成 shell 脚本

在使用脚本之前，先要对网站根目录做一些处理

```
# 打开网站根目录
cd /home/wwwroot/domain.com
# 采用 Git 文件控制
git init
# 添加远程 Git 仓库地址
git remote add origin https://xx.git 
```

参考地址：
[How do I force “git pull” to overwrite local files?](https://stackoverflow.com/questions/1125968/how-do-i-force-git-pull-to-overwrite-local-files)

然后再创建 `deploy.sh`，与 `webhook.js` 在同一个目录下

```bash
#!/bin/bash
# 网站的根目录
WEB_PATH='/home/wwwroot/domain.com'
 
echo "start deployment"
cd $WEB_PATH
echo "fetching from remote..."
# 为了避免冲突，强制更新本地文件
git fetch --all
git reset --hard origin/master
echo "done"
```

> 由于 Linux 文件权限问题，可能无法执行，建议先执行 `chmod 777 ./deploy.sh`

## 使用pm2进行进程守护

安装pm2：
```
npm i pm2 -g
```

运行webhook.js
```
pm2 start webhook.js
```


## 进入Gtihub后台进行设置

进入需要自动部署的项目的github地址添加webhook，进入Settings设置页面，点击左侧的 Webhooks

![image](http://wx2.sinaimg.cn/large/85f4065cgy1ftv4wb3sbsj21jy172gtd.jpg)


参考地址：

[利用Github的Webhook进行静态网站的自动化部署](http://zhuxiblog.com/2017/03/13/Github-webhook-vps/)



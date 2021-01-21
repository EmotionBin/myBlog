1. service worker 是什么
2. service worker 与 web worker 区别(service worker是一个新的进程，可以在多个网页 page 中共享；web worker 只是一个不同于 js 主线程的新线程)
3. service worker 的注册与安装
4. service worker 信息通讯
5. service worker 静态资源缓存

参考:
1. https://juejin.cn/post/6844903613270081543

# Service Worker

之前讲了 Web worker，后来我还发现有一个东西叫 Service Worker，最近花了点时间去研究学习它，把关于 Service Worker 的内容整理了一下，本文就介绍一下 Service Worker。  

----

## Service Worker 是什么

Service Worker 到底是什么？它与 Web Worker 有什么区别？又有什么联系？  

Service Worker 可以理解为一个介于客户端和服务器之间的一个代理服务器。在 Service Worker 中我们可以做很多事情，比如拦截客户端的请求、向客户端发送消息、向服务器发起请求等等，其中最重要的作用之一就是离线资源缓存。  

与 Web Worker 相比，它有以下相同点：  

1. Service Worker 工作在 worker context 中，是没有访问 DOM 的权限的，所以我们无法在 Service Worker 中获取 DOM 节点，也无法在其中操作 DOM 元素
2. 可以通过 `postMessage` API 把数据传递给其他 JS 文件
3. Service Worker 中运行的代码不会被阻塞，也不会阻塞其他页面的 JS 文件中的代码

不同的地方在于，Service Worker 是一个浏览器中的进程而不是浏览器内核下的线程，因此它在被注册安装之后，**能够被在多个页面中使用，也不会因为页面的关闭而被销毁**。因此，Service Worker 很适合被用与多个页面需要使用的复杂数据的计算。  

另外有一点需要注意的是，出于对安全问题的考虑，**Service Worker 只能被使用在 https 或者本地的 localhost 环境下。**  

----

## 注册和安装

下面就让我们来使用 Service Worker  

如果当前使用的浏览器支持 Service Worker ，则在 window.navigator 下会存在 serviceWorker 对象，我们可以使用这个对象的 register 方法来注册一个 Service Worker。  

```javascript
// index.js
if ('serviceWorker' in window.navigator) {
  navigator.serviceWorker.register('./sw.js', { scope: './' })
    .then(function (reg) {
      console.log('success', reg)
    })
    .catch(function (err) {
      console.log('fail', err)
    })
}
```

在这段代码中，我们先使用 if 判断下浏览器是否支持 Service Worker ，避免由于浏览器不兼容导致的 bug。  

register 方法接受两个参数，第一个是 service worker 文件的路径，请注意：**这个文件路径是相对于 Origin，也就是相对于根路径，而不是当前 JS 文件的目录的；第二个参数是 Serivce Worker 的配置项，可选填，其中比较重要的是 scope 属性。**按照文档上描述，它是 Service Worker 控制的内容的子目录，这个属性所表示的路径不能在 service worker 文件的路径之上，默认是 Serivce Worker 文件所在的目录。  

register 方法返回一个 Promise。如果注册失败，可以通过 catch 来捕获错误信息；如果注册成功，可以使用 then 来获取一个 ServiceWorkerRegistration 的实例。  

注册完 Service Worker 之后，浏览器会为我们自动安装它，因此我们就可以在 service worker 文件中监听它的 install 事件了。  

```javascript
// sw.js
this.addEventListener('install', function (event) {
  console.log('Service Worker install')
})
```

同样的，Service Worker 在安装完成后会被激活，所以我们也可监听 activate 事件  

```javascript
// sw.js
this.addEventListener('activate', function (event) {
  console.log('Service Worker activate')
})
```

这时，我们可以在 Chorme 的开发者工具中看到我们注册的 Service Worker。  

F12 打开控制台，选择 Application，再选择 Service Workers 就可以看到我们注册的 Service Worker。  

在同一个 Origin 下，我们可以注册多个 Service Worker。但是请注意，这些 Service Worker 所使用的 **scope 必须是不相同的。**  

```javascript
// index.js
if ('serviceWorker' in window.navigator) {
  navigator.serviceWorker.register('./sw/sw.js', { scope: './sw' })
    .then(function (reg) {
      console.log('success', reg)
    })
  navigator.serviceWorker.register('./sw2/sw2.js', { scope: './sw2' })
    .then(function (reg) {
      console.log('success', reg)
    })
}
```

----

## 信息通讯

使用 postMessage 方法可以进行 Service Worker 和页面之间的通讯，下面来看一下如何实现。  

----

### 从页面到 Service Worker

首先是从页面发送信息到 Serivce Worker 。  

```javascript
// index.js
if ('serviceWorker' in window.navigator) {
  navigator.serviceWorker.register('./sw.js', { scope: './' })
    .then(function (reg) {
      console.log('success', reg)
      navigator.serviceWorker.controller && navigator.serviceWorker.controller.postMessage("this message is from page")
    })
    .catch(function (err) {
      console.log('fail', err)
    })
}
```

为了保证 Service Worker 能够接收到信息，我们在它被注册完成之后再发送信息，和普通的 `window.postMessage()` 的使用方法不同，为了向 Service Worker 发送信息，我们要在 ServiceWorker 实例上调用 `postMessage()` 方法，这里我们使用到的是 `navigator.serviceWorker.controller` 。  

```javascript
// sw.js
this.addEventListener('message', function (event) {
  console.log(event.data) // this message is from page
})
```

在 service worker 文件中我们可以直接在 `this` 上绑定 message 事件，这样就能够接收到页面发来的信息了。  

对于不同 scope 的多个 Service Worker ，也可以给指定的 Service Worker 发送信息。  

```javascript
if ('serviceWorker' in window.navigator) {
  navigator.serviceWorker.register('./sw.js', { scope: './sw' })
    .then(function (reg) {
      console.log('success', reg)
      reg.active.postMessage("this message is from page, to sw")
    })
  navigator.serviceWorker.register('./sw2.js', { scope: './sw2' })
    .then(function (reg) {
      console.log('success', reg)
      reg.active.postMessage("this message is from page, to sw 2")
    })
}

// sw.js
this.addEventListener('message', function (event) {
  console.log(event.data) // this message is from page, to sw
})

// sw2.js
this.addEventListener('message', function (event) {
  console.log(event.data) // this message is from page, to sw 2
})
```

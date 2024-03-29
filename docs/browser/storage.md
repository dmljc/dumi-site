---
toc: content
order: 6
---

# 存储

> 涉及面试题：有几种方式可以实现存储功能，分别有什么优缺点？什么是 Service Worker？

## cookie，localStorage，sessionStorage，indexDB

我们先来通过表格学习下这几种存储方式的区别

|     特性     |                   cookie                   |       localStorage       | sessionStorage |         indexDB          |
| :----------: | :----------------------------------------: | :----------------------: | :------------: | :----------------------: |
| 数据生命周期 |     一般由服务器生成，可以设置过期时间     | 除非被清理，否则一直存在 | 页面关闭就清理 | 除非被清理，否则一直存在 |
| 数据存储大小 |                     4K                     |            5M            |       5M       |           无限           |
| 与服务端通信 | 每次都会携带在 header 中，对于请求性能影响 |          不参与          |     不参与     |          不参与          |

`cookie` 通常用于存储用户身份，登录状态等。从上表可以看到，已经不建议用于存储，。如果没有大量数据存储需求的话，可以使用 `localStorage` 和 `sessionStorage` 。对于不怎么改变的数据尽量使用 `localStorage` 存储，否则可以用 `sessionStorage` 存储。

对于 `cookie`，我们还需要注意安全性。

|   属性    |                              作用                              |
| :-------: | :------------------------------------------------------------: |
|   value   | 如果用于保存用户登录态，应该将该值加密，不能使用明文的用户标识 |
| http-only |             不能通过 JS 访问 Cookie，减少 XSS 攻击             |
|  secure   |                只能在协议为 HTTPS 的请求中携带                 |
| same-site |     规定浏览器不能在跨域请求中携带 Cookie，减少 CSRF 攻击      |

## Service Worker

`Service Worker` 是运行在浏览器背后的 `独立线程`，一般可以用来实现 `缓存` 功能。使用 Service Worker 的话，传输协议必须为 `HTTPS`。因为 Service Worker 中涉及到请求拦截，所以必须使用 HTTPS 协议来保障安全。

Service Worker 实现缓存功能一般分为三个步骤：

-   首先需要先注册 Service Worker
-   然后监听到 install 事件以后就可以缓存需要的文件
-   那么在下次用户访问的时候就可以通过拦截请求的方式查询是否存在缓存，存在缓存的话就可以直接读取缓存文件，否则就去请求数据。

```js
// index.js
if (navigator.serviceWorker) {
    navigator.serviceWorker
        .register('sw.js')
        .then(function (registration) {
            console.log('service worker 注册成功');
        })
        .catch(function (err) {
            console.log('servcie worker 注册失败');
        });
}
// sw.js
// 监听 `install` 事件，回调中缓存所需文件
self.addEventListener('install', (e) => {
    e.waitUntil(
        caches.open('my-cache').then(function (cache) {
            return cache.addAll(['./index.html', './index.js']);
        }),
    );
});

// 拦截所有请求事件
// 如果缓存中已经有请求的数据就直接用缓存，否则去请求数据
self.addEventListener('fetch', (e) => {
    e.respondWith(
        caches.match(e.request).then(function (response) {
            if (response) {
                return response;
            }
            console.log('fetch source');
        }),
    );
});
```

打开页面，可以在开发者工具中的 Application 看到 Service Worker 已经启动了!

![](/images/browser/render/ServiceWorkers.png)

在 Cache 中也可以发现我们所需的文件已被缓存

![](/images/browser/storage/myCache.png)

当我们重新刷新页面可以发现我们缓存的数据是从 Service Worker 中读取的

![](/images/browser/storage/ServiceIndex.png)

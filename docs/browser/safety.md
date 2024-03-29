---
toc: content
order: 9
---

# 安全相关

## XSS 攻击

`XSS` 即 Cross Site Script，中译是 `跨站脚本攻击`；其原本缩写是 CSS，但为了和层叠样式表(Cascading Style Sheet)有所区分，因而在安全领域叫做 `XSS`。

`XSS` 攻击是指`攻击者在网站上注入恶意的客户端代码`，通过恶意脚本对客户端网页进行篡改，从而在用户浏览网页时，对用户浏览器进行控制或者获取用户隐私数据的一种攻击方式。

攻击者对客户端网页注入的恶意脚本一般包括 `JavaScript`，有时也会包含 `HTML` 和 `Flash`。有很多种方式进行 XSS 攻击，但它们的共同点为：将一些隐私数据像 `cookie`、`session` 发送给攻击者，将受害者重定向到一个由攻击者控制的网站，在受害者的机器上进行一些恶意操作。

`XSS` 攻击可以分为 2 类：`反射型（非持久型）`、`存储型（持久型）`。

### 反射型

`反射型 XSS` 只是简单地把用户输入的数据 “反射” 给`浏览器`，这种攻击方式往往需要攻击者诱使用户点击一个恶意链接，或者提交一个表单，或者进入一个恶意网站时，注入脚本进入被攻击者的网站。最简单的示例是访问一个链接，服务端返回一个可执行脚本：

```js
const http = require('http');
function handleReequest(req, res) {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.writeHead(200, { 'Content-Type': 'text/html; charset=UTF-8' });
    res.write('<script>alert("反射型 XSS 攻击")</script>');
    res.end();
}

const server = new http.Server();
server.listen(8001, '127.0.0.1');
server.on('request', handleReequest);
```

### 存储型

`存储型 XSS` 会把用户输入的数据 "存储" 在`服务器端`，当浏览器请求数据时，脚本从服务器上传回并执行。这种 XSS 攻击具有很强的稳定性。比较常见的一个场景是攻击者在社区或论坛上写下一篇包含恶意 JavaScript 代码的文章或评论，文章或评论发表后，所有访问该文章或评论的用户，都会在他们的浏览器中执行这段恶意的 JavaScript 代码：

```js
// 例如在评论中输入以下留言
// 如果请求这段留言的时候服务端不做转义处理，请求之后页面会执行这段恶意代码
<script>alert('xss 攻击')</script>
```

### XSS 防范

-   `HttpOnly 防止劫取 Cookie`：浏览器将禁止页面的 `Javascript` 访问带有 `HttpOnly` 属性的`Cookie`。上文有说到，攻击者可以通过注入恶意脚本获取用户的 Cookie 信息。通常 Cookie 中都包含了用户的登录凭证信息，攻击者在获取到 Cookie 之后，则可以发起 Cookie 劫持攻击。所以，严格来说，HttpOnly 并非阻止 XSS 攻击，而是能阻止 XSS 攻击后的 Cookie 劫持攻击。

> HttpOnly属性是Set-Cookie HTTP 响应标头的可选属性，由 Web 服务器在 HTTP 响应中与网页一起发送到 Web 浏览器。下面是使用Set-Cookie标头设置会话 cookie 的示例：Set-Cookie: sessionid=QmFieWxvbiA1; HttpOnly


-   `输入检查`：不要相信用户的任何输入。对于用户的任何输入要进行`检查`、`过滤`和`转义`。建立可信任的字符和 HTML 标签`白名单`，对于不在白名单之列的字符或者标签进行过滤或编码。在 XSS 防御中，输入检查一般是检查用户输入的数据中是否包含 <，> 等特殊字符，如果存在，则对特殊字符进行过滤或编码，这种方式也称为 XSS Filter。而在一些前端框架中，都会有一份 decodingMap， 用于对用户输入所包含的特殊字符或标签进行编码或过滤，如 <，>，script，防止 XSS 攻击：

```js
// vuejs 中的 decodingMap
// 在 vuejs 中，如果输入带 script 标签的内容，会直接过滤掉
const decodingMap = {
    '&lt;': '<',
    '&gt;': '>',
    '&quot;': '"',
    '&amp;': '&',
    '&#10;': '\n',
};
```

-   `输出检查`：用户的输入会存在问题，服务端的输出也会存在问题。一般来说，除富文本的输出外，在变量输出到 HTML 页面时，可以使用编码或转义的方式来防御 XSS 攻击。

PS：对于显示`富文本`来说，不能通过上面的办法来`转义所有字符`，因为这样会把需要的`格式也过滤掉`。这种情况通常采用前后端穷举有限标签和属性，进行`白名单`过滤。
<!-- https://www.codercto.com/a/84431.html -->
## CSRF

`CSRF`，即 Cross Site Request Forgery，即 `跨站请求伪造`；`利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作`。

完成 CSRF 攻击必须要有三个条件：

-   用户已经登录了站点 A，并在本地记录了 cookie
-   在用户没有登出站点 A 的情况下（也就是 cookie 生效的情况下），访问了恶意攻击者提供的引诱危险站点 B (B 站点要求访问站点 A)
-   站点 A 没有做任何 CSRF 防御

### Cookie

`Cookie` 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。Cookie 主要用于以下三个方面：

-   会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
-   个性化设置（如用户自定义设置、主题等）

-   浏览器行为跟踪（如跟踪分析用户行为等）

[点击进一步了解 Cookie 和 Session 的区别](https://juejin.im/post/5cd9037ee51d456e5c5babca)

### CSRF 防范

-   `验证码`： 验证码被认为是对抗 CSRF 攻击最简洁而有效的防御方法。从上述示例中可以看出，CSRF 攻击往往是在用户不知情的情况下构造了网络请求。而验证码会强制用户必须与应用进行交互，才能完成最终请求。因为通常情况下，验证码能够很好地遏制 CSRF 攻击。但验证码并不是万能的，因为出于用户考虑，不能给网站所有的操作都加上验证码。因此，验证码只能作为防御 CSRF 的一种辅助手段，而不能作为最主要的解决方案。

-   `同源检测`：在浏览器 HTTP 请求头中有一个字段叫 `origin` 和 `referer`，它记录了该 HTTP 请求的来源地址。通过浏览器的校验，可以检查请求是否来自合法的"源"。
    > origin: 'https://juejin.cn' <br />
    > referer: 'https://juejin.cn/'

-   `添加 token 验证`： 要抵御 `CSRF`，关键在于在`请求中放入攻击者所不能伪造的信息`，并且该信息`不存在于 Cookie 之中`。可以在 HTTP 请求中以参数的形式加入一个随机产生的 `token`，并在服务器端建立一个`拦截器来验证这个 token`，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。

小提示：Vue 中的 v-html 防范 XSS 攻击。

[常见六大 Web 安全攻防解析](https://juejin.im/post/5c446eb1e51d45517624f7db#heading-16) [egg.js 关于安全的处理方案](https://eggjs.org/zh-cn/core/security.html#%E5%AE%89%E5%85%A8%E5%A8%81%E8%83%81csrf%E7%9A%84%E9%98%B2%E8%8C%83)

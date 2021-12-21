###  常见Web安全攻防解析

#### XSS

XSS (Cross-Site Scripting)，跨站脚本攻击，因为缩写和 CSS 重叠，所以只能叫 XSS。XSS 的原理是恶意攻击者往 Web 页面里插入恶意可执行网页脚本代码，当用户浏览该页之时，嵌入其中的脚本代码会被执行，从而可以达到攻击者盗取用户信息或其他侵犯用户安全隐私的目的

XSS 有可能造成以下影响：
* 利用虚假输入表单骗取用户个人信息
* 利用脚本窃取用户的 Cookie 值，被害者在不知情的情况下，帮助攻击者发送恶意请求
* 显示伪造的文章或图片

XSS攻击方式很多，可以大致细分为几种类型：

##### 1. 非持久型（反射型）XSS
一般是指通过给别人发送带有恶意脚本代码参数的 URL，当 URL 地址被打开时，特有的恶意代码参数被 HTML 解析、执行。一些浏览器如 Chrome 内置了一些 XSS 过滤器，可以防止大部分反射型 XSS

非持久型 XSS 漏洞攻击有以下几点特征：

* 即时性，不经过服务器存储，直接通过 HTTP 的 GET 和 POST 请求就能完成一次攻击，拿到用户隐私数据。
* 攻击者需要诱骗点击,必须要通过用户点击链接才能发起
* 反馈率低，所以较难发现和响应修复
* 盗取用户敏感保密信息

为了防止出现非持久型 XSS 漏洞，需要确保这么几件事情：

* Web 页面渲染的所有内容或者渲染的数据都必须来自于服务端
* 尽量不要从 `URL`，`document.referrer`，`document.forms` 等这种 DOM API 中获取数据直接渲染
* 尽量不要使用 `eval, new Function()，document.write()，document.writeln()，window.setInterval()，window.setTimeout()，innerHTML，document.createElement() `等可执行字符串的方法
* 如果做不到以上几点，也必须对涉及 DOM 渲染的方法传入的字符串参数做 escape 转义
* 前端渲染的时候对任何的字段都需要做 escape 转义编码

##### 2. 持久型（存储型）XSS
一般存在于 Form 表单提交等交互功能，如文章留言，提交文本信息等，黑客利用的 XSS 漏洞，将内容经正常功能提交进入数据库持久保存，当前端页面获得后端从数据库中读出的注入代码时，恰好将其渲染执行

主要注入页面方式和非持久型 XSS 漏洞类似，只不过持久型的不是来源于 URL，referer，forms 等，而是来源于后端从数据库中读出来的数据。持久型 XSS 攻击不需要诱骗点击，黑客只需要在提交表单的地方完成注入即可，但是这种 XSS 攻击的成本相对还是很高

攻击成功需要同时满足以下几个条件：

* POST 请求提交表单后端没做转义直接入库
* 后端从数据库中取出数据没做转义直接输出给前端
* 前端拿到后端数据没做转义直接渲染成 DOM

持久型 XSS 有以下几个特点：

* 持久性，植入在数据库中
* 盗取用户敏感私密信息
* 危害面广

##### 3. 如何防御
对于 XSS 攻击来说，通常有3种方式可以用来防御

1. **CSP**

    CSP 本质上就是建立白名单，开发者明确告诉浏览器哪些外部资源可以加载和执行。我们只需要配置规则，如何拦截是由浏览器自己实现的。我们可以通过这种方式来尽量减少 XSS 攻击

    通常可以通过两种方式来开启 CSP：

    * 设置 HTTP Header 中的 `Content-Security-Policy`
    * 设置 meta 标签的方式

    这里以设置 HTTP Header 来举例：
    * 只允许加载本站资源

      ```Content-Security-Policy: default-src 'self'```
      
    * 只允许加载 HTTPS 协议图片
    
      ```Content-Security-Policy: img-src https://*```
    
    * 允许加载任何来源
    
      ```Content-Security-Policy: child-src 'none'```
    
    如需了解更多属性，请查看 [Content-Security-Policy文档](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
    
    对于这种方式来说，只要开发者配置了正确的规则，那么即使网站存在漏洞，攻击者也不能执行它的攻击代码，并且 CSP 的兼容性也不错
    
2. **转义字符**

    用户的输入永远不可信任的，最普遍的做法就是转义输入输出的内容，对于引号、尖括号、斜杠进行转义

    ```js
    function escape(str) {
      str = str.replace(/&/g, '&amp;')
      str = str.replace(/</g, '&lt;')
      str = str.replace(/>/g, '&gt;')
      str = str.replace(/"/g, '&quto;')
      str = str.replace(/'/g, '&#39;')
      str = str.replace(/`/g, '&#96;')
      str = str.replace(/\//g, '&#x2F;')
      return str
    }
    ```

    但是对于显示富文本来说，显然不能通过上面的办法来转义所有字符，因为这样会把需要的格式也过滤掉。对于这种情况，通常采用白名单过滤的办法，当然也可以通过黑名单过滤，但是考虑到需要过滤的标签和标签属性实在太多，更加推荐使用白名单的方式

    ```js
    const xss = require('xss')
    let html = xss('<h1 id="title">XSS Demo</h1><script>alert("xss");</script>')
    // -> <h1>XSS Demo</h1>&lt;script&gt;alert("xss");&lt;/script&gt;
    console.log(html)
    ```

    以上示例使用了 js-xss 来实现，可以看到在输出中保留了 h1 标签且过滤了 script 标签

3. **HttpOnly Cookie**

    这是预防 XSS 攻击窃取用户 cookie 最有效的防御手段。Web 应用程序在设置 cookie 时，将其属性设为 HttpOnly，就可以避免该网页的 cookie 被客户端恶意 JavaScript 窃取，保护用户 cookie 信息

#### CSRF

CSRF(Cross Site Request Forgery)，即跨站请求伪造，是一种常见的 Web 攻击，它利用用户已登录的身份，在用户毫不知情的情况下，以用户的名义完成非法操作


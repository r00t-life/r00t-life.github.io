---
title: CORS
date: 2018-12-06 21:22:33
tags:
---


# 跨域资源漏洞

## 同源策略

    同源的必要条件：同协议，同域名，同端口

## 需要使用跨域的情况
 - `XMLHttpRequest`或`Fetch`发起的跨域HTTP请求
 - Web字体(CSS中通过`@font-face`使用跨域字体资源)
 - WebGL贴图
 - 使用`drawImage`将Image/video画面绘制到canvas
 - 使用样式表(使用CSSOM)
 - 引用外部资源

    ```
    <script src="..."> 
    <img src="...">
    <video src="...">
    <embed src="...">
    <frame src="...">
    <iframr src="...">
    <link rel="stylesheet" href="...">
    <applet code="...">
    <object data="...">
    ```
## 功能描述

    跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）

## 简单请求
    CORS不会发起预检请求，需要满足一些条件：
    - GET
    - HEAD
    - POST
    - Accept, Accept-Language, Content-Language 需要为标准值
    Conten-Type接收三种类型的值：
    - text/plain
    - multipart/form-data
    - application/x-www-form-urlencoded
##### REQUEST HEADER:
```
Host: a.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://foo.example/index.html
Origin: http://foo.example
Connection: keep-alive
Cache-Control: max-age=0
```
##### RESPONSE HEADER:
```
HTTP/1.1 200 OK
Server: nginx
Date: Sun, 09 Dec 2018 08:57:00 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/7.2.6
Access-Control-Allow-Origin: http://foo.example
Content-Length: 6
```
## 预检请求
    需预检请求要求必须先使用OPTIONS方法发起一个预检请求到服务器，以获取服务器是否允许该实际的请求，当请求满足下述条件之一时，就需要发送预检请求：
    - PUT
    - DELETE
    - CONNECT
    - OPTIONS
    - TRACE
    - PATCH
    Content-Type不属于下列之一：
    - application/x-www-form-urlencoded
    - multipart/form-data
    - text/plain
```
OPTIONS REQEUST:

Host: a.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Access-Control-Request-Method: POST
Access-Control-Request-Headers: content-type,x-pingother
Origin: http://foo.example
Connection: keep-alive
Cache-Control: max-age=0

OPTIONS RESPONSE:

HTTP/1.1 200 OK
Server: nginx
Date: Sun, 09 Dec 2018 08:43:23 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/7.2.6
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Headers: Content-Type, X-PINGOTHER
Content-Length: 0
```
```
POST REQUEST:
Host: a.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://foo.example/OPTIONS.html
X-PINGOTHER: pingpong
Content-Type: application/xml
Content-Length: 62
Origin: http://foo.example
Connection: keep-alive
Cache-Control: max-age=0

POST RESPONSE:

HTTP/1.1 200 OK
Server: nginx
Date: Sun, 09 Dec 2018 08:43:23 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/7.2.6
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Headers: Content-Type, X-PINGOTHER
Content-Length: 0
```

## 带身份凭证的请求
客户端发送请求使用方法`withCredentials`，服务的的相应中携带`Access-Control-Allow-Credentials: true`
```
REQUEST:

Host: a.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://foo.example/withcred.html
Origin: http://foo.example
Connection: keep-alive
Cache-Control: max-age=0

RESPONSE:

HTTP/1.1 200 OK
Server: nginx
Date: Sun, 09 Dec 2018 09:43:02 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/7.2.6
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: http://foo.example
Content-Length: 13
```
如果`Access-Control-Allow-Credentails`为`false`，或者`Access-Control-Allow-Origin`设置为`*`，则响应失败。

## HTTP响应首部字段语法
- Access-Control-Allow-Origin: <origin> | *



- Access-Control-Expose-Headers: X-MY-Custom-header, X-My-Custom-header2

    在跨域访问时，XMLHttpRequest对象的getResponseHeader()方法只能拿到一些最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。
这样浏览器就能够通过getResponseHeader访问X-My-Custom-Header和 X-Another-Custom-Header 响应头
了。

- Access-Control-Max-Age: <delta-seconds>

    `Access-Control-Max-Age` 头指定了preflight请求的结果能够被缓存多久, delta-seconds 参数表示preflight请求的结果在多少秒内有效。

- Access-Control-Allow-Methods: <method>[, <method>]*

    `Access-Control-Allow-Methods` 首部字段用于预检请求的响应。其指明了实际请求所允许使用的 HTTP 方法。

- Access-Control-Allow-Headers: <field-name>[, <field-name>]*

    `Access-Control-Allow-Headers` 首部字段用于预检请求的响应。其指明了实际请求中允许携带的首部字段。


## HTTP请求首部字段
- Origin: <origin>

    `Origin` 首部字段表明预检请求或实际请求的源站。
- Access-Control-Request-Method: <method>

    `Access-Control-Request-Method` 首部字段用于预检请求。其作用是，将实际请求所使用的 HTTP 方法告诉服务器。

- Access-Control-Request-Headers: <field-name>[, <field-name>]*
    
    `Access-Control-Request-Headers` 首部字段用于预检请求。其作用是，将实际请求所携带的首部字段告诉服务器。


## JSONP和CORS漏洞
- CORS
- JSONP

## JSONP漏洞

server:
```
<?php
$callback = $_GET['callback'];
print $callback."({'username':'admin', 'password':'123456'});";
print
?>
```

attacker:
```
<script type="text/javascript">
	function info(data)  {
		alert(data.username);
	}
</script>
<script type="text/javascript" src="http://a.com/jsonp.php?callback=info"></script>
```

### 防御方式
当服务端没有验证Referer头的时候，会存在JSONP漏洞，所以任何网站都可以从这个域下获取数据。
- 要防御这种攻击方式，可以在函数返回数据的时候，返回一个`while(1)`，这样利用js代码就会陷入无线循环中，攻击者无法获得数据; 
- 对Callback参数进行过滤检测，不允许callback可定义，过滤JSON里面的敏感数据; 
- 检测referer头，还需要检测referer是否为空，，一般情况下浏览器直接访问URL是不带Referer的，所以很多防御体系允许空Referer，而跨协议调用JS的时候，Referer也是空
    ```
    <iframe src="javascript:'<script>function info(data){alert(data.username);}</script><script src="http://a.com/jsonp.php?callback=info"></script>'"></iframe>
    ```
- 部署一次性TOKEN


## CORS错误配置
服务器(`a.com`)：
```
<?php
header("Access-Control-Allow-Origin: http://foo.example");
echo "secret";
?>

```
攻击者(`foo.example`)：
```
<script type="text/javascript">
	var xhr = new XMLHttpRequest();
	xhr.onreadystatechange = function() {
		if (xhr.readyState === 4) {
			alert(xhr.responseText);
		}
	}
	xhr.open("GET", "http://a.com/secret.php");
	xhr.send();
</script>
```
上面配置了一个白名单域名`foo.example`，这样就允许了`foo.example`跨域访问`a.com`，如果配置错误了，将`Access-Control-Allow-Origin`设置为`*`，那么任何人都可以跨域访问到`a.com`的资源。如果服务端把`Access-Control-Allow-Credentials`设置为`true`，允许客户端带上cookie，那么攻击者很容易能够获得用户的数据，如果这样配置了那么浏览器会阻止数据返回。
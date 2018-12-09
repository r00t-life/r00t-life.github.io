---
title: CORS
date: 2018-12-06 21:22:33
tags:
---


# CORS跨域资源共享漏洞

## 同源策略

    同源的必要条件：同协议，同域名，同端口

## 什么情况下需要使用CORS
 - `XMLHttpRequest`或`Fetch`发起的跨域HTTP请求
 - Web字体(CSS中通过`@font-face`使用跨域字体资源)
 - WebGL贴图
 - 使用`drawImage`将Image/video画面绘制到canvas
 - 使用样式表(使用CSSOM)

## 功能描述

    跨域资源共享标准新增了一组 HTTP 首部字段，允许服务器声明哪些源站通过浏览器有权限访问哪些资源。另外，规范要求，对那些可能对服务器数据产生副作用的 HTTP 请求方法（特别是 GET 以外的 HTTP 请求，或者搭配某些 MIME 类型的 POST 请求），浏览器必须首先使用 OPTIONS 方法发起一个预检请求（preflight request），从而获知服务端是否允许该跨域请求。服务器确认允许之后，才发起实际的 HTTP 请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）


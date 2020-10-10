---
title: 子域名接管漏洞
date: 2018-12-14 05:43:49
tags:
- dns
- sec
---


### CNAME子域名接管
这个漏洞的成因，主要是因为域名（源域名）配置了CNAME，但是CNAME指向的域名并没有被注册。那么攻击者可以注册这个CNAME指向的域名，就可以控制了源域名

这样攻击者就可以绕过`HttpOnly`和`Secure Cookie`的安全配置，盗取用户的cookie，CNAME子域名接管的整个流程如下：
    
    1. 源域名(sub.example.com)设置了一个CNAME，CNAMED指向的域名为sub.cname.com
    2. 检查cname.com，如果cname.com没有注册，那么sub.example.com可以成功被接管
    3. 攻击者注册cname.com，创建恶意页面

### NS子域名接管
NS记录同样也会收到影响，如果域名的NS记录中，如果有一个域名没有被注册，那么这个域名就可能被接管。

    加入sub.example.com有两个NS记录，分别是ns1.vuln.com, ns2.novuln.com，如果攻击者注册了vuln.com这个域名，那么就有50%的几率受到攻击。如果DNS解析选择了ns1.vuln.com，那么攻击者可以返回一个钓鱼页面，并非原来sub.example.com的页面，并且会缓存很长的时间，攻击者可以设置TTL的时长。

### MX子域名接管
如果MX记录的子域名被接管了，那么攻击者可以接收到发送到源域名的邮件

### 云提供商
这里有个github(https://github.com/EdOverflow/can-i-take-over-xyz)项目，列出了会受到子域名接管漏洞影响的提供商


### 参考链接
https://0xpatrik.com/subdomain-takeover-basics/

https://0xpatrik.com/subdomain-takeover/
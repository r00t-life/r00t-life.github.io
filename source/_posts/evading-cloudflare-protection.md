---
title: evading-cloudflare-protection
date: 2017-09-11 21:56:44
tags: cdn, web security
---

1.老旧的cloudflare配置
cloudflare有一个直连DNS记录:`direct-connect` ,可以直接访问目标网站？不过随着DDOS攻击的泛滥,现在cloudflare把这个'绕过子域名'替换成更加随机的域名(dc-######-随机的hex哈希)

2.MX记录
MX记录一般会显示网站的邮箱服务器地址,基本上邮箱服务器地址都是目标的真实IP段,不过现在越来越多的邮箱服务都是托管在云上。如果目标使用的是office 365作为邮件服务器，那么可以到office 365上随便输入该网站一个不存在的邮箱，那么会自动跳转到目标的邮箱服务器。如果不存在，则提示没有这个ID；如果是托管，显示不变，也不跳转。

3.子域名
FTP/SCP/VPN等子域名可能会暴露目标真实IP

4.WEBSOCKET服务
cloudflare提供了针对websockets技术服务的访问，但是有些用户可能还不知道或者还未从自己的WS服务器上迁移过来。加上WS服务需要始终保持客户端服务端的连接，真实IP暴露的几率很大。

5.以前的DNS记录
可以在https://www.netcraft.com/网站查询网站的历史纪录，可能会包含真实IP地址

6.多地PING
目标可能购买的CDN服务没有覆盖全球，如果使用全球多个节点来ping，那么可能会得到真实的IP地址

7.SSRF
如果目标存在SSRF漏洞，则可以对我们设置的服务器发起一个请求，在服务器上用NC监听，服务器接收到请求后即可输出目标的IP地址

8.发邮件
给目标发送邮件，如果目标回复邮件，是不是可以得到目标的邮件服务器地址？


https://rhinosecuritylabs.com/cloud-security/cloudflare-bypassing-cloud-security/
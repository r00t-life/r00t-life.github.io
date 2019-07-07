---
title: 使用openwrt搭建上网网关
date: 2019-07-07 10:23:21
tags: ssr, iptables, openwrt, proxy
---


前阵子买了一个openwrt路由器，就捣鼓了一下，使用`ssr + chinadns + dnsmasq + iptables`搭建上网代理，这里简单记录一下

1.配置ssr-redir

2.配置chinadns
    ```
    chinadns -c chnroute.txt -p 5353
    curl 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | grep ipv4 | grep CN | awk -F\| '{ printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > chnroute.txt
    可以使用crontab每周更新一次
    0 0 * * 0 command
    ```

3.dnsmasq配置
    ```
    vim /etc/dnsmasq.d/dnsmasq.conf
    no-resolv
    server=127.0.0.1#5353
    ```

4.iptables
    ```
    #!/bin/bash
    iptables -t nat -N SS
    #iptables -t nat -A SS -p tcp --dport 443 -j RETURN	#ssr服务器端口
    iptables -t nat -A SS -d ssr-server -j RETURN		#ssr服务器IP	二选1

    iptables -t nat -A SS -d 0.0.0.0/8 -j RETURN
    iptables -t nat -A SS -d 10.0.0.0/8 -j RETURN
    iptables -t nat -A SS -d 127.0.0.0/8 -j RETURN
    iptables -t nat -A SS -d 169.254.0.0/16 -j RETURN
    iptables -t nat -A SS -d 172.16.0.0/12 -j RETURN
    iptables -t nat -A SS -d 192.168.0.0/16 -j RETURN
    iptables -t nat -A SS -d 224.0.0.0/4 -j RETURN
    iptables -t nat -A SS -d 240.0.0.0/4 -j RETURN
    #iptables -t nat -A SS -d chinaIP -j RETURN

    iptables -t nat -A SS -p tcp -j REDIRECT --to-ports 10800
    iptables -t nat -I PREROUTING -p tcp -j SS
    #iptables -t nat -I PREROUTING -p udp -j SS				
    #iptables -t nat -I PREROUTING -p udp --dport 53 -j DNAT --to-destination 1.2.3.4:53 # 可以配合dnsforward，把dns查询转换成TCP
    ```

这样设置好之后，是全局翻墙模式，要分IP进行代理的话，需要在iptables中加入china IP，使用APNIC生成的IP太庞大了，所以这里使用的是 [bestroutetb](https://github.com/ashi009/bestroutetb)，把生成的net列表替换到`iptables`中即可。
```
#!/bin/bash

bestroutetb \
  --output 20-route \
  --route.net=cn \
  --route.vpn=us \
  --profile=custom \
  --no-default-gateway \
  --group-gateway \
  --group-name.net=wan \
  --group-name.vpn=vpn \
  --group-header=$'%name)\n' \
  --rule-format=$'iptables -t nat -A SS -d  %prefix/%length -j RETURN\n' \
  --gateway.net='$5' \
  --gateway.vpn='$5' \
  -vvf
```

如果要设置透明代理，使用下面的iptables规则
```
ens38 lan ens33 wan
iptables -A FORWARD -i ens38 -o ens33 -j ACCEPT
iptables -A FORWARD -i ens33 -o ens38 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE

iptables -t nat - N SS
iptables -t nat -A SS -d 1.1.1.1/24 -j  RETURN
iptables -t nat -A SS -p tcp -j REDIRECT --to-ports 1080
iptables -t nat -A PREROUTING -p tcp -j SS
iptables -t nat -A PREROUTING -p udp --dport 53 -j DNAT --to-destination 1.2.3.4:53 #配合DNSFORWARD
```
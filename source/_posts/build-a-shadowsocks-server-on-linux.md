---
title: 如何搭建Shadowsocks服务器
date: 2017-10-30 15:04:16
tags: [shadowsocks, proxy, 代理]
categories: network
published: false
---

想要科学上网，就需要有个代理账号，账号怎么来，要么去买一个代理账号，便宜方便，缺点是同一个IP用的人多，容易被gfw封，不够稳定；要么自己买服务器搭建一个代理服务器，服务相对稳定些，但需要一点技术基础。
对于程序员，建议自己搭建一个代理服务器。

## 1. 购买服务器

可以去亚马逊云，Godaddy，阿里云购买，按自己的需求来选择配置，系统请选择Ubuntu，一般搭建代理服务器的主要影响因素是带宽，对于CPU和内存影响不大，为了节约开支一般选最低就可以，如果嫌贵，可以和几个人一起购买一个服务器公用。

## 2. 搭建Shadowsocks服务端

买好服务器后开始搭建Shadowsocks服务端，先通过ssh登录代理服务器
```
//安装shadowsocks
sudo apt-get update
sudo apt-get install shadowsocks
```
```
//修改shadowsocks服务器配置
sudo vim /etc/shadowsocks/config.json
```
```
{
    "server":"0.0.0.0",   //这里很多教程说填服务器对外的IP，我试了很多次都不行，最后换成0.0.0.0可以
    "server_port":8388,   //对外端口，可以自己定义，注意服务器要先配置此端口可被访问
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"you_password",    //客户端密码
    "timeout":300,
    "method":"aes-256-cfb",       //客户端加密类型
    "fast_open": false,
    "workers": 1
}
```
<!-- more -->

## 3. 启动和停止Shadowsocks服务器

```
sudo /etc/init.d/shadowsocks start
sudo /etc/init.d/shadowsocks stop
```

## 4. Shadowsocks客户端配置

下载一个Shadowsocks客户端: https://sourceforge.net/projects/shadowsocksgui
安装好后，按如下方式配置，然后开启Shadowsocks

{% asset_img 01.png %}

配置shadowsocks白名单，指定哪些网站是需要代理的
```
vim .ShadowsocksX/gfwlist.js
```
打开浏览器，开始科学上网😉。

---
title: 自建Bitwarden密码管理服务
abbrlink: 
categories: 实用工具
tags: 信息安全
cover: 'https://cdn.jsdelivr.net/gh/laixiaoyu-v/ImgHosting/blog/Cover/vaultwarden.jpg'
date: 2022-01-15 14:44:05
updated: 2022-01-15 14:44:05
---

信息发达的现代，密码可以说是和我们的生活息息相关。无论是经常使用到的支付应用，社交账号，网站登录，银行卡，乃至出入的门禁，保险箱都会经常用到。俗话说，好记性不如烂笔头。越来越多的密码使得我们会偶尔忘记、记错甚至记混。（反正我经常记混。）设置成一样的又不安全。设置太复杂又难记。所以我觉得是有必要拥有一款密码管理软件来帮你记住密码和账号的。对没错，是密码和账号，我就有忘记账号的尴尬时刻。所以接下来要介绍Bitwarden。

## 什么是Bitwarden
Bitwarden是一款自由且開源的密碼管理服務，用戶可在加密的保管庫中儲存敏感資訊（例如網站登入憑據）。Bitwarden平台提供有多種客戶端應用程式，包括網頁用戶介面、桌面應用，瀏覽器擴充、流動應用以及命令列介面。Bitwarden提供雲端寄存服務，並支援自行部署解決方案。（维基百科）

官网：[Bitwarden](https://bitwarden.com/)


## 为什么选择Bitwarden
- 支持Chrome、Firefox、Opera、 Edge 等常见浏览器的插件，拥有 windows、macOS、iOS、Android 、Linux客户端，多设备同步。
- 不单只是网页登录密码，还能保存支付卡，身份信息，安全笔记。
-  Bitwarden是开源的，支持自建服务。
-  可按要求一键生成复杂密码，提高安全性。

其实我之前是使用谷歌的密码管理，但是他依附于谷歌浏览器。这就使得它只能记录网站密码，而生活中有很多其他的密码无法被储存。而其他的密码管理软件其实有很多。它们要么插件或者户端不够齐全，要么需要付费才能拥有高级功能，还有的曾经爆出过泄露用户数据。综合考虑之下，才选择了 Bitwarden自建服务。

## docker搭建Bitwarden
这次我们搭建的是Bitwarden的一个第三方开源版本：vaultwarden(原：bitwarden_rs)。这个vaultwarden比起官方版本该有的功能都有还自带官方的高级会员功能。而且比官方提供的自建方式，系统资源的要求上要低的多。

GitHub：[vaultwarden](https://github.com/dani-garcia/vaultwarden)


先安装docker，由于我这边使用的是Ubuntu20.04的系统，所以我直接使用它的包管理工具安装：
```
sudo apt update
sudo apt install docker
``` 

安装好docker后拉取`vaultwarden`镜像：
```
docker pull vaultwarden/server:latest
```

创建容器
```
docker run -d --name vaultwarden -v /vaultwarden/data/:/data/ -p 8000:80 bitwardenrs/server:latest
````
其中`/vaultwarden/data/`是Bitwarden的数据储存位置可自定义，端口`8000`也可自定义

完成以上步骤如跟没有报错的话你访问 ` ip:8000 `就能访问Bitwarden服务

![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/端口登录.png "")

## 宝塔添加反向代+启用https
创建站点，填写域名，php版本选纯静态
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/创建网站.png "")

添加反向代理，目标URL填写 `127.0.0.1:8000 `，如果上面你设置的端口不是`8000`要填写相应的端口
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/反向代理.png "")

选择SSL点申请开启https
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/开启https.png "")


做完以上步骤我们就可以通过域名访问Bitwarden服务了

![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/域名访问.png "")


## 开启Web Vault Admin界面
要开启Bitwarden的管理界面，需要在创建容器时指定环境变量： `-e ADMIN_TOKEN=一个用于登录Web Vault Admin的密码`
```
# 停止旧容器
docker stop vaultwarden
# 删除旧容器
docker rm vaultwarden
# 创建新容器
docker run -d --name vaultwarden -e ADMIN_TOKEN=用于登录Web Vault Admin的密码 -v /vaultwarden/data/:/data/ -p 8000:80 vaultwarden/server:latest
```

输入：`域名/admin `并输入自己设置的密码进入管理界面
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/管理界面.png "")

Web Vault Admin界面能配置Bitwarden设定和管理已注册的用户
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/管理界面2.png "")

例如我只是自己使用不想让别人注册我们可以禁用注册

![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/禁用注册.png "")


## 客户端登录
下载官方客户端在设置中填写自己的服务端地址才能使用自己搭建的服务端
我这边以ios客户端为例，点击左上角的设置按钮在服务器URL中填写自己的服务端网址点击保存再登录：
![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/登录.jpg "")

![alt text](https://od.likexy.tk/ImgHosting/Bitwarden/登录2.jpg "")



其他客户端及插件同理设置
.
## 最后
做完以上步骤其实你就可以使用一个属于自己的密码储存服务了，但其实如果你不想那么麻烦的搭建也可以使用Bitwarden官方提供的免费服务和了解一下它们的付费服务，支出一下Bitwarden官方：8bit Solutions LLC 公司毕竟，开源项目实属不易。







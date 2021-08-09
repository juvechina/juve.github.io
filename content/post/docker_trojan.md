---
title: "在N1盒子上运行docker搭建trojan server"
date: 2021-07-11T18:11:35+08:00
draft: false
---
#### 1、关于N1盒子：
![image](https://cdn.jsdelivr.net/gh/Pockies/pic/741f9461ly1g0u2heavhxj212w0pyx6p.jpg#pic_left)
##### N1盒子配置高，价格低，目前转转上120包邮可以拿到成色不错的版本，先来看下他的硬件配置：

模块 | 参数
---|---
CPU | Amlogic S905，ARMv8 4核 Cortex-A53
GPU | ARM Mali™-450，4K@60fps硬件
内存 | 2G
闪存 | 8G
数据接口 | usb2.0 x 2
视频接口 | HDMI 2.0
网络接口 | RTL8211F，千兆
WIFI | 5G 80Mhz 433Mbps
固件推荐|基于flippy大神 https://wxf2088.xyz/2235.html
#### 2、关于trojan
##### trojan是一款代理软件，网上有很多测速的文章，小编实测发现trojan对于硬(laji)路由的cpu是非常友好的，即使在30元包邮的newifi-mini上仍能流畅观看油管1440P，速度可维持在5万左右（小编使用的是俄罗斯justhost主机，下方有推荐链接），虽然不少文章认为Xray的速度最快，但我这边实测下来Xray对cpu的消耗要更大，所以硬(laji)路由上trojan速度更快。

主机 | 推广链接 | 价格(人民币) | 优势
---|---|---|---
JustHost | https://justhost.ru/?ref=70664 | 90元/年 | 选TTK线路直连国内/200M不限流量/联通ping 100ms/IP不通可随时更换

#### 3、在N1盒子上部署docker版的trojan
##### 3.1、docker介绍：
###### docker是个好东西，在这个ARM架构的路由器上，你甚至可以运行CentOS，并安装nginx搭建下载站；今天来介绍如何在N1盒子上用docker搭建trojan的server端，首先要刷好flippy的固件，具体方法请参考表格链接，刷好以后可在左侧菜单栏看到docker按钮。
![image](https://download.juve.cc:8888/pic/001.jpg#pic_left)
##### 3.2、N1盒子上的docker配置如下图：
![image](https://download.juve.cc:8888/pic/002.jpg#pic_left)
##### 3.3、部署docker的torjan：
###### 支持跨平台的trojan的docker镜像有很多，今天给大家介绍的是一款比较主流的版本，teddysun大神的作品，**接下来我们将在路由器的ssh界面(黑白屏)上进行相关的操作**，首先执行下面的命令来拉取镜像
```
docker pull teddysun/trojan-go
```
###### 然后为trojan的配置文件创建目录并添加配置文件

```
mkdir -p /etc/trojan
```
###### 添加配置文件，注意是json格式，下面是一个简单例子，其中要注意证书存放路径要与配置文件路径一致，本案例中配置文件和证书都放在路由器的 /etc/trojan/ 目录下：

```
cat > /etc/trojan/config.json <<EOF
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443, #注意：这个是通信端口，默认是443，也可改为其他端口，在开启实例时要跟系统保持一致，否则无法通信
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "password1", #trojan的密码，可以自己改，可以自己加多个密码
        "password2"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/etc/trojan/certificate.crt", #证书的位置，注意路径跟配置文件路径保持一致
        "key": "/etc/trojan/private.key",
        "key_password": "", 
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
EOF
```
###### 运行实例，注意端口配置要与配置文件的端口相同

```
docker run -d -p 443:443 --name trojan --restart=always -v /etc/trojan:/etc/trojan teddysun/trojan
#此处有必要说明一下，-p为端口映射，格式为8000:443，其中8000为系统端口，443为docker实例内部端口，如果你想开路由器的8000端口来代理docker实例的443端口，就写为 -p 8000:443；
--restart=always 表示如果实例如果跑挂了可自动重启，当然如果真的有问题需要查看详细日志来定位。
日志查看可以在路由器web界面中通过docker-容器中点trojan容器的名称，再点顶部菜单中日志按钮进行查看。
```
#### 4、tips
##### 4.1、申请免费证书：可以去腾讯云申请免费证书，一年有效期，到期后手动申请新证书，永久免费。各大云厂商的后台，小编只对腾讯云情有独钟，将域名解析到dnspod以后可支持一键申请证书，非常方便。

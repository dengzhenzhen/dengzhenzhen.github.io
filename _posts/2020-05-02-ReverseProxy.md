---
layout:     post
title:      用V2ray配置反向代理实现外网访问PC服务
subtitle:   
date:       2020-05-02
author:     dengzhenzhen
header-img: img/block_of_reverse-doko.6abc2d13.png
catalog: 	 true
tags:
    - Reverse Proxy
    - V2ray
---

## V2ray

自从SS频繁被封端口之后，我找到了一个更加年轻且强大的工具**V2ray**。

在github上开源而且官方文档肥肠详细，可以看看 https://www.v2ray.com/ 文档面向小白，虽然配置起来比SS要稍微复杂一点，但是也不算很难，而且功能太强了！ 它的绝不仅限于我们平时的科学上网用途，通过配置文件，我们可以实现很多自定义的需求。

这里记录一下搭反向代理的过程，搭梯子比较容易理解感觉没什么好写的

## 反向代理

话不多说先看图

![](https://s1.ax1x.com/2020/05/02/JvAr0x.png)

这里**A**指的是**没有公网ip**的用来提供服务的设备，通常来说就是我们的PC；**B**指的是**拥有公网ip**的服务器，通常来说就是我们的VPS

过程相当于：
- A向B发起一个连接，两者建立连接
- B收到其他地方访问A的服务的请求，从dokodemo进来通过路由转到portal
- portal用第一步建立的连接访问A上的服务(图中绿色的路线)
- 从外网直接访问B配置的dokodemo的端口就能访问到A的服务了

### 具体配置

#### Client：
```json
{
  "inbounds": [
    {
      "port": 555, // SOCKS 代理端口，在浏览器中需配置代理并指向这个端口
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
        "udp": false
      }
    },
	{
      "port": 556, // http 代理端口，在浏览器中需配置代理并指向这个端口
	  "tag":"common",
      "listen": "127.0.0.1",
      "protocol": "http",
      "settings": {
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "dengzhenzhen.ml", // 服务器地址，请修改为你自己的服务器 ip 或域名
            "port": 443, // 服务器端口
            "users": [
              {
                "id": "uuid"
              }
            ]
          }
        ]
      },
      "mux": {
        "enabled": false
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls" // 客户端的 security 也要设置为 tls
      }
    },
	{
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "my.domain", // 服务器地址，请修改为你自己的服务器 ip 或域名
            "port": 16823, // 服务器端口
            "users": [
              {
                "id": "uuid"
              }
            ]
          }
        ]
      },
	  "tag":"tunnel"
    },
    {
      "protocol": "freedom",
      "tag": "direct",
      "settings": {}
    },
	{
      "protocol": "blackhole",
      "tag": "adblock",
      "settings": {}
    },
	{
      "tag": "out",
      "protocol": "freedom",
      "settings": {
        "redirect": "127.0.0.1:80"  // 将所有流量转发到网页服务器
      }
    }
  ],
  "reverse":{ 
    // 这是 A 的反向代理设置，必须有下面的 bridges 对象
    "bridges":[  
      {  
        "tag":"bridge", // 关于 A 的反向代理标签，在路由中会用到
        "domain":"private.cloud.com" // A 和 B 反向代理通信的域名，可以自己取一个，可以不是自己购买的域名，但必须跟下面 B 中的 reverse 配置的域名一致
      }
    ]
  },
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct"
      },
	   {  
        // 配置 A 主动连接 B 的路由规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "domain":[  
          "full:private.cloud.com"
        ],
        "outboundTag":"tunnel"
      },
      {  
        // 反向连接访问私有网盘的规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "outboundTag":"out"
      }
    ]
  },
  "log": {
    "loglevel": "info", // 日志级别
    "access": "./log/access.log", // 可以用相对路径
    "error": "./log/error.log"
  }
}
```

#### Server:
```json
{
  "inbounds": [
    {
      "port": 443,
      "protocol": "vmess",
      "tag":"wallbreakin",
      "settings": {
        "clients": [
          {
            "id": "****************",
            "level": 1,
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls", // security 要设置为 tls 才会启用 TLS
        "tlsSettings": {
          "certificates": [
            {
              "certificateFile": "/etc/v2ray/v2ray.crt", // 证书文件
              "keyFile": "/etc/v2ray/v2ray.key" // 密钥文件
            }
          ]
        }
      }
    },
    {  
      // 接受 C 的inbound
      "tag":"external", // 标签，路由中用到
      "port":80,
      // 开放 80 端口，用于接收外部的 HTTP 访问 
      "protocol":"dokodemo-door",
        "settings":{  
          "address":"127.0.0.1",
          "port":80, //假设 client 监听的端口为 80
          "network":"tcp"
        }
    },
    // 另一个 inbound，接受 A 主动发起的请求  
    {  
      "tag": "tunnel",// 标签，路由中用到
      "port":16823,
      "protocol":"vmess",
      "settings":{  
        "clients":[  
          {  
            "id":"******************",
            "alterId":64
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag":"wallbreakout",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "reverse":{  //这是 B 的反向代理设置，必须有下面的 portals 对象
    "portals":[  
      {  
        "tag":"portal",
        "domain":"private.cloud.com"        // 必须和上面 A 设定的域名一样
      }
    ]
  },
  "routing": {
    "rules": [
      {
        "type":"field",
        "inboundTag":["wallbreakin"],
        "outboundTag":"wallbreakout"
      },
      {  //路由规则，接收 C 请求后发给 A
        "type":"field",
        "inboundTag":[  
          "external"
        ],
        "outboundTag":"portal"
      },
      {  //路由规则，让 B 能够识别这是 A 主动发起的反向代理连接
        "type":"field",
        "inboundTag":[  
          "tunnel"
        ],
        "outboundTag":"portal"
      },
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked"
      }
    ]
  },
  "log": {
    "loglevel": "info",
    "access": "/var/log/v2ray/access.log", // 这是 Linux 的路径
    "error": "/var/log/v2ray/error.log"
  }
}
```

client和server即A和B的配置

配置文件更改后重启V2ray即可

## 遇到的问题

一开始配置好之后总是访问不了，然后经历了这么几步

- 用telnet查对应端口服务有没有开 
    - ```telnet 127.0.0.1 80```
    - ```telnet 127.0.0.1 443```
    - ```telnet 127.0.0.1 16823```
- 结果发现都是好的，说明服务都开起来了
- 想到防火墙没开16823端口，赶紧重新打开
    ```bash
    firewall-cmd --permanent --zone=public --add-port=16823/tcp
    firewall-cmd --reload
    ```
- 结果还是不行，我要崩溃了！
- 打了一下debug日志，发现问题出在从B到A那一步上，B根本没能把请求发到A上！然后发现B路由里有一条，把私有ip都转到blocked了，我们访问的IP是 127.0.0.1:80 所以可能是这个原因，抱着试试看的想法，我把那一条路由规则放到了最后。
- It works!!!
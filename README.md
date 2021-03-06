# easyProxy

轻量级、高性能http代理服务器，主要应用于**内网穿透**。支持五种模式，**http代理请求**、**tcp隧道模式**、**udp隧道模式**、**sock5代理模式**、**http代理模式**，可根据自身需求进行选择，同时支持sock5验证，gzip、snnapy压缩等。

支持客户端与服务端连接中断自动重连，多路传输，大大的提高请求处理速度，go语言编写，无第三方依赖，经过测试内存占用小，普通场景下，仅占用10m内存。

水平很有限，不足指出请大家指出，交流QQ群：206610006

## 背景	  
1. 我有一个小程序的需求，但是小程序的数据源必须从内网才能抓取到，但是内网服务器没有公网ip，所以只能内网穿透了。----> [http反向代理请求](#http代理请求)

2. 想在外网通过ssh连接内网的机器，做云服务器到内网服务器端口的映射，或者做微信公众号开发---->[tcp隧道模式](#tcp隧道模式)

3. 在非内网环境下使用内网dns，或者需要通过udp访问内网机器等---->[udp隧道模式](#udp隧道模式)

4. 在外网使用HTTP代理访问内网站点---->[http代理模式](#http代理模式)

5. 搭建一个内网穿透ss，在外网如同使用内网vpn一样访问内网资源或者设备----> [socks5代理模式](#socks5代理模式)

## 特点
- [x] 支持gzip、snnapy压缩,减小传输过程流量消耗
- [x] 支持多站点配置,兼容多个内网网站，可处理相互之间的跳转包含关系
- [x] 断线自动重连
- [x] 支持多路传输,提高并发
- [x] 跨站自动匹配替换
- [x] 支持tcp隧道,提升访问效率
- [x] 支持udp隧道
- [x] 支持http代理
- [x] 支持内网穿透sock5代理，配合proxifer可达到vpn的效果，在外网访问内网资源或者设备，同时可以设置用户名和密码验证


## 目录

1. [安装](#安装)
2. [http反向代理请求](#http代理请求)
3. [tcp隧道模式](#tcp隧道模式)
4. [udp隧道模式](#udp隧道模式)
5. [sock5代理模式](#sock5代理模式)
6. [http代理模式](#http代理模式)
7. [数据压缩支持](#数据压缩支持)
8. [操作系统支持](#操作系统支持)



## 安装

1. release安装
> https://github.com/cnlh/easyProxy/releases

下载对应的系统版本即可（目前linux和windows只编译了64位的），服务端和客户端共用一个程序，go语言开发，无需任何第三方依赖

2. 源码安装
- 安装源码
> go get github.com/cnlh/easyProxy
- 编译（无第三方模块）
> go build

## http代理请求

### 场景及原理

较为适用于http，也就是web站点的穿透，服务端与客户端之间建立连接，服务端收到http请求后，将请求发送到客户端，客户端再执行这个请求，并将结果返回给服务端，服务端收到后再返回。

<html>
<span style="color:red">特点：支持同时代理多个站点，不同站点之间有联系还可以实现匹配替换</span>
</html>

![image](https://github.com/cnlh/easyProxy/blob/master/image/http.png?raw=true)

**最终效果**：
- 访问a.server.com和访问10.1.50.203的80端口相同
- 访问b.server.com和访问10.1.50.202的80端口相同
- 访问c.server.com和访问10.1.50.201的80端口相同
### 使用 
- 服务端 

```
./easyProxy -mode httpServer -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=8024
```

名称 | 含义
---|---
mode | 运行模式(client、server不写默认client)
vkey | 验证密钥
tcpport | 服务端与客户端通信端口
httpport | 代理的http端口（与nginx配合使用）

- 客户端

```
建立配置文件 config.json
```


```
./easyProxy -config config.json  
```


 名称 | 含义
---|---
config | 配置文件路径
### 配置文件config.json

```
{
  "Server": {
    "ip": "123.206.77.88",
    "tcp": 8284,
    "vkey": "DKibZF5TXvic1g3kY",
    "num": 10
  },
  "SiteList": [
    {
      "host": "a.ourcauc.com",
      "url": "10.1.50.203",
      "port": 80
    },
    {
      "host": "b.ourcauc.com",
      "url": "10.1.50.202",
      "port": 80
    },
    {
      "host": "c.ourcauc.com",
      "url": "10.1.50.203",
      "port": 80
    }
  ],
  "Replace": 0
}
```
 名称 | 含义
---|---
ip | 服务端ip地址
tcp | 服务端与客户端通信端口
vkey | 验证密钥
num | 服务端与客户端通信连接数
SiteList | 本地解析的域名列表
host | 域名地址
url | 内网代理的地址
port | 内网代理的地址对应的端口
Replace | 是否自动匹配替换[（查看场景）](https://github.com/cnlh/easyProxy/issues/1)


### nginx代理配置示例
```
upstream nodejs {
    server 127.0.0.1:8024;
    keepalive 64;
}
server {
    listen 80;
    server_name a.ourcauc.com b.ourcauc.com c.ourcauc.com ;
    location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host  $http_host:8024;
            proxy_set_header X-Nginx-Proxy true;
            proxy_set_header Connection "";
            proxy_pass      http://nodejs;
        }
}
```
## 域名配置示例
> -a	    A	    123.206.77.88

> -b	    A	    123.206.77.88

> -c	    A	    123.206.77.88

### 跨站自动匹配替换说明

例如，访问：a.ourcauc.com，该页面里面有一个超链接为10.1.50.202:80,将根据配置文件自动该将url替换为b.ourcauc.com，以达到跨站也可访问的效果，但需要提前在配置文件中配置这些站点。

如需开启，请加配置文件Replace值设置为1
>注意：开启可能导致不应该被替换的内容被替换，请谨慎开启

### 二级域名示范

[二级域名](https://github.com/cnlh/easyProxy/wiki/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B)


## tcp隧道模式

### 场景及原理
较为适用于处理tcp连接，例如ssh，同时也适用于http等，访问服务端的8024端口相当于访问内网10.1.50.202机器的4000端口，构成如下所示的隧道。

![image](https://github.com/cnlh/easyProxy/blob/master/image/tcp.png?raw=true)

例如：

**背景:**

- 内网机器10.1.50.203提供了web服务80端口

- 有VPS一个,公网IP:123.206.77.88

**需求:**

在家里能够通过访问VPS的8024端口访问到内网机器A的80端口

### 使用 
- 服务端 

```
./easyProxy -mode tunnelServer -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=8024 -target=10.1.50.203:80
```

名称 | 含义
---|---
mode | 运行模式(client、server不写默认client)
vkey | 验证密钥
tcpport | 服务端与客户端通信端口
httpport | 代理的http端口（与nginx配合使用）
target | 目标地址，格式如上

- 客户端

```
建立配置文件 config.json
```


```
./easyProxy -config config.json  
```


 名称 | 含义
---|---
config | 配置文件路径
### 配置文件config.json




```
{
  "Server": {
    "ip": "123.206.77.88",
    "tcp": 8284,
    "vkey": "DKibZF5TXvic1g3kY",
    "num": 10
  }
}
```
 名称 | 含义
---|---
ip | 服务端ip地址
tcp | 服务端与客户端通信端口
vkey | 验证密钥
num | 服务端与客户端通信连接数

## udp隧道模式

### 场景及原理

**背景**
- 内网机器A提供了DNS解析服务,10.1.50.210:53端口

- 有VPS一个,公网IP:123.206.77.88

**需求:**
在家里能够通过设置本地dns为123.206.77.88,使用内网机器A进行域名解析服务.

访问vps的53端口相当于访问10.1.50.210的53端口，构成如下所示的隧道。

![image](https://github.com/cnlh/easyProxy/blob/master/image/udp.png?raw=true)


### 使用 
- 服务端 

```
./easyProxy -mode udpServer -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=53 -target=10.1.50.210:53
```

名称 | 含义
---|---
mode | 运行模式(client、server不写默认client)
vkey | 验证密钥
tcpport | 服务端与客户端通信端口
httpport | 公网vpn的端口
target | 目标地址，格式如上

- 客户端

```
建立配置文件 config.json
```


```
./easyProxy -config config.json  
```


 名称 | 含义
---|---
config | 配置文件路径

- 本机

```
设置本机dns解析服务器地址为123.206.77.88
```


### 配置文件config.json


```
{
  "Server": {
    "ip": "123.206.77.88",
    "tcp": 8284,
    "vkey": "DKibZF5TXvic1g3kY",
    "num": 10
  }
}
```
 名称 | 含义
---|---
ip | 服务端ip地址
tcp | 服务端与客户端通信端口
vkey | 验证密钥
num | 服务端与客户端通信连接数


## socks5代理模式

### 场景及原理

**原理**

主要用于socks5代理，也就是和ss类似，不过是代理内网。使用此模式时，可在非内网环境下配置本机的socks5代理（服务器ip、sock5代理端口），即可实现socks5代理，达到访问内网的网站的效果，配合proxifer等全局代理软件，即可如同使用内网vpn一样，访问内网网站，通过ssh连接内网机器等等……。
![image](https://github.com/cnlh/easyProxy/blob/master/image/sock5.png?raw=true)

### 使用 
- 服务端 

```
./easyProxy -mode sock5Server -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=8024
```

名称 | 含义
---|---
mode | 运行模式(client、server不写默认client)
vkey | 验证密钥
tcpport | 服务端与客户端通信端口
httpport | 代理的http端口（与nginx配合使用）
u | 验证的用户名
p | 验证的密码

**说明**：用户名和密码验证模式，仅部分socks5客户端支持，例如proxifer。
如需验证，在服务端命令后加上
```
-u=user -p=password
```
即可

- 客户端

```
建立配置文件 config.json
```


```
./easyProxy -config config.json  
```
 名称 | 含义
---|---
config | 配置文件路径

- 需要使用内网代理的机器

```
配置sock5代理即可，ip为外网服务器ip，端口为httpport，即可在外网环境使用内网啦！也可使用proxifer等全局代理软件。
```
如果设置了用户名和密码，记得填上用户名和密码

### 配置文件config.json

```
{
  "Server": {
    "ip": "123.206.77.88",
    "tcp": 8284,
    "vkey": "DKibZF5TXvic1g3kY",
    "num": 10
  }
}
```
 名称 | 含义
---|---
ip | 服务端ip地址
tcp | 服务端与客户端通信端口
vkey | 验证密钥
num | 服务端与客户端通信连接数


## http代理模式

### 场景及原理
主要用于HTTP代理，区别也就是HTTP代理和sock5代理的区别。使用此模式时，可在非内网环境下配置本机的HTTP代理（服务器ip、HTTP代理端口），即可实现HTTP代理，达到访问内网的网站的效果。
![image](https://github.com/cnlh/easyProxy/blob/master/image/httpProxy.png?raw=true)


### 使用 
- 服务端 

```
./easyProxy -mode httpProxyServer -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=8024
```

名称 | 含义
---|---
mode | 运行模式(client、server不写默认client)
vkey | 验证密钥
tcpport | 服务端与客户端通信端口
httpport | 代理的http端口（与nginx配合使用）

- 客户端

```
建立配置文件 config.json
```

```
./easyProxy -config config.json  
```

- 需要使用内网代理的机器


```
配置HTTP代理即可，ip为外网服务器ip，端口为httpport，即可在外网环境访问内网啦！
```


 名称 | 含义
---|---
config | 配置文件路径
### 配置文件config.json

```
{
  "Server": {
    "ip": "123.206.77.88",
    "tcp": 8284,
    "vkey": "DKibZF5TXvic1g3kY",
    "num": 10
  }
}
```
 名称 | 含义
---|---
ip | 服务端ip地址
tcp | 服务端与客户端通信端口
vkey | 验证密钥
num | 服务端与客户端通信连接数

## 数据压缩支持

### 场景及原理
由于是内网穿透，内网客户端与服务端之间的隧道存在大量的数据交换，为节省流量，加快传输速度，由此本程序支持GZIP、SNNAPY两种形式的压缩，两者差异请自行选择。

### 注意点

- 所有模式均支持数据压缩
- 如使用数据压缩，客户端与服务端的压缩方式必须相同，否则将无法正常工作。

### 如何使用

**GZIP压缩**

- 在server端加上参数 -compress=gzip，例如在TCP隧道模式
```
./easyProxy -mode tunnelServer -vkey DKibZF5TXvic1g3kY -tcpport=8284 -httpport=8024 -target=10.1.50.203:80 -compress=gzip
```
- 在client端同时加上参数 -compress=gzip，例如
```
./easyProxy -config=config.json -compress=gzip
```

**SNNAPY压缩**

将参数修改为snnapy即可

## 操作系统支持
支持Windows、Linux、MacOSX等，无第三方依赖库。

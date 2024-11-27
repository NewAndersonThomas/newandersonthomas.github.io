---
layout: default
title: 科学上网
parent: VPN
nav_order: 2
last_modified_date:   2023-05-23 17:09:29 +0800
---

<details  markdown="block">
  <summary>
    TOC
  </summary>

1. [OpenVPN](#openvpn)
    1. [安装](#安装)
        1. [服务端](#openvpn-server)
        2. [客户端](#openvpn-client)
2. [代理](#代理)
    1. [shadowsocks](#shadowsocks)
        1. [服务端](#shadowsocks-server)
        2. [客户端](#shadowsocks-client)
    2. [v2ray](#v2ray)
        1. [服务端](#v2ray-server)
        2. [客户端](#v2ray-client)

</details>

# 科学上网
---
## OpenVPN
### 安装
#### 服务端<a id="openvpn-server"></a>
- 参考 [https://github.com/angristan/openvpn-install](https://github.com/angristan/openvpn-install)
- 全部默认即可
- vpn 会安装虚拟网卡接管全部流量，如果不需要全部走vpn, 需要配置route, 可以在客户端也可以在服务端配置,但最终连上服务端时，服务端会把 push "route ip mask " 这些配置发送给客户端配置文件起效, 同时需要注释掉配置文件里的 ```push "redirect_gateway def1 bypass-dhcp" ```
- 防火墙需要开放 1194/udp, 1194/tcp
#### 客户端<a id="openvpn-client"></a>
- 建议使用最新版的 [OpenVPN GUI](https://openvpn.net/community-downloads/)
- 安装后在网络适配器里会多出 OpenVPN Data Channel Offload， 旧版本(2.4)多出 本地连接2, 而且默认都有红叉(Network cable unplugged)，需要等连上服务端后才会变为可用
- 配置文件**需要去掉 ```setenv opt block-outside-dns```**， 否则会导致未走 vpn 的流量全部被block掉，而无法访问网络
- 配置route ```route 10.0.0.0 255.255.255.0 vpn_gateway```, 所有 10.*开头的ip都会走 vpn gateway, 该配置可以直接写死在客户端配置文件，也可以写在服务端 ```push "route 10.0.0.0 255.255.255.0 vpn_gateway"```
- 客户端会先将域名解析成 ip 后，才会走 vpn, 所以通过 vpn 的http 请求已是通过 DNS 解析后的 ip, vpn 只是服务器和客户端之间的一条流量通道，它不是代理，它和 shadowsocks, v2ray 这些代理有本质区别，这些代理是直接代理所请求url的服务器，vpn 查看浏览器的 remote-ip url原服务器的ip, 但代理的是代理的ip，一般是 127.0.0.1, 它们共同点都是加密通信，访问网络最终由服务器端的网络负责。 

---
## 代理
### shadowsocks
#### 服务端<a id="shadowsocks-server"></a>
- 参考 [https://github.com/shadowsocks/go-shadowsocks2](https://github.com/shadowsocks/go-shadowsocks2)
- v2ray 插件[https://github.com/shadowsocks/v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin)
- 生成自验证的ssl证书

```bash

# 生成CA根证书(用来签发服务器证书，供客户端验证证书是否可信)
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=Acme Root CA" -out ca.crt

# 签发证书，-subj 参数添加example.com域名, 强制客户端必须携带example.com域名的参数才能连接，用来伪装成正常网站的https访问
openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=GD/L=SZ/O=Acme, Inc./CN=*.example.com" -out server.csr
openssl x509 -req -extfile <(printf "subjectAltName=DNS:example.com,DNS:www.example.com") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

# 注意： ca.crt 需要发送给客户端(windows 系统则导入到 certmgr的Trusted Root Certification，以便让系统信任有该CA签发的证书。而 server.crt server.key 需要发送给服务端，server.key 是私钥，server.crt 是公钥，服务端需要用私钥解密客户端发送的数据，它不需要我们手动发送给客户端，因为客户端在连接上服务端后，服务端会发送 server.crt 给客户端，客户端会使用 ca.crt 验证 server.crt 是否可信，如果可信则会使用 server.crt 的公钥加密数据发送给服务端，服务端使用 server.key 解密数据，这样就实现了双向加密通信。

```

- 运行脚本

```bash
#/bin/sh
./shadowsocks2-linux -s 'ss://AEAD_CHACHA20_POLY1305:yourpassword@:443' -verbose     -plugin ./v2ray-plugin_linux_amd64 -plugin-opts "server;tls;host=example.com;cert=./cert/server.crt;key=./cert/server.key" &

# -s 参数是加密方式和密码，-plugin 是插件，-plugin-opts 是插件参数，server 表示服务端，tls 表示使用 tls 加密，host 是域名，cert 是证书，key 是私钥
```

#### 客户端<a id="shadowsocks-client"></a>
- 参考 [https://github.com/shadowsocks/shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows)
- v2ray 插件[https://github.com/shadowsocks/v2ray-plugin](https://github.com/shadowsocks/v2ray-plugin)
- 配置

```js
{
  "configs": [
    {
      "server": "yourserverip",
      "server_port": 443,
      "password": "yourpassword",
      "method": "chacha20-ietf-poly1305",
      "plugin": "C:\\v2ray\\v2ray-plugin_windows_amd64.exe",
      "plugin_opts": "tls;host=example.com",//ca.crt 提前导入到 windows 系统的受信任的根证书颁发机构，这里就不用配置 cert 了
      // "plugin_opts": "tls;host=example.com;cert=./cert/ca.crt", 
      "plugin_args": "",
      "remarks": "",
      "timeout": 5
    }
  ],
}
```

### v2ray
#### 服务端<a id="v2ray-server"></a>
- 参考 x-ui [https://github.com/vaxilu/x-ui/](https://github.com/vaxilu/x-ui/)
- 命令 ```bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)```
- 安装(支持docker)并且使用内置的ssl证书申请功能，它使用了 letsentrypted， 会自动续签，不用担心证书过期，也不需要安装 nginx，x-ui 内置了web服务器
- 安装后保证 v2ray 插件的状态是 running，否则可能入站配置问题
- 打开面板，配置入站：protocal: vless, 监听ip: 留白, 端口：443，开启 tls，tls 配置：域名：指向你代理服务器的域名，路径：选择ssl证书申请成功的路径,一般是在 /root/cert/你的域名.cer
- 然后查看 v2ray 的状态

#### 客户端<a id="v2ray-client"></a>
- 参考 [https://www.v2ray.com/awesome/tools.html](https://www.v2ray.com/awesome/tools.html)
- windows 平台推荐使用 [v2rayN](https://github.com/2dust/v2rayN)
- 除了及时检查更新（自动把core，geo更新）外，无需修改其它配置
- 导入订阅地址即可
---
layout: default
title: Scientific Internet Access
parent: VPN
nav_order: 2
last_modified_date:   2024-11-27 11:49:29 +0800
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

# Scientific Internet Access
---
## OpenVPN
### Install
#### Server<a id="openvpn-server"></a>
- Reference  [https://github.com/angristan/openvpn-install](https://github.com/angristan/openvpn-install)
- All defaults
- VPN will install a virtual network card to take over all traffic. If you don't want all traffic to go through VPN, you need to configure the route. You can configure it on the client or on the server. But when you finally connect to the server, the server will push "route ip mask" to the client configuration file to take effect. At the same time, you need to comment out the configuration in the configuration file.push "redirect_gateway def1 bypass-dhcp"
- The firewall needs to open 1194/udp, 1194/tcp
#### Client<a id="openvpn-client"></a>
- It is recommended to use the latest version of [OpenVPN GUI](https://openvpn.net/community-downloads/)
- After installation, there will be an additional OpenVPN Data Channel Offload in the network adapter. The old version (2.4) has an additional Local Area Connection 2, and by default there is a red cross (Network cable unplugged). You need to wait until you connect to the server before it becomes available.
- The configuration file **needs to be removed ```setenv opt block-outside-dns```**， otherwise all traffic that does not go through the VPN will be blocked and the network will not be accessible
- Configure route```route 10.0.0.0 255.255.255.0 vpn_gateway```, all IPs starting with 10.* will go through the VPN gateway. This configuration can be written directly in the client configuration file or in the server. ```push "route 10.0.0.0 255.255.255.0 vpn_gateway"```
- The client will first resolve the domain name into an IP before going through the VPN, so the http request through the VPN is the IP resolved by DNS. The VPN is just a traffic channel between the server and the client, it is not a proxy. It is fundamentally different from proxies such as shadowsocks and v2ray, which directly proxy the server of the requested URL. The VPN checks the IP of the original server of the remote-ip URL of the browser, but the proxy is the IP of the proxy, which is usually 127.0.0.1. What they have in common is that they are encrypted communications, and the server-side network is ultimately responsible for accessing the network. 

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
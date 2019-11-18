# Shadowsocks+ Privoxy在Ubuntu中使用socks5代理


## 背景

在日常工作经常碰到需要使用的依赖包下载不下来的情况，这时候就需要使用代理服务了，shadowsocks是一个socks5代理软件，而日常使用中的很多软件都需要的是HTTP代理。所需需要借助Privoxy将socks5代理转化成HTTP代理。

> 这里假设你已经拥有一个shadowsocks代理服务器

## 安装shadowsocks

``` bash
sudo apt install shadowsocks
```
安装shadowsocks后会有一个`sslocal`的命令，通过修改`/etc/shadowsocks/config.json`来配置shadowsocks服务器信息。

需要配置的config.json的文件如下：

``` json
{
    "server":"*.*.*.*",
    "server_port": *,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"*",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": true,
    "workers": 1,
    "prefer_ipv6": false
}
```

执行如下命令既可以启动服务：
``` bash
sslocal -c /etc/shadowsocks/config.json -v 
```


## 使用Privoxy将socks5代理转化成HTTP代理

### 安装和配置Privoxy：

``` bash
sudo apt install privoxy
```

修改 `/etc/privoxy/config`


```  bash
## 监听本地本端（即本机HTTP代理端口）
listen-address 127.0.0.1:8008

# 将所有转发到socks5代理中
forward-socks5 / 127.0.0.1:1080 .
```

通过上述配置我们即得到`HTTP_proxy`,使用HTTP代理所有的请求都会转发到本地`8008`端口，然后转发给shadowsocks服务。


## 设置环境变量

我这里修改的是`~/.zshrc`,在文件最后面添加：

``` bash
export HTTPs_proxy=HTTP://127.0.0.1:8008
export HTTP_proxy=HTTP://127.0.0.1:8008
```

然后执行`source ~/.zshrc`  让环境变量生效。

到这里我们已经完成了所有配置，但是问题是，所有的HTTP请求都被被转到到shadowsocks中，这是不要的，我们期望的是需要被代理的的才经过shadowsocks。

## 改进

具体参照`HTTPs://github.com/zfl9/gfwlist2privoxy`，`/etc/privoxy/config`的最终配置如下：
``` bash
## 监听本地本端（即本机HTTP代理端口）
listen-address 127.0.0.1:8008

# 将所有转发到socks5代理中
# forward-socks5 / 127.0.0.1:1080 .

## 需要被转发的文件
actionsfile gfwlist.action
```

> 修改完`/etc/privoxy/config`配置文件记得重启 privoxy服务。


## 将shadowsocks做成开启服务

创建文件`/etc/systemd/system/shadowsocks.service`，并做如下修改：

``` bash
[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks/config.json

[Install]
WantedBy=multi-user.target
```

然后再执行如下命令，将服务作为开启启动：

``` bash
systemctl enable /etc/systemd/system/shadowsocks.service
```

然后重启机器，查看运行状态：
``` bash
service shadowsocks status
```
如果是`active`则表示加入开启启动成功。


# 欢迎关注微信公账号

![](../images/qrcode_for_gh_8a4c1ef089cc_258.jpg)







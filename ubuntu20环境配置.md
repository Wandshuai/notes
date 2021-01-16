## 网卡驱动

联想笔记本安装 ubuntu 20.04  之后发现没有无线网卡驱动。

因为坑爹的Secure Boot 导致安装过程中装网卡驱动失败了，需要从BIOS把 Secure Boot

 给禁掉。

```
sudo mokutil --disable-validation
```

会要求你输入密码,简单的12345678就行，因为改BIOS的Secure Boot Status的时候会让你输入刚刚输入的密码的第几个字符。

禁掉之后可能驱动版本不太对，需要手动安装

查询网卡型号：

```
lspci -nnk | grep -iA2 net
```

其中`Ethernet Controller`表明你的电脑有以太网卡，`Network controller`表明你的电脑有无线网卡。

我的电脑无线网卡型号是

```
Network controller [0280]: Realtek Semiconductor Co., Ltd. RTL8821CE 802.11ac PCIe Wireless Network Adapter
```

默认网卡驱动太高，据说这是一个坑爹的网卡，从[github](https://github.com/M000M/Driver-Realtek-RTL8821CE)上随便搜了一个版本，需要手动编译安装

```
sudo make

sudo make install

sudo modprobe -a 8821ce

sudo reboot

```

重启就可以看到wifi了,装完之后不要随便删，以防下次重启失效。



## 代理配置

首先需要安装shadowsocks

```
apt-get install shadowsocks-libev -y
```

安装完成后编辑配置文件，配置文件存放路径没有要求，主要是方便启动客户端

```
vi /etc/shadowsocks-libev/config.json
```

服务器配置样例如下

```
{
    "server":"xxx.xx.xx.xx",
    "mode":"tcp_and_udp",
    "server_port":xxxx,
    "local_port":1080,
    "password":"xxxxxxxx",
    "timeout":60,
    "method":"aes-256-cfb"
}
```

启动代理服务器

```
ss-local -c  /etc/shadowsocks-libev/config.json &
```

不想通过配置文件启动也可以

```
ss-local -s xxx.xx.xx.xx（ip） -p xxxxx(server_port) -l 1080 -k xxxxx(passwd) -m aes-256-cfb &
```

命令行不能直接使用，安装代理转换 `privoxy`，将 `Socks5 ` 代理转化为 `http ` 代理

```
sudo apt-get install -y privoxy
```

编辑 `privoxy` 的配置文件。

```bash
sudo vim /etc/privoxy/config
```

*这里有一点需要注意的，请看下面的备注*

```bash
# 下面一行是默认存在的，如果没有，则需要添加
# 如果 `8118` 端口被占用了，则更换一个未被占用的端口就好
# 但是要记住端口号，后面需要对此做配置的
listen-address 127.0.0.1:8118

# 下面一行是默认存在的，如果没有，那恭喜你
# 如果有，那么需要你默默的在最前面加个 `#`
# 不然启动会失败
listen-address [::1]:8118

# 下面的内容是需要添加进配置文件的，需要注意的是 `.` 必须要
forward-socks5 / 127.0.0.1:1080 .
```

配置完成后，直接启动 `privoxy`

```bash
sudo service privoxy start
```

如若启动失败，那就看日志继续查吧，下面是日志的地址：

```bash
/var/log/privoxy/logfile
```

最后一步了，我们需要设置环境变量，指定 `http`、`https` 等链接走我们指定的代理。

```bash
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
```

OK，来试一下成功没有。

```bash
curl www.google.com
curl https://www.google.com
```

浏览器代理配置，我使用的forefox,chrome和它配置类似

安装 SwitchyOmega插件

通过 [Firefox Add-Ons](https://addons.mozilla.org/zh-CN/firefox/addon/switchyomega/) 在线安装，如果无法访问，也可以从 Github [直接下载](https://github.com/FelisCatus/SwitchyOmega/releases)安装包

配置 Proxy

- `Server`填写`shadowsocks.json`配置中的`local_address`
- `Port`填写`shadowsocks.json`配置中的`local_port`
- 左边`Apply changes`保存。

配置 Auto Switch

Rule list rules的Profile填`proxy`
Default的Profile填`Direct`
Rule List Format选择`AutoProxy`

`Rule List URL`填写

```
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

引用https://www.vpnto.net/posts/firefox-shadowsocks/

### 自动启动

privoxy 默认是自动启动的，需要配置下ss-local自动启动

以下使用Systemd来实现shadowsocks开机自启。

```
sudo vim /etc/systemd/system/shadowsocks.service
```

在里面填写如下内容：

```
[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/bin/ss-local -c  /etc/shadowsocks-libev/config.json

[Install]
WantedBy=multi-user.target
```

配置生效：

```
systemctl enable /etc/systemd/system/shadowsocks.service
```

输入管理员密码就可以了。

## 火狐浏览器看视频

preferences -> general ->Play DRM-controlled content 勾上

```
sudo apt install libavcodec-extra
```


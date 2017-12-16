---
title: Ubuntu下使用SS
date: 2017-12-09 13:56:57
categories: [笔记]
tags: [Linux, ss]
---

# 环境

Ubuntu 16.04，连接到互联网。

# 安装、配置、启动ss

安装命令：

> sudo apt-get install python-pip
> pip install shadowsocks



运行`sslocal`时需要指定几个参数：

> "server": "服务器的ip",
> "server_port": 服务器的端口,
> "local_port": "127.0.0.1",
> "password": 1080
> "timeout": 500,
> "method": "aes-256-cfb"

上述参数可以在命令行中指定，也可使用配置文件，然后用指定配置文件启动`sslocal`

> sslocal -c /home/test/shadowsocks.json

每次输入太麻烦，可以写个脚本：

```bash
#! /bin/bash
#shadow.sh
sslocal -c /home/test/shadowsocks.json
```

不管在终端中输入还是运行脚本，整个终端会被占据，所以可能需要后台运行：

> nohup sh /home/usr/shadow.sh > /home/usr/log.out 2>&1 &


但是我们不想每次都手动去输入上面的命令，解决的办法就是加入开机运行，将上述命令加到`rc.local`文件。编辑`rc.local`文件，在`exit`前输入上述命令。然后`reboot`。


上述设置后，一切运行正常，但是想通过`chrome`上外网，发现还是不行。有两个方法可以让`chrome`上外网，第一个是`HTTP`代理，第二个是`SwitchOmega`。

## HTTP代理
安装`polipo`进行`HTTP`代理：

> sudo apt-get install polipo

修改`polipo`配置文件(/etc/polipo/config)如下：

```bash
# This file only needs to list configuration variables that deviate
# from the default values. See /usr/share/doc/polipo/examples/config.sample
# and "polipo -v" for variables you can tweak and further information.
logSyslog = false
logFile = "/var/log/polipo/polipo.log"

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5

chunkHighMark = 50331648
objectHighMark = 16384

serverMaxSlots = 64
serverSlots = 16
serverSlots1 = 32

proxyAddress = "0.0.0.0"
proxyPort = 8123
```

重启`polipo`:

> /etc/init.d/polipo restart

此时可一检测下代理是否正常运行：

> export http_proxy="http://127.0.0.1:8123/"
> curl www.google.com

如果一切正常，终端会显示抓取到的`Google`网页的内容。

### Chrome参数

上述一切正常运行，从左侧启动`Chrome`，依旧无法上外网。上面能正常抓取`Google`网页是因为在那个`Terminal`中加入了`http_proxy`变量。所以也要把类似`http_proxy`的代理加入到启动`Chrome`参数中。`Luancher`中的启动项可以在`usr/share/applications`下找到。编辑此目录下的`google-chrome.desktop`文件，找到`Exec`命令行，在末尾加入`http`代理服务器配置：

```bash
Exec=/usr/bin/google-chrome-stable --incognito --proxy-server="http://127.0.0.1:8123"
```

从`Luancher`的`Chrome`图标中，有三处可以启动，因此需要找到另外两个`Exec`命令行，加入：

> --proxy-server="http://127.0.0.1:8123"


## SwitchOmega
可以从[Github](https://github.com/FelisCatus/SwitchyOmega/releases)上下载最新的`chrome`版。在`Chrome`地址栏输入`chrome://extensions`打开扩展程序，拖动 .crx 后缀的`SwitchyOmega`安装文件到扩展程序中进行安装。

安装成功后，增加一个类型为`Proxy Profile`的配置文件，或者直接在原有的修改：
- Protocol: SOCKS5
- Server: 127.0.0.1
- Port: 1080

然后点击`Apply Changes`。

同时也可以增加一个`Switch Profile`的配置文件，这个可以自动根据一个域名的列表清单自动选择是否需要代理。配置文件进行如下修改：
- Switch rules的 Rule list rules的Profile列选择上述的proxy配置文件名
- Rule List URL: https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt，点击下载

然后点击`Apply Changes`。

这是就可以在扩展图标中选择`SwitchOmega`，然后选择自动切换选项。 


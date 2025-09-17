+++
date = '2024-11-28T21:08:39+08:00'
draft = false
title = 'nvda远程服务器在linux的搭建'
categories = ['经验分享']
tags = ['linux', 'nvda']
slug = 'nrs'
+++
今天来说说NVDA远程服务器在linux的搭建，windows自行看说明研究。
主要是针对debian、ubuntu等系统，centOS的大同小异。
## 学习之前
你需要掌握
- linux的基本知识
- 熟练使用WinSCP
- 魔法{非必须}
## 获取文件
可以从[NVDARemoteServer的release页面](https://github.com/tech10/nvdaRemoteServer/releases/latest/)找到最新版本的文件，通常，linux64位的就下载[nvdaRemoteServer_linux_amd64.tar.gz](https://github.com/tech10/nvdaRemoteServer/releases/latest/download/nvdaRemoteServer_linux_amd64.tar.gz)，32位就下载386的，arm处理器的就选择带arm的，这里不再赘述。
如只是想安装，不想要那么多自定义功能，或者只想直接使用，下载困难，可从文末下载。
## 解压相关文件并赋予权限
我们仍使用[WinSCP](https://winscp.net/eng/download.php)进行相关操作。
### 对于小白用户
从文墨下载我的两个安装脚本，把nrs.tgz复制到vps的home文件夹，在命令行中输入
```sh
tar -xzvf nrs.tgz && cd nrs && chmod +x install.sh && ./install.sh
```
这样就安装完成了，运行脚本的时候要输入证书和私钥的绝对路径，如果无证书文件可直接留空。
把check-cert-update.tgz复制到任意位置，然后执行命令
```sh
tar -xzvf check-cert-update.tgz && cd check-cert-update && chmod +x install.sh && ./install.sh
```
同样，输入证书的绝对路径，即可配置完成。
### 对于想自定义使用或者想自己动手的高级用户：
从github下载好相关文件，连接到VPS后，找个你想放的文件夹，例如我放在/home里，右键菜单、文件自定义命令、UnTar/GZip，回车两次搞定。
我们可以重命名一下文件夹，这名字太长了，比如改成nrs，然后f9给文件夹权限，改成0777，并选中递归设置所有者、分组与权限复选框，然后回车确定。
进入文件夹内部，进入systemd文件夹，修改nvdaRemoteServer.service文件，把
User=sample
和
Group=sample
从sample改成root，然后更改ExecStart的内容。
ExecStart=/home/sample/bin/nvdaRemoteServer -cert-file /home/sample/nvdaRemoteServer/cert.crt -key-file /home/sample/nvdaRemoteServer/cert.key -log-level=3
首先当然是主进程的位置，我们放在了/home/nrs里面，那么就要更改指向路径的位置，就变成了
/home/nrs/nvdaRemoteServer
### 更改-cert-file的路径
这里分几种情况：
#### 没有ssl证书
如果你没有ssl证书，那么就可以把cert-file和cert-key这两个参数直接删掉。
#### 有ssl证书
那么就可以把后面的路径改成你指向你ssl证书和秘钥的路径，格式不限，不管是.crt还是.cer，都行，能打开查看里面的内容就行。
### 使用配置
-conf-file是使用配置文件的意思，后面加上配置文件的路径，至于如何得到配置文件，后面在说，首先我这么写：
-conf-file /home/nrs/nvdaRemoteServer.json
### 日志相关
#### -log-level
日志等级，-1到4，-1就是关闭日志，数字月高月详细。
#### 程序输出日志
默认是
StandardOutput=append:/home/sample/nvdaRemoteServer/stdout.log
改成你想放这个日志的地方，比如我改成
StandardOutput=append:/home/nrs/stdout.log
注意你设置的输出日志的文件夹必须存在，比如放在/home/log/nrs-log里面，就要保证有log和nrs-log这两个文件夹并有相关权限，没有就要先创建在运行服务，否则会报错
#### 程序错误日志
同上，比如我改成
StandardError=append:/home/nrs/stderr.log
当然，最后日志的文件名也可以随意更改。
### 最终配置
最后改好的配置就是
```ini
ExecStart=/home/nrs/nvdaRemoteServer -cert-file /home/nrs/cert.crt -key-file /home/nrs/cert.key -log-level=3 -conf-file /home/nrs/nvdaRemoteServer.json
StandardOutput=append:/home/nrs/stdout.log
StandardError=append:/home/nrs/stderr.log
```
注意证书文件不能直接用，要自己证书的位置，没有证书就删掉cert-file和key-file这两个参数吧，注意后面的路径要一起删掉，如果你以后或得了证书文件可以直接改配置文件，不需要在到找这里改了。
### 复制文件
把nvdaRemoteServer.service重命名为nrs.service，这样方便命令输入，放到
/etc/systemd/system/里面。
## 程序配置
我会给一个配置文件的样本，后面也会解释相关参数，只需要在电脑上新建一个文本文件，粘贴配置的内容，文件名改成nvdaRemoteServer.json
复制到服务器的/home/nrs目录下即可，当然这是我在上面所写的路径，如果你不想放在这里或者文件名不想这么长，也可以自行修改，当然也要记得同时修改nrs.service里面的`-conf-file`及其后面的路径。
```json
{
	"pid_file": "",
	"log_file": "",
	"log_level": 0,
	"addresses": [
		":6837"
	],
	"cert_file": "",
	"key_file": "",
	"motd": "1",
	"motd_always_display": false,
	"send_origin": true
}
```
注：命令行执行的参数高于配置文件的参数，相信细心的你已经看到，在这个配置文件里也有证书秘钥和日志等级的参数，但其实可以不必理会，因为在上面的nrs.service里面已经指定了位置，那个优先级要高于配置文件的，除非你想缩短ExecStart的内容，及删掉证书秘钥和日志等级的相关参数。
下面我会逐一解释配置的每个参数：
### pid_file:
这是程序pid的位置，我没有更改它，保持了留空。
### log_file:
日志文件的位置，注意不包含错误日志，所以没必要也不需要改，因为在nrs.service改过了。
### log_level:
日志等级，高一级的日志也会同时记录比它低的内容，比如设成1，就会同时记录0和1的内容，2就是记录0、1和2的内容，不过日志等级也在上面的service文件里，想改就改ExecStart的内容，除非你删掉了service文件里关于日志等级相关内容，具体数字涵义如下：
- -1 将禁用日志记录，但错误日志除外，这些信息始终会被记录。这包括导致程序崩溃的严重错误。
- 0 会在服务器启动、停止或发生不严重到需要始终记录的错误时进行日志记录。
- 1 会记录连接的客户端的信息，包括它们的ID和IP地址。
- 2 会记录每个客户端加入和离开的频道，其中包含频道密码。不要在生产环境中使用此日志级别。
- 3 会记录程序在每个操作阶段所做的事情。仅用于调试目的。
- 4 会记录服务器和客户端之间交换的协议。除非你是开发人员或者想要分析所使用的协议，否则不要使用这个级别。这可能会导致性能下降，因为交换协议的信息也会发送到控制台，或者如果你的操作系统设置了重定向，也会写入文件。
### addresses:
无必要一般不用修改。
程序监听传入连接的地址，格式为IP地址:端口。默认情况下，使用所有地址，服务器接受端口6837上的连接。
如果你想监听IPV6地址，该地址必须用方括号括起来。例如：
\[fd80::ffe8]:6837
只要您的某个网络接口上有一个有效的IPV6地址，例如IPV6地址的本地前缀，您就可以监听到服务器的传入连接。要只监听IPV6地址，请使用以下示例。
\[::]:6837
要监听IPV4地址，只需使用以下示例中的有效参数即可，该参数将仅监听所有IPV4地址。
0.0.0.0:6837
如果没有声明地址，默认端口为6837。有效端口号介于1和65536之间。在声明服务器监听的地址时，您还必须声明端口，否则参数将无效。但是，您不需要声明地址。例如，如果你想监听所有地址，但使用端口5000，请使用以下命令。
:5000
### cert_file:
证书文件位置
### key_file:
秘钥文件位置
### motd:
每日消息内容
### motd_always_display:
设置是否每次连接都显示没日消息，如果为true则每次连接都显示，false为第一次连接显示。
### send_origin:
默认情况下，当服务器收到来自客户端的消息时，会将相同的消息发送给所有需要接收的客户端，但会在该消息中添加一个来源字段。这要求将消息解码为程序更易操作的值，添加来源信息后，再编码回要发送给所有客户端的值。这可能会导致轻微的性能损失。如果需要，可以通过设置为 false 来禁用此功能，不过这样做可能会导致某些功能无法正常工作。如果将其设置为 false，服务器会警告您，在需要来源字段时，这可能会影响客户端的功能。
没必要不需要改。
## nvda远程服务器需要用到的命令
### 设置开机自启
```sh
systemctl enable nrs
```
### 启动nvda远程服务器
```sh
systemctl start nrs
```
### 停止nvda远程服务器
```sh
systemctl stop nrs
```
### 重启nvda远程服务器
```sh
systemctl restart nrs
```
### 禁止nvda远程服务器开机自启动
```sh
systemctl disable nrs
```
## 后记
一切设置完成之后就成功搭建了nvda远程服务器，更详细的说明可以去看压缩包里的readme.md文件。
## 附：当ssl证书更新时自动重启nvda远程服务器已加载最新的ssl证书
如果你是用caddy申请的证书，可以看看[晴天老师的文章](https://www.qt06.com/post/424/)
如果你用的是acme.sh申请的免费的证书，如ZeroSsl、Let's Encrypt等，通常证书文件只有90天，一般来说，会在证书有效期剩余30天左右acme.sh之类的脚本会给你自动续期新证书，那么问题来了，如果只是证书更新，但是nrs服务没有重启，它加载的还是以前的证书，等到90天过去，就会提示证书失效了，该怎么解决这个问题呢？
我的思路是这样的：
- 创建一个检查证书有效期的脚本文件
- 将其作为系统服务开机自启
- 设置一个定时器定期运行这个脚本检查证书的有效期
下面我会具体展开说说
首先创建一个名为
check-cert-update.sh
的脚本，内容如下
```sh
#!/bin/bash

CERT_FILE="/usr/local/nginx/conf/ssl/www.yydjtc.top/fullchain.cer"
SERVICE_NAME="nrs"
TIMESTAMP_FILE="/var/lib/check-cert-update.timestamp"

if [ ! -f "$TIMESTAMP_FILE" ]; then
    # If timestamp file doesn't exist, create it with the current timestamp of the cert file
    stat -c %Y "$CERT_FILE" > "$TIMESTAMP_FILE"
    exit 0
fi

LAST_MODIFIED=$(stat -c %Y "$CERT_FILE")
LAST_CHECK=$(cat "$TIMESTAMP_FILE")

if [ "$LAST_MODIFIED" -ne "$LAST_CHECK" ]; then
    echo "$LAST_MODIFIED" > "$TIMESTAMP_FILE"
    systemctl restart "$SERVICE_NAME"
fi
```
如果你的证书位置不同自行修改，注意续签的证书和现在证书的文件名要一样。
把它放在
/usr/local/bin/
里面，并给相应的权限。
创建一个服务
check-cert-update.service
内容如下：
```ini
[Unit]
Description=Check for SSL certificate updates and restart nrs service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/check-cert-update.sh
```
继续，创建一个定时器让他定时执行，创建一个名字叫
check-cert-update.timer
的文件，内容如下：
```ini
[Unit]
Description=Run check-cert-update.service every 15 minutes

[Timer]
OnBootSec=5min
OnUnitActiveSec=15min

[Install]
WantedBy=timers.target`
```
把它和
check-cert-update.service
放在同一文件夹，就是
/etc/systemd/system/

然后命令行输入
```bash
systemctl daemon-reload && systemctl enable check-cert-update.timer && systemctl start check-cert-update.timer
```
### 解析：
1、  变量定义：
- `CERT_FILE`
：定义证书文件的路径。
- `SERVICE_NAME`
：定义需要重启的服务名称。
- `TIMESTAMP_FILE`
：定义存储上次检查时间戳的文件路径。
2、  检查时间戳文件是否存在：
- 如果不存在，创建它并记录证书文件的当前修改时间戳，然后退出脚本。
3、  获取时间戳：
- `LAST_MODIFIED`
：获取证书文件的当前修改时间戳。
- `LAST_CHECK`
：读取上次检查时记录的时间戳。
4、  比较时间戳：
- 如果当前时间戳和上次检查的时间戳不同，更新时间戳文件，并重启  `SERVICE_NAME` 服务。
这样，不仅仅是nrs，理论上所有需要重新加载证书的程序都可以用这个脚本，如果需要多个程序都重启，就改一改脚本文件，定义多个变量，例如`SERVICE1_NAME`、`SERVICE2_NAME`，相应的在下面加上对应程序需要的命令就行了。
## 文件下载
若无法直接下载可从网盘进行下载。
### nvda远程服务器一键安装脚本
[本地下载我整理好的nrs一件安装脚本](https://yydjtc.top/download/nrs.tgz)
[123网盘下载我整理好的nrs一件安装脚本](https://www.123pan.com/s/egE9-Rr1Jv.html)
### 证书自动监控一件安装脚本
[本地下载证书监控一件安装脚本](https://yydjtc.top/download/check-cert-update.tgz)
[123云盘下载证书监控一键安装脚本](https://www.123pan.com/s/egE9-G81Jv.html)
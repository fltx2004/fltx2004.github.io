+++
date = '2024-11-28T21:08:29+08:00'
draft = false
title = '说说teamtalk服务器的安装'
 categories = ['经验分享']
tags = ['teamtalk', 'linux']
+++

这里我主要说说linux teamtalk服务器的配置，windows也会顺便说一下。
## 下载Teamtalk:
首先要下载Teamtalk，可以从[bearware](https://bearware.dk/?page_id=353)下载。
带Portable的是windows的绿色版，
.pkg的是mac版，
linux中，debian和ubuntu用后面带ubuntu的，cent用后面是centos的。
## 安装teamtalk：
### windows
#### 便携版
这个没啥好说的，便携版打开直接用，客户端在
Client
文件夹里，服务器在
Server
里，直接运行
tt5svc_install.bat
，就有服务器的配置向导了。
#### 安装板
运行安装包的时候选择组件的时候，如果只需要客户端那默认就行，如果想要带服务器的，组件就选下面的那个，
TeamTalk 5 Client & Server
，安装好，在开始菜单的程序组里，打开teamtalk5文件夹，找
install nt service
类似的文件，我已经好长时间不用windows的tt服务器了，大概是这样的，反正带install的就是了。
### linux
#### 安装WinSCP
方便新手，可以用[WinSCP](https://winscp.net/eng/downloads.php)，很好的图形化文件管理软件，
安装好了打开，新建站点这里，后面就有可填写的选项了，文件协议选择SCP，后面的主机地址、端口、用户名和密码自行填写，在保存的地方可以展开选择保存，里面可以选择创建桌面快捷方式、保存密码等，自行按需选择。
#### 复制解压tt安装文件
连接好了之后，复制下载回来的tgz文件，放到一个你喜欢的地方，比如我放到了home里，你问怎么复制？就像资源管理器一样的操作，复制粘贴。
复制好了，在tt压缩包右键选择
文件自定义命令
，展开，点击UnTar/GZip，在点击确定即可。
同样，解压开也是Client和Server两个文件夹，如果不需要客户端可以删掉，如果想更改server文件夹名字可以在文件夹按f2，修改即可，后面的配置的文件路径也自行修改成你自己的。
在本教程中，我把tt共享文件存放在server里的files文件夹，所以我要进入server文件夹，按f7创建一个新文件夹，名称就叫files，如果你不想放在这里也可自行在后面修改。
#### 赋予文件夹权限
由于tt需要在teamtalk组和隶属于teamtalk组的同名用户下运行，所以创建用户组，
可以用putty或者windows自带的ssh工具，这里已windows的ssh为例：假设服务器地址是123.456.789.0，win+r打开运行对话框，然后输入
```cmd
ssh root@123.456.789.0
```
，这只是用于默认22端口，如果你更改过ssh端口,假设改成了12345，就这样：
```cmd
ssh -p 12345 root@123.456.789.0
```
第一次连接可能会让确认指纹，输入
yes
即可，然后输入root账户密码，这里不会显示输入的字符，输入好了直接回车就行。
然后开始添加用户和组了，输入
```sh
groupadd teamtalk && useradd -g teamtalk teamtalk
```
这样就可以了。
我们回到WinSCP，退回到server的上一级文件夹，也就是home，找到server，按f9，在确定的前面的编辑框，由
0755
改成
0777
，并选中后面的递归设置所有者、分组与权限，确定后面的两个编辑框，现在应该是root，都改成teamtalk，点击确定即可。
#### teamtalk配置修改
##### 默认配置
如果想遵循默认安装，那么把tt5srv复制到
/usr/bin
里面。
##### 自定义配置
进入server/systemd文件夹，编辑tt5server.service，把
ExecStart=/usr/bin/tt5srv -nd -c /etc/teamtalk/tt5srv.xml -l /var/log/teamtalk/tt5srv.log
改成
```ini
ExecStart=/home/server/tt5srv -nd -c /home/tt5srv.xml -l /home/server/tt5srv.log
```
这里如果你tt放置的文件夹不同，修改的路径也不同，
-l
后面的是日志文件，
-c
后面的是配置文件。
如果不想都放在server文件夹，也可以不修改
-c
和
-l
后面的路径。
然后把systemd里面的
tt5server.service
复制到
/etc/systemd/system
里面。
### 运行tt服务器配置向导
执行
```sh
/home/server/tt5srv -wd /home/server -c /home/server/tt5srv.xml -l /home/server/tt5srv.log -wizard -nd
```
，这里如果tt服务器文件位置不同，也自行修改。
这样就会弹出和windows一样的设置向导了，都可以用翻译看明白，注意如果启用了文件共享，就要在路径里输入
```path
/home/server/files
```
，同理，如果你共享文件放的位置与我不同，也自行改成你的路径。
### 设置tt服务器开机自启动
命令行输入
```sh
systemctl enable tt5server
```
就可以开机自启动了。
## 结束
至此，tt服务器就安装成功了，当服务器重启之后tt服务器也会开机自启动，不用去手动启动，非常方便。
## 附：tt服务器相关命令
让tt服务器开机自启动
```sh
systemctl enable tt5server
```
禁用tt服务器开机自启动
```sh
systemctl disable tt5server
```
启动tt服务器
```sh
systemctl start tt5server
```
停止tt服务器
```sh
systemctl stop tt5server
```
重启tt服务器
```sh
systemctl restart tt5server
```

+++
date = '2026-07-10T11:05:18+08:00'
title = 'TeamTalk服务器docker版'
categories = ['软件分享']
tags = ['TeamTalk','服务器','docker']
slug = 'tt5srv-for-docker'
+++

项目地址：
- [github]
- [Docker]

昨天下午突然想把tt的服务器做成一个docker版的，不为别的，就为了系统兼容。

在软路由上想运行tt服务器除了用docker没啥太好的办法，之前也有好多朋友想在软路由上挂tt服务器，但由于官方没出docker版的，就只能自己构建。

昨天我先在本地构建了它，然后正常跑起来了，我转念一想，为什么不把它公开出去，方便自己以后直接拉取，也方便我的朋友们。

于是，这个项目诞生了。

注意，仅支持x86/64处理器

## 实现方式

采用[github action]进行构建。

- 通过 `version.json`的版本信息自动下载[TeamTalk]对应版本压缩包
- 解压出里面的 `/server/tt5srv`
- 登录到DockerHub
- 构建并推送

### 具体使用方法：

支持平台：`linux/amd64`

镜像内部使用 Ubuntu 24.04 提供 TeamTalk 所需的 glibc，因此可以运行在
Ubuntu、Debian、Fedora、Rocky Linux、Arch Linux 和 OpenWrt x86_64 等安装了
Docker 的 Linux 系统上。

## 数据目录

容器内的持久化目录是：

```text
/data
```

下面的示例将它映射到宿主机的 `/opt/tt5srv/data`：

| 内容 | 宿主机路径 | 容器内路径 |
| --- | --- | --- |
| 配置文件 | `/opt/tt5srv/data/tt5srv.xml` | `/data/tt5srv.xml` |
| 日志文件 | `/opt/tt5srv/data/tt5srv.log` | `/data/tt5srv.log` |

宿主机目录可以自行修改，但容器内路径应保持为 `/data`。在 OpenWrt 上应将
宿主机目录放在持久化存储中。

先创建数据目录：

```sh
mkdir -p /opt/tt5srv/data
```

## 首次配置

首次启动前运行 TeamTalk 配置向导：

```sh
docker run --rm -it \
  --network host \
  -v /opt/tt5srv/data:/data \
  fltx2004/tt5srv:latest \
  -wizard -wd /data
```

向导询问文件存储目录时，最好在/data 里提前创建共享文件夹，就可以填写：

```text
/data/files
```

向导生成的 `tt5srv.xml` 会保存在宿主机的 `/opt/tt5srv/data` 中。

## 使用 docker run 启动

完成配置后启动服务器：

```sh
docker run -d \
  --name tt5srv \
  --restart unless-stopped \
  --network host \
  -v /opt/tt5srv/data:/data \
  fltx2004/tt5srv:latest \
  -nd -wd /data -l /data/tt5srv.log -verbose
```

以上命令以前台模式运行服务，并读取 `/data/tt5srv.xml`。

如果不希望使用 host 网络，可以将 `--network host` 替换为端口映射：

```sh
-p 10333:10333/tcp -p 10333:10333/udp
```

如果向导中修改了端口，需要同步修改映射端口。

## 使用 Docker Compose 启动

创建 `/opt/tt5srv/compose.yaml`：

```yaml
services:
  tt5srv:
    image: fltx2004/tt5srv:latest
    container_name: tt5srv
    network_mode: host
    restart: unless-stopped
    volumes:
      - /opt/tt5srv/data:/data
    command:
      - -nd
      - -wd
      - /data
      - -l
      - /data/tt5srv.log
      - -verbose
```

如果尚未运行配置向导：

```sh
docker compose -f /opt/tt5srv/compose.yaml run --rm \
  tt5srv -wizard -wd /data
```

启动服务器：

```sh
docker compose -f /opt/tt5srv/compose.yaml up -d
```

如果已经通过 `docker run` 创建了同名容器，第一次切换到 Compose 前需要先
删除旧容器：

```sh
docker rm -f tt5srv
```

绑定目录中的配置和上传文件不会被删除。

旧版 Compose 使用 `docker-compose` 命令，其余参数相同。

## 日常管理

查看运行状态：

```sh
docker ps --filter name=tt5srv
```

查看实时日志：

```sh
docker logs -f tt5srv
```

重启、停止或启动服务器：

```sh
docker restart tt5srv
docker stop tt5srv
docker start tt5srv
```

查看镜像中的 TeamTalk 完整版本：

```sh
docker run --rm fltx2004/tt5srv:latest --version
```

## 升级

升级前建议备份数据目录：

```sh
tar -C /opt/tt5srv -czf \
  /opt/tt5srv-backup-$(date +%Y%m%d-%H%M%S).tar.gz data
```

### 使用 Docker Compose 升级

```sh
docker compose -f /opt/tt5srv/compose.yaml up -d --pull always
```

旧版 Compose 可以使用：

```sh
docker-compose -f /opt/tt5srv/compose.yaml pull && \
docker-compose -f /opt/tt5srv/compose.yaml up -d
```

### 使用 docker run 升级

先拉取新镜像：

```sh
docker pull fltx2004/tt5srv:latest
```

删除旧容器并使用相同的数据目录重新创建：

```sh
docker rm -f tt5srv

docker run -d \
  --name tt5srv \
  --restart unless-stopped \
  --network host \
  -v /opt/tt5srv/data:/data \
  fltx2004/tt5srv:latest \
  -nd -wd /data -l /data/tt5srv.log -verbose
```

## 固定版本和回滚

生产环境可以使用固定版本标签：

```text
fltx2004/tt5srv:5.22
```

Compose 中相应修改为：

```yaml
image: fltx2004/tt5srv:5.22
```

修改版本后重新创建容器：

```sh
docker compose -f /opt/tt5srv/compose.yaml up -d --pull always
```

如果新版本出现问题，将镜像标签改回之前的版本并再次执行上述命令即可回滚。

自动切换到新镜像，仍然需要重新创建容器。

## SELinux 系统

Fedora、Rocky Linux、RHEL 等启用 SELinux 的系统如果遇到目录权限错误，
可以给绑定目录添加 `:Z`：

```yaml
volumes:
  - /opt/tt5srv/data:/data:Z
```

对应的 `docker run` 参数为：

```sh
-v /opt/tt5srv/data:/data:Z
```

## 结语

至于具体啥效果，我还不知，自己都没测试过哈哈哈，有甚么问题欢迎提出。

[github action]: https://docs.github.com/en/actions
[DockerHub]: https://hub.docker.com
[TeamTalk]: https://bearware.dk/?page_id=327
[github]: https://github.com/fltx2004/tt5srv-for-docker
[docker]: https://hub.docker.com/r/fltx2004/tt5srv/
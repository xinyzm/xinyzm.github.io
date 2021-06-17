---
title: CentOS7 环境下快速搭建服务节点
keywords: docker docker-compose
catalog: true
comments: true
date: 2020-10-17 16:20:00
subtitle: 
header-img: snail-bg.jpg
tags:
- Linux
- Docker
categories:
- 运维
---
### 1、安装 docker  环境

```shell
1）安装 docker
	curl -sSL https://get.daocloud.io/docker | sh

2）启动服务
	systemctl start docker
	
3）查看服务状态
	systemctl status docker
	
4）设置开机自启动
	systemctl enable docker
```

官方文档：[https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)

### 2、实现 docker 镜像加速

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
```

### 3、安装 docker-compose

```shell
curl -L https://get.daocloud.io/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 4、开放指定端口

```shell
# 请将 * 修改成需要开放的端口（请自行查看 docker-compose.yml 文件中的 ports 属性）
firewall-cmd --permanent --zone=public --add-port=*/tcp

# 重载防火墙配置
firewall-cmd --reload
```

### 5、将 docker-compose.yml 文件上传至 home 目录下

### 6、根据  docker-compose.yml 文件的 volumes 属性创建目录或上传文件到指定目录（只关注 home 目录相关）

以 “**/home/tomcat/conf/server.xml:/usr/local/tomcat/conf/server.xml**”为例，根据这句脚本命令，我们需要在 home 目录下创建 tomcat 目录，然后在 tomcat 目录下创建 conf 目录，进入到 /home/tomcat/conf 目录，将 server.xml 文件上传到当前目录

### 7、创建网络实现各容器之间的正常通信

```shell
docker network create arfysica
```

### 8、执行 docker-compose.yml 文件

```shell
cd /home

docker-compose up -d
```

### 9、查看容器是否启动成功

```shell
docker ps
```

![查看容器是否启动成功](查看容器是否启动成功.png)

### 10、执行 SQL 脚本

（1）使用可视化 SQL 工具（推荐使用 navicat）

（2）连接数据库，数据库密码自行查看 docker-compose.yml 文件的 MYSQL_ROOT_PASSWORD 属性

（3）连接成功后，执行 SQL 脚本

### 11、服务应用升级

```shell
# 直接将新的应用包替换原来的，如果是后端升级再执行下面的命令，前端升级直接替换即可
docker restart tomcat
```

### 附：

如果在首次安装启动服务时，后端应用访问异常，可能原因是当前应用在启动加载过程中需要访问数据库，这时数据库并不存在，从而导致服务异常，只需要重启服务即可，执行 **docker restart tomcat** 命令


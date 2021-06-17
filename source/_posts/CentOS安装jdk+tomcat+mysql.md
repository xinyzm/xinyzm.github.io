---
title: CentOS 安装 jdk+tomcat+mysql
keywords: jdk tomcat mysql
catalog: true
comments: true
date: 2020-05-12 14:00:00
subtitle: 
header-img: snail-bg.jpg
tags:
- Linux
categories:
- 运维
---
### 安装  jdk1.8

（1）前往 [Oracle 官网](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)，选择下载 **xxx-linux-xxx.tar.gz** 压缩包到本地（以 jdk-8u152-linux-x64.tar.gz 为例）

（2）使用 **[filezilla](https://filezilla-project.org/)** 将下载的 jdk 压缩包上传至 Linux 系统下（/home/java/jdk-8u152-linux-x64.tar.gz）

（3）解压 jdk 到当前目录

```shell
tar -zxvf jdk-8u152-linux-x64.tar.gz
```

（4）配置 jdk 的环境变量

```shell
vi /etc/profile

# 将以下配置放在文件最后
export JAVA_HOME=/home/java/jdk-8u152-linux-x64
export JRE_HOME=/home/java/jdk-8u152-linux-x64/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

（5）执行上述文件，使其生效

```shell
source /etc/profile
```

（6）查看 java 版本，验证是否安装成功

```shell
java -version
```

### 安装  tomcat8.5

（1）前往 [Apache Tomcat](https://tomcat.apache.org/download-80.cgi)，选择下载 **Core - tar.gz** 压缩包到本地（以 apache-tomcat-8.5.24.tar.gz 为例），如下图

![tomcat下载](tomcat下载.png)

（2）使用 **[filezilla](https://filezilla-project.org/)** 将下载的 tomcat 压缩包上传至 Linux 系统下（/home/tomcat/apache-tomcat-8.5.24.tar.gz）

（3）解压 tomcat 到当前目录

```shell
tar -zxvf apache-tomcat-8.5.24.tar.gz
```

（4）启动和关闭 tomcat

```shell
# 启动 tomcat
./home/tomcat/apache-tomcat-8.5.24/bin/startup.sh

# 关闭 tomcat
./home/tomcat/apache-tomcat-8.5.24/bin/shutdown.sh
```

### 安装  mysql5.7

（1）前往 [MySQL官网](https://downloads.mysql.com/archives/community/)，选择如下图所示的 mysql 版本下载到本地（以 mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz 为例）

![mysql下载](mysql下载.png)

（2）使用 **[filezilla](https://filezilla-project.org/)** 将下载的 mysql 压缩包上传至 Linux 系统下（/home/mysql/mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz）

（3）解压 mysql 到 **/usr/local/mysql** 目录下

```shell
tar -zxvf mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz -C /usr/local/mysql/
```

（4）添加系统组和用户

```shell
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```

（5）修改 mysql 目录拥有者为 mysql 用户

```shell
cd /usr/local/mysql
chown -R mysql:mysql ./
```

（6）数据库初始化，并且保存最后生成的密码

```shell
cd /usr/local/mysql/bin
./mysqld --initialize --datadir=/usr/local/mysql/data --user=mysql --basedir=/usr/local/mysql
```

（7）修改 data 目录拥有者为 mysql 用户

```shell
chown -R mysql:mysql data
```

（8）复制启动文件到服务文件夹

```shell
cp support-files/mysql.server /etc/init.d/mysql
```

（9）启动 mysql 服务，并设置开机自启动

```shell
service mysql start
systemctl enable mysql
```

（10）添加 mysql 软连接

```shell
ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
```

（11）登录 mysql，密码是在第六步生成的

```shell
mysql -uroot -p
```

（12）修改 root 密码，并且开发权限使用 navicat 可以连接 mysql

```shell
# 修改root密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 赋予root用户远程连接权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

# 刷新权限
FLUSH PRIVILEGES;
```




---
title: Jenkins 和 Gitlab 配合执行项目任务
keywords: jenkins gitlab
catalog: true
comments: true
date: 2020-05-13 11:00:00
subtitle: 
header-img: snail-bg.jpg
tags:
- Jenkins
categories:
- 运维
---
### 在 Gitlab 创建 git 项目

### 使用 sshkey 命令创建一对 ssh 公私钥，用于 deploy

```git
ssh-keygen -t rsa -b 4096 -C "youremail@xxx.com" -f "sshkey_filename"
```

### 在 Jenkins 中添加 ssh 私钥凭据

（1）选择凭据 - 系统 - 全局凭据 - 添加凭据

![添加凭据-1](添加凭据-1.jpg)

（2）选择类型 **SSH Username with private key**，填写用户名和 ssh 私钥，如果生成公私钥的时候设置了 passphrase 就填写，否则不填

![填写凭证-2](填写凭证-2.jpg)

### 在 Gitlab 中配置 Deploy Keys（添加 ssh 公钥）

（1） 此步骤的前提是你拥有这个 git 项目的管理权限

（2） 选择 git 项目 - 设置 - 仓库 - Deploy Keys ，填写 ssh 公钥，保存

![配置deploy keys](配置deploy keys.jpg)

（3）激活新增的 deploy key 

![激活deploy key](激活deploy key.jpg)

### 在 Jenkins 中新建任务

### 配置任务 - 源码管理

（1） 选择 Git

（2） 设置仓库地址，仓库地址从 Gitlab 中获取

（3） 凭据选择你刚才使用 ssh 私钥添加的凭据

（4） 指定分支，建议设置成 master

（5） 如果需要部署的项目目录不是根目录，通过 Additional Behaviours -> Sparse Checkout paths（安装 Git Parameter 插件） -> Path 设置

![配置任务-源码管理](配置任务-源码管理.jpg)

### 配置任务 - 构建触发器

（1）勾选 Build when a change is pushed to Gitlab.

（2）查看 Jenkins 的 webhooks 的地址，点击高级按钮，生成 Secret token

![获取webhooks地址](获取webhooks地址.jpg)![生成Secret token](生成Secret token.jpg)

（3） 选择 git 项目 - 设置 - 集成 ，填写 Webhooks 的 URL 和 Secret Token，取消勾选 Enable SSL verification

![填写webhooks和token](填写webhooks和token.jpg)

### 配置任务 - 构建

（1） 增加构建步骤 - 执行 shell 

```shell
echo 'Hello World!'
```

### 在 Gitlab 进行测试

（1）选择 git 项目 - 设置 - 集成，在新创建的 Webhooks 下，点击 Test，选择 Push events

（2） 在 Jenkins 中查看新创建的任务是否正确执行并打印 **Hello World!**
---
title: windows搭建gitblit服务
tags:
  - git
  - windows
abbrlink: 5f2ddb0
date: 2024-06-16 17:02:53
---

# 下载

- git
- gitblit
- jar

https://www.gitblit.com/

```shell
E:\Users\evanl>java -version
java version "1.8.0_361"
Java(TM) SE Runtime Environment (build 1.8.0_361-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.361-b09, mixed mode)

E:\Users\evanl>git --version
git version 2.34.1.windows.1
```
# 配置

新建路径存放仓库 `F:/GitRepository`

```bash
git.repositoriesFolder = F:/GitRepository
server.httpPort = 8080
server.httpBindInterface = 127.0.0.1
```

运行`gitblit.cmd` 查看日志运行成功后，再进行下一步配置

## 后台服务

安装服务 `installService.cmd`
```
@REM arch = x86, amd64, or ia32
SET ARCH=amd64
SET CD=D:\tools\GIT\gitblit-1.9.3
```


卸载服务 `uninstallService.cmd`

```
@REM arch = x86, amd64, or ia32
SET ARCH=amd64
SET CD=D:\tools\GIT\gitblit-1.9.3
```

以管理员身份运行`installService.cmd`，`win + r` 输入 `services.msc`，找到 `gitlit` 服务启动

# 使用

`127.0.0.1:8080`，默认账号密码为`admin`，管理员账号用来管理仓库权限和用户团队，不参与实际仓库修改提交推送操作。

## 创建test仓库

版本库 -> 创建版本库

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240629/20240629155437.png)

## 创建用户

右上角倒三角 -> 管理 -> 用户

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240629/20240629155705.png)

### 管理员为test用户添加仓库权限

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240629/20240629160346.png)


切换到 `test` 用户就可以查看到这个仓库了。

## 克隆仓库

### 为test配置ssh


切换到`test`用户。

打开个人资料 -> SSH密钥 -> 填写密钥

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240629/20240629161021.png)


**生成和查看id_rsa.pub：**

```shell
ssh-keygen -t rsa -C "juzhango@outlook.com"
```

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240629/20240629160936.png)


测试连接情况

```shell
ssh test@127.0.0.1 -p 29418
```

### 克隆报错问题

```shell
$ git clone ssh://test@127.0.0.1:29418/test.git
Cloning into 'test'...
Unable to negotiate with 127.0.0.1 port 29418: no matching host key type found. Their offer: ssh-rsa,ssh-dss
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

报错处理：

`vim ~/.ssh/config`

```shell
Host 127.0.0.1
HostkeyAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
```

成功克隆

```shell
$ git clone ssh://test@192.168.1.11:29418/test.git
Cloning into 'test'...
remote: Counting objects: 4, done
remote: Finding sources: 100% (4/4)
remote: Getting sizes: 100% (3/3)
remote: Compressing objects: 100% (2806/2806)
remote: Total 4 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (4/4), done.
```
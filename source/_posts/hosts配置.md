---
title: hosts配置
date: 2024-06-25 08:20:01
tags: windows
---



# hosts配置入口

右键菜单 -> Windows PowerShell(管理员)

```shell
cd C:\Windows\System32\drivers\etc
notepad hosts
```


# 参考格式

```shell
52.85.151.28 　api.themoviedb.org
13.35.166.111 　api.themoviedb.org
108.157.4.76 　api.themoviedb.org
13.224.167.16 　api.themoviedb.org
99.86.178.63 　api.themoviedb.org
```

# DNS checker


[DNS Checker官网](https://dnschecker.org/)

例如查找github：
```
github.com
github.global.ssl.fastly.net
gist.github.com
help.github.com
docs.github.com
desktop.github.com
vscode-auth.github.com
education.github.com
status.github.com
```

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240625/20240625083019.png)


将检查到的结果追加到`hosts`
```shell
20.205.243.166   github.com
151.101.77.194   github.global.ssl.fastly.net
```

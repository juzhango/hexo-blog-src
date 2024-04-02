---
title: linux新建用户
tags: linux
abbrlink: 21b2690f
date: 2024-04-02 22:49:10
---
# useradd

- m: 创建用户的主目录
- u: 新账户的用户 ID
- c: 新账户的 GECOS 字段
- s: 新账户的登录 shell


```shell
# 创建用户
$ sudo useradd -m lzw -u 1001  -c "lzw" -s /bin/bash
# 设置密码
$ sudo passwd lzw
```

```shell
$ cat /etc/passwd|grep "lzw"
lzw:x:1001:1001:lzw:/home/lzw:/bin/bash
```

## 帮助信息


```shell
$ useradd -h
用法：useradd [选项] 登录
      useradd -D
      useradd -D [选项]

选项：
  -b, --base-dir BASE_DIR       新账户的主目录的基目录
  -c, --comment COMMENT         新账户的 GECOS 字段
  -d, --home-dir HOME_DIR       新账户的主目录
  -D, --defaults                显示或更改默认的 useradd 配置
  -e, --expiredate EXPIRE_DATE  新账户的过期日期
  -f, --inactive INACTIVE       新账户的密码不活动期
  -g, --gid GROUP               新账户主组的名称或 ID
  -G, --groups GROUPS   新账户的附加组列表
  -h, --help                    显示此帮助信息并推出
  -k, --skel SKEL_DIR   使用此目录作为骨架目录
  -K, --key KEY=VALUE           不使用 /etc/login.defs 中的默认值
  -l, --no-log-init     不要将此用户添加到最近登录和登录失败数据库
  -m, --create-home     创建用户的主目录
  -M, --no-create-home          不创建用户的主目录
  -N, --no-user-group   不创建同名的组
  -o, --non-unique              允许使用重复的 UID 创建用户
  -p, --password PASSWORD               加密后的新账户密码
  -r, --system                  创建一个系统账户
  -R, --root CHROOT_DIR         chroot 到的目录
  -s, --shell SHELL             新账户的登录 shell
  -u, --uid UID                 新账户的用户 ID
  -U, --user-group              创建与用户同名的组
  -Z, --selinux-user SEUSER             为 SELinux 用户映射使用指定 SEUSER

```

# usermode

参考 id 为 1000 的用户，将新建的用户追加到与 1000 用户相同的组。

```shell
# 查看第一位用户
$ cat /etc/passwd|grep "1000"
evan:x:1000:1000:evan,,,:/home/evan:/bin/bash

# 查看他加入的组
$ cat /etc/group|grep "evan"
adm:x:4:syslog,evan
cdrom:x:24:evan
sudo:x:27:evan
dip:x:30:evan
plugdev:x:46:evan
lpadmin:x:116:evan
evan:x:1000:
sambashare:x:126:evan
```

- a: 将用户追加至上边 -G 中提到的附加组中，并不从其它组中删除此用户
- G: 新的附加组列表 GROUPS

```shell
sudo usermod -a -G sambashare lzw
sudo usermod -a -G lpadmin  lzw
sudo usermod -a -G plugdev lzw
sudo usermod -a -G dip lzw
sudo usermod -a -G sudo lzw
sudo usermod -a -G cdrom lzw
sudo usermod -a -G adm lzw
```

## 帮助信息

```shell
$ usermod --help
用法：usermod [选项] 登录

选项：
  -c, --comment 注释            GECOS 字段的新值
  -d, --home HOME_DIR           用户的新主目录
  -e, --expiredate EXPIRE_DATE  设定帐户过期的日期为 EXPIRE_DATE
  -f, --inactive INACTIVE       过期 INACTIVE 天数后，设定密码为失效状态
  -g, --gid GROUP               强制使用 GROUP 为新主组
  -G, --groups GROUPS           新的附加组列表 GROUPS
  -a, --append GROUP            将用户追加至上边 -G 中提到的附加组中，
                                并不从其它组中删除此用户
  -h, --help                    显示此帮助信息并推出
  -l, --login LOGIN             新的登录名称
  -L, --lock                    锁定用户帐号
  -m, --move-home               将家目录内容移至新位置 (仅于 -d 一起使用)
  -o, --non-unique              允许使用重复的(非唯一的) UID
  -p, --password PASSWORD       将加密过的密码 (PASSWORD) 设为新密码
  -R, --root CHROOT_DIR         chroot 到的目录
  -s, --shell SHELL             该用户帐号的新登录 shell
  -u, --uid UID                 用户帐号的新 UID
  -U, --unlock                  解锁用户帐号
  -v, --add-subuids FIRST-LAST  add range of subordinate uids
  -V, --del-subuids FIRST-LAST  remvoe range of subordinate uids
  -w, --add-subgids FIRST-LAST  add range of subordinate gids
  -W, --del-subgids FIRST-LAST  remvoe range of subordinate gids
  -Z, --selinux-user  SEUSER       用户账户的新 SELinux 用户映射

```


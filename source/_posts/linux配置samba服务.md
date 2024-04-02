---
title: linux配置samba服务
tags: linux
abbrlink: 107925f6
date: 2024-04-02 23:19:05
---

# 配置

```shell
sudo vim /etc/samba/smb.conf
sudo service smbd restart
```

## 需用户登入

```bash
[lzw_work] # 映射出来的文件夹名称
   comment = /home/lzw/work # 描述
   path = /home/lzw/work # 路径
   valid users = lzw # samba用户
   browseable = yes # 路径可见的
   guest ok = no # 需要登陆才能访问
   writable = yes # 可写
```

## 配置项说明

- `writable` 和 `read only`：配置客户端访问的权限，两个配置项存在互斥关系，一般只使用 `writable` 即可。
- `guest ok` 和 `public`：表示是否可以匿名访问，即不需要登陆进行访问，在现代 Samba 配置中更倾向使用 `guest ok`。
- `browseable`：是否要在网络上可见。

## valid users和force users

在 Samba 配置文件（`smb.conf`）中，`valid users` 和 `force users` 两个选项分别用于控制对共享资源的不同访问权限：

1. **valid users**
   - 描述：`valid users` 选项定义了哪些用户允许访问共享资源。只有在这个列表中的用户才能成功登录并访问指定的共享目录。可以在这里列出单个用户账号，也可以使用组名（如果用户属于该组）或通配符（例如 `%users` 表示所有系统用户）。多个用户之间用逗号分隔。
   - 示例：`valid users = user1, user2, @group_name`
   - 当 `security = user` 时，此选项尤为关键，因为它决定了哪个系统用户能够通过Samba访问共享资源。

2. **force users**
   - 描述：`force users` 选项指定了即使不在 `valid users` 列表中，也始终会被强制登录并访问共享资源的用户。当设置 `force users` 时，即使在 `valid users` 中没有列出这些用户，它们也能无条件地访问共享。
   - 示例：`force users = admin_user`
   - 注意：通常情况下，`force users` 选项不是为了日常访问控制设计的，更多的是用于特殊场景下的调试或者紧急访问目的。

总结来说，`valid users` 是一个白名单机制，用来限定允许访问的用户范围，而 `force users` 则是一种特殊情况下的强制定向授权，它会无视常规的访问控制规则，强制指定用户获得访问权限。在大多数正常情况下，只需要配置 `valid users` 即可满足基本的访问控制需求。


# samba用户创建

`smbpasswd` 是一个在 Linux 和类 Unix 系统中管理 Samba 服务用户账户和密码的命令行工具。它主要用于创建、删除、禁用、启用 Samba 用户账户以及更改其 Samba 密码。以下是一些 `smbpasswd` 命令的常见用法及其选项说明：

- **添加用户**：
   ```bash
   smbpasswd -a [username]
   ```
   这个命令会为已经存在于系统的用户添加一个 Samba 密码，使其能够通过 Samba 访问共享资源。


>**注意：** Samba 用户基于系统用户存在，所以在使用 `smbpasswd` 添加 Samba 密码前，需要确保该用户已经在系统中创建。可以通过 `useradd` 或 `adduser` 命令先创建系统用户。


- **删除用户**：
   ```bash
   smbpasswd -x [username]
   ```
   删除指定的 Samba 用户账号，使其无法再通过 Samba 进行认证。

- **冻结用户**（禁止登录）：
   ```bash
   smbpasswd -d [username]
   ```
   将指定的 Samba 用户账户冻结，暂时阻止其通过 Samba 登录。

- **恢复用户**（解冻，允许登录）：
   ```bash
   smbpasswd -e [username]
   ```
   恢复之前被冻结的 Samba 用户账户，使其能够再次通过 Samba 登录。

- **修改密码**：
   ```bash
   smbpasswd [username]
   ```
   不带选项直接执行 `smbpasswd` 并输入用户名后，将会提示用户输入新的 Samba 密码并确认。

- **设置空密码**（通常不建议这样做，因为很多配置默认不允许空密码）：
   ```bash
   smbpasswd -n [username]
   ```
   设置用户的 Samba 密码为空。请注意，这通常需要在 Samba 配置文件（如 `/etc/samba/smb.conf`）中允许空密码才能生效。

这些操作通常需要具有管理员权限（root 或者通过 sudo 执行）。

如果您想了解更详细的帮助信息或查看所有可用选项，请运行：
```bash
smbpasswd --help
```

根据不同的 Samba 版本和系统环境，一些选项可能有所不同或有额外的选项，请参照实际系统中的 `smbpasswd` 命令的手册页或官方文档。



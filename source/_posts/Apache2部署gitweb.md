---
title: Apache2部署gitweb
tags:
  - git
abbrlink: c9fc1918
date: 2024-06-02 02:39:02
---


# 创建git仓库

```shell
sudo apt install git
```

```shell
cd ~
mkdir git-repo
cd git-repo
git init --bare test.git
```

# 安装apache2

```shell
sudo apt install apache2
sudo apache2ctl start
```

如果有这条信息，可在配置文件中追加`ServerName = 127.0.1.1`
```
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1. Set the 'ServerName' directive globally to suppress this message
```

apache2 的配置目录`/etc/apache2`

>**说明：通过`apt install`安装，配置目录才是这种结构**
>
```shell
.
├── apache2.conf
├── conf-available
├── conf.d
├── conf-enabled
├── envvars
├── magic
├── mods-available
├── mods-enabled
├── ports.conf
├── sites-available
└── sites-enabled
```

主要通过`apache.conf`文件去包含子目录下的配置

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240602/20240602025346.png)

## cgi模块

gitweb是用cgi脚本运行的，所以要启用cgi配置。

apache2.conf 有这样一条信息

```shell
# Include module configuration:
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
```

模块都放在了`mods-available`目录下，可通过软链接到`mods-enabled`目录下，

```shell
lrwxrwxrwx 1 root root 27  6月  1 18:32 cgid.conf -> ../mods-available/cgid.conf
lrwxrwxrwx 1 root root 27  6月  1 18:32 cgid.load -> ../mods-available/cgid.load
lrwxrwxrwx 1 root root 26  6月  1 17:46 cgi.load -> ../mods-available/cgi.load
```

`conf-enabled/serve-cgi-bin.conf` 配置如下：

```xml
<IfModule mod_alias.c>
        <IfModule mod_cgi.c>
                Define ENABLE_USR_LIB_CGI_BIN
        </IfModule>

        <IfModule mod_cgid.c>
                Define ENABLE_USR_LIB_CGI_BIN
        </IfModule>

        <IfDefine ENABLE_USR_LIB_CGI_BIN>
                ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
                <Directory "/usr/lib/cgi-bin">
                        AllowOverride None
                        Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
                        Require all granted
                </Directory>
        </IfDefine>
</IfModule>
```

写一个脚本检查 `cgi` 模块是否正常

`sudo vim /usr/lib/cgi-bin/hello.cgi`

```perl
#!/usr/bin/perl
print "Content-type: text/html\n\n";
print "<html><head><title>CGI Test</title></head>";
print "<body><h1>Hello, world!</h1></body></html>";
```

访问`http://localhost/cgi-bin/hello.cgi`

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240602/20240602030620.png)

# gitweb

```shell
sudo apt install gitweb
```

gitweb的页面文件存放在`/usr/share/gitweb`

```shell
.
├── gitweb.cgi
├── index.cgi -> gitweb.cgi
└── static
    ├── git-favicon.png
    ├── git-logo.png
    ├── gitweb.css
    └── gitweb.js

1 directory, 6 files
```

**配置gitweb**

`/etc/gitweb.conf`
```shell
# path to git projects (<project>.git)
$projectroot = "/home/[user]/git-repo";
```

`$projectroot`就是存放git仓库的路径。


**配置apache2**

创建apache的gitweb配置文件

`sudo vim /etc/apache2/conf.d/gitweb`

---

```xml
Alias /gitweb /usr/share/gitweb

<Directory /usr/share/gitweb>
  AllowOverride None
  Options +ExecCGI +SymLinksIfOwnerMatch
  Require all granted
  Options +FollowSymLinks +ExecCGI
  AddHandler cgi-script .cgi
</Directory>
```

- Alias：配置URL别名，`/gitweb`指向`/usr/share/gitweb`资源
- <Directory /usr/share/gitweb>：配置目录块
- AllowOverride None：配置不运行被`.htaccess`覆盖
- Options：访问项，运行执行CGI脚本，仅相同用户ID文件允许跟随符号链接
- AddHandler：将`.cgi`扩展名的文件作为CGI脚本来处理
---

将其软链接到`conf-enabled`目录下

```shell
lrwxrwxrwx 1 root root 16  6月  2 01:28 gitweb.conf -> ../conf.d/gitweb
```

重启`apache2`，访问`http://localhost/gitweb/gitweb.cgi`

![image.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240602/20240602032712.png)


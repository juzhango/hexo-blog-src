---
title: hexo博客记录1——搭建
tags: hexo
abbrlink: df608854
date: 2024-03-26 22:33:36
---

# 必要软件

```shell
wget https://nodejs.org/download/release/v18.19.1/node-v18.19.1-linux-x64.tar.xz
export PATH=$PATH:/home/hexo/tools/node-v18.19.1-linux-x64/bin
npm config set registry https://registry.npm.taobao.org/
npm set strict-ssl false
```
# 初始化

[hexo.io](https://hexo.io/zh-cn/)

```shell
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```

# 部署


github创建仓库命名 `juzhango.github.io`

```yml
deploy:
  type: 'git'
  repo: git@github.com:juzhango/juzhango.github.io.git
  branch: main
```


```shell
cd blog
npm install hexo-deployer-git --save
hexo clean
hexo d
```

# 其他配置

## 1 主题

[butterfly](https://butterfly.js.org/)

安装butterfly主题需要安装 pug和stylus的渲染器

```shell
cd blog
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

## 2 文章链接格式

防止永久链接为中文。

[hexo-abbrlink](https://github.com/rozbo/hexo-abbrlink)

```shell
npm install hexo-abbrlink --save
```

`_config.yml`

```yaml
url: http://juzhango.cn
permalink: :abbrlink/
abbrlink:
  alg: crc32 #support crc16(default) and crc32
  rep: hex  #support dec(default) and hex
```

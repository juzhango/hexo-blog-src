---
title: hexo博客记录2——部署nginx
tags: hexo
abbrlink: 3da8bd06
date: 2024-03-26 22:36:53
---


使用了一段时间 halo 来部署博客，但总有种顿挫感，最后还是回到了用 hexo 来部署博客。

之前部署 hexo 的网页都是使用 github.io 的方式，这次决定使用 nginx 来部署网页。

流程就是：`hexo d` 将博客页面文件 push 到 github.io 仓库，之后 VPS 从 github 拉取网页文件，提供给 nginx 部署，当 github 仓库有更新时，可以写一个定时脚本去自动 pull 仓库文件。

---
# 操作

新建一个站点，站点的目录我配置为 `/www/wwwroot/blog`，将 github.io 仓库的页面文件放入此目录下即可。

![1.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240324/1.png)


配置 SSL：将下载的证书文本填写上去，宝塔面板会自动为我配置好 nginx，配置好后博客的地址就可以通过 `IP:433`或域名(`juzhango.cn`)直接访问了。

![2.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240324/2.png)

## 脚本

```shell
#!/bin/bash
dir_path="/home/ubuntu/blog"

# 使用if条件判断该路径是否存在
if [ ! -d "$dir_path" ]; then
    mkdir -p "$dir_path"
    cd $dir_path
    git clone git@github.com:juzhango/juzhango.github.io.git
fi

cd /home/ubuntu/blog/juzhango.github.io
git pull origin main
# sudo rm -r /www/wwwroot/blog/*
sudo cp -rf /home/ubuntu/blog/juzhango.github.io/* /www/wwwroot/blog/
```



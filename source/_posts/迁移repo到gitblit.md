---
title: 迁移repo到gitblit
tags:
  - git
abbrlink: 3989ab0
date: 2024-07-01 00:29:03
---



```bash
#!/bin/bash

# 定义基础目录和远程仓库前缀
REPO_BASE_DIR="/home/evan/linux"
REMOTE_URL_PREFIX="/home/evan/work/list"

# 切换到基础目录
cd "$REPO_BASE_DIR" || exit

# 使用 repo forall 命令获取所有仓库的路径
mapfile -t paths < <(.repo/repo/repo forall -c 'pwd')

# 遍历每个仓库路径
for path in "${paths[@]}"; do
    echo "======================================================="
    echo "进入目录: $path"
    cd "$path" || exit

    echo "当前目录: $(pwd)"

    # 获取远程仓库URL并替换前缀
    REMOTE_URL=$(git remote -v | grep push | awk '{print $2}' | sed "s/http:\/\/192.168.1.71/$REMOTE_URL_PREFIX/")
    REMOTE_URL="${REMOTE_URL}.git"

    # 初始化裸仓库
    if [ ! -d "${REMOTE_URL}" ]; then
        mkdir -p "${REMOTE_URL}"
        cd "${REMOTE_URL}" || exit
        git init --bare
        cd "$path" || exit
    fi

    # 清理并重新初始化本地仓库
    rm -rf .git
    git init
    git add .

    # 提交更改
    git commit -m "🎉 初始化: 新分支"
    git remote add origin "$REMOTE_URL"

    # 推送到远程仓库
    git push -u origin master

    echo ""
done
```

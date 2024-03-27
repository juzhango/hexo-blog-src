---
title: hexo博客记录3——Appveyor持续集成
abbrlink: da95da28
date: 2024-03-26 22:48:32
tags: hexo
---

>[持续集成服务](https://en.wikipedia.org/wiki/Continuous_integration)（全名`Continuous Integration`，简称 CI），它其实是一个云端自动化构建和测试服务，**它绑定 Github 上面的项目，只要有新的代码，就会自动抓取。然后，提供一个运行环境，执行测试，完成构建，还能部署到服务器。**


# 仓建仓库

将我的`blog`文件夹初始化为`git`仓库，添加`.gitignore`文件。

```
.deploy*/
node_modules/
public/
db.json
package-lock.json
*.log
```

关联`github`

```shell
git branch -M main
git remote add origin git@github.com:juzhango/hexo-blog-src.git
git push -u origin main
```

# 建立 CI

[appveyor](https://ci.appveyor.com/)

`project` 添加博客源文件仓库

![1.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240326/1.png)

## 添加Access Token

`github` 获取`Access Token`

![6.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240326/6.png)

`appveyor` 对`Access Token`加密处理

![2.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240326/2.png)


# 配置CI工作流

环境配置

- STATIC_SITE_REPO：博客站点Github仓库地址；
- TARGET_BRANCH：博客站点仓库的branch（默认是main）；
- GIT_USER_EMAIL：GitHub账号的信息；
- GIT_USER_NAME：GitHub账号的信息；

![3.png](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240326/3.png)

在博客源文件根目录新建`appveyor.yml`文件

```yml
clone_depth: 5

environment:
  access_token:
    secure: A/jGNBats9WD7w/e4ehDBAZJDMaBW59qErPx07/rYLdtWyAA2gtz+szoVrTmjVLW

install:
  - node --version
  - npm --version
  - npm install hexo-cli -g
  - npm install
  - npm install hexo-abbrlink --save
  - npm install hexo-renderer-pug hexo-renderer-stylus --save

build_script:
  - hexo clean
  - hexo g

artifacts:
  - path: public

on_success:
  - git config --global credential.helper store
  - ps: Add-Content "$env:USERPROFILE\.git-credentials" "https://$($env:access_token):x-oauth-basic@github.com`n"
  - git config --global user.email "%GIT_USER_EMAIL%"
  - git config --global user.name "%GIT_USER_NAME%"
  - git clone --depth 5 -q --branch=%TARGET_BRANCH% %STATIC_SITE_REPO% %TEMP%\static-site
  - cd %TEMP%\static-site
  - del * /f /q
  - for /d %%p IN (*) do rmdir "%%p" /s /q
  - SETLOCAL EnableDelayedExpansion & robocopy "%APPVEYOR_BUILD_FOLDER%\public" "%TEMP%\static-site" /e & IF !ERRORLEVEL! EQU 1 (exit 0) ELSE (IF !ERRORLEVEL! EQU 3 (exit 0) ELSE (exit 1))
  - git add -A
  - if "%APPVEYOR_REPO_BRANCH%"=="main" if not defined APPVEYOR_PULL_REQUEST_NUMBER (git diff --quiet --exit-code --cached || git commit -m "Update Static Site" && git push origin %TARGET_BRANCH% && appveyor AddMessage "Static Site Updated")
```



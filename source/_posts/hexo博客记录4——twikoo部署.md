---
title: hexo博客记录4——twikoo部署
tags: hexo
abbrlink: 99cce692
date: 2024-04-05 16:03:45
---

[twikoo](https://twikoo.js.org/)，一个简洁、安全、免费的静态网站评论系统。

# 后端部署

本地 nas 使用 docker 部署。

```yml
version: '3'
services:
  twikoo:
    image: imaegoo/twikoo
    container_name: twikoo
    restart: unless-stopped
    ports:
      - 8080:8080
    environment:
      TWIKOO_THROTTLE: 1000
    volumes:
      - ./data:/app/data
```

通过 frp 服务，将本地端口映射到云服务器。

## 域名解析 & 反向代理

twikoo.juzhango.cn --> 服务器ip --> 反向代理到frps

1. 添加一条DNS解析
主机记录：`twikoo`，主机类型：`A`，记录值：`云服务器ip`。
2. 宝塔面板配置反向代理
新建站点`twikoo.juzhango.cn`，配置反向代理

![](https://juzhango-1257120269.cos.ap-shanghai.myqcloud.com/240405/1.png)

3. 添加SSL证书，与配置博客域名证书相同。


# 前端部署


在 [hexo](https://hexo.io/zh-tw/) 的 [butterfly](https://butterfly.js.org/) 部署，前端部署请看完[Butterfly 安裝文檔(四) 主題配置-2 #评论](https://butterfly.js.org/posts/ceeb73f/#%E8%A9%95%8%AB%96)

修改主题配置文件`_config.landscape.yml`

```yml
comments:
  use: twikoo 
  text: true
  lazyload: false
  count: true
  card_post_count: false
```

| 參數              | 解釋                                                                             |
| --------------- | ------------------------------------------------------------------------------ |
| use             | 使用的評論（請注意，最多支持兩個，如果不需要請留空）<br>_注意：雙評論不能是 Disqus 和 Disqusjs 一起，由於其共用同一個 ID，會出錯_ |
| text            | 是否顯示評論服務商的名字                                                                   |
| lazyload        | 是否為評論開啟lazyload，開啟後，只有滾動到評論位置時才會加載評論所需要的資源（開啟 lazyload 後，評論數將不顯示）              |
| count           | 是否在文章頂部顯示評論數<br>livere、Giscus 和 utterances 不支持評論數顯示                            |
| card_post_count | 是否在首頁文章卡片顯示評論數<br>gitalk、livere 、Giscus 和 utterances 不支持評論數顯示                  |


配置 twikoo 后端 https 网址

```yml
# Twikoo
# https://github.com/imaegoo/twikoo
twikoo:
  envId: https://twikoo.juzhango.cn/
  region:
  visitor: false
  option:
```

| 参数      | 解释                              |
| ------- | ------------------------------- |
| envId   | 环境ID                            |
| region  | 环境地域，默认为 ap-shanghai，如果不是，需传此参数 |
| visitor | 是否显示文章阅读数                       |
| option  | 可选配置                            |
>開啟 visitor 後，文章頁的訪問人數將改為 Twikoo 提供，而不是**不蒜子**

修改记录
```diff
diff --git a/_config.butterfly.yml b/_config.butterfly.yml
index aa969d8..00b8331 100644
--- a/_config.butterfly.yml
+++ b/_config.butterfly.yml
@@ -409,7 +409,7 @@ addtoany:
 comments:
   # Up to two comments system, the first will be shown as default
   # Choose: Disqus/Disqusjs/Livere/Gitalk/Valine/Waline/Utterances/Facebook Comments/Twikoo/Giscus/Remark42/Artalk
-  use: # Valine,Disqus
+  use: twikoo # Valine,Disqus
   text: true # Display the comment name next to the button
   # lazyload: The comment system will be load when comment element enters the browser's viewport.
   # If you set it to true, the comment count will be invalid
@@ -487,7 +487,7 @@ facebook_comments:
 # Twikoo
 # https://github.com/imaegoo/twikoo
 twikoo:
-  envId:
+  envId: https://twikoo.juzhango.cn/
   region:
   visitor: false
   option:
```

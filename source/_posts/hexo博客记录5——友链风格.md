---
title: hexo博客记录5——友链风格
tags: hexo
abbrlink: ad8ef30a
date: 2024-04-05 18:24:00
---

1. 新建友链页面。已开的可以跳过，从第2步开始。<br>参照参考教程中的[Butterfly友链界面配置教程](https://butterfly.js.org/posts/dc584b87/#%E5%8F%8B%E6%83%85%E9%8F%88%E6%8E%A5)先配置好默认友链页面。
2. 在Hexo博客根目录`[Blogroot]`下打开终端，输入`hexo new page link`。
```bash
	hexo new page link
```
3. 打开`[Blogroot]\source\link\index.md`,添加一行`type: 'link'`:
```markdown
---  
title: link  
date: 2020-12-01 22:19:45  
type: 'link'  
---
```
4. 新建文件`[Blogroot]\source\_data\link.yml`,没有`_data`文件夹的话也请自己新建。以下是默认友链格式示例(~~自己写的教程，夹带点私货不过分吧，嘻嘻~~)。打开`[Blogroot]\source\_data\link.yml`，输入：
```yml
- class_name:
  class_desc:
  link_list:
    - name: # 昵称
      link: # 博客地址
      avatar: # 头像
      descr: # 描述
```
5. 取消`[Blogroot]\_config.butterfly.yml`中`menu`配置项内`link`页面的注释。
```yml
menu:  
  # Home: / || fas fa-home  
  # Archives: /archives/ || fas fa-archive  
  # Tags: /tags/ || fas fa-tags  
  # Categories: /categories/ || fas fa-folder-open  
  # List||fas fa-list:  
  #   - Music || /music/ || fas fa-music  
  #   - Movie || /movies/ || fas fa-video  
  Link: /link/ || fas fa-link  
  # About: /about/ || fas fa-heart
```
6. 修改`[Blogroot]\themes\butterfly\layout\includes\page\flink.pug`,此处添加判断机制，使得可以通过修改配置文件来切换友链风格。同时为了方便管理，把各个友链样式放到新建的文件目录`[Blogroot]\themes\butterfly\layout\includes\page\flink_style`下。


{% tabs s %}
<!-- tab flink.pug -->
修改`[Blogroot]\themes\butterfly\layout\includes\page\flink.pug`，全部内容替换为：
```plaintext
case theme.flink_style  
  when 'volantis'  
    include ./flink_style/volantis.pug  
  when 'flexcard'  
    include ./flink_style/flexcard.pug  
  when 'byer'  
    include ./flink_style/byer.pug  
  default  
    include ./flink_style/butterfly.pug
```
<!-- endtab -->

<!-- tab butterfly.pug -->

新建`[Blogroot]\themes\butterfly\layout\includes\page\flink_style\butterfly.pug`
```plaintext
#article-container  
  if top_img === false  
    h1.page-title= page.title  
  .flink  
    if site.data.link  
      each i in site.data.link  
        if i.class_name  
          h2!= i.class_name  
        if i.class_desc  
          .flink-desc!=i.class_desc  
        .flink-list  
          each item in i.link_list  
            .flink-list-item  
              a(href=url_for(item.link)  title=item.name target="_blank")  
                .flink-item-icon  
                  img.no-lightbox(src=url_for(item.avatar) onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.flink) + `'` alt=item.name )  
                .flink-item-name= item.name  
                .flink-item-desc(title=item.descr)= item.descr  
  != page.content
```
<!-- endtab -->

<!-- tab volantis.pug -->
新建`[Blogroot]\themes\butterfly\layout\includes\page\flink_style\volantis.pug`
```plaintext
#article-container  
  if top_img === false  
    h1.page-title= page.title  
  .flink  
    if site.data.link  
      each i in site.data.link  
        if i.class_name  
          h2!= i.class_name  
        if i.class_desc  
          .flink-desc!=i.class_desc  
        .site-card-group  
          each item in i.link_list  
            a.site-card(target='_blank' rel='noopener' href=url_for(item.link))  
              .img  
                - var siteshot = item.siteshot ? url_for(item.siteshot) : 'https://image.thum.io/get/width/400/crop/800/allowJPG/wait/20/noanimate/' + item.link  
                img.no-lightbox(src=siteshot onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.post_page) + `'` alt='' )  
              .info  
                img.no-lightbox(src=url_for(item.avatar) onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.flink) + `'` alt='' )  
                span.title= item.name  
                span.desc(title=item.descr)= item.descr  
  != page.content
```
<!-- endtab -->

<!-- tab flexcard.pug -->
新建`[Blogroot]\themes\butterfly\layout\includes\page\flink_style\flexcard.pug`
```plaintext
#article-container  
  if top_img === false  
    h1.page-title= page.title  
  .flink  
    if site.data.link  
      each i in site.data.link  
        if i.class_name  
          h2!= i.class_name  
        if i.class_desc  
          .flink-desc!=i.class_desc  
        .flink-list  
          each item in i.link_list  
            a.flink-list-card(href=url_for(item.link) target='_blank' data-title=item.descr)  
              .wrapper.cover  
                - var siteshot = item.siteshot ? url_for(item.siteshot) : 'https://image.thum.io/get/width/400/crop/800/allowJPG/wait/20/noanimate/' + item.link  
                img.no-lightbox.cover.fadeIn(src=siteshot onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.post_page) + `'` alt='' )      
              .info  
                img.no-lightbox(src=url_for(item.avatar) onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.flink) + `'` alt='' )  
                span.flink-sitename= item.name  
  != page.content
```
<!-- endtab -->
<!-- tab byer.pug -->
新建`[Blogroot]\themes\butterfly\layout\includes\page\flink_style\byer.pug`
```plaintext
#article-container  
  .flink  
    if site.data.link  
      each i in site.data.link  
        if i.class_name  
          h2!= i.class_name  
        if i.class_desc  
          .flink-desc!=i.class_desc  
        .flink-list  
          each item in i.link_list  
            .flink-list-item  
              a(href=url_for(item.link)  title=item.name target="_blank")  
                .flink-item-bar  
                  sapn.flink-item-bar-yellow   
                  sapn.flink-item-bar-green   
                  sapn.flink-item-bar-red  
                  sapn.flink-item-bar-x +  
                .flink-item-content  
                  .flink-item-text  
                    .flink-item-name= item.name   
                    .flink-item-desc(title=item.descr)= item.descr  
                  .flink-item-icon  
                    img.no-lightbox(src=url_for(item.avatar) onerror=`this.onerror=null;this.src='` + url_for(theme.error_img.flink) + `'` alt=item.name )  
  != page.content
```
<!-- endtab -->
{% endtabs %}


7. 修改`[Blogroot]\themes\butterfly\source\css\_page\flink.styl`,同理，将样式文件也放到新建的`[Blogroot]\themes\butterfly\source\css\_flink_style`目录下方便管理。

{% tabs s %}
<!-- tab flink.styl -->
修改`[Blogroot]\themes\butterfly\source\css\_page\flink.styl`
```stylus
if hexo-config('flink_style') == 'butterfly'  
  @import './_flink_style/butterfly'  
else if hexo-config('flink_style') == 'volantis'  
  @import './_flink_style/volantis'  
else if hexo-config('flink_style') == 'flexcard'  
  @import './_flink_style/flexcard'  
else if hexo-config('flink_style') == 'byer'  
  @import './_flink_style/byer'
```
<!-- endtab -->
<!-- tab butterfly.styl -->
新建`[Blogroot]\themes\butterfly\source\css\_flink_style\butterfly.styl`
```stylus
.flink#article-container  
  margin-bottom: 20px  
.flink-desc  
  margin: .2rem 0 .5rem  
  
.flink-list  
  overflow: auto  
  padding: 10px 10px 0  
  text-align: center  
  
  & > .flink-list-item  
    position: relative  
    float: left  
    overflow: hidden  
    margin: 15px 7px  
    width: calc(100% / 3 - 15px)  
    height: 90px  
    border-radius: 8px  
    line-height: 17px  
    -webkit-transform: translateZ(0)  
  
    +maxWidth1024()  
      width: calc(50% - 15px) !important  
  
    +maxWidth600()  
      width: calc(100% - 15px) !important  
  
    &:hover  
      .flink-item-icon  
        margin-left: -10px  
        width: 0  
  
    &:before  
      position: absolute  
      top: 0  
      right: 0  
      bottom: 0  
      left: 0  
      z-index: -1  
      background: var(--text-bg-hover)  
      content: ''  
      transition: transform .3s ease-out  
      transform: scale(0)  
  
    &:hover:before,  
    &:focus:before,  
    &:active:before  
      transform: scale(1)  
  
    a  
      color: var(--font-color)  
      text-decoration: none  
  
      .flink-item-icon  
        float: left  
        overflow: hidden  
        margin: 15px 10px  
        width: 60px  
        height: 60px  
        border-radius: 35px  
        transition: width .3s ease-out  
  
        img  
          width: 100%  
          height: 100%  
          transition: filter 375ms ease-in .2s, transform .3s  
          object-fit: cover  
  
      .img-alt  
        display: none  
  
.flink-item-name  
  @extend .limit-one-line  
  padding: 16px 10px 0 0  
  height: 40px  
  font-weight: bold  
  font-size: 1.43em  
  
.flink-item-desc  
  @extend .limit-one-line  
  padding: 16px 10px 16px 0  
  height: 50px  
  font-size: .93em  
.flink-name  
  margin-bottom: 5px  
  font-weight: bold  
  font-size: 1.5em
```
<!-- endtab -->
<!-- tab volantis.styl -->

新建`[Blogroot]\themes\butterfly\source\css\_flink_style\volantis.styl`
```stylus
trans($time = 0.28s)  
  transition: all $time ease  
  -moz-transition: all $time ease  
  -webkit-transition: all $time ease  
  -o-transition: all $time ease  
  
  
.site-card-group  
  display: flex  
  flex-wrap: wrap  
  justify-content: flex-start  
  margin: -0.5 * 16px  
  align-items: stretch  
.site-card  
  margin: 16px * 0.5  
  width: "calc(100% / 4 - %s)" % 16px  
  @media screen and (min-width: 2048px)  
      width: "calc(100% / 5 - %s)" % 16px  
  @media screen and (max-width: 768px)  
      width: "calc(100% / 3 - %s)" % 16px  
  @media screen and (max-width: 500px)  
      width: "calc(100% / 2 - %s)" % 16px  
  display: block  
  line-height: 1.4  
  height 100%  
  .img  
    width: 100%  
    height 120px  
    @media screen and (max-width: 500px)  
      height 100px  
    overflow: hidden  
    border-radius: 12px * 0.5  
    box-shadow: 0 1px 2px 0px rgba(0, 0, 0, 0.2)  
    background: #f6f6f6  
    img  
      width: 100%  
      height 100%  
      pointer-events:none;  
      // trans(.75s)  
      transition: transform 2s ease  
      object-fit: cover  
  
  .info  
    margin-top: 16px * 0.5  
    img  
      width: 32px  
      height: 32px  
      pointer-events:none;  
      border-radius: 16px  
      float: left  
      margin-right: 8px  
      margin-top: 2px  
    span  
      display: block  
    .title  
      font-weight: 600  
      font-size: var(--global-font-size)  
      color: #444  
      display: -webkit-box  
      -webkit-box-orient: vertical  
      overflow: hidden  
      -webkit-line-clamp: 1  
      trans()  
    .desc  
      font-size: var(--global-font-size)  
      word-wrap: break-word;  
      line-height: 1.2  
      color: #888  
      display: -webkit-box  
      -webkit-box-orient: vertical  
      overflow: hidden  
      -webkit-line-clamp: 2  
  .img  
    trans()  
  &:hover  
    .img  
      box-shadow: 0 4px 8px 0px rgba(0, 0, 0, 0.1), 0 2px 4px 0px rgba(0, 0, 0, 0.1), 0 4px 8px 0px rgba(0, 0, 0, 0.1), 0 8px 16px 0px rgba(0, 0, 0, 0.1)  
    .info .title  
      color: #ff5722
```
<!-- endtab -->
<!-- tab flexcard.styl -->
新建`[Blogroot]\themes\butterfly\source\css\_flink_style\flexcard.styl`
```stylus
#article-container img  
  margin 0 auto!important  
.flink-list  
  overflow auto  
  & > a  
    width calc(25% - 15px)  
    height 130px  
    position relative  
    display block  
    margin 15px 7px  
    float left  
    overflow hidden  
    border-radius 10px  
    transition all .3s ease 0s, transform .6s cubic-bezier(.6, .2, .1, 1) 0s  
    box-shadow 0 14px 38px rgba(0, 0, 0, .08), 0 3px 8px rgba(0, 0, 0, .06)  
    &:hover  
      .info  
        transform translateY(-100%)  
      .wrapper  
        img  
          transform scale(1.2)  
      &::before  
        position: fixed  
        width:inherit  
        margin:auto  
        left:0  
        right:0  
        top:10%  
        border-radius: 10px  
        text-align: center  
        z-index: 100  
        content: attr(data-title)  
        font-size: 20px  
        color: #fff  
        padding: 10px  
        background-color: rgba($theme-color,0.8)  
  
    .cover  
      width 100%  
      transition transform .5s ease-out  
    .wrapper  
      position relative  
      .fadeIn  
        animation coverIn .8s ease-out forwards  
      img  
        height 130px  
        pointer-events none  
    .info  
      display flex  
      flex-direction column  
      justify-content center  
      align-items center  
      width 100%  
      height 100%  
      overflow hidden  
      border-radius 3px  
      background-color hsla(0, 0%, 100%, .7)  
      transition transform .5s cubic-bezier(.6, .2, .1, 1) 0s  
      img  
        position relative  
        top 22px  
        width 66px  
        height 66px  
        border-radius 50%  
        box-shadow 0 0 10px rgba(0, 0, 0, .3)  
        z-index 1  
        text-align center  
        pointer-events none  
      span  
        padding 20px 10% 60px 10%  
        font-size 16px  
        width 100%  
        text-align center  
        box-shadow 0 0 10px rgba(0, 0, 0, .3)  
        background-color hsla(0, 0%, 100%, .7)  
        color var(--font-color)  
        white-space nowrap  
        overflow hidden  
        text-overflow ellipsis  
.flink-list>a .info,  
.flink-list>a .wrapper .cover  
  position absolute  
  top 0  
  left 0  
  
@media screen and (max-width:1024px)  
  .flink-list  
    & > a  
      width calc(33.33333% - 15px)  
  
@media screen and (max-width:600px)  
  .flink-list  
    & > a  
      width calc(50% - 15px)  
  
[data-theme=dark]  
  .flink-list a .info,  
  .flink-list a .info span  
    background-color rgba(0, 0, 0, .6)  
  .flink-list  
    & > a  
      &:hover  
        &:before  
          background-color: rgba(#121212,0.8);  
.justified-gallery > div > img,  
.justified-gallery > figure > img,  
.justified-gallery > a > a > img,  
.justified-gallery > div > a > img,  
.justified-gallery > figure > a > img,  
.justified-gallery > a > svg,  
.justified-gallery > div > svg,  
.justified-gallery > figure > svg,  
.justified-gallery > a > a > svg,  
.justified-gallery > div > a > svg,  
.justified-gallery > figure > a > svg  
  position static!important
```
<!-- endtab -->
<!-- tab byer.styl -->
新建`[Blogroot]\themes\butterfly\source\css\_flink_style\byer.styl`
```stylus
#article-container  
  .flink  
    margin-bottom: 20px  
  
    .flink-list  
      overflow: auto  
      padding: 10px 10px 0  
      text-align: center  
  
      & > .flink-list-item  
        position: relative  
        float: left  
        overflow: hidden  
        margin: 15px 7px  
        width: calc(100% / 3 - 15px)  
        height: 120px  
        border-radius: 2px  
        line-height: 17px  
        -webkit-transform: translateZ(0)  
        border: 1px solid  
        box-shadow: 3px 3px 1px 1px #fee34c;  
  
        +maxWidth1024()  
          width: calc(50% - 15px) !important  
  
        +maxWidth600()  
          width: calc(100% - 15px) !important  
  
        a  
          color: var(--font-color)  
          text-decoration: none  
          .flink-item-bar  
            height: 15px  
            border-width: 0 0 1px 0  
            border-style: none none solid none  
            background: #fde135  
            display: flex;  
            align-items: center;  
            flex-direction: row;  
            flex-wrap: nowrap;  
            padding: 0 3px 0 3px  
            sapn  
              width: 10px;  
              height: 10px;  
              margin: 0 1px 0 1px  
              border-radius: 50%;  
              display: block;  
              border: 1px solid;  
              display: flex;  
              align-items: center;  
              justify-content: flex-start;  
              &.flink-item-bar-yellow  
                background: #fde135  
              &.flink-item-bar-green  
                background: #249a33  
              &.flink-item-bar-red  
                background: #f13b06  
              &.flink-item-bar-x  
                background: transparent  
                border: 0px  
                margin-left: auto  
                transform: rotate(45deg);  
                font-size: 23px;  
                padding: 0px 0px 6px 0px;  
          .flink-item-content  
            display: flex;  
            height: 105px  
            flex-direction: row;  
            align-items: center;  
            justify-content: space-between;  
            padding: 0 5px 0 5px;  
            .flink-item-text  
              width: 60%;  
              display: flex;  
              flex-direction: column;  
              align-items: center;  
              .flink-item-name  
                @extend .limit-one-line  
                max-width: 100%;  
                padding: 0px 5px 0px 5px;  
                margin: 0px 0 6px 0;  
                height: 50%;  
                font-weight: bold;  
                font-size: 1.43em;  
                border-width: 0 0 7px 0;  
                border-style: solid;  
                border-color: #fbf19f;  
              .flink-item-desc  
                @extend .limit-one-line  
                max-width: 100%;  
                height: 50%;  
                padding: 5px 5px 5px 5px;  
                font-size: 0.93em;  
                position: relative  
                &:before  
                  content: "";  
                  background: transparent;  
                  display: block;  
                  height: calc(100% - 4px);  
                  width: calc(100% - 4px);  
                  position: absolute;  
                  left: 0;  
                  top: 0;  
                  border-radius: 2px;  
                  border: 1px solid;  
                  clip-path: polygon(0 0, 100% 0, 100% 100%, 95% 100%, 95% 50%, 90% 50%, 90% 100%, 0 100%);  
  
  
            .flink-item-icon  
              overflow: hidden;  
              margin: 0px 5px;  
              width: 70px;  
              height: 70px;  
              border: 1px solid;  
              border-radius: 2px;  
              transition: width .3s ease-out  
              box-shadow: 2px 2px 1px 1px #fee34c;  
              img  
                width: 50px;  
                height: 50px;  
                margin: 9px 9px;  
                transition: filter 375ms ease-in .2s, transform .3s  
                object-fit: cover  
  
              .img-alt  
                display: none
```
<!-- endtab -->
{% endtabs %}



8. 因为`Volantis`的`site-card`比`Butterfly`的`flink-card`多出了一个站点缩略图，所以需要再额外添加一条配置项。修改`[Blogroot]\source\_data\link.yml`,添加一条名为`siteshot`的配置项。
```yml
- class_name:
  class_desc:
  link_list:
    - name:
      link:
      avatar:
      descr:
      siteshot: # siteshot就是站点缩略图的链接。
```
9. 在`[Blogroot]\_config.butterfly.yml`中添加配置项：
```yml
# 友链样式，butterfly为默认样式，volantis为站点卡片样式,flexcard为弹性卡片样式,byer为粉丝彩蛋！  
flink_style: volantis # butterfly | volantis | flexcard | byer
```
10. 站点卡片添加了懒加载和图片失效替换。对应配置项为`[Blogroot]\_config.butterfly.yml`中的：
```yml
# Replace Broken Images (替換無法顯示的圖片)  
error_img:  
  flink:   # 头像失效替换图  
  post_page:  # 站点缩略图
```
11. 可能遇到的bug：使用`flexcard`样式时，因为全站字体大小配置与本站不一致的关系，可能导致友链卡片的头像位置偏移较大。请读者按照`flexcard.styl`里的注释内容自己微调。




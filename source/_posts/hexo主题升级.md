---
title: hexo主题升级
date: 2018-03-01 12:25:10
tags: 
  - Hexo
---



偶然看到最新的next主题，甚佳，决定对自己的博客主题进行升级，简要记录升级过程。

<!--more-->

1. 下载最新next 6.0.4

2. 备份_config.yml，同时对配置进行修改，相关图片位于next/source/images

3. 页面增加3D渲染，next默认提供两种渲染效果，theme-next-three和canvas_nest

   ```
   cd themes/next
   git clone https://github.com/theme-next/theme-next-three source/lib/three
   git clone https://github.com/theme-next/theme-next-canvas-nest source/lib/canvas-nest
   ```

   启用以下任意项：

   ```
   three_waves: true
   ```

   ```
   canvas_lines: true
   ```

   ```
   canvas_sphere: true
   ```

   ```
   canvas_nest: true
   ```

4. 添加访问人数

   打开`\themes\next\layout\_partials\footer.swig`文件,在copyright前加上画红线这句话

   ```js
   <script async src="https://dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
   ```

   然后再合适的位置添加显示统计的代码，如图：

   ```javascript
   <div class="powered-by">
   <i class="fa fa-user-md"></i><span id="busuanzi_container_site_uv">
     本站访客数:<span id="busuanzi_value_site_uv"></span>
   </span>
   </div>
   ```

   - **pv**的方式，单个用户连续点击n篇文章，记录n次访问量
   - **uv**的方式，单个用户连续点击n篇文章，只记录1次访客数

5. 每篇文章末尾统一添加“本文结束”标记

   **实现方法**

   在路径 `\themes\next\layout\_macro` 中新建 `passage-end-tag.swig` 文件,并添加以下内容：

   ```javascript
   <div>
       {% if not is_index %}
           <div style="text-align:center;color: #ccc;font-size:15px;">--------------都看到这了，请我喝杯咖啡吧！<i class="fa fa-coffee"></i>--------------</div>
       {% endif %}
   </div>
   ```

   接着打开`\themes\next\layout\_macro\post.swig`文件，在`post-body` 之后， `post-footer` 之前添加

   ```javascript
   <div>
     {% if not is_index %}
       {% include 'passage-end-tag.swig' %}
     {% endif %}
   </div>
   ```

   打开主题配置文件，文章末尾添加标记

   ```yaml
   passage_end_tag:
     enabled: true
   ```

6. 增加评论系统


   主题配置

   ```yaml
   gitment:
     enable: true
     mint: true # RECOMMEND, A mint on Gitment, to support count, language and proxy_gateway
     count: true # Show comments count in post meta area
     lazy: false # Comments lazy loading with a button
     cleanly: false # Hide 'Powered by ...' on footer, and more
     github_user: silloy  # MUST HAVE, Your Github Username
     github_repo: repo  # MUST HAVE, The name of the repo you use to store Gitment comments
     client_id: xxx # MUST HAVE, Github client id for the Gitment
     client_secret: xxx 
   ```

   问题：

   - Error：validation failed

     修改 next/layout/_third-party/comments/gitment.swig中`id: window.location.pathname`为`id: '{{ page.date }}'`

7. 文章底部增加版权信息

**参考文章**：

[Hexo-next](http://theme-next.iissnan.com)

[利用Gulp来压缩你的Hexo博客](https://leaferx.online/2017/06/16/use-gulp-to-minimize/)

[hexo的next主题个性化教程:打造炫酷网站](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)

[老高博客](https://gaoyuhao.ga)

[gitment](https://github.com/imsun/gitment#methods)

[Gitment评论功能接入踩坑教程](http://ihtc.cc/2018/02/25/2018-02-25%20_Gitment%E8%AF%84%E8%AE%BA%E5%8A%9F%E8%83%BD%E6%8E%A5%E5%85%A5%E8%B8%A9%E5%9D%91%E6%95%99%E7%A8%8B/)

[实现 Hexo next 主题博客本地站内搜索](https://zetaoyang.github.io/post/2016/07/08/hexo-localsearch.html)


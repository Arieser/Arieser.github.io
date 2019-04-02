---
title: hexo主题升级
date: 2018-03-01 12:25:10
tags: 
  - hexo
---



偶然看到最新的next主题，甚佳，决定对自己的博客主题进行升级，简要记录升级过程。

<!--more-->

1. 下载最新next 6.0.4, —> **参见11**，使用模块管理主题

2. 备份_config.yml，同时对配置进行修改，相关图片位于next/source/images

   ```
   cd themes/next
   git clone https://github.com/theme-next/theme-next-fancybox3 source/lib/fancybox
   git clone https://github.com/theme-next/theme-next-pace source/lib/pace
   git clone https://github.com/theme-next/theme-next-algolia-instant-search source/lib/algolia-instant-search
   ```

3. 其他插件

   1. 增加阅读进度

      ```shell
      git clone https://github.com/theme-next/theme-next-reading-progress source/lib/reading_progress
      ```

   2. 填补字符间空白

      ```shell
      git clone https://github.com/theme-next/theme-next-pangu.git source/lib/pangu
      ```

   3. 页面增加3D渲染，next默认提供两种渲染效果，theme-next-three和canvas_nest

      ```shell
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

6. 添加访问人数（6.0已原生支持busuanzi，启用即可）

   打开`\themes\next\layout\_partials\footer.swig`文件,在copyright前加上画红线这句话

   ```js
   <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
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

7. 每篇文章末尾统一添加“本文结束”标记

   **实现方法**

   在路径 `\themes\next\layout\_macro` 中新建 `passage-end-tag.swig` 文件,并添加以下内容：

   ```javascript
   <div>
       {% if not is_index %}
           <div style="text-align:center;color: #ccc;font-size:15px;">--------------都看到这了，请我喝杯咖啡吧！<i class="fa fa-coffee"></i>--------------</div>
       {% endif %}
   </div>
   ```

   接着打开`\themes\next\layout\_macro\post.swig`文件，在`{### END POST BODY ###}` 之后， `<footer class="post-footer">` 之前添加

   ```javascript
   <div>
     {% if not is_index %}
       {% include 'passage-end-tag.swig' %}
     {% endif %}
   </div>
   ```

   打开主题配置文件，文章末尾添加标记（不用设置）

   ```yaml
   passage_end_tag:
     enabled: true
   ```

8. 增加评论系统

   **gitment**

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

     修改 next/layout/_third-party/comments/gitment.swig中`id: window.location.pathname`为

     ```
     id: '{{ page.date }}'
     ```

   **valine**

9. 文章底部增加版权信息

10. 修改文章底部的那个带#号的标签

   修改模板`/themes/next/layout/_macro/post.swig`，搜索 `rel="tag">#`，将 # 换成`<i class="fa fa-tag"></i>` 

11. 修改打赏字体不闪动，next 7.0 已支持配置

   修改文件`next/source/css/_common/components/post/post-reward.styl`， 注释`wechat:hover` 和`alipay:hover`， 如下：

   ```css
   /* 注释文字闪动函数
    #wechat:hover p{
       animation: roll 0.1s infinite linear;
       -webkit-animation: roll 0.1s infinite linear;
       -moz-animation: roll 0.1s infinite linear;
   }
    #alipay:hover p{
      animation: roll 0.1s infinite linear;
       -webkit-animation: roll 0.1s infinite linear;
       -moz-animation: roll 0.1s infinite linear;
   }
   */
   ```

12. 模块化主题管理（以next主题为例）

   1. 备份next主题 `mv next next-bak`，提交代码

   2. **增加子模块** `git submodule add git@github.com:silloy/hexo-theme-next.git themes/next`

   3. 查看状态 `git status`

      ```shell
      Changes to be committed:
        (use "git reset HEAD <file>..." to unstage)
      
      	new file:   .gitmodules
      	new file:   themes/next
      ```

   4. 提交

      ```shell
      $  git commit -m "add next module"
      [hexo adbe36e] add next module
       3 files changed, 19 insertions(+), 1 deletion(-)
       create mode 100644 .gitmodules
       create mode 160000 themes/next
      ```

   5. **更新子模块** `git submodule update --remote`

   6. **拉取含子模块的项目**，git clone 后执行以下操作

      1. `git submodule init` 初始化本地配置文件

      2. `git submodule update` 从该项目中抓取所有数据并检出父项目中列出的合适的提交

         也可在 clone 使用 `git clone --recursive` 命令, git 就会自动初始化并更新仓库中的每一个子模块.

      3. 若子分支仓库中有未同步的更新, 可通过 `git submodule update --remote --rebase` 来同步最新的内容

   7. **同步源主题的修改**

      1. 增加源

         ```shell
         cd theme/next
         git remote add source  git@github.com:theme-next/hexo-theme-next.git
         ```

      2. 拉取更新

         ```shell
         git pull source master
         ```

         等同于

         ```shell
         git fetch source master
         git checkout master
         git merge source/master
         ```

   8. **发布子模块的修改**

      1. 使用 `git push --recurse-submodules=check` 命令 检查没有推送的子模块
      2. 使用 `git push --recurse-submodules=on-demand` git 会自动尝试推送变更的子项目


**参考文章**：

[NexT 使用文档](http://theme-next.iissnan.com/)

[利用Gulp来压缩你的Hexo博客](https://leaferx.online/2017/06/16/use-gulp-to-minimize/)

[hexo的next主题个性化教程:打造炫酷网站](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)

[老高博客](https://gaoyuhao.ga)

[gitment](https://github.com/imsun/gitment#methods)

[Gitment评论功能接入踩坑教程](http://ihtc.cc/2018/02/25/2018-02-25%20_Gitment%E8%AF%84%E8%AE%BA%E5%8A%9F%E8%83%BD%E6%8E%A5%E5%85%A5%E8%B8%A9%E5%9D%91%E6%95%99%E7%A8%8B/)

[实现 Hexo next 主题博客本地站内搜索](https://zetaoyang.github.io/post/2016/07/08/hexo-localsearch.html)

[我的个人博客之旅：从jekyll到hexo](http://blog.csdn.net/u011475210/article/details/79023429)

[在 hexo 中使用 git submodules 管理主题](https://juejin.im/post/5c2e22fcf265da615d72c596)
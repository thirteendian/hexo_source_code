---
title: git+github+hexo配置手册
date: 2017-10-02 12:25:02
tags: config
---
## git+github+hexo集成配置手册

_-Git跟Github是什么关系?_

_-大概就是「魔兽争霸」与「对战平台」的关系吧。_

_摘自知乎mzlogin_
<!-- more -->
本文<>内为此处占位内容提示
### git使用手册

1. 在根目录下创建.git

        git init
1. 仓库跟踪新文件时
        
        git add <filename>
        git add .//将所有文件暂存
1. 仓库取消跟踪文件时
        
        git rm <filename>//删除文件
        git rm -r <documname>//删除文件夹
1. 提交

        git commit -m "comment"
        //当需要大量编写时不加-m且自动打开默认编辑器
        历次comment用
        git log
        查看
1. 查看状态

        git status
    * 跟随status的指示解决问题 
        
        * untracked files：未用add加到跟踪列中的新文件
        * changes to be committed：新加入跟踪中的文件
        * changes not staged for commit：(1)跟踪了改动的文件(2)跟踪了不存在文的件，通常是本地删除了但是本地仓库没有删除

    
1. 本地新分支

分支是用来隔绝编写以期防止破坏主分支的，一般可以在新分支上编写，成功后再合并(merge)到主分支上

        git checkout -b <branchname>//创建分支
        git checkout -d <branchname>//删除分支
        git checkout <existbranshname>//切换分支
        创建新分支->编写->切换回主分支合并->删除新分支

        git branch//查看仓库有几个分支
### github 配置

1. 关联git

        git config --global username "<username>"
        git config --global useremail "<username@address.com>"
1. 设置密钥
这一步与hexo设置一样

1. 新建仓库
1. 将本地仓库关联到远端仓库git remote

        git remote add <shortname> <url>
        对于github可以这样设置
        git remote add origin git@github.com:<username>/<reponame>.git
        \\如果不设置密钥会弹出用户名密码框


        git remote -v\\会显示所有关联的远程仓库及其地址 
        \\origin是git默认从远端clone下来的名字
        git remote show <shortname>\\显示仓库详细信息
        git remote rename <shortname1> <shortname2>\\远程仓库重命名
        git remote rm <shortname>\\删除远程仓库
1. 上传前先推送本地(主)分支
        
        git push <shortname> <branchname>
        一般<branchname>直接master即可
        git push origin master

        [注意]:如果提示fail则应先pull再push
        git pull -u <shortname> master
1. 上传代码

        git push -u <shortname> master
    -u的含义其实将两个命令达到的效果结合起来了：创建+关联

        git branch --set-upstream-to <reponame>/<branchname> <localbranchname>
        这条命令是创建内外branch关联，这样在local上直接pushpull而不用加额外指示，但当外面没有branchname时则失效，而用git push相当于创建+关联

1. 更新代码
        
        不停的git push即可，一定注意先将本地仓库commit好后再push远程端

### github灰色文件夹

说明此文件夹下还有github的repo，如果想由灰变白可以将其下的`.git`和`.gitignore`删去

### 快速配置环境

更换电脑或本地数据丢失时只需要clone源代码再配置hexo环境即可

    git clone git@github.com:<username>/<reponame>.git
当本地版本不是最新时，进行本地仓库更新

    git fetch <shortname>-----(1)
合并前可查看新版本的不同

    git diff <branch1> <branch2>
然后将仓库内容合并到本地分支

    git merge <shortname>/master-----(2)
(1)(2)两步可以合为一步

    git pull <shortname> master-----(1)+(2)
若有冲突则检查有冲突的文件路径

    git checkout
然后就是

    add ./rm --> commit --> status --> push  
配置hexo

        npm install hexo --save

### hexo持续集成

防止电脑、hexo源文件崩溃导致网页源代码消失，可以在github中再建一个repo专门放hexo的源代码。这样我们发现每次hexo编辑博客都会有两个过程

    git push
    hexo deploy
我们将采用持续集成(Continuous integration)，使得当执行git push时自动运行CI脚本直接发布,于是采用对windows友好的[AppVeyor](https://ci.appveyor.com/login)

* 注册账号
采用github账号登陆即可
* 添加github project
* 在hexo源代码的repo下新增文件appveyor.yml

        clone_depth: 5
        environment:
        access_token:
            secure: *******************
        install:
        - node --version
        - npm --version
        - npm install
        - npm install hexo-cli -g
        build_script:
        - hexo generate
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
        - if "%APPVEYOR_REPO_BRANCH%"=="master" if not defined APPVEYOR_PULL_REQUEST_NUMBER (git diff --quiet --exit-code --cached || git commit -m "Update Static Site" && git push origin %TARGET_BRANCH% && appveyor AddMessage "Static Site Updated")


        进入github的setting-->左侧Personal access tokens-->Generate new token-->复制过来即可
appveyor中的[加密页面](https://ci.appveyor.com/tools/encrypt)应该输入并生成

此时将生成的代码替换**************

然后在刚刚添加的project下的setting选项Environment下创建四个变量

        GIT_USER_EMAIL <username@address.com>
        GIT_USER_NAME  <username>
        STATIC_SITE_REPO <repo url>
        TARGET_BRANCH <master>
## hexo内置标签

使用hexo时，相比原来的markdown，应首先使用内置标签，标签的开始终止类似latex的box模式

### 引用

    {% blockquote [author], [sourcename] [link-url] [link_title] %}
    content
    {% endblockquote %}
    //注意这里的链接url和链接名link_title顺序正好和markdown反过来

    居中引用
    {% centerquote %}
    content
    {% endcenterquote %}
    或简称
    {% cq %}
    {% endcq %}
### 内容省略显示

通常在主页上省略显示内容

    <!-- more -->

### 代码块

    {% codeblock [title] [lang:language] [url] [link text] %}
    code snippet
    {% endcodeblock %}

    也可以用反引号
    ```[language][title][url][link text]```

{% codeblock test lang:markdown http://test.com test_url %}
this is a test code
{% endcodeblock %}

### 图片

用标签可以插入指定大小的图片，如果不介意图片压缩的话
一些图片网站直接提供图片链接(可以直接右键新标签页打开图片)，一些收费网站可以利用审查元素获得内链接地址，完全可以直接拉过来

也可利用一些插件比如chrome的500px download可以直接将500px的图片拉下来

自己上传图片图床可用[七牛云](https://portal.qiniu.com/)

    {% img [class names] [url] [width] [height] [title text [alt text]] %}

    突破容器宽度限制的图片，用于强调
    {% fullimage /image-url, alt, title %}
如
{% fullimage https://drscdn.500px.org/photo/174588983/q%3D80_m%3D2000/v2?webp=true&sig=5cf9d11a986f5731051f23503c2de4105a46f5c8f9cdb17e8277b100041d4c92 %}

### Bootstrap Callout样式

{% note default %} Content (md partial supported) {% endnote %}

    {% note class_name %} Content (md partial supported) {% endnote %}
    其中class_name可替换成
    无           浅灰
    default     深灰
    primary     蓝
    success     绿
    info        天蓝
    warning     黄
    danger      红

### 字体

_config.yml->fond

### 背景

_config.yml->canvas

### 文章结束语

主题下`\layout\_macro`中新建`passage-end-tag.swig`添加

        <div>
        {% if not is_index %}
                <div style="text-align:center;color: #ccc;font-size:14px;">
                -------------往者不可諫<i class="fa fa-paw"></i>來者猶可追-------------
                </div>
        {% endif %}
        </div>
并在`、layout\_macro\post.swig`中的`END POST BODY`中添加

        <div>
        {% if not is_index %}
                {% include 'passage-end-tag.swig' %}
        {% endif %}
        </div>
在主题配置文件添加

        # 文章末尾添加“本文结束”标记
        passage_end_tag:
        enabled: true

### 文章底部版权

在`next/layout/_macro/`下修改`post-copyright.swig`为

        {% if page.copyright %}
        <ul class="post-copyright">
        <li class="post-copyright-author">
        <strong>本文标题:</strong>
        <a href="{{ url_for(page.path) }}">{{ page.title }}</a>
        </li>
        <li class="post-copyright-author">
        <strong>{{ __('post.copyright.author') + __('symbol.colon') }}</strong>
        {{ config.author }}
        </li>
        <li class="post-copyright-author">
        <strong>发布时间:</strong>
        {{ page.date.format("YYYY年MM月DD日 - HH:MM") }}
        </li>
        <li class="post-copyright-author">
        <strong>最后更新:</strong>
        {{ page.updated.format("YYYY年MM月DD日 - HH:MM") }}
        </li>
        <li class="post-copyright-link">
        <strong>{{ __('post.copyright.link') + __('symbol.colon') }}</strong>
        <a href="{{ post.permalink }}" title="{{ post.title }}">{{ post.permalink }}</a>
        </li>
        <li class="post-copyright-license">
        <strong>{{ __('post.copyright.license_title') + __('symbol.colon') }} </strong>
        {{ __('post.copyright.license_content', theme.post_copyright.license_url, theme.post_copyright.license) }}
        </li>
        </ul>
        {% endif %}

### 修改md默认头部

根目录`/scaffolds/post.md`修改为

        ---
        title: {{ title }}
        date: {{ date }}
        tags:                #标签
        categories:      #分类
        copyright: true #版权声明
        permalink: 01  #文章链接，有默认值
        top: 0              #置顶优先级
        password:      #密码保护
        ---
### 博文置顶

* 安装插件

                npm install hexo-generator-index--save

* 替换代码

找到`node_modules/hexo-generator-index/lib/generator.js`修改为

                'use strict';
                var pagination = require('hexo-pagination');
                module.exports = function(locals){
                var config = this.config;
                var posts = locals.posts;
                posts.data = posts.data.sort(function(a, b) {
                        if(a.top && b.top) { // 两篇文章top都有定义
                        if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
                        else return b.top - a.top; // 否则按照top值降序排
                        }
                        else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
                        return -1;
                        }
                        else if(!a.top && b.top) {
                        return 1;
                        }
                        else return b.date - a.date; // 都没定义按照文章日期降序排
                });
                var paginationDir = config.pagination_dir || 'page';
                return pagination('', posts, {
                perPage: config.index_generator.per_page,
                layout: ['index', 'archive'],
                format: paginationDir + '/%d/',
                data: {
                __index: true
                }
                });
                };

* 在文章中设置top值即可，越大越靠前

                ---
                top: 100
                ---

### 增强底部标签

修改`/themes/next/layout/_macro/post.swig`下的`rel=”tag”>#`
将#换成

                <i class="fa fa-tag"></i>
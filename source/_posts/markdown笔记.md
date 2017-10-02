---
title: markdown+网站配置手册
date: 2017-10-01 10:43:51
tags: config
mathjax: true
---
# Markdown+网站配置手册

## 1.VScode

采用markdown编写程序进行及时预览
    cmd(CTRL)+K+V

## 2.标题headers

标题行上下应该空出两行

标题采用

# 一级标题

    # 一级标题
    或
    一级标题
    =========（几个无所谓）

## 二级标题

    ## 二级标题
    或
    ---------（几个无所谓）

### 三级标题

    ### 三级标题
以此类推

## 3.序号

有以下两种

1.无序序号

    *
    *
    *
2.有序序号

1. 这是一个序号
这还是同一个序号

        1. 这是一个序号
        这还是同一个序号
        [注意]时的命令行格式就应该缩进两次
        直接用1.(空格)这种方式，无论前面数字是几
        避免1. 生成序号可以在"."前面加\

## 4.特殊字符

1. 转义

输入\\、\!、\@、\#等

    \\ \! \@ \# //与latex相似x
    [注意]网址中的&标记也要转义
1. 特殊字符仍符合html格式
如 &copy;  \&

    &copy；
    虽然对于"&"而言markdown会将其转换为&amp;但仍可用\&d的方法进行转义

## 5.引用

>这是一个引用
>这还是一个引用
这又是一个引用


>这是另一个引用
>>>>>这是五重引用

    >这是一个引用
    >这还是一个引用
    这又是一个引用


    >这是另一个引用
    >>>>>这是五重引用
    [注意]与代码框相似的，引用遇到序号应该缩进一次 遇到代码框应该缩进两次

## 6. 分割线

这是一个分割线
***
这还是一个分割线
- - -

    ***
    - - -

## 7. 链接

* 行内式

这是[百度](http://www.baidu.com)搜索引擎

    这是[百度](http://www.baidu.com)搜索引擎

* 参考式

这还是[百度][a]

[a]:http://www.baidu.com

    这还是[百度][a]
    [a]:http://www.baidu.com

这依然是[百度][]

[百度]: http://www.baidu.com/

    这依然是[百度][]
    [百度]: http://www.baidu.com/

* 直接确定链接
凡是用 <尖括号.com>（注意带有.com等链接尾的才可以）括起来的都会转为链接，邮箱同理<address@xxx.com>

## 8.强调

*em强调*

    *em强调*
    _em强调_

**strong强调**

    **strong强调**
    __strong强调__
    [注意]如果*或_两边都有空格的话就不再是强调

## 9. 代码

代码块只需要缩进一格

如果块内代码需要指定语言
```c
#include <stdio.h>
 int main ()
 {
     int a,i;
     for(i=1;i<10;i++>)
     {
         a++;
     }
 }
```

应该用

    (空格)
    ```language
    (内容)
    ```
    (空格)
如果有`行内代码`用"`"括起
## 10.图片

行内式图片用

    ！[asdf](path.jpg)、
这样的语句来表明，参考式模仿网址的参考式加上[id]然后在后部另外补上

注意markdown没法指定宽高，只有在hexo中利用

## 公式

### 配置公式页面启用mathjax

* 换用新渲染工具

        npm uninstall hexo-renderer-marked --save
        npm install hexo-renderer-pandoc --save

更改node_modules\kramed\lib\rules\inline.js
将11行的escape和第21行的em分别换成
        
        //  escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
        escape: /^\\([`*\[\]()#$+\-.!_>])/

        //  em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
        em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/

* theme/主题/_config.yml配置开启mathjax

        # MathJax Support
        mathjax:
        enable: true
        per_page: true

* 文章中开启mathjax开关

        title: markdown笔记
        date: 2017-10-01 10:43:51
        tags: markdown
        mathjax: true

* 采用latex编码编写即可

如

$$x^{y^z}=(1+e^x)^{-2xy^w}$$

        $$x^{y^z}=(1+e^x)^{-2xy^w}$$

$$ \begin{cases}

n/2, & \text{如果x<=2}\\
3n+1 , & \text{如果 x>2 }

\end{cases}
$$

        $$ \begin{cases}

        n/2, & \text{如果x<=2}\\
        3n+1 , & \text{如果 x>2 }

        \end{cases}
        $$
        [注意]这里$$是使公式居中

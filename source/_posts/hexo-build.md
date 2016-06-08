---
title: hexo build
date: 2016-05-19 07:51:09
tags: 
 - code
 - web
---


## 首先你得有个自己的github账户

[www.github.com](www.github.com)

注册完成后，新建一个名为`YOURUSERNAME.github.io`仓库，并在设置内将该仓库设置为gitpage

{% asset_img 2.png %}

{% asset_img 3.png %}

## 在linux本地设置好git

推荐文档

[http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)

## hexo安装 

推荐官方安装文档。
[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)

{% asset_img 1.png %}

## 博客初始化

还是推荐官方文档
[https://hexo.io/zh-cn/docs/setup.html](https://hexo.io/zh-cn/docs/setup.html)

## 主题切换

[https://github.com/Winter3un/hexo-theme-yilia](https://github.com/Winter3un/hexo-theme-yilia)

## 备份插件

[https://github.com/coneycode/hexo-git-backup](https://github.com/coneycode/hexo-git-backup)

## 写文章

hexo分为原始文件和发布文件两种。

首先需要用户写好原始文件，然后使用`hexo g`来生成发布文件。

接着将生成好的发布文件，`hexo d` 推送到服务器上。

简略命令`hexo g -d`

资料：[https://hexo.io/zh-cn/docs/writing.html](https://hexo.io/zh-cn/docs/writing.html)
[https://hexo.io/zh-cn/docs/generating.html](https://hexo.io/zh-cn/docs/generating.html)
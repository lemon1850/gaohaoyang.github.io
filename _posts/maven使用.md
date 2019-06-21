---
layout: post
title: 生活帖子
categories: 面试
tags:  面试
---

* content
{:toc}

> 我的技术博客，主要是记载看过的书以及对写的好的博客文章的搬运整理，方便自己他人查看，也方便别人指出我文章中的错误，达到一起学习的目的。
> 技术永无止境


1 解决冲突的首要方式是不要引入冲突
合理使用bom, dependency management , 建立优雅的项目结构。API打包排除无关依赖，不要把无关依赖打入包。
2 排包intellij idea maven helper 插件可以方便排除一些。
3 插件工具找不出的，手动 mvn dependency:tree -Dverbose -X > tree.txt
人肉找冲突的包，然后排除。
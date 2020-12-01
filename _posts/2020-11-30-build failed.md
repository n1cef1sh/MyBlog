---
layout: post
title: "GitHub Pages failed to build your site"
categories: [blog]
---

## 报错
自从更新了一篇学习博客后，疯狂报错。  

The tag extends on line 5 in _posts/2020-11-30-flask-web学习记录.md is not a recognized Liquid tag.  

改着改着突然9.24的Vue学习记录也开始报错，所幸把所有引用的代码全部删除了，不知道为什么突然md格式上传时会出现冲突和错误，也不太想深究了。  

突然myVue库也报错了……可能是因为最近刚把他从private设成了public，所以检测出了不安全的代码部分，也直接把该目录删除了。  

但是flask-web这里的错误一直不知道为什么，尝试了几十次commit，未果。直接删除了那个md，然后现在来写个乱七八糟不知所以的博文，试试到底能不能成功上传了。
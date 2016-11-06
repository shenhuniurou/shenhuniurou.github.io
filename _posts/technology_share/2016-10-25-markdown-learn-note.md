---
layout: post
title: markdown语法学习笔记
category: 技术分享
tags: markdown语法
---


- 标题前面用`#`，#越多字体越小，要注意的是，#后面要空一格，不空格在markdown编辑器上或许没什么问题，但在博客上就会有问题，#后面的文字没有被解析成对应的字体大小。

- 开头用`-`表示无序列表，比如前面实心的小黑点，就是这样出来的，后面也要空格。

    - 如果是需要空心的点，需要先一个`tab`键后再`-加空格`

- 如要要表示强调，比如某个单词

    - 使用`**`，被两个`*`包裹的内容会显示粗体，比如`**shenhuniurou**`会显示成**shenhuniurou**

    - 使用`*`，被一个`*`包裹的内容会显示斜体，比如`*shenhuniurou*`会显示成*shenhuniurou*
 
	- 使用`tab`键上面的那个字符，被这个字符包裹的内容会有一个阴影
	
	- `~~删除线~~`，效果是这样的~~删除线~~
	
	- `> 引用`，效果是 >引用

	
- 代码高亮，代码块用三个tab键上面那个字符加语言名再加三个这种字符来包裹代码，比如:

```xml
<!--index.wxml-->
<view class="container">
    <view bindtap="bindViewTap" class="userinfo">
        <image class="userinfo-avatar" src="{{userInfo.avatarUrl}}" background-size="cover"></image>
        <text class="userinfo-nickname">{{userInfo.nickName}}</text>
    </view>
    <view class="usermotto">
        <text class="user-motto">{{motto}}</text>
    </view>
</view>
```

- 表格 
   
|名称|年龄|性别|
|:----|:--------|:-----|
|小明小明|2222222|男
|小明|2111111|男
|小明小明小明小明|24444|男
|小明小明|2666|男
|小明小明小明|2777777|男


格式`|title1|title2|title3|...`，换行，`|:---:|:----|---:|`，其中`:`表示对齐的方向，在左边，表示左对其，在右边表示右对齐，两边都有，表示居中。然后表格中的数据就一条条写下来，要以`|`隔开。

- 链接和图片，方式是差不多的

    - 链接是“`[名称](url地址)`”这样的，比如[shenhuniurou‘s bolg](http://shenhuniurou.com)。

	- 图片是“`![名称](url地址)`”，还有一个方式，`<img src="http://sjdhfjksdhfksdlfsj" width="40%" />`比如你图片本身尺寸较大，而你又不需要图片显示很大时，可以用这种方式设置width。
	




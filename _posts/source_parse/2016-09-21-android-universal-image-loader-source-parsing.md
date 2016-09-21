---
layout: post
title: Android-Universal-Image-Loader框架源码解析
category: 源码解析
tags: 图片框架
keywords: 
---


Android-Universal-Image-Loader（UIL）作为一款强大的被普遍运用的可高度定制的第三方图片缓存显示框架，研究它的底层实现是很有必要的，研究透它的源码，能帮助我们更好地在项目中运用它，也能让我们学习到作者的设计思路，达到能够自定义我们想要的图片显示器的水准，因为今后我们有可能会遇到一些不太大的项目，对图片的处理也没有那么多要求，如果直接导入UIL可能会使得我们的项目体积变大，这时候自己写一个比较简单的图片显示器就更好了。
#####使用步骤：
* 导入jar包或者在主moudle的build.gradle中dependencies部分加上
```
compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.5'
```
* 然后在Application中进行相关配置
```
ImageLoaderConfiguration configuration = new ImageLoaderConfiguration.Builder(this)
            // 添加你的配置需求
            .build();
ImageLoader.getInstance().init(configuration);
```
ImageLoaderConfiguration就是用户自定义的配置信息， 这个类在```
package com.nostra13.universalimageloader.core;```包中，可配置的信息包括：
```java
//下面这四个是图片在disk和memory中的最大缓存尺寸
final int maxImageWidthForMemoryCache;
final int maxImageHeightForMemoryCache;
final int maxImageWidthForDiskCache;
final int maxImageHeightForDiskCache;
//默认的线程池大小
final int threadPoolSize;
//默认的线程池优先级
final int threadPriority;
//任务处理类型 默认是QueueProcessingType#FIFO先进先出
final QueueProcessingType tasksProcessingType;
//创建内存缓存DefaultConfigurationFactory#createMemoryCache(android.content.Context, int) 
final MemoryCache memoryCache;
//创建本地硬盘缓存com.nostra13.universalimageloader.cache.disc.impl.UnlimitedDiskCache
final DiskCache diskCache;
//下面三个是下载器
final ImageDownloader downloader;
final ImageDownloader networkDeniedDownloader;
final ImageDownloader slowNetworkDownloader;
//解码器DefaultConfigurationFactory#createImageDecoder(boolean)
final ImageDecoder decoder;
//显示选项 DisplayImageOptions#createSimple() Simple options
final DisplayImageOptions defaultDisplayImageOptions;
```
* 使用UIL还需要添加下面这两个权限：
```java
//网络权限
<uses-permission android:name="android.permission.INTERNET" />
//如果允许了diskCache，还需要添加写外部存储的权限
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
* 最后在需要显示图片的地方使用语句
```java
//根据图片url下载图片并解析为bitmap，最后显示在imageview上
ImageLoader.getInstance().diaplayImage(imageurl, imageview);
```

#####工作流程及原理
* Overall design
![overall-design.png](http://upload-images.jianshu.io/upload_images/1159224-6bb86e4a0a354666.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由总体设计图可知，这套框架可分为ImageLoaderEngine，Cache，Decoder，ImageDownloader，BitmapDisplayer，BitmapPrecessor这几个大的部分，Cache又可分为MemoryCache和DiskCache两块，整个工作流程就是ImageLoader接收到加载显示图片的任务，然后将任务交给ImageLoaderEngine，ImageLoaderEngine分发任务到具体的线程池去执行，任务通过Cache或者ImageDowner获取到图片，经过BitmapDecoder或者BitmapProcessor处理解析为Bitmap后再交给BitmapDisplayer显示到ImageAware上。

* Load&Display Task Flow
![uil-flow.png](http://upload-images.jianshu.io/upload_images/1159224-12f6ec0888a73faf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
加载及显示图片的任务流：
1、首先会从内存中取，如果bitmap对象缓存在内存里，则经过BitmapProcessor后处理再交给BitmapDisplayer显示
2、Bitmap对象不在内存中，则会去disk中查找图片，找到后BitmapDecoder将图片解码为Bitmap对象， 再经过BitmapProcessor预处理缓存到内存中，再经过BitmapProcessor后处理交给BitmapDisplayer显示。
3、如果disk中也找不到图片时，就要从网络上下载了。ImageDowner下载完图片后先缓存图片到disk上，然后BitmapDecoder将图片解码为Bitmap对象， 再经过BitmapProcessor预处理缓存到内存中，再经过BitmapProcessor后处理交给BitmapDisplayer显示。

* 包结构

![universal-image-loader包结构.png](http://upload-images.jianshu.io/upload_images/1159224-5a34b8d67cab1354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 
```javacache分为MemoryCeche和DiskCache，内存图片缓存和本地图片缓存；
core中包括：
图片解码器BitmapDecder，负责将图片输入流inputstream转换为Bitmap对象
图片显示器BitmapDisplayer，将bitmap对象显示到ImageAware
图片下载器ImageDowner，从图片的各个来源获取输入流
图片显示容器ImageAware，承载bitmap对象
图片处理器BitmapProcessor，从缓存中读取bitmap对象和写入缓存时对bitmap进行处理
任务引擎ImageLoaderEngine，将具体的任务分发给不同的task去完成
用于加载及显示图片任务LoadAndDisplayImageTask
用于处理和显示图片的任务ProcessorAndDisplayImageTask
用于显示图片的任务DisplayBitmapTask
```

#####核心类介绍
* ######ImageLoader
单例对象
```java
private volatile static ImageLoader instance;
/** Returns singleton class instance */
public static ImageLoader getInstance() {   
        if (instance == null) {      
                synchronized (ImageLoader.class) {        
                       if (instance == null) {            
                              instance = new ImageLoader();         
                       }      
                }   
        }   
        return instance;
}
```
初始化方法
```java
public synchronized void init(ImageLoaderConfiguration configuration) {
       if (configuration == null) {
            throw new IllegalArgumentException(ERROR_INIT_CONFIG_WITH_NULL);   
       }   
       if (this.configuration == null) {      
            L.d(LOG_INIT_CONFIG);      
            engine = new ImageLoaderEngine(configuration);                        
            this.configuration = configuration;   
       } else {      
            L.w(WARNING_RE_INIT_CONFIG);   
      }
}
```
这就是我们当时在Application中自定义的configuration然后再调用
```java
ImageLoader.getInstance().init(configuration);
```
初始化时会创建一个任务引擎ImageLoaderEngine。
显示图片方法：
```java
public void displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageSize targetSize, ImageLoadingListener listener, ImageLoadingProgressListener progressListener)
```
另外ImageLoader提供三种加载显示图片的方法，但是这三种方法最后都是调用的上面displayImage
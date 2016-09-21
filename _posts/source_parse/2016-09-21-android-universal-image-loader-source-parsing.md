---
layout: post
title: Android-Universal-Image-LoaderԴ�����
category: Դ�����
tags: Android-Universal-Image-Loader
keywords: 
---

Android-Universal-Image-Loader��UIL����Ϊһ��ǿ��ı��ձ����õĿɸ߶ȶ��Ƶĵ�����ͼƬ������ʾ��ܣ��о����ĵײ�ʵ���Ǻ��б�Ҫ�ģ��о�͸����Դ�룬�ܰ������Ǹ��õ�����Ŀ����������Ҳ��������ѧϰ�����ߵ����˼·���ﵽ�ܹ��Զ���������Ҫ��ͼƬ��ʾ����ˮ׼����Ϊ��������п��ܻ�����һЩ��̫�����Ŀ����ͼƬ�Ĵ���Ҳû����ô��Ҫ�����ֱ�ӵ���UIL���ܻ�ʹ�����ǵ���Ŀ��������ʱ���Լ�дһ���Ƚϼ򵥵�ͼƬ��ʾ���͸����ˡ�
#####ʹ�ò��裺
* ����jar����������moudle��build.gradle��dependencies���ּ���
```java
compile 'com.nostra13.universalimageloader:universal-image-loader:1.9.5'
```
* Ȼ����Application�н����������
```java
ImageLoaderConfiguration configuration = new ImageLoaderConfiguration.Builder(this)
            // ��������������
            .build();
ImageLoader.getInstance().init(configuration);
```
ImageLoaderConfiguration�����û��Զ����������Ϣ�� �������```
package com.nostra13.universalimageloader.core;```���У������õ���Ϣ������
```java
//�������ĸ���ͼƬ��disk��memory�е���󻺴�ߴ�
final int maxImageWidthForMemoryCache;
final int maxImageHeightForMemoryCache;
final int maxImageWidthForDiskCache;
final int maxImageHeightForDiskCache;
//Ĭ�ϵ��̳߳ش�С
final int threadPoolSize;
//Ĭ�ϵ��̳߳����ȼ�
final int threadPriority;
//���������� Ĭ����QueueProcessingType#FIFO�Ƚ��ȳ�
final QueueProcessingType tasksProcessingType;
//�����ڴ滺��DefaultConfigurationFactory#createMemoryCache(android.content.Context, int) 
final MemoryCache memoryCache;
//��������Ӳ�̻���com.nostra13.universalimageloader.cache.disc.impl.UnlimitedDiskCache
final DiskCache diskCache;
//����������������
final ImageDownloader downloader;
final ImageDownloader networkDeniedDownloader;
final ImageDownloader slowNetworkDownloader;
//������DefaultConfigurationFactory#createImageDecoder(boolean)
final ImageDecoder decoder;
//��ʾѡ�� DisplayImageOptions#createSimple() Simple options
final DisplayImageOptions defaultDisplayImageOptions;
```
* ʹ��UIL����Ҫ�������������Ȩ�ޣ�
```java
//����Ȩ��
<uses-permission android:name="android.permission.INTERNET" />
//���������diskCache������Ҫ���д�ⲿ�洢��Ȩ��
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
* �������Ҫ��ʾͼƬ�ĵط�ʹ�����
```java
//����ͼƬurl����ͼƬ������Ϊbitmap�������ʾ��imageview��
ImageLoader.getInstance().diaplayImage(imageurl, imageview);
```

#####�������̼�ԭ��
* Overall design
![overall-design.png](http://upload-images.jianshu.io/upload_images/1159224-6bb86e4a0a354666.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
���������ͼ��֪�����׿�ܿɷ�ΪImageLoaderEngine��Cache��Decoder��ImageDownloader��BitmapDisplayer��BitmapPrecessor�⼸����Ĳ��֣�Cache�ֿɷ�ΪMemoryCache��DiskCache���飬�����������̾���ImageLoader���յ�������ʾͼƬ������Ȼ�����񽻸�ImageLoaderEngine��ImageLoaderEngine�ַ����񵽾�����̳߳�ȥִ�У�����ͨ��Cache����ImageDowner��ȡ��ͼƬ������BitmapDecoder����BitmapProcessor�������ΪBitmap���ٽ���BitmapDisplayer��ʾ��ImageAware�ϡ�

* Load&Display Task Flow
![uil-flow.png](http://upload-images.jianshu.io/upload_images/1159224-12f6ec0888a73faf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
���ؼ���ʾͼƬ����������
1�����Ȼ���ڴ���ȡ�����bitmap���󻺴����ڴ���򾭹�BitmapProcessor�����ٽ���BitmapDisplayer��ʾ
2��Bitmap�������ڴ��У����ȥdisk�в���ͼƬ���ҵ���BitmapDecoder��ͼƬ����ΪBitmap���� �پ���BitmapProcessorԤ�����浽�ڴ��У��پ���BitmapProcessor������BitmapDisplayer��ʾ��
3�����disk��Ҳ�Ҳ���ͼƬʱ����Ҫ�������������ˡ�ImageDowner������ͼƬ���Ȼ���ͼƬ��disk�ϣ�Ȼ��BitmapDecoder��ͼƬ����ΪBitmap���� �پ���BitmapProcessorԤ�����浽�ڴ��У��پ���BitmapProcessor������BitmapDisplayer��ʾ��

* ���ṹ

![universal-image-loader���ṹ.png](http://upload-images.jianshu.io/upload_images/1159224-5a34b8d67cab1354.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 
```javacache��ΪMemoryCeche��DiskCache���ڴ�ͼƬ����ͱ���ͼƬ���棻
core�а�����
ͼƬ������BitmapDecder������ͼƬ������inputstreamת��ΪBitmap����
ͼƬ��ʾ��BitmapDisplayer����bitmap������ʾ��ImageAware
ͼƬ������ImageDowner����ͼƬ�ĸ�����Դ��ȡ������
ͼƬ��ʾ����ImageAware������bitmap����
ͼƬ������BitmapProcessor���ӻ����ж�ȡbitmap�����д�뻺��ʱ��bitmap���д���
��������ImageLoaderEngine�������������ַ�����ͬ��taskȥ���
���ڼ��ؼ���ʾͼƬ����LoadAndDisplayImageTask
���ڴ������ʾͼƬ������ProcessorAndDisplayImageTask
������ʾͼƬ������DisplayBitmapTask
```

#####���������
* ######ImageLoader
��������
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
��ʼ������
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
��������ǵ�ʱ��Application���Զ����configurationȻ���ٵ���
```java
ImageLoader.getInstance().init(configuration);
```
��ʼ��ʱ�ᴴ��һ����������ImageLoaderEngine��
��ʾͼƬ������
```java
public void displayImage(String uri, ImageAware imageAware, DisplayImageOptions options, ImageSize targetSize, ImageLoadingListener listener, ImageLoadingProgressListener progressListener)
```
����ImageLoader�ṩ���ּ�����ʾͼƬ�ķ��������������ַ�������ǵ��õ�����displayImage
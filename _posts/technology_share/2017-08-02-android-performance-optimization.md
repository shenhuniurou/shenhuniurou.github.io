---
layout: post
title: Andorid性能优化总结
category: blog
tags: 性能优化
---




最近大半个月都在做app的优化，最主要是从apk包大小、界面过度绘制、掉帧、内存抖动、主线程IO这几个方面来入手的。相比开发新功能，做优化真的是更费脑力和心神，因为也许你做了大量的修改和优化操作，能看见的效果却微乎其微，但又不得不做。我想对于大多数Android开发者来说，开发出一款app并不难，但是开发出一款高性能体验棒的app却并不是每个开发者都能做到的。在开发过程中，可能会因为各种各样的原因会使你开发出来的app性能不佳，体验很差，开发者写代码的水平肯定是最重要的因素，另外还有一些第三方SDK也可以能会引起你的app出现问题。


今天抽空把这大半个月以来优化的过程和心得记录一下。

## apk包大小

无论是个人开发者自己开发的产品还是公司开发的产品，尽可能减小apk包的大小可以大大提高app的下载转化率，所以优化过程中apk体积是开发者必须要注意的一点。我们先看看apk包是由哪些部分组成的。我们可以使用Android Studio自带的功能Build->Analyze APK，下图是微信的apk组成：

![wechat_apk](http://offfjcibp.bkt.clouddn.com/wechat_apk.png)


- lib 存放app所需的native库文件
- classes.dex 开发者编写的java文件最后都会转化成dex文件运行在Android虚拟机上
- assets 存放需要保持原始文件的资源文件
- res 存放所有的资源文件
- resource.arsc  所有资源文件的id映射
- META-INF 签名校验文件
- AndroldManifest.xml   Android应用全局配置文件
- 其它一些配置文件和第三方库生成的文件


#### 删除无用资源

首先我从资源文件入手。我们可以使用Android Studio中的工具搜索项目中没有使用的的资源文件：

![run-inspection-by-name](http://offfjcibp.bkt.clouddn.com/run-inspection-by-name.png)

然后通过unused resources这个功能来查找：

![unused-resources](http://offfjcibp.bkt.clouddn.com/unused-resources.png)

查找完成后会把整个项目中未被使用的资源列出来，但是这个清理的时候我们需要注意下，有些资源文件可能是在你的项目中有用到的，但是资源id未出现在项目中的，比如你在代码中是通过资源名来获取这个资源而不是通过资源id，这种情况下的资源就不能删了，否则就会出错。另外一些第三方库的资源文件也要小心误删。

#### 关于保留几种屏幕分辨率的资源文件

随着目前手机市场的发展，屏幕越来越大，分辨率越来越高，但是基于成本的考虑，仍然还有一些相对较低的分辨率的机型，而我们的app是适配所有分辨率的机型，还是针对一些主流的机型做适配，这取决于我们app所适用的人群范围。我们UI设计师一般是设计xxhdpi(480dpi)和xhdpi(320dpi)两套，基本上可以满足绝大部分的手机屏幕了，如果你的app对apk体积有极高的要求，那么你也可以只选择一套分辨率的素材。

#### 资源压缩

图片压缩：一般UI给我们的图片都可以压缩，我一般采用[tinypng](https://tinypng.com/)在线压缩，支持png/jpg格式，为了避免失真，不要对同一张图片压缩多次。如果是Mac系统下，还可以选择[ImageOptim](https://imageoptim.com/mac)。

使用SVG图片：SVG图片即矢量图，简单的说，就是缩放不失真的图像格式。使用矢量图的好处有很多，可被非常多的工具读取和修改（比如记事本）。SVG与JPEG和GIF图像比起来，尺寸更小，且可压缩性更强。SVG是可伸缩的，SVG图像可在任何的分辨率下被高质量地打印，SVG可在图像质量不下降的情况下被放大，SVG图像中的文本是可选的，同时也是可搜索的（很适合制作地图），SVG可以与Java技术一起运行，SVG文件是纯粹的XML。使用Android Studio也可以将下载的svg格式的矢量图转化为xml格式。比如我一般在阿里的[Iconfont](http://www.iconfont.cn/)上下载一些简单的图标，它可以选择svg格式下载，然后在Android Studio的drawable文件夹右键NEW->Vector Asset:

![](http://offfjcibp.bkt.clouddn.com/Vector-Asset.png)

然后会进到下面这个界面：

![](http://offfjcibp.bkt.clouddn.com/Configure-Vector-Asset.png)

其中Asset Type类型中Material Icon是通过系统自带的一些符合Material Design设计的icon来制作，Local file就是通过我们自己下载的svg文件来制作，完成后svg格式的文件就转化成xml格式保存在我们的目标文件夹里了。当然普通的ImageView是无法使用矢量图的，必须使用v7包下的AppCompatImageView，而且不能使用src属性，必须是app:srcCompat属性才可以。


用jpg代替png：因为jpg没有alpha通道，所以文件更小，比较适合于不需要透明度的图片。

用shape、color来代替图片：如果是渐变背景或者纯颜色的控件背景，都可以使用shape或者color做背景，一个xml文件相比一张位图要小得多。

#### 使用混淆

混淆除了代码压缩代码混淆的功能还有资源压缩的作用，在app module的build.gradle文件中配置shrinkResources true和minifyEnabled true这两个参数，就可以达到混淆压缩的目的。关于混淆，参看这篇文章 [Android代码混淆与进阶](http://shenhuniurou.com/2017/07/27/android-proguard)

#### native库文件

native库都是为了支持不同架构的CPU，虽然native库文件是占整个apk体积最大的部分，减少对其中一些CPU架构的支持可以很直观的看到变化。关于so文件的知识和它的适配，可以看看下面几篇文章，会有很大帮助。

- [谈谈Android的so](http://allenfeng.com/2016/11/06/what-you-should-know-about-android-abi-and-so/)
- [Android SO文件的兼容和适配](http://blog.coderclock.com/2017/05/07/android/Android-so-files-compatibility-and-adaptation/)
- [关于Android的.so文件你所需要知道的](http://www.jianshu.com/p/cb05698a1968)


## 过度绘制
	
首先我们得知道，过度绘制这个概念，到底什么是过渡绘制？UI过度绘制简单的来说是指在一个界面中有很多元素，但是我们只需要更新某一小块的元素，app却把所有的元素都刷新一遍，这就造成过度绘制。过度绘制会造成GPU资源浪费，引起我们的app页面卡顿，掉帧现象。

![overdraw_options_view](http://offfjcibp.bkt.clouddn.com/overdraw_options_view.png)

上图是UI界面的过度绘制的情况，不同的颜色代表被过度绘制的次数，开发者可以在手机设置的开发者选项中将“调式过度绘制”选项开启，然后就可以看到界面上显示不同的颜色了。我们优化的目的就是尽可能让界面上只看到蓝色或绿色，尽量不要出现大量的红色。

怎么做才能减少红色部分？尽量将我们的layout布局层次简单化，移除Window默认的background，移除layout中非必须的background等，UI布局层次不应该过多，否则系统绘制UI时会占用较多时间，引起掉帧。

更详细的优化方法请看

- [Android性能优化之渲染篇](http://hukai.me/android-performance-render/)
- [实战 Android中的UI过度绘制](http://www.jianshu.com/p/9e095bacf44a)



## 内存抖动及优化

内存抖动是因为在短时间内大量的对象被创建又马上被释放。瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，会触发GC从而导致刚产生的对象又很快被回收。即使每次分配的对象占用了很少的内存，但是他们叠加在一起会增加Heap的压力，从而触发更多其他类型的GC。这个操作有可能会影响到帧率，并使得用户感知到性能问题。如果你在Memory Monitor里面查看到短时间发生了多次内存的涨跌，这意味着很有可能发生了内存抖动。同时我们还可以通过Allocation Tracker来查看在短时间内，同一个栈中不断进出的相同对象。这是内存抖动的典型信号之一。当你大致定位问题之后，接下去的问题修复也就显得相对直接简单了。例如，你需要避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外，自定义View中的onDraw方法也需要引起注意，每次屏幕发生绘制以及动画执行过程中，onDraw方法都会被调用到，避免在onDraw方法里面执行复杂的操作，避免创建对象。对于那些无法避免需要创建对象的情况，我们可以考虑对象池模型，通过对象池来解决频繁创建与销毁的问题，但是这里需要注意结束使用之后，需要手动释放对象池中的对象。


## 启动时间（冷启动，暖启动，热启动）

app的启动分为三种状态，冷启动即应用从零开始加载运行，而其它状态则是应用从后台运行回到前台运行。

冷启动状态：系统不存在该应用的进程，启动应用才创建出应用的进程。冷启动一般指的就是应用在开机后或者被系统停止后的第一次启动过程。因为系统和应用在冷启动时需要做更多的工作，所以减少它的启动时间的难度是最大的。
	
冷启动初始时，系统完成三个任务：

- 启动和加载应用(这里泛指的是应用本身)
- 创建应用的专属进程
- 启动后立刻显示启动视图(通常是个空白屏)
	
一旦系统创建了应用的专属进程，该进程开始创建应用：

- 创建应用对象
- 启动主线程 (MainThread)
- 创建 Launcher Activity
- 加载视图 (Inflating views)
- 渲染布局 (Laying out)
- 执行初始绘制

当应用完成了第一次绘制，系统进程就把当前显示的启动视图切换为应用界面，用户就可以使用应用了。

应用进程的启动流程可以分为：Application的创建和Launcher Activity的创建，Application的创建是从Application.onCreate()开始的，重写onCreate()方法通常我们会在这里完成一些通用组件和第三方SDK的初始化操作。接着应用程序生成主线程，并开始创建Launcher Activity。创建Activity的过程分为初始化、调用构造方法、调用当前生命周期的回调方法。通常onCreate()方法对加载时间的影响最大，因为它要执行加载、渲染和初始化Activity所需要的对象等开销最大的任务，如果Activity的布局过于复杂，那么就很可能会导致启动性能问题。

	
暖启动状态：应用程序的暖启动与冷启动类似，但比冷启动开销低。在暖启动中，系统只需要把 Activity 切换到前台运行。如果应用的该 Activity 之前驻留在内存中，那么应用程序就不用重新初始化对象和渲染布局。但是，如果由于响应了低内存事件，例如在 onTrimMemory() 方法中清除了资源对象，那么这些对象就需要在热启动时重新创建。
	
热启动状态：热启动为冷启动的过程操作的子集，而且开销比暖启动稍小。以下这些情况可以认为是热启动：

- 用户退出应用，但随后重新启动它。应用的进程还在运行，但应用必须重新从 onCreate() 开始创建 Activity。
- 系统从内存中清除了应用(非用户主动)，然后用户重新启动它。进程和 Activity 需要重新启动，但 onCreate() 将接收到保存状态的 Bundle。事实上，savedInstanceState 在用户未主动销毁 Activity 时系统就会调用。

启动时间的优化：

一般我们优化启动时间都是基于冷启动时间的，所以主要是创建应用和创建Activity这两部分所花时间的优化，在Application.onCreate()方法中，要尽量避免创建太多临时变量、执行IO操作、以及反序列化操作、多重for循环等。而在创建Launcher Activity过程中，布局视图要尽量层次简单，让绘制过程尽量短，加载大量资源时可以选择懒加载方式。

关于启动视图：

之前说过当应用启动后而且Launcher Activity的布局还没有被渲染完成时，会出现一个空白屏，如果Launcher Activity的布局非常复杂，渲染消耗的时间很长，那用户可能会感知到这个空白屏的存在，体验上来说，是很不友好的，那么怎么解决这个问题呢？我们可以使用windowBackground这个属性，一般我们会设置成一张图片，然后在AndroidManifest.xml中的application节点的Theme中加上这个属性。这样做之后，当应用被点击后立马就可以看到这张图片，而不是一个空白屏了。

## 主线程IO

如果在主线程中有读写文件等这些耗时的操作，就会阻塞主线程，影响app的性能。所以在主线程中，不要进行IO操作，也不要进行for循环创建大量的对象。如果必须要操作IO，也要放在子线程中或者Service中进行，以保证主线程不会被阻塞。


## 64K 引用限制

关于64k引用限制，官方的说明文档：[配置方法数超过 64K 的应用](https://developer.android.com/studio/build/multidex.html?hl=zh-cn)，规避64k限制的办法就是采用Dalvik可执行文件分包。具体的配置方法请看文档说明。


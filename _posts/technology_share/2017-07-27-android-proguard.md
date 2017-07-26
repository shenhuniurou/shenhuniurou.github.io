---
layout: post
title: Android代码混淆与进阶
category: 技术分享
tags: Android、混淆
---



混淆是 Android 打包过程中最重要的流程之一，基本上所有 app 都应该开启混淆，增加app的安全性。混淆其实是包括了代码压缩、代码混淆以及资源压缩等的优化过程。谷歌官方文档[Shrink Your Code and Resource](https://developer.android.com/studio/build/shrink-code.html)，依靠 ProGuard，混淆流程将主项目以及依赖库中未被使用的类、类成员、方法、属性移除，这有助于规避64K方法数的瓶颈；同时，将类、类成员、方法重命名为无意义的简短名称，增加了逆向工程的难度。而依靠 Gradle 的 Android 插件，我们将移除未被使用的资源，可以有效减小 apk 安装包大小。

## 配置方法

在 app module 目录中的 `build.gradle`文件中的 `android` 节点下配置如下：

```
buildTypes {
    release {
        shrinkResources true
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

因为混淆会大大减慢编译的速度，所以 debug 模式下不应该开启混淆，`minifyEnabled` 设置为 true 表示开启混淆，`shrinkResources` 设置为 true 表示开启压缩资源，`proguardFiles` 表示指定给项目的 ProGuard 文件，默认情况下是指定了两个 Proguard rules 文件，一个是 Android 系统自带的 `proguard-android.txt`，另外一个就是在 app module目录下的 proguard-rules.pro，`proguard-android.txt` 我们可以在 Android SDK 的目录中搜索即可找到该文件，完整路径是 SDK 目录下的 <sdk-root>/tools/proguard/。我们先看看 proguard-rules.pro 这个文件，它是留给开发者来自定义混淆规则的，一般开发者的混淆都写在这个文件中。


## 常见的混淆命令及含义

- optimizationpasses 指定压缩级别，默认是5
- dontoptimize 混淆时不优化类文件
- dontusemixedcaseclassnames 不使用大小写混合类名
- dontskipnonpubliclibraryclasses 混淆jars中的非public classes
- dontpreverify 混淆时不做预校验
- dontwarn 不提示警告，避免打包时某些警告出现
- ignorewarnings  忽略警告，避免打包时某些警告出现
- verbose 混淆时记录日志
- assumenosideeffects 优化时允许访问并修改类和成员的访问修饰符，可能作用域会变大。
- repackageclasses 可以把你的代码以及所使用到的各种第三方库代码统统移动到同一个包下。
- obfuscationdictionary 后面加一个纯文本文件路径，它的作用是指定一个字典文件作为混淆字典。默认情况下我们的代码命名会被混淆成 abcdefg... 字母组合的内容，需要修改可以使用这个配置项将字典修改成乱码或中文内容。
- keep 防止类和成员被移除或重命名
- keepnames 防止类和成员被重命名
- keepclassmembers 防止成员被移除或重命名
- keepclassmembernames 防止成员被重命名
- keepclasseswithmembers 防止拥有该成员的类被移除或重命名
- keepclasseswithmembernames 防止拥有该成员的类被重命名
- keepattributes 当“Annotation、Exceptions, Signature, Deprecated, SourceFile, SourceDir, LineNumberTable，InnerClasses”这些东西可能被移除时如果想保留，就使用该属性keep住



混淆的格式一般是下面这两种：

```
[混淆命令] [类]
```

和

```
[混淆命令] [类] {
    [成员];
}
```

其中"类"是代表类相关的限制条件，它最终会指定到某些符合限定条件的类，它可以是下列内容：

- 具体的类
- 类访问修饰符(public private protected)
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- extends，即可以指定类的基类
- implement，匹配实现了某接口的类
- $，内部类

"成员"代表类成员相关的限定条件，它将最终定位到某些符合该限定条件的类成员。它的内容可以使用：
- <init> 匹配所有构造器
- <fields> 匹配所有域
- <methods> 匹配所有方法
- 通配符*，匹配任意长度字符，但不含包名分隔符(.)
- 通配符**，匹配任意长度字符，并且包含包名分隔符(.)
- 通配符***，匹配任意参数类型
- ...，匹配任意长度的任意类型参数。比如void test(…)就能匹配任意 void test(String a) 或者是 void test(int a, String b) 这些方法。
- 访问修饰符（public、protected、private）


## 常用的自定义混淆规则

- 不混淆某个类

```
-keep public class com.shenhuniurou.DemoActivity { *; }
```

- 不混淆某个包所有的类

```
-keep class com.shenhuniurou.demo.** { *; }
```

- 不混淆某个类的子类

```
-keep public class * extends com.shenhuniurou.activity.BaseActivity { *; }
```

- 不混淆所有类名中包含了“model”的类及其成员

```
-keep public class **.*model*.** {*;}
```

- 不混淆某个接口的实现

```
-keep class * implements com.shenhuniurou.demo.DemoInterface { *; }
```

- 不混淆某个类的构造方法

```
-keepclassmembers class com.shenhuniurou.demo.DemoClass { 
    public <init>(); 
}
```

- 不混淆某个类的特定的方法

```
-keepclassmembers class com.shenhuniurou.demo.DemoClass { 
    public void test(java.lang.String); 
}
```


下面我们再看看SDK自带的默认混淆文件中定义了哪些混淆规则

```
# 不使用大小写混合类名
-dontusemixedcaseclassnames
# 混淆jars中的非public classes
-dontskipnonpubliclibraryclasses
# 混淆时记录日志
-verbose

# 混淆时不优化类文件
-dontoptimize
# 混淆时进行预校验
-dontpreverify

# 不混淆注解
-keepattributes *Annotation*
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# 保持 native 方法不被混淆
-keepclasseswithmembernames class * {
    native <methods>;
}

# 保持自定义View的getter/setter不被混淆
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# 保持Activity和其子类的View作为参数的方法不被混淆
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# 保持枚举不被混淆
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# 保持通过Parcelable序列化的类不被混淆
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

# 保持R文件不被混淆
-keepclassmembers class **.R$* {
    public static <fields>;
}

-dontwarn android.support.**


# Keep注解的相关的类和方法字段以及构造方法
-keep class android.support.annotation.Keep

-keep @android.support.annotation.Keep class * {*;}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <methods>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <fields>;
}

-keepclasseswithmembers class * {
    @android.support.annotation.Keep <init>(...);
}
```

有了这些默认的混淆规则之后，在开发者自定义的混淆文件中就不需要再写了，开发者一般需要关注的混淆规则是引用了第三方库后或者使用了jni的类、webview中调用了js方法，这些都需要开发者来定义混淆规则。第三方库的接入文档一般都已经写好了混淆规则，开发者只需要拷贝至proguard-rules.pro文件中即可。




完成混淆后，混淆过的包必须进行检查，避免因混淆引入的bug。

在使用上面的配置进行混淆打包后在 <module-name>/build/outputs/mapping/release/ 目录下会输出以下文件：

- dump.txt 描述APK文件中所有类的内部结构
- mapping.txt 提供混淆前后类、方法、类成员等的对照表
- seeds.txt 列出没有被混淆的类和成员
- usage.txt 列出被移除的代码

如果有使用渠道打包的话，那么输出目录是 <module-name>/build/outputs/mapping/channel/release/。我们可以根据 seeds.txt 文件检查未被混淆的类和成员中是否已包含所有期望保留的，再根据 usage.txt 文件查看是否有被误移除的代码。另外还需要从测试方面检查。将混淆过的包进行全方面测试，检查是否有 bug 产生。


## 混淆资源文件

一般使用上述两个混淆文件的配置混淆出来的apk，只是混淆了代码，而对于项目中的资源文件名却没有混淆的，所以一般别人反编译你的apk后，是可以清楚的看到你的资源文件的（包括命名和内容）。微信团队开源了一个能混淆资源文件的库叫[AndResGuard](https://github.com/shwenzhang/AndResGuard)，AndResGuard是一个帮助你缩小APK大小的工具，他的原理类似Java Proguard，但是只针对资源。他会将原本冗长的资源路径变短，例如将res/drawable/wechat变为r/d/a下面是微信apk的资源目录结构：

![wechat_res_proguard](http://upload-images.jianshu.io/upload_images/1159224-affb7954e5035283.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 解码混淆过的堆叠追踪

混淆后的类、方法名等等难以阅读，这固然会增加逆向工程的难度，但对追踪线上 crash 也造成了阻碍。我们拿到 crash 的堆栈信息后会发现很难定位，这时需要将混淆反解。
在 <sdk-root>/tools/proguard/bin 路径下有附带的的反解工具（Window 系统为 proguardgui.bat，Mac 或 Linux 系统为 proguardgui.sh）。
双击运行 proguardgui.bat 后，可以看到左侧的一行菜单。点击 ReTrace，选择该混淆包对应的 mapping 文件（混淆后在<module-name>/build/outputs/mapping/release/ 路径下会生成 mapping.txt 文件，它的作用是提供混淆前后类、方法、类成员等的对照表），再将 crash 的 stack trace 黏贴进输入框中，点击右下角的 ReTrace ，混淆后的堆栈信息就显示出来了。如下图：

![proguardgui](http://upload-images.jianshu.io/upload_images/1159224-f14b51bcc783f1ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
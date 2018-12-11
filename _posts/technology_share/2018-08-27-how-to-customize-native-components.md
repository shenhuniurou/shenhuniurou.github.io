---
layout: post
title: ReactNative如何封装原生组件并发布到npm
category: blog
tags: ReactNative、native、npm
---



在使用 ReactNative 开发过程中，由于 ReactNative 自带组件有限，有些效果使用自带组件无法实现，这时就需要使用 Android、iOS 各自平台上的原生组件来实现了。


## 封装原生组件

- 首先要创建一个 ReactNative 工程：

```
react-native init ReactNativeScrollRuler --verbose --version 0.51.0
```

- 在这个 ReactNative 工程中的 node_modules 目录中新建一个目录 `react-native-scroll-ruler`，注意，这个目录即为你到时候发布到 npmjs 时的库名，一般还是以 `react-native` 开头。然后在这个目录下建 `android` 和 `ios` 两个文件夹。


### iOS平台

- iOS 原生组件是以静态库形式提供给 js 调用的，所以需要创建一个 iOS 静态库。使用 Xcode 创建静态库，命名为 `RCTScrollRuler`，选择新创建项目时，选择 `Cocoa Touch Static Library`:

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_1.png)


- 接着在 `Build Settings` 中搜索 `Header Search Paths`，添加一行Header Search Paths，值为$(SRCROOT)/../../react-native/React，并设置为recursive。


![](http://offfjcibp.bkt.clouddn.com/Snip20180831_3.png)

这一步的目的是设置头文件的路径，因为当我们封装原生组件时会使用到一些 React 的头文件，所以需要在这里指定头文件的路径，`$(SRCROOT)`实际上就是指向.xcodeproj所在的路径。


- 然后，将静态库中的文件（包括 RCTScrollRuler.xcodeproj 和 RCTScrollRuler 文件夹）全部拷贝到上面一步创建的 `ios` 目录中。


- 开始编写原生模块代码

写原生模块代码之前需要先了解这些：

因为原生视图都需要被一个 RCTViewManager 的子类来创建和管理，但它们本质上都是单例，ReactNative 只会为每个管理器创建一个实例。它们创建原生的视图并提供给 RCTUIManager，RCTUIManagr 则会反过来委托它们在需要的时候去设置和更新视图的属性。RCTViewManager 还会代理视图的所有委托，并给 JavaScript 发回对应的事件。所以每一个原生视图都必须对应一个 RCTViewManager。

创建原生视图的步骤：

1、创建一个子类，命名规范为“视图名称+Manager”. 视图名称可以加上自己的前缀，比如我创建的视图为 RCTScrollRuler，那么视图管理器则命名为 RCTScrollRulerManager

2、添加 RCT_EXPORT_MODULE()标记宏 —— 让模块接口暴露给 JavaScript，这个宏括号内不写模块名则默认为视图名。

3、使用 RCT_EXPORT_VIEW_PROPERTY 宏来映射和导出组件属性：RCT_EXPORT_VIEW_PROPERTY(minValue, int);

> React Native 用 RCTConvert 来在 JavaScript 和原生代码之间完成类型转换。
支持的默认转换类型(部分)如下：
string (NSString)、number (NSInteger, float, double, CGFloat, NSNumber)、boolean (BOOL, NSNumber)、array (NSArray) 、map (NSDictionary)

4、实现-(UIView *)view方法 —— 创建并返回组件视图

5、封装属性及传递事件
与组件交互的事件可以通过 RCTBubblingEventBlock 来传递。
1) 在封装的原生视图中添加 RCTBubblingEventBlock 的 block 属性（on开头才有效）

```
@property (nonatomic, copy) RCTBubblingEventBlock onSelect;
```

2) 在 Manager 类中通过宏 RCT_EXPORT_VIEW_PROPERTY 完成 Block 属性的映射和导出

```
RCT_EXPORT_VIEW_PROPERTY(onSelect, RCTBubblingEventBlock)
```

3) 实现事件调用

```
rulerView.onSelect(@{@"value": @((int)value)});
```

> 记住这里的 key 是“value”，到js中调用时再讲这个 value。


### Android平台

- 新建一个 Android Library

- 在 build.gradle 中添加依赖 react-native:+

- 创建视图管理器，和 iOS 一样，Android 平台也需要一个 Manager，不过它是 SimpleViewManager 的子类，还可以是 ViewGroupManager 的子类，在管理器中需要返回模块名称和一个视图实例

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_4.png)

- 建立 Package，实现 ReactPackage 接口，并且在 createViewManagers 方法中添加自己的 Module

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_5.png)

- 使用 `ReactProp` 注解声明要封装的属性，并实现 setter 方法

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_6.png)

- 事件传递
在 Android平台上原生模块是通过 RCTEventEmitter 来发送的。

```
WritableMap event = Arguments.createMap();
event.putString("value", result);
reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(ruler.getId(), "topSelect", event);
```

`topSelect`对应着在js端调用的方法名为 `onSelect`，这个对应关系在 com.facebook.react.uimanager.UIManagerModuleConstants.java文件中，如图

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_7.png)

- 将该 Android Library 目录下的所有文件拷贝到之前创建的 android 目录下


### 在js端调用

还是在/node_modules/react-native-scroll-ruler/目录下创建这几个文件，index.js、index.ios.js、index.android.js，一般来说，如果封装的属性和传递事件的方法都相同的话，那么我们在js端调用 NativeModule 的代码都是相同的，那么就只需要在 index.js 文件中将 NativeModule 导出即可供js调用了，如果两个平台实现的方法不一样，那么就需要用两个文件分别导出，然后在index.js中根据 Platform 来区分使用哪个平台的 NativeModule。

![](http://offfjcibp.bkt.clouddn.com/Snip20180831_8.png)


在js中声明组件，包括声明属性方法，属性默认值，把通过 NativeModule 获取到的原生组件封装起来，然后导出。

```
import React, {Component} from 'react';
import PropTypes from 'prop-types';
import {requireNativeComponent, View, ViewPropTypes} from 'react-native';

type PropsType = {
    minValue?: number;
    maxValue?: number;
    defaultValue?: number;
    step?: number;
    num?: number;
    unit?: string;
} & typeof
(View);

export default class ReactScrollRuler extends Component {
    static propTypes = {
        minValue: PropTypes.number.isRequired,
        maxValue: PropTypes.number.isRequired,
        defaultValue: PropTypes.number.isRequired,
        step: PropTypes.number.isRequired,
        num: PropTypes.number.isRequired,
        unit: PropTypes.string,
        onSelect: PropTypes.func,
        ...ViewPropTypes,
    };
    props: PropsType;
    rulerRef: any;

    setNativeProps(props: PropsType) {
        this.rulerRef.setNativeProps(props);
    }

    _onSelect = (event) => {
        if (!this.props.onSelect) {
            return;
        }
        this.props.onSelect(event.nativeEvent.value);
    }

    render() {
        const {
            minValue,
            maxValue,
            defaultValue,
            step,
            num,
            unit,
            onSelect,
            ...otherProps
        } = this.props;

        return (
            <RNScrollRuler
                ref={(component) => {
                    this.rulerRef = component;
                }}
                minValue={minValue}
                maxValue={maxValue}
                defaultValue={defaultValue}
                step={step}
                num={num}
                unit={unit}
                onSelect={this._onSelect}
                {...otherProps}
            />
        );
    }
}

const RNScrollRuler = requireNativeComponent('RNScrollRuler', ReactScrollRuler, {
    nativeOnly: {onSelect: true}
});
```

- 添加README

这个文件需要告诉别人我们这个原生组件是干什么的、如何安装、API、使用手册等等。




## 发布组件到npmjs

- 将组件上传到 github 仓库

- 使用npm init命令来创建package.json（在刚开始创建的项目里面的node_modules/react-native-scroll-ruler/目录下），系统会提示我们输入所需的信息，不想输入的直接按回车跳过。

package.json中的内容是关于这个库的配置信息：
name-组件名
version-当前版本号，如果以后有更新，只需要改这个version，然后重新发布即可。
description-库的描述
main-库的入口文件
repository-github仓库地址

```
{
  "name": "react-native-scroll-ruler",
  "version": "1.1.9",
  "description": "ReactNative版选择身高体重的横向刻度尺组件，兼容Android和iOS",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "react-native",
    "react-native-scroll-ruler"
  ],
  "author": "shenhuniurou",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": 
  },
}

```

- 注册 npmjs 账号，https://www.npmjs.com/

- npm add user 添加刚注册的账号密码，验证通过后会把authToken存储在~/.npmrc文件中

- npm publish --registry http://registry.npmjs.org  发布
如果发布不成功则改成npm publish







































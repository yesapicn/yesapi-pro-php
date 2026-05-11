# 接口大师开发者App源码说明（需要单独购买）

> 接口大师开发者App源代码，如有需要，请单独购买。    

## 概述
接口大师专属客户端，采用Flutter跨平台开发框架实现，同时支持Android/IOS两端。

参考[《Flutter开发文档》](https://flutter.cn/docs)

## 搭建开发环境
推荐使用Flutter官方推荐的Android Studio进行开发。

#### 1.安装Android Studio
	
	下载地址：[https://developer.android.com/studio/](https://developer.android.com/studio/)

#### 2.安装flutter插件

点击Android Studio菜单，进入属性界面：  
![安装flutter插件](./img/img2.png)

选择【Plugins】选项，搜索Flutter，安装Flutter插件和国际化插件：  
![安装flutter插件](./img/img1.png)

安装完成后需要重启Android Studio才能启用插件。

#### 3.加载工程

1. 用Android Studio选择工程根目录打开。打开工程根目录下的 pubspec.yaml 文件

2. 点击【Pub get】下载项目相关的依赖库。  

![安装flutter插件](./img/img3.png)

3. 安装完毕即可以连接真机或者模拟器运行了。

## 修改应用信息

### Android端修改方法

1. 修改应用名：直接AndroidManifest.xml文件下修改：  
![安装flutter插件](./img/img4.png)

2. 修改包名：打开android/app/build.gradle 文件，找到applicationId，设置相应的包名。  

![安装flutter插件](./img/img6.png)

3. Android App图标存放位置：

`.../android/app/src/main/res/`


### IOS端修改方法

1. 修改包名、应用名：
	
	IOS端需要在Mac系统下，使用xcode打开`/ios/Runner.xcworkspace`工程文件，打开以下选项中进行修改：

![安装flutter插件](./img/img5.png)

2. IOS App图标存放位置：

`.../ios/Runner/Assets.xcassets/`

## 国际化
App国际化多语音采用 intl v0.16.1 自动化插件实现，在以下文件分别定义多语音字段：

```
// 中文配置
lib/l10n/intl_zh.arb

// 英文配置
lib/l10n/intl_en.arb
```
编辑完成，保存后会自动生成代码。然后便可以在代码中引用相应的字段了。
引用方法如下：

```
'${S.of(context).home_1}'	//home_1为多语音字段
```

注意：同一个字段需要在中英文配置文件中都有定义才会生效。

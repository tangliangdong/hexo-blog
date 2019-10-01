---
title: phonegap 使用方法
date: 2017-07-12 11:49:13
toc: true
category: Learn
tags:
    - phonegap
---

> 提醒广大的Mac党，如果执行命令的时候 遇到问题，在每个命令的最前面加上 `sudo`，以管理员权限运行可以解决很多因权限不够产生的问题。

<!--more-->

## 一、准备

开始之前，有几个基本软件需要提前安装：

 - Node.js – JavaScript 运行时工具，用于编译 JavaScript 代码
 - git – PhoneGap CLI 使用它在后台下载需要的资源。它已经预装在某些系统里面，你可以输入“git”命令查看是否已经被安装。

## 二、安装 PhoneGap CLI

`npm install -g phonegap`

## 三、创建 PhoneGap 项目

PhoneGap CLI 为初学者准备了一个默认的 Hello World 项目。通过这个项目，可以简单快捷的理解怎么构建一个移动 PhoneGap 应用。让我们使用 CLI 来创建一个默认项目。

1、在命令窗口执行以下命令：

```
phonegap create myApp
```

2、验证命令的执行结果

如果项目创建成功会显示下列信息：

```
Creating a new cordova project.
```

3、进入新建的项目目录

```
cd myApp/
```

4、确认目录下包含以下文件和子目录

```
hooks
platforms
plugins
www
config.xml  
```

5、进入 www 目录可以一些文件和子目录，它们就是 app 的内容，其中 index.html 文件是访问入口。后面的开发工作主要就在这个 www 目录下进行。

```
cd www/
```

## 四、启动预览服务

PhoneGap CLI 有一个 serve 命令可以启动一个小型的 web 服务器，用于在桌面浏览器和移动设备中预览项目。

进入项目根目录，执行 $ phonegap serve 命令，你将得到用于预览的服务器地址（如 192.168.1.11:3000）：

```
phonegap serve
[phonegap] starting app server...
[phonegap] listening on 192.168.191.1:3000
[phonegap] listening on 10.0.151.244:3000
[phonegap] listening on 192.168.0.110:3000
[phonegap]
[phonegap] ctrl-c to stop the server
[phonegap]
```

## 六、编译、打包和发布产品

运行ios `phonegap run ios`

运行android `phonegap run android`

打包ios `phonegap build ios`

打包apk `phonegap build android`

如何找到apk

```
MyApp > platforms > android > build > outputs > apk > android-debug.apk
```

![android apk路径](2.png)

如果要修改app的名字之类的信息，需要在根目录下的 `config.xml`

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="hdu.edu.smartAPP" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:gap="http://phonegap.com/ns/1.0">
    <!-- app的名字 -->
    <name>SmartAPP</name>
    <description>
        空气净化器智能APP
    </description>
    <author email="xiaotang188@gmail.com" href="http://zhizhi.tangliangdong.me">
        PhoneGap Team
    </author>
    <content src="index.html" />
    <preference name="DisallowOverscroll" value="true" />
    <preference name="android-minSdkVersion" value="19" />
    <!-- android app的logo 和 启动页面 -->
    <platform name="android">
        <icon density="ldpi" src="www/res/icon/android/drawable-ldpi-icon.png" />
        <icon density="mdpi" src="www/res/icon/android/drawable-mdpi-icon.png" />
        <icon density="hdpi" src="www/res/icon/android/drawable-hdpi-icon.png" />
        <icon density="xhdpi" src="www/res/icon/android/drawable-xhdpi-icon.png" />
        <icon density="xxhdpi" src="www/res/icon/android/drawable-xxhdpi-icon.png" />
        <icon density="xxxhdpi" src="www/res/icon/android/drawable-xxxhdpi-icon.png" />
        <splash density="land-ldpi" src="www/res/screen/android/drawable-land-ldpi-screen.png" />
        <splash density="land-mdpi" src="www/res/screen/android/drawable-land-mdpi-screen.png" />
        <splash density="land-hdpi" src="www/res/screen/android/drawable-land-hdpi-screen.png" />
        <splash density="port-ldpi" src="www/res/screen/android/drawable-port-ldpi-screen.png" />
        <splash density="port-mdpi" src="www/res/screen/android/drawable-port-mdpi-screen.png" />
        <splash density="port-hdpi" src="www/res/screen/android/drawable-port-hdpi-screen.png" />
        <splash density="port-xhdpi" src="www/res/screen/android/drawable-port-xhdpi-screen.png" />
        <splash density="port-xxhdpi" src="www/res/screen/android/drawable-port-xxhdpi-screen.png" />
    </platform>
    <!-- ios app的logo 和 启动页面  -->
    <platform name="ios">
        <icon height="57" platform="ios" src="www/res/icon/ios/icon.png" width="57" />
        <icon height="114" platform="ios" src="www/res/icon/ios/icon@2x.png" width="114" />
        <icon height="40" platform="ios" src="www/res/icon/ios/icon-40.png" width="40" />
        <icon height="80" platform="ios" src="www/res/icon/ios/icon-40@2x.png" width="80" />
        <icon height="50" platform="ios" src="www/res/icon/ios/icon-50.png" width="50" />
        <icon height="100" platform="ios" src="www/res/icon/ios/icon-50@2x.png" width="100" />
        <icon height="60" platform="ios" src="www/res/icon/ios/icon-60.png" width="60" />
        <icon height="120" platform="ios" src="www/res/icon/ios/icon-60@2x.png" width="120" />
        <icon height="180" platform="ios" src="www/res/icon/ios/icon-60@3x.png" width="180" />
        <icon height="72" platform="ios" src="www/res/icon/ios/icon-72.png" width="72" />
        <icon height="144" platform="ios" src="www/res/icon/ios/icon-72@2x.png" width="144" />
        <icon height="76" platform="ios" src="www/res/icon/ios/icon-76.png" width="76" />
        <icon height="152" platform="ios" src="www/res/icon/ios/icon-76@2x.png" width="152" />
        <icon height="29" platform="ios" src="www/res/icon/ios/icon-small.png" width="29" />
        <icon height="58" platform="ios" src="www/res/icon/ios/icon-small@2x.png" width="58" />
        <icon height="87" platform="ios" src="www/res/icon/ios/icon-small@3x.png" width="87" />
        <splash height="1136" platform="ios" src="www/res/screen/ios/Default-568h@2x~iphone.png" width="640" />
        <splash height="1334" platform="ios" src="www/res/screen/ios/Default-667h.png" width="750" />
        <splash height="2208" platform="ios" src="www/res/screen/ios/Default-736h.png" width="1242" />
        <splash height="768" platform="ios" src="www/res/screen/ios/Default-Landscape~ipad.png" width="1024" />
        <splash height="1024" platform="ios" src="www/res/screen/ios/Default-Portrait~ipad.png" width="768" />
        <splash height="960" platform="ios" src="www/res/screen/ios/Default@2x~iphone.png" width="640" />
        <splash height="480" platform="ios" src="www/res/screen/ios/Default~iphone.png" width="320" />
    </platform>
    <!-- wp8 其实不用管  -->
    <platform name="wp8">
        <icon height="99" platform="wp8" src="www/res/icon/wp8/ApplicationIcon.png" width="99" />
        <icon height="159" platform="wp8" src="www/res/icon/wp8/Background.png" width="159" />
        <splash height="1280" platform="wp8" src="www/res/screen/wp8/screen-portrait.jpg" width="768" />
    </platform>
    <!-- 这个也是不用管 -->
    <platform name="windows">
        <icon height="150" platform="windows" src="www/res/icon/windows/Square150x150Logo.scale-100.png" width="150" />
        <icon height="30" platform="windows" src="www/res/icon/windows/Square30x30Logo.scale-100.png" width="30" />
        <icon height="50" platform="windows" src="www/res/icon/windows/StoreLogo.scale-100.png" width="50" />
        <splash height="300" platform="windows" src="www/res/screen/windows/SplashScreen.scale-100.png" width="620" />
        <icon height="120" platform="windows" src="www/res/icon/windows/StoreLogo.scale-240.png" width="120" />
        <icon height="44" platform="windows" src="www/res/icon/windows/Square44x44Logo.scale-100.png" width="44" />
        <icon height="106" platform="windows" src="www/res/icon/windows/Square44x44Logo.scale-240.png" width="106" />
        <icon height="70" platform="windows" src="www/res/icon/windows/Square70x70Logo.scale-100.png" width="70" />
        <icon height="71" platform="windows" src="www/res/icon/windows/Square71x71Logo.scale-100.png" width="71" />
        <icon height="170" platform="windows" src="www/res/icon/windows/Square71x71Logo.scale-240.png" width="170" />
        <icon height="360" platform="windows" src="www/res/icon/windows/Square150x150Logo.scale-240.png" width="360" />
        <icon height="310" platform="windows" src="www/res/icon/windows/Square310x310Logo.scale-100.png" width="310" />
        <icon height="150" platform="windows" src="www/res/icon/windows/Wide310x150Logo.scale-100.png" width="310" />
        <icon height="360" platform="windows" src="www/res/icon/windows/Wide310x150Logo.scale-240.png" width="744" />
        <splash height="1920" platform="windows" src="www/res/screen/windows/SplashScreenPhone.scale-240.png" width="1152" />
    </platform>
    <access origin="*" />
    <allow-intent href="http://*/*" />
    <allow-intent href="https://*/*" />
    <allow-intent href="tel:*" />
    <allow-intent href="sms:*" />
    <allow-intent href="mailto:*" />
    <allow-intent href="geo:*" />
    <platform name="android">
        <allow-intent href="market:*" />
    </platform>
    <platform name="ios">
        <allow-intent href="itms:*" />
        <allow-intent href="itms-apps:*" />
    </platform>
    <engine name="ios" spec="~4.0.0" />
    <plugin name="cordova-plugin-battery-status" spec="~1.1.1" />
    <plugin name="cordova-plugin-camera" spec="~2.1.1" />
    <plugin name="cordova-plugin-media-capture" spec="~1.2.0" />
    <plugin name="cordova-plugin-console" spec="~1.0.2" />
    <plugin name="cordova-plugin-contacts" spec="~2.0.1" />
    <plugin name="cordova-plugin-device" spec="~1.1.1" />
    <plugin name="cordova-plugin-device-motion" spec="~1.2.0" />
    <plugin name="cordova-plugin-device-orientation" spec="~1.0.2" />
    <plugin name="cordova-plugin-dialogs" spec="~1.2.0" />
    <plugin name="cordova-plugin-file-transfer" spec="~1.5.0" />
    <plugin name="cordova-plugin-geolocation" spec="~2.1.0" />
    <plugin name="cordova-plugin-globalization" spec="~1.0.3" />
    <plugin name="cordova-plugin-inappbrowser" spec="~1.3.0" />
    <plugin name="cordova-plugin-media" spec="~2.2.0" />
    <plugin name="cordova-plugin-network-information" spec="~1.2.0" />
    <plugin name="cordova-plugin-splashscreen" spec="~3.2.1" />
    <plugin name="cordova-plugin-statusbar" spec="~2.1.2" />
    <plugin name="cordova-plugin-vibration" spec="~2.1.0" />
    <plugin name="cordova-plugin-whitelist" spec="~1.2.1" />
    <engine name="android" spec="~6.1.2" />
</widget>
```

我们还需要设置app的图标和启动页，不可能用默认丑丑的图标和启动页，我们就得设置自己的一套logo和app启动页，

![app图标和启动页的路径](1.png)

在 `www/res`路径下应该有两个文件夹：`icon`(app图标)、`screen`（app启动页）  **没有就新建**

其下应该还有 `ios`、`android` 文件夹，就把之前根目录里提到的 app图标和启动页，比如这个

```
<icon height="57" platform="ios" src="www/res/icon/ios/icon.png" width="57" />
```

路径是 `www/res/icon/ios/` 且是命名为 `icon.png`的图片，依次类推，

如果你要打包`android app` 那么，你就要保证在 *config.xml* 里被`<platform name="android"></platform>`包裹的所有`<icon></icon>`和`<splash></splash>`的标签在相应的路径下 都能找到对应的图标或启动页，

不然的话 执行 `phonegap build android` 或 `phonegap build ios` 是会报错的，打包失败。

> `config.xml` 里的 `<icon></icon>`对应app图标，`<splash></splash>` 对应app启动页



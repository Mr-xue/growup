# React Native手记

## 一、项目配置

> 开发环境搭建：[参考此处](https://www.react-native.cn/docs/environment-setup)

### 1.安装react-navigation 4.x

#### 1-1.路由的安装配置

- 安装核心组件及用到的组件库

  ```sh
  //安装主库
  $ yarn add react-navigation
  
  //安装核心库
  $ yarn add react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view
  
  //根据需要引入各导航组件库
  $ yarn add react-navigation-stack
  $ yarn add react-navigation-drawer
  $ yarn add react-navigation-tabs
  ```

  > React Native 0.60以上会自动link包，反之需要手动link

- ios目录下执行 `pod install`

- 为react-native-screens添加相关依赖

  - 为了在Android上完成安装，还需要在`android/app/build.gradle`中为`react-native-screens`添加相关依赖

    ```java
    implementation 'androidx.appcompat:appcompat:1.1.0-rc01'
    implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0-alpha02'
    ```

    

- 配置react-native-gesture-handler

  为了在Android上能够使用react-native-gesture-handler，需要修改MainActivity.java(加号部分为新增代码)

  ```java
  package com.reactnavigation.example;
  
  import com.facebook.react.ReactActivity;
  + import com.facebook.react.ReactActivityDelegate;
  + import com.facebook.react.ReactRootView;
  + import com.swmansion.gesturehandler.react.RNGestureHandlerEnabledRootView;
  public class MainActivity extends ReactActivity {
  
    @Override
    protected String getMainComponentName() {
      return "Example";
    }
  +  @Override
  +  protected ReactActivityDelegate createReactActivityDelegate() {
  +    return new ReactActivityDelegate(this, getMainComponentName()) {
  +      @Override
  +      protected ReactRootView createRootView() {
  +        return new RNGestureHandlerEnabledRootView(MainActivity.this);
  +      }
  +    };
  +  }
  }
  ```

  

- 在`index.js`中导入`react-native-gesture-handler`



#### 1-2.获取路由参数的两种方式

- `this.props.navigation.state.params`获取的是整个参数对象
- `this.props.navigation.getParam('参数名')`通过key获取参数

### 2.配置redux

- 安装redux：`yarn add redux react-redux redux-thunk`

- 使用以下代码覆盖App.js，AppNavigators存放的是路由配置代码

  ```javascript
  import React, {Component} from 'react';
  import {Provider} from 'react-redux';
  import AppNavigators from './navigator/AppNavigators';
  import store from './store';
  
  class App extends Component {
    render() {
      /**
       * 将store传递给App框架
       */
      return (
        <Provider store={store}>
          <AppNavigators />
        </Provider>
      );
    }
  }
  
  export default App;
  ```

  

### 3.配置icon图标库

- 安装 `yarn add react-native-vector-icons`

- 配置ios

  ios目录下的info.plist添加以下代码

  ```xml
  <key>UIAppFonts</key>
  <array>
    <string>AntDesign.ttf</string>
    <string>Entypo.ttf</string>
    <string>EvilIcons.ttf</string>
    <string>Feather.ttf</string>
    <string>FontAwesome.ttf</string>
    <string>FontAwesome5_Brands.ttf</string>
    <string>FontAwesome5_Regular.ttf</string>
    <string>FontAwesome5_Solid.ttf</string>
    <string>Foundation.ttf</string>
    <string>Ionicons.ttf</string>
    <string>MaterialIcons.ttf</string>
    <string>MaterialCommunityIcons.ttf</string>
    <string>SimpleLineIcons.ttf</string>
    <string>Octicons.ttf</string>
    <string>Zocial.ttf</string>
    <string>Fontisto.ttf</string>
  </array>
  ```

  

- ios目录下执行pod install

  

- 配置android

  编辑 `android/app/build.gradle`文件，增加以下代码

  ```java
  apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
  ```

  

### 4.本地存储AsyncStorage

原官方api已废弃，建议使用以下扩展

AsyncStorage：`yarn add @react-native-async-storage/async-storage`



### 5.配置UI库

项目中使用的UI库为 React Native Elements [文档地址](https://reactnativeelements.com/docs/)



### 6.配置极光社交分享

[官方配置文档](https://github.com/jpush/jshare-react-native#manually-configure-part)

- 安卓以下报错解决方案

  ```java
  .../android/app/build/generated/rncli/src/main/java/com/facebook/react/PackageList.java:79: error: constructor JSharePackage in class JSharePackage cannot be applied to given types;
        new JSharePackage(),
        ^
    required: boolean,boolean
    found: no arguments
    reason: actual and formal argument lists differ in length
  1 error
  ```

  Fix方案

  在`node_modules/jshare-react-native/android/src/main/java/..../JsharePackage.java`文件中添加如下代码

  ```java
  public JSharePackage() {
  	Logger.SHUTDOWNTOAST = false;
  	Logger.SHUTDOWNLOG = false;
  }
  ```

  同时不需要在 `项目目录/android/app/src…/MainApplication.java`文件中添加 new JSharePackage(xx,xx)



- 安卓配置分享平台id和key

  `项目目录/android/app/assets/JGShareSDK.xml`



## 二、Code Push热更新

> App Center CLI [官方文档](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/cli)
>
> React NativeCLient SDK [官方文档](https://docs.microsoft.com/en-us/appcenter/distribution/codepush/rn-get-started)
>
> [参考整理](https://www.jianshu.com/p/84307d3aa824)  [参考整理2](https://blog.csdn.net/qq_42076140/article/details/90408047)
>
> 目前https://codepush.appcenter.ms/服务被墙，可能导致热更新失败

### 1.项目配置

#### 1-1.服务端配置

- 安装`App Center CLI`，用于服务端信息管理

  ```sh
  $ sudo yarn global add appcenter-cli
  ```

  

- 登陆app center

  ```sh
  $ appcenter login
  ```

  

- 运行以上命令并在命令行确认后，网页会弹出一个要求登陆的页面，登陆后，会得到一串`Access code`，复制粘贴回命令行，成功的话会返回登陆账号。

  ```sh
  $ appcenter login
  Opening your browser... 
  ? [Visit]:https://appcenter.ms/cli-login?hostname=assetfundeMacBook-Pro.local and enter the code:
  ? Access code from browser:  0cd185da****36a****7295b3****c8da9ba766a
  Logged in as xuechongwei-hotmail.com（登录账号）
  ```

  

- 添加`App`信息，这里要分别添加`Android`与`iOS(i小写)`，`app`名字是HbAppBaby,以此为例

  ```sh
  // -d 后面接的是app显示的名字，为了区分不同平台后面也写上平台命
  // -o 表示运行系统（operation） Android/iOS
  // -p 表示平台（Platform）这里是 react-native
  $ appcenter apps create -d HbAppBaby-android -o Android -p React-Native
  $ appcenter apps create -d HbAppBaby-ios -o iOS -p React-Native
  ```

  

- 接下来运行一下`appcenter apps list`检测是否添加成功

  ```sh
  $ appcenter apps list
  xuechongwei-hotmail.com/HbAppBaby-android
  xuechongwei-hotmail.com/HbAppBaby-ios
  ```

  

- 将已添加的`app`部署热更新服务，一般会部署两个用于灰度更新，和正式更新，这里分别叫做`Staging`与`Production`。分别给安卓和iOS部署，所以一共要运行四行命令

  ```sh
  // -a 是指应用（application），这里要写上“用户名和程序名”
  
  // 部署IOS
  $ appcenter codepush deployment add -a xuechongwei-hotmail.com/HbAppBaby-ios Staging
  $ appcenter codepush deployment add -a xuechongwei-hotmail.com/HbAppBaby-ios Production
  // 部署安卓
  $ appcenter codepush deployment add -a xuechongwei-hotmail.com/HbAppBaby-android Staging
  $ appcenter codepush deployment add -a xuechongwei-hotmail.com/HbAppBaby-android Production
  ```



#### 1-2.IOS、安卓端配置

安装依赖包：`yarn add react-native-code-push`



**ios配置方法**

------

- 切换到ios目录下执行`pod install`

- 打开`AppDelegate.m`文件，添加以下代码

  ```objective-c
  #import <CodePush/CodePush.h>
  ```

  

- 找到以下代码，进行替换

  ```objective-c
  return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
  //替换成
  return [CodePush bundleURL];
  ```

  

- 修改 `/项目目录/ios/项目名称/Info.plist` 文件,`CodePushDeploymentKey` 标签的值。（如果没有的话则新增）

  ```xml
  <key>CodePushDeploymentKey</key>
  <string>Rbw0ct6cIQS******sKb0laMQNMVju</string>
  ```

  

- 修改版本号versionName（设置成1.0.0）

  ![](https://gitee.com/thelife/pic-oss/raw/master/pic/2021-01-18-7450593-b5f2026cbb424a8f.png)



**android配置方法**

------



- 在`android/settings.gradle`文件中，添加以下代码

  ```java
  include ':app', ':react-native-code-push'
  project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
  ```

  

- 在 `android/app/build.gradle` 文件中，添加以下代码,

  ```java
  ...
  apply from: "../../node_modules/react-native/react.gradle"
  apply from: "../../node_modules/react-native-code-push/android/codepush.gradle" -->新增
  ...
  ```

  

- 修改`MainApplication.java`

  ```java
  ...
  // 1. 导入codepush插件类
  import com.microsoft.codepush.react.CodePush;
  
  public class MainApplication extends Application implements ReactApplication {
      private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
          ...
          // 2. 新增这段代码
          @Override
          protected String getJSBundleFile() {
              return CodePush.getJSBundleFile();
          }
      };
  }
  ```



- 在 `strings.xml`文件中新增部署的key

  ```xml
  <string moduleConfig="true" name="CodePushDeploymentKey">真实的key</string>
  
  <!-- 查看部署的key-->
  appcenter codepush deployment list -a <ownerName>/<appName> <deploymentName> -k
  例：appcenter codepush deployment list -a xuechongwei-hotmail.com/HbAppBaby-android -k
  ```



- 修改版本号versionName

  打开`android/app/build.gradle`,搜索 `defaultConfig`定位到以下代码，修改`versionName`为 1.0.0（默认是1.0，codepush需要三位数）

  ```java
  android{
      defaultConfig{
          versionName "1.0.0"
      }
  }
  ```

  

### 2.常用命令

旧版code push命令

------

```sh
//发布新版本：
$ code-push release-react GithubRN-IOS ios --t 1.0.0 --dev false --d Production --des "1.提交信息" --m true

//查看已发布的包信息：
$ code-push deployment ls GithubRN-Android

//可以看到指定版本更新的时间、描述等等属性：
$ code-push deployment history appName Staging/Production

//查看部署密钥：
$ code-push deployment ls [APP_NAME] -k
```



新版appcenter命令

------



```sh
// 查看创建app列表
$ appcenter apps list

// 查看部署key
$ appcenter codepush deployment list -a <ownerName>/<appName> <deploymentName> -k
例：appcenter codepush deployment list -a xuechongwei-hotmail.com/HbAppBaby-ios -k

// 发布新版本：在默认情况下，更新会推送到Staging的部署。
// 指定版本: -t 版本号  -d 要部署的环境
$ appcenter codepush release-react -a xuechongwei-hotmail.com/HbAppBaby-ios -d Production -t 1.0.0 --description '更新内容'

// 设置更新日志，供前端读取
$ appcenter codepush release-react -a xuechongwei-hotmail.com/HbAppBaby-ios  --description '更新'

// 强制更新
// 多了个-m true 参数，强制更新的默认效果是，用弹窗确认更新时候，只有确认键，并且安装成功后是立即生效，所以app可能会闪一下。
$ appcenter codepush release-react -a xuechongwei-hotmail.com/splashExample-ios -m true  --description '更新'

// 查看更新历史
$ appcenter codepush deployment history -a <ownerName>/<appName> <deploymentName>

// 显示历史
$ appcenter codepush deployment history -a xuechongwei-hotmail.com/HbAppBaby-ios Staging

// 清空历史
$ appcenter codepush deployment clear Staging -a xuechongwei-hotmail.com/HbAppBaby-ios
```



### 3.配置更新方式

```javascript

import {AppRegistry} from 'react-native';
import CodePush from "react-native-code-push";
// 静默方式，app每次启动的时候，都检测一下更新 'ON_APP_RESUME'
const codePushOptions = { checkFrequency: CodePush.CheckFrequency.ON_APP_RESUME };
 
// 手动方式接收更新的方式
//const codePushOptions = { checkFrequency: CodePush.CheckFrequency.ON_APP_RESUME };
import _App from './App';
const App = CodePush(codePushOptions)(_App);
import {name as appName} from './app.json';

```

### 4.部署code-push-server

微软云服务在中国太慢，并且部分服务被墙，导致无法正常热更新，建议自部署更新服务器。[参考文档](https://github.com/lisong/code-push-server)





## 三、技术点

### 1.判断ios全面屏

- 官方DeviceInfo即将废弃，目前可以使用:`DeviceInfo.isIPhoneX_deprecated`进行判断

- 后续可通过安装扩展:`react-native-device-info` 通过`DeviceInfo.hasNotch()`判断
- 用屏幕的长宽比判断 长度比宽度的值 大于1.8是全面屏

### 2.导航层级无法跳转问题

- 路由跳转提取到NavigatorUtil.js

  在HomePage主入口文件的render方法中加入 `NavigationUtil.navigation = this.props.navigation;`

  需调用对应页面的navigation

### 3.iOS边框圆角注意事项

请注意下列边框圆角样式目前在 iOS 的图片组件上还不支持：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

### 4.图片url需为静态字符串

为了使新的图片资源机制正常工作，require 中的图片名字必须是一个静态字符串（不能使用变量！因为 require 是在编译时期执行，而非运行时期执行！）。[参考地址](https://www.react-native.cn/docs/images)

```react
// 正确
<Image source={require("./my-icon.png")} />;

// 错误
const icon = this.props.active ? "my-icon-active" : "my-icon-inactive";
<Image source={require("./" + icon + ".png")} />;

// 正确
const icon = this.props.active
  ? require("./my-icon-active.png")
  : require("./my-icon-inactive.png");
<Image source={icon} />;
```

### 5.判断开发环境

通过全局变量`__DEV__`判断开发环境



### 6.打包安卓和ios安装包

打包安卓：[参考地址](https://blog.csdn.net/CC1991_/article/details/103285684)

打包iOS：[参考地址](https://www.devio.org/2020/03/15/React-Native-releases-packaged-iOS-apps-for-apps/)

- ios证书申请 [参考地址](https://www.jianshu.com/p/e72b3e91afca)

- ios打包注意事项

  通过Product -> Archive 进行归档打包，打包后通过 Window -> Organizer打开归档结果页进行导出



### 7.检测手机系统版本

```javascript
import { Platform } from "react-native";

//在 Android 上，Version属性是一个数字，表示 Android 的 api level：
if (Platform.Version === 25) {
  console.log("Running on Nougat!");
}

//在 iOS 上，Version属性是-[UIDevice systemVersion]的返回值，具体形式为一个表示当前系统版本的字符串。比如可能是"10.3"。
const majorVersionIOS = parseInt(Platform.Version, 10);
if (majorVersionIOS <= 9) {
  console.log("Work around a change in behavior");
}
```



### 8.屏幕适配

1.简单的单位转换

以750宽度的设计稿为例：

如果要让750代表RN设备的宽度，可以这样设计你的代码：

```react
const viewPortWidth = 750;//这里是设计稿的宽度
const px2dp = (px: number): number => {
    return px * Dimensions.get('window').width / viewPortWidth
}
 
<View style={{height:200,width:px2dp(750),backgroundColor:'yellow'}}>
	<Text>750</Text>
</View>
```



2.第三方库 [参考地址1](https://bingoootang.github.io/blog/2018/08/31/react-native-adaptive/)  [参考地址2](https://juejin.cn/post/6844903662188396551)

建议使用以下库： [Github地址](https://github.com/bingoootang/react-native-adaptive-stylesheet#readme)

```react
//安装
yarn add react-native-adaptive-stylesheet

//引入
import StyleSheet from 'react-native-adaptive-stylesheet';
StyleSheet.setGuidelineBaseWidth(750); //设置基准尺寸

//全局配置
StyleSheet.configure({
  width: 375,
  scaleFont: true,
});

//在视图层直接使用
<View style={{ width: StyleSheet.scaleView(60) }}>
  <Text style={{ fontSize: StyleSheet.scaleFont(18) }}>This is am example!</Text>
</View>

//定义样式与原本用法相同
StyleSheet.create({
  container: {
    width: 375,
    borderWidth: StyleSheet.hairlineWidth,
    fontSize: 18,
  },
});
```



### 9.将安卓的路由切换效果设置为ios模式

[react-navigation4.X 参考文档](https://reactnavigation.org/docs/stack-navigator/#transitionpresets)

```react
import { TransitionPresets } from 'react-navigation-stack';
{
  //路由配置
  ...
},
{
    initialRouteName: 'Index',
    defaultNavigationOptions: {
      ...TransitionPresets.SlideFromRightIOS,
    },
}
```



### 10.获取元素位置尺寸

```react
<View ref={(filter) => (this.filter = filter)}></View>

this.filter && this.filter.measure((x, y, width, height, left, top) => {
  console.log('===x===', x); //组件相对父布局的X坐标??
  console.log('===y===', y); //组件相对父布局的Y坐标??
  console.log('===width===', width); //组件的宽度
  console.log('===height===', height); //组件的高度
  console.log('===left===', left); //组件在屏幕中的X坐标
  console.log('===top===', top); //组件在屏幕中的Y坐标
});
```



### 11.动画教程

[官方文档](https://www.react-native.cn/docs/animated)

[简书](https://www.jianshu.com/p/7fd37d8ed138)

## 四、常见问题

### 1.多个依赖安卓解析冲突

![](https://gitee.com/thelife/pic-oss/raw/master/pic/2021-01-07-1.png)



### 2.ios依赖解析错误

尝试切到ios目录下执行 pod install

### 3.模拟器错误弹窗不消失

有时候因代码错误导致模拟器错误弹窗，还原代码后，错误弹窗依旧存在。可尝试以下方法：

- 关闭所有终端窗口
- 项目根目录启动终端，运行 `yarn cache clean`,然后再启动项目

### 4.TextInput文字显示不全

安卓下存在的问题，添加样式 `paddingVertical: 0`

### 5.如何调试http请求

> 注意：使用 Chrome 调试目前无法观测到 React Native 中的网络请求，你可以使用第三方的[react-native-debugger](https://github.com/jhen0409/react-native-debugger)来进行观测。


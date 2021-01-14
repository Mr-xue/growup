# React Native手记

## 一、项目配置

> 开发环境搭建：[参考此处](https://www.react-native.cn/docs/environment-setup)

### 1.安装react-navigation 4.x

1.安装核心组件及用到的组件库

```javascript
//安装核心库
yarn add react-native-reanimated react-native-gesture-handler react-native-screens react-native-safe-area-context @react-native-community/masked-view

//根据需要引入各导航组件库
yarn add react-navigation-stack
yarn add react-navigation-drawer
yarn add react-navigation-tabs
```

> React Native 0.60以上会自动link包，反之需要手动link



2.ios目录下执行 `pod install`

3.为react-native-screens添加相关依赖

为了在Android上完成安装，还需要在`android/app/build.gradle`中为`react-native-screens`添加相关依赖

```java
implementation 'androidx.appcompat:appcompat:1.1.0-rc01'
implementation 'androidx.swiperefreshlayout:swiperefreshlayout:1.1.0-alpha02'
```

4.配置react-native-gesture-handler

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

5.在`index.js` or `App.js`中导入`react-native-gesture-handler`



### 2.配置redux

1.安装redux：`yarn add redux react-redux redux-thunk`

2.使用以下代码覆盖App.js，AppNavigators存放的是路由配置代码

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

1.安装 `yarn add react-native-vector-icons`

2.配置ios

- ios目录下的info.plist添加以下代码

  ```swift
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

  

3.配置android

- 编辑 `android/app/build.gradle`文件，增加以下代码

  ```java
  apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
  ```



### 4.本地存储AsyncStorage

AsyncStorage：`yarn add @react-native-async-storage/async-storage`



### 5.配置UI库

项目中使用的UI库为 React Native Elements [文档地址](https://reactnativeelements.com/docs/)



### 6.配置极光社交分享

[配置文档](https://github.com/jpush/jshare-react-native#manually-configure-part)

1.安卓以下报错解决方案

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

2.安卓配置分享平台id和key

`项目目录/android/app/assets/JGShareSDK.xml`



## 二、技术点

### 1.判断ios全面屏

`DeviceInfo.hasNotch()`

### 2.导航层级无法跳转问题

- 路由跳转提取到NavigatorUtil.js

  在HomePage主入口文件的render方法中加入 `NavigationUtil.navigation = this.props.navigation;`

  需调用对应页面的navigation

### 3.iOS边框圆角的注意事项

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



## 三、Code Push热更新相关命令

**发布新版本**

`code-push release-react GithubRN-IOS ios --t 1.0.0 --dev false --d Production --des "1.提交信息" --m true`



**查看已发布的包信息**

`code-push deployment ls GithubRN-Android`



**可以看到指定版本更新的时间、描述等等属性**

`code-push deployment history appName Staging/Production`



**查看部署密钥**

`code-push deployment ls [APP_NAME] -k`





## 四、常见问题

#### 1.安卓依赖项解析错误

![1](https://gitee.com/thelife/pic-oss/raw/master/pic/2021-01-07-1.png)



#### 2.ios依赖解析错误

尝试切到ios目录下执行 pod install
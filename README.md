# OCRN
原生OC 集成React Native 

#### 知识技能储备
1. 需要了解OC基础
1. ios pod三方库管理插件
1. 需要了解React React-Native组件
1. React组件运行方式
1. JavaScript开发基础
1. css开发基础
1. xcode的日常使用
1. ES2015、ES6、ES7语法
1. npm创建工程
1. Android日常开发基础(Android集成)
1. ...
#### 你能学到什么？
1. OC如何集成React-Native
1. javascript如何与底层OC通信 **(函数调用)**
1. OC如何传递数据给JavaScript **(数据传递)**

#### 集成步骤
1. 通过xcode创建一个原生的IOS项目
1. 关闭xcode,将xcode创建的原生项目对应的文件夹改成ios
1. vscode 编辑器打开ios 文件夹对应的上层文件夹
1. 在上层文件夹里面新建package.json文件,代码如下
    ```javascaript
      {
          "name": "test",
          "version": "0.0.1",
          "private": true,
          "scripts": {
            "start": "node node_modules/react-native/local-cli/cli.js start"
          }
        }
    ```
1. 安装react-native组件库 `yarn add react-native` 这样默认会安装最新版本的 React Native，同时会打印出类似下面的警告信息,这个表示我们也要安装指定版本的react.
    `warning " > react-native@0.58.4" has unmet peer dependency "react@16.6.3".`
1. 安装指定的react版本 `yarn add react@16.6.3`
1. 命令行切换到ios目录下初始化podfile。
    ```
      cd ios
      pod init
    ```
1. 打开生成的Podfile文件，在`do`和`end`之间粘贴如下代码。这个表示将`react native `三方库加入到ios工程
    ```
      # 'node_modules'目录一般位于根目录中
      # 但是如果你的结构不同，那你就要根据实际路径修改下面的`:path`
      pod 'React', :path => '../node_modules/react-native', :subspecs => [
        'Core',
        'CxxBridge', # 如果RN版本 >= 0.47则加入此行
        'DevSupport', # 如果RN版本 >= 0.43，则需要加入此行才能开启开发者菜单
        'RCTText',
        'RCTNetwork',
        'RCTWebSocket', # 调试功能需要此模块
        'RCTAnimation', # FlatList和原生动画功能需要此模块
        # 在这里继续添加你所需要的其他RN模块
      ]
      # 如果你的RN版本 >= 0.42.0，则加入下面这行
      pod 'yoga', :path => '../node_modules/react-native/ReactCommon/yoga'

      # 如果RN版本 >= 0.45则加入下面三个第三方编译依赖
      pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
      pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
      pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'
    ```
1. 运行`pod install `安装三方依赖库，静静等待安装完成即可
1. 切换项目的根目录下创建`index.js`文件.代码如下.用于展示React Native页面
    ```
      import React from 'react';
      import {AppRegistry, StyleSheet, Text, View} from 'react-native';
      class App extends React.Component{
          render(){
              return (
                  <View style={Styles.center}>
                      <Text style={Styles.text}>我是React Native的文本控件</Text>
                  </View>
              )
          }
      }
      const Styles = {
          center:{
              flex:1,
              alignItems:"center",
              justifyContent:"center",
          },
          text:{
              fontSize:40,
              color:"red"
          }
      }
      // 整体js模块的名称
      AppRegistry.registerComponent('Test', () => App);
    ```
1. 打开通过xcode打开ios文件夹下的项目，文件后缀名称为`*.xcworkspace`;
1. 在`Main.storyboard`文件中拖拽一个`Button`组件，并为其绑定`Touch Up Inside`事件
1. 在`ViewController.m`中引入`react-native`头文件，否则可能会报错
    ```
      #import <React/RCTRootView.h>
    ```
1. 在`Button`事件里面写下如下代码
    ```
       NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.bundle?platform=ios"]; 

    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL: jsCodeLocation
                                  moduleName: @"Test"
                           initialProperties:
                             @{
                               @"books" : @[
                                 @{
                                   @"name" : @"IOS从入门到放弃",
                                   @"price": @"40.0"
                                  },
                                 @{
                                   @"name" : @"Android从入门到放弃",
                                   @"price": @"50.0"
                                 }
                               ]
                             }
                               launchOptions: nil];
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view = rootView;
    [self presentViewController:vc animated:YES completion:nil];
    ```
1. `command+r`运行`ios`项目可能会报错,错误原因可能如下
    ```
    1. 没有启动react native packager server 
       解决：npm run start

    2. 没有添加 App Transport Security 例外
       解决：在info.plist添加 App Transport Security
        <key>NSAppTransportSecurity</key>
        <dict>
            <key>NSExceptionDomains</key>
            <dict>
                <key>localhost</key>
                <dict>
                    <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
                    <true/>
                </dict>
            </dict>
        </dict>
    ```
1. 重新`command+r`运行`ios`即可查看效果
1. 新建一个`Cocoa Touch Class`类，将其命名为`Cache`
    ```
    // Cache.h
    #import <UIKit/UIKit.h> // 此处添加UIKit 头文件
    NS_ASSUME_NONNULL_BEGIN
    @interface Cache : NSObject
    +(void) setVC:(UIViewController *)vc; // 新增方法
    +(UIViewController *) getVC ; // 新增方法
    @end
    NS_ASSUME_NONNULL_END

    //Cache.m
    #import "Cache.h"
    #import <UIKit/UIKit.h> // 此处添加UIKit 头文件
    static UIViewController *vc = nil ; // 定义静态变量用来存储ViewController
    @implementation Cache
    +(void) setVC:(UIViewController *)viewController{ // 实现方法
        vc = viewController ;
    }
    +(UIViewController *) getVC{ // 实现方法
        return  vc ;
    }
    @end

    ```

1. 新建一个`Cocoa Touch Class`类，将其命名为`ReactNativeManager`
    ```
      // ReactNativeManager.h
        #import <Foundation/Foundation.h>
        #import <React/RCTBridgeModule.h> //导入ReactNaitve 导出的头文件
        NS_ASSUME_NONNULL_BEGIN
        @interface ReactNativeManager :NSObject<RCTBridgeModule> // 继承react native module
        @end
        NS_ASSUME_NONNULL_END

        // ReactNativeManager.m
        #import "ReactNativeManager.h"
        #import "Cache.h" // 导入我们创建好的Cache 头文件
        @implementation ReactNativeManager 
        RCT_EXPORT_MODULE();// 将该模块导出

        RCT_EXPORT_METHOD(closeWindow){ //需要导出的方法
            UIViewController *vc = [Cache getVC] ;
            if(vc != nil){
                [vc dismissViewControllerAnimated:true completion:nil] ;
            }
        }
        @end
    ```
1. 修改`ViewController.m`如下
    ```
      1. 加入头文件
        #import "Cache.h"
      2. 在按钮的点击事件里面加入以下代码 
        [Cache setVC:vc];

    ```
1. 修改`index.js`
    ```
    import React from 'react';
    import {AppRegistry, StyleSheet, Text, View,NativeModules } from 'react-native';
    class App extends React.Component{
        onPress = ()=>{
            NativeModules.ReactNativeManager.closeWindow(); //此处调用IOS原生封装的方法
        }
        render(){
            return (
                <View style={Styles.center}>
                    <Text style={Styles.text}>我是React Native的文本控件</Text>
                    <Text onPress={this.onPress} style={Styles.text}>点我调用原生方法关闭当前视图</Text>
                </View>
            )
        }
    }
    const Styles = {
        center:{
            flex:1,
            alignItems:"center",
            justifyContent:"center",
        },
        text:{
            fontSize:40,
            color:"red"
        }
    }
    // 整体js模块的名称
    AppRegistry.registerComponent('Test', () => App);
    ```
1. 重新运行查看效果即可。

1. 推荐基于我们联客的UI规范实现一套SaaS React Web 组件[RUI](https://github.com/reapeatlink-front/rui)



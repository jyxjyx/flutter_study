# 目录
- [简介](#简介)
- [安装过程中可能出现的几个问题](#安装过程中可能出现的几个问题)
- [dart语法](#dart语法)
- [flutter基础知识](#flutter基础知识)
    - [Widget](#widget)
    - [BuildContext和Key](#buildcontext和key)
- [路由](#路由)
- [静态资源](#静态资源)
- [状态管理](#状态管理)
- [网络请求](#网络请求)
- [App打包](#app打包)
- [项目中碰到的坑](#项目中碰到的坑)


# 简介
本文从一个前端的角度，记录了个人学习的flutter过程，以快速搭建一个flutter项目为出发点，供快速入门，本文除了一些个人看法外，还会在文中推荐几篇我觉得不错的文章，以供学习。本文不会有对api和widget的详细介绍，想知道更多api信息还请移步flutter官网。这里先介绍下2个网站：
1. flutter中文网：[https://flutterchina.club/android-release/](https://flutterchina.club/android-release/)
2. flutter官网：[https://flutter-io.cn/docs/deployment/android](https://flutter-io.cn/docs/deployment/android)

推荐在安装时查看前者，深入学习时查看后者。目录中`安装过程中可能出现的几个问题`这一节，均为个人安装时出现过得问题，但不一定包含所有可能产生的问题。

PS：如果觉得本文对您有用，请给个star。本文仅供参考学习使用，文章中也会推荐一些个人觉得比较不错的博客，希望大家能看下来，毕竟学习最重要是的耐心。文档编写开始于2019/6/27。

# 安装过程中可能出现的几个问题
安装过程根据[flutter中文官网](https://flutterchina.club/get-started/install/)的步骤即可。在此谈一下几个注意点：
- 1.初次运行flutter doctor的时候应该还会有一个报错，根据命令行提示输入命令即可（PS：记不清了啥错误）
- 2.初次运行flutter run时，如遇到Initializing gradle...时间特别长的，是因为国内无法访问谷歌导致的。可通过挂代理解决，或者修改项目目录下的`/android/build.gradle`文件的`buildscript`中的`repositories`这一段代码：
```js
buildscript {
    repositories {
        // google()
        // jcenter()
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://maven.aliyun.com/repository/jcenter' }
        maven { url 'http://maven.aliyun.com/nexus/content/groups/public' }
    }


    ...
}
```

- 3.模拟器选择：可以使用`android stdio`自带的模拟器，或者第三方模拟器。
    - 3.1 如选择AS自带的模拟器，配置好AS自带的模拟器后，可以在VS code的右下角有个`No devices`，可以快速启动模拟器，不用进入AS启动。
    - 3.2 如选择第三方模拟器的，需要进行设备连接，这里我采用的是夜神模拟器:

    先打开夜神模拟器

    然后打开cmd进入夜神模拟器的安装目录下的bin目录
    ```
    D:\yeshen\Nox\bin
    ```
    然后运行(不同的模拟器可能命令会有不同)：
    ```
    nox_adb.exe connect 127.0.0.1:62001
    ```
    为了以后能够方便连接（为了偷懒），我们手写一个bat文件执行命令：
    ```
    cmd /c "D: & cd D:\yeshen\Nox\bin & nox_adb.exe connect 127.0.0.1:62001"
    ```
- 4.开发快捷键介绍：r 热更新；p 显示网格线；o 切换ios/android；q 
关闭flutter。在这里可能会觉得觉得每次热更新都要按r会很麻烦，遵循偷懒原则，我们只需在vscode中开启调试模拟，就不需要按r刷新了。
- 5.关闭项目时请结束调试模式，结束flutter命令，不然下次使用时可能会提示`Waiting for another flutter command to release the startup lock`。该问题解决方案：任务管理器中关闭所有的dart.exe进程，然后删除flutter的安装目录下的`/bin/cache/lockfile`文件。
- 6.vscode插件：除了官网提供的flutter插件和dart插件外，还推荐安装Awesome Flutter Snippets。详细请自行查看插件说明。
- 7.debug模式下的热更新代码报错后，需要restart重启一下。

# dart语法
虽然官网上说：
> 如果您熟悉面向对象和基本编程概念（如变量、循环和条件控制），则可以完成本教程，您无需要了解Dart或拥有移动开发的经验。

我信你个鬼，在这里还是推荐看一下dart语法，附上[官网中文地址](http://dart.goodev.org/guides/language/language-tour)

有两个点特别说明：
1. dart中的箭头函数和es6中的不一样，只有函数体中只有return语句时才可以使用，且可以省略return，如下：
```dart
sayName (name) {
    return 'my name is $name';
}

sayName(name) => 'my name is $name';
```
2. dart中`new`关键字可以省略，以下两句话是一样的：
```dart
const a = new Person('name')

const a = Person('name')
```

3. 模块化
使用import导入库即可，无须导出。也可以导出部分功能或者不导出部分功能。
```dart
// Import only foo.
import 'package:lib1/lib1.dart' show foo;

// Import all names EXCEPT foo.
import 'package:lib2/lib2.dart' hide foo;
```

还有一种方案，使用part导入和part of导出，文件之间会共享所有代码结构，而且part之间貌似不能相互导入，part应为最小的代码结构，貌似官方不推荐使用这个。
```dart
// home文件
part './detail.dart';
const detail = new Detail();

// detail文件
part of './home.dart';
class Detail {}
```

4. dart中也有`async/await`的语法，不过与js稍微有所不同
```dart
void test() async {
  final result = await asyncFunction();
}
```

# flutter基础知识
flutter中所有部分均由Widgets构成，同Web前端开发中的组件的概念。一开始学习时，由于Widget种类繁多，建议先看一些常用的组件，其余在项目中慢慢接触即可。

## Widget
以下为个人对flutter中Widget的看法：
Widget大致分为三种：基础组件（Basic Components）、质感组件（Material Components）、IOS风格组件（Cupertino）。
1. [基础组件](https://flutterchina.club/widgets/basics/)：包括Container,Row,Column,Image,Text等等，顾名思义，这些组件为容器组件，水平排列组件，垂直排列组件，图片组件，文本组件。以下为创建一个Text组件的例子，通过注释进行剖析：
[dart函数语法](http://dart.goodev.org/guides/language/language-tour#functions%E6%96%B9%E6%B3%95)
```dart
// 这里采用了省略new的写法
// 对于可选参数请自行查阅上面的链接查看
Text(
  // 必填参数，文本内容，$_name 表示变量_name
  'Hello, $_name! How are you?',
  // 可选参数textAlign
  textAlign: TextAlign.center,
  // 可选参数overflow
  overflow: TextOverflow.ellipsis,
  // 可选参数style
  style: TextStyle(fontWeight: FontWeight.bold)
)
```
可以看到，flutter中，样式全部作为可选参数传入类中，样式结构大致与css相同，并不完全一样，具体请查看[flutter官方文档](https://api.flutter.dev/flutter/widgets/Text-class.html)

2. [质感组件](https://flutterchina.club/widgets/material/)：在讲Material Components前，先了解一下Material Design。
> Material Design是由Google推出的全新的设计语言，谷歌表示，这种设计语言旨在为手机、平板电脑、台式机和“其他平台”提供更一致、更广泛的“外观和感觉”。

以MaterialApp组件为例：
```dart
new MaterialApp(
  // 应用名称
  title: 'Welcome to Flutter',
  // Scaffold：Material Design布局结构的基本实现。此类提供了用于显示drawer、snackbar和底部sheet的API。
  home: new Scaffold(
    // AppBar: 一个Material Design应用程序栏，由工具栏和其他可能的widget（如TabBar和FlexibleSpaceBar）组成。
    appBar: new AppBar(
      title: new Text('Welcome to Flutter'),
    ),
    // 居中布局组件
    body: new Center(
      child: new Text('Hello World'),
    ),
  ),
)
```

3. [IOS风格组件](https://flutterchina.club/widgets/cupertino/): 用法和上面差不多，有ios风格需求的可以看一下。

## BuildContext和Key
BuildContext和Key会是写代码过程中看到最多的组件构造函数中的参数，对这两个参数比较懵逼的可以看一下这两篇文章，便于理解：[BuildContext](https://juejin.im/post/5c665cb651882562914ec153)、[Key](https://juejin.im/post/5ca2152f6fb9a05e1a7a9a26)


# 路由
路由分为两种：创建式路由和命名式路由。一般项目中使用命名式路由。路由对象[Navigator](https://api.flutter.dev/flutter/widgets/Navigator-class.html)，点击可以查看官网信息。

创建式路由Navigator.push，通过传入临时创建的一个新的路由页面，不推荐
```js
bool value = await Navigator.push(context, MaterialPageRoute<bool>(
  builder: (BuildContext context) {
    return Center(
      child: GestureDetector(
        child: Text('OK'),
        onTap: () { Navigator.pop(context, true); }
      ),
    );
  }
));
```
命名式路由，在MaterialApp的参数中配置routes：
```js
void main() {
  runApp(MaterialApp(
    home: MyAppHome(), // becomes the route named '/'
    routes: <String, WidgetBuilder> {
      '/a': (BuildContext context) => MyPage(title: 'page A'),
      '/b': (BuildContext context) => MyPage(title: 'page B'),
      '/c': (BuildContext context) => MyPage(title: 'page C'),
    },
  ));
}
```
跳转时通过Navigator.pushNamed(context, routename, { arguments })方法，该方法接收2个必填参数，第一个参数context，第二个参数路由信息；第三个为跳转时传递的参数，是一个可选命名参数，为Object类型，调用方法时需要使用这种形式`paramName: value`来指定命名参数，这是dart语法，详情请查看[dart中文官网](http://dart.goodev.org/guides/language/language-tour#functions%E6%96%B9%E6%B3%95)。
```js
class Params{};

// 跳转至/a，并传递一个Params的实例
Navigator.pushNamed(context, '/a', arguments: Params('this is parmas'));

//这种写法和上面的写法是一样的
Navigator.of(context).pushNamed('/a', arguments: Params('this is parmas'));

// 在page a中接收参数
Params obj = ModalRoute.of(context).settings.arguments;
```

# 静态资源
对于静态资源，需要在`pubspec.yaml`文件中引入静态资源所在文件/文件夹，如下：
```
flutter:
  assets:
   - assets/
```

# 状态管理
flutter中的状态管理工具有很多，[官网](https://flutter-io.cn/docs/development/data-and-backend/state-mgmt/options)中均由提及，前两种只适合widget内使用：
1. setState
2. InheritedWidget & InheritedModel
3. Provider & Scoped Model
4. Redux
5. BLoC / Rx
6. MobX
7. Provider

这里官方推荐使用Provider，[点击查看官方文档](https://flutter-io.cn/docs/development/data-and-backend/state-mgmt/simple)。或者[点击查看这篇文章](https://juejin.im/post/5d00a84fe51d455a2f22023f)，我个人觉得写得蛮不错的。

# 网络请求
网络请求我推荐使用由Flutter中文网开源的第三方库dio，因为他比较符合前端人员的编码风范，支持Restful API、FormData、拦截器、请求取消、Cookie管理、文件上传/下载、超时等...

附上github地址：[https://github.com/flutterchina/dio/blob/master/README-ZH.md](https://github.com/flutterchina/dio/blob/master/README-ZH.md);

如果你想要虚拟机上访问本地的服务器，需要将localhost改为本地的ip地址，cmd输入ipconfig查看，因为虚拟机上的localhost是它自己。


# App打包
因为我木有Mac，所以以下只涉及到了Android下的打包。

首先根据[官网](https://flutter-io.cn/docs/deployment/android)的步骤下来，然后我们就会在创建密钥库的时候踩坑，这里我是这样解决的：
1. 运行flutter run -v查看你的java路径，就是`Java binary at:`后面那部分路径，即:
```
你的Android Studio路径\jre\bin\java
```
2. cmd进入到上面的bin目录下，输入命令：
```js
// D:\key.jks此为要生成的keystore存放目录，这里存放到D盘根目录下
// 结尾的key为keystore的别名，下文中的keyAlias
keytool.exe -genkey -v -keystore D:\key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias key
```
3. 然后根据提示输入即可
4. 创建完成后，在项目中配置keystore：在项目目录下的/android下创建文件key.properties，写入如下内容：
```
storePassword=<上一步骤中的密码>
keyPassword=<上一步骤中的密码>
keyAlias=key
storeFile=<密钥库的位置，e.g. /Users/<用户名>/key.jks>
```
例如：
```js
storePassword=123321
keyPassword=123321
keyAlias=key
storeFile=D:/key.jks
```
5. 在gradle中配置签名：按照[官网](https://flutter-io.cn/docs/deployment/android)即可
6. 运行flutter build apk，打包后的文件放在项目目录下的build\app\outputs\apk\release\app-release.apk

# 总结
学习新的技术是一个漫长的过程，耐心是必要的，所以不要急于求成，本文的这一过程我个人是觉得比较合理，也是一个较快的入门方案，文中推荐的几篇文章还是推荐大家读一下，对相关部分的剖析还是比较透彻的。最后希望本文对你能有所帮助。
# chapters1

## 1.1 系统架构

### 1.1.1 Linux内核层

安卓系统是基于Linux内核的，这一层为安卓设备的各种硬件提供了底层驱动

### 1.1.2 系统运行库层

这一层通过一些C/C++库为安卓系统提供了主要的特性支持

### 1.1.3 应用框架层

这一层主要提供了构建应用程序可能用到的API，安卓自带的一些核心应用就是使用这些API完成的

### 1.1.4 应用层 

所有安装在手机上的应用程序都属于这一层。

## 1.2 开发特色

### 1.2.1 四大组件

​	活动（activity） 服务（service） 广播接收器（Broadcast Receiver）内容提供器（Content Provider）

### 1.2.2 丰富的系统控件

### 1.2.3 SQLite 数据库

### 1.2.4 强大的多媒体

### 1.2.5 地理位置定位 

## 1.3 第一个Android 程序

### 1.3.1 程序分析

​	1 .gradle和.idea

​	这两个目录下放置的是Andriod Studio 自动生成的一些文件

​	 2 app

​	项目中的代码，资源等内容都放在这个目录下面，后面开发工作也是在这个目录下进行的

​    3 build 

​	编译时自动生成的文件

​    4 gradle 

​	这个目录下包含了gradle wrapper 的配置文件，使用gradle wrapper的方式不需要提前将gradle下载好，会自动根据本地缓存决定是否需要联网下载

​	5 .gitignore

​	该文件将指定的目录或文件排除在版本控制之外

​	6 build.grade

​	这是项目全局的gradle构建脚本，通常这个内容是不需要修改的

​	7 gradle.properties

​	这个文件时全局的gradle的配置文件，在这里配置的属性将会影响到项目中所有的gradle编译脚本

​	8 gradlew 和gradlew.bat 

​	这两个文件是用来在命令行界面中执行gradle命令，其中gradlew是在Linux或Mac系统中使用的，gradlew.bat是在windows系统中使用的

​	9 local.properties 

​	这个文件用于指定本机中的Andorid SDK路径，通常都是自动生成的，如果SDK默认位置发生改变，将路径改成新的位置即可

​	10 settings.gradle 

​	这个文件用于指定项目中所引入的模块。

### 1.3.2 app目录下分析

​	1 build 

​	编译时自动生成的文件

​	2 libs 

​	如果你的项目中使用到了第三方jar包，就需要把这些jar包放在libs中，放在这个目录下的jar包会自动添加到构建路径中

​	3 AndroidTest

​	此处用来写Android Test测试用例，可对项目进行一些自动化测试

​	4 java

​	java目录是我们放置所有JAVA代码的地方，展开该目录，你能看到刚才创建的文件

​	5 res 

​	在项目中用到的所有图片，布局，字符串等资源都放置在这个目录下面。

​	6 AndroidManifest.xml

​	整个Android项目的配置文件，在程序中定义的四大组件都需要在这个文件里注册，另外还可以在这个文件中给应用程序添加权限声明。

​	7 test

​	用来编写Unit Test测试用例

​	8 .gitignore

​	这个文件用于将APP模块内的指定目录或文件排除在版本控制之外

​	9 build.gradle

​	这是app模块的gradle构建脚本，这个文件中会指定很多项目构建相关的配置

​	10 proguard-rule.pro

​	这个文件用于指定项目代码的混淆规则，当代码开发完成后打成安装包文件，如果不希望代码被人破解，通常将其混淆，让阅读者难以阅读

### 1.3.3 项目中资源分析

​	res中以drawable开头的文件用来放图片，以mipmap开头的用来放应用图标，以values开头的用来放字符串、样式和颜色等配置，layout文件夹用来放置布局

​	对于用于程序名字符串可用以下两种

```
代码中R.string.app_name获得该字符串的引用
在XML中通过@string/app_name获得该字符串的引用
```

​	其中string可以进行更换，引用图片改成drawable 应用图标mipmap 布局文件改成layout

```
android:label="@string/app_name" 修改应用名称
android:icon="@mipmap/ic_lanucher"  修改应用图标
```

### 1.3.4 build.gradle 详解

​	（1）最外层的build.gradle 

​	android 闭包用来配置项目构建的各种属性

​	defaultConfig闭包用来对项目的更多细节进行配置

```
applicationId 指定项目的包名
minSdkVersion 用于指定项目最低兼容的Android版本
targetSdkversion 在该目标版本上已经做了充分测试，系统会为你的应用程序启动一些最新的功能和特性
versionCode 指定项目的版本号 
versionName 指定项目的版本名
```

​	buildTypes闭包 应开指定安装文件的相关配置 

​		debug 指定生成测试版安装文件的配置

​		release 用于指定生成正式安装文件的配置

```
minifyEnabled 指定对项目代码是否进行混淆
proguardFiles 指定混淆是采用的规则 proguard-andriod.txt 使用SDK目录下的规则
								proguard-rules.pro  在当前项目的根目录下编写混淆规则
```

​	dependencies闭包 指定当前项目所有的依赖关系 

​	1 本地依赖 对本地的jar包或目录添加依赖关系  

```
compile fileTree(dir: 'lib',include:['*.jar'])
将lib目录下所有.jar文件添加到项目的构建路径中
```

​	2 库依赖 对项目中的库模块添加依赖关系 

```
complie project (':helper') helper 依赖库的名称
```

​	3 远程依赖 对jcenter库上的开源项目添加依赖关系

```
complie 'com.android.support:appcompat-v7:24.2.1'
com.android.support 域名部分用来和其他公司库区分
appcompat-v7 组名称，用来和同一个公司的不同库区分
24.2.1 版本号 用来和同一个库的不同版本做区
```

## 1.4 掌握日志工具使用

### 1.4.1 Android日志工具 Log

```java
Log.v() 用于打印最为琐碎的日志信息 是android日志里最低价的一种
Log.d() 用于打印一些调试信息，对应级别debug
Log.i() 用于打印一些比较重要的数据 对应几倍 info
Log.w() 用于打印一些警告信息，提示程序在这个地方可能会有潜在的危险，最好修复以下 对应warn
Log.e() 用于打印程序中的错误信息，比如程序进入catch语句中.对应 error
```


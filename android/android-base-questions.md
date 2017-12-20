# Android基础问题

##  Activity&View系列

### 谈谈Activity的生命周期

### 介绍不同场景下Activity生命周期的变化过程

### Activity销毁但Task如果没有销毁掉，当Activity重启时这个AsyncTask该如何解决？

### 内存不足时,系统会杀死后台的Activity,此时如何保存当前Activity的某些状态？如何恢复？

### `onSaveInstanceState()`的工作原理是什么?


### 请介绍下Activity中的luancherMode

### 请介绍下LaunchMode的使用场景

### 简单描述Activity,Window,View三者的关系

### 如何实现Activity窗口变暗效果,简单写一下代码?

### RemoteView应用及与普通View的区别?

### LinearLayout和RelativeLayout性能对比?

### Constraintlayout和RelativeLayout对比?


### 从哪几个角度进行布局优化?

### Android中动画的分类?

### Android属性动画特性

### 属性动画和补间动画的有什么区别?

### 在补间动画执行的过程中,调用其invalidate()方法会出现什么现象?为什么?

### SurfaceView与View的区别?

### 在哪些场景下需要使用SurfaceView?

### 介绍下自定义View的基本流程?以实现一个刻度表为例.

### View中setOnTouchListener钟的onTouch,onTouchEvent,onClick的执行顺序?

### 自定义控件这中滑动冲突该如何解决?

### ListView常用的优化策略

### 如何实现一个局部更新的ListView

### ListView中的ViewHolder为什么要设计成静态内部类？



## Fragment系列

### 谈谈Fragment的用途以及为什么google要设计Fragment?

### 简述Fragment的生命周期

### Fragment重叠问题遇到过么?为什么?

### Fragment和父Activity通信方式有哪些?



## Service系列

### 谈谈Service的生命周期

### 启动Service的两种方式以及区别是什么?

### IntentService的用途是什么?简述其工作原理.

### 如何提升Service优先级?

### Service和Activity通信的手段有哪些?

### 如何保证Service在后台不被kill?



## Broadcast系列

### 广播注册的几种方式?有什么区别?

### 本地广播和全局广播有什么区别?


### Android为什么要引入广播机制?



## ContentProvider系列

### ContentProvider实现数据共享过程简述?

### ContentProvider和SQL的联系与区别?



## Context系列

### 简述Android中Context的体系?


### 你经常使用Context的哪些方法?

### Context可以手动创建么?在哪些情况下需要手动创建?



## 数据存储系列

### 简单介绍你所知的本地数据存储方式

### 如何发布数据库文件到apk中?


### 如何使用apk资源目录Raw下的数据库文件?


### SQLite支持事务么?如何有效的提供操作数据库的性能?

### 你对SQLite有优化经验么?如果让你来优化,你会从哪些角度进行?



## 网络相关

### 对WebView熟么?简单讲讲你在使用WebView中遇到过哪些坑?

### 你知道哪些WebView中的漏洞

### XML解析方式和对比?

### 你所知道的网络数据传输协议有哪些?比较一下

### Http工作原理及缺点?

### Http中Etag的用途是什么?

### 如何实现多线程下载框架?简述思路

### 对HTTPS了解么?用户以及原生HttpUrlConnection如何支持HTTPS

### 对网络请求有过优化方案么？有哪些思路？

  ### 如何防止重复发送网络请求？


### 如何解决域名被运营商拦截问题？


### 网页被运行商篡改问题？



## 数据结构类

### 简述Bundle结构,并说明为什么Google不采用Map的原因.




## 适配相关

### dp,dip,dpi,px,sp含义及换算公式


### 屏幕适配

### RTL布局适配

### mipmap文件夹和drawable文件加的区别

### 谈谈你对Bitmap的理解，以及bitmap.recycle()的工作原理？

### 如何高效的加载图片资源？



## 系统机制

### Linux中常见的进程通信方式有哪些呢?Android呢?

### 什么时AIDL?使用AIDL的基本流程是什么?


### 如何实现跨进程回调?

### 你了解NDK开发么?简单说说开发流程(eclipse和AS中)

### NDK常见的应用场景是什么?

### NDK开发中如何输出日志，以及如何调试？

### NDK开发中有那些注意事项？

### 简单聊聊CPU 架构和ABI


### 如何通过JNI传递String对象？

### 造成ANR的问题有哪些？如果在主线成进行耗时操作，一定会造成ANR么？


### 给你一段ANR，如何定位问题？

### 如何避免ANR问题？分别从代码层次和设计层次去介绍下？

### Dalvik虚拟机和JVM虚拟机的区别是什么？

### Dalvik虚拟机和ART虚拟机的对比？

### 如何解决方法数超过65K问题？

### Android两个应用能在同一个任务栈吗？

###现在有两个应用A和B,先打开A应用,假设依次打开其A-Page1,A-Page2,A-Page3三个Activity,此时回到桌面,打开B应用,假设由于B应用非常大,促使系统杀死了A应用,此时回到桌面,点击A应用图标,请问出现什么情况?

### 假设A类存在一个静态变量,在变量被赋值后,该应用进程被杀死,请问应用重启后该变量的状态?



## 调试与测试

### 如何调试Android应用程序？so调试呢？

### 如何进行压力测试？从数据库和网络两方介绍下

### 如何进行UI性能测试？如何试用Android Profiler排查？

### 如何进行内存测试？

### 如何评估应用是否发生了内存泄漏？

### 内存泄漏产生的原因是什么？如何避免？

### 如何排查OOM么？OOM的产生的原因是什么？


### 如何制造一个OOM的场景？

### 如何避免OOM？



## 安全加固


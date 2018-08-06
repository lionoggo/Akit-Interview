# Android基础问题

##  Activity&View系列

### 谈谈Activity的生命周期

### 介绍不同场景下Activity生命周期的变化过程

### Activity销毁但Task如果没有销毁掉，当Activity重启时这个AsyncTask该如何解决？

### 内存不足时,系统会杀死后台的Activity,此时如何保存当前Activity的某些状态？如何恢复？

### `onSaveInstanceState()`的工作原理是什么?

### 简述Activity任务栈

为了对应用中的Activity进行管理,Google采用了栈结构(后进先出)来管理Activity,即所谓的任务栈.默认情况下,每启动一个应用,系统便会为其创建一个包名为名的任务栈.

默认情况下，一个Activity启动另一个Activity时，两个Activity是放置在同一个task中的，后者被压入前者所在的task栈，当用户按下后退键，后者从task被弹出，前者又显示在幕前，特别是启动其他应用中的Activity时，两个Activity对用户来说就好像是属于同一个应用；系统task和task之间是互相独立的，当我们运行一个应用时，按下Home键回到主屏，启动另一个应用，这个过程中，之前的task被转移到后台，新的task被转移到前台，其根Activity也会显示到幕前，过了一会之后，在此按下Home键回到主屏，再选择之前的应用，之前的task会被转移到前台，系统仍然保留着task内的所有Activity实例，而那个新的task会被转移到后台，如果这时用户再做后退等动作，就是针对该task内部进行操作了。

处于栈顶的Activity为当前活动栈,处在焦点状态.当按下Back键或者返回按钮时,栈内Activity会依次出栈,并调用其onDestory().当所有的Activity被销毁后,系统会回收该栈.



### 请介绍下Activity中的luancheMode

LancheMode即所谓的启动模式,系统会根据该模式来控制Activity打开时对任务栈的影响.该选项有以下四个值选项:standard,singleTop,singleTask,singleInstance.其含义如下:

1、standard：默认模式：每次启动都会创建一个新的activity对象，放到目标任务栈中

2、singleTop：判断当前的任务栈顶是否存在相同的activity对象，如果存在，则直接使用，如果不存在，那么创建新的activity对象放入栈中

3、singleTask：在任务栈中会判断是否存在相同的activity，如果存在，那么会清除该activity之上的其他activity对象显示，如果不存在，则会创建一个新的activity放入栈顶

4、singleIntance：会在一个新的任务栈中创建activity，并且该任务栈种只允许存在一个activity实例，其他调用该activity的组件会直接使用该任务栈种的activity对象

控制任务栈的方式

方法一：
使用android:launchMode="standard|singleInstance|single Task|singleTop"来控制Acivity任务栈。
方法二：
Intent Flags：

```java
Intent intent=new Intent();
intent.setClass(MainActivity.this, MainActivity2.class);
intent.addFlags(Intent. FLAG_ACTIVITY_CLEAR_TOP);
startActivity(intent);
```

Flags有很多，比如：
Intent.FLAG_ACTIVITY_NEW_TASK   相当于singleTask
Intent. FLAG_ACTIVITY_CLEAR_TOP   相当于singleTop


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

### 动画的两种实现:setX()和setTranslation的区别?



### SurfaceView与View的区别?

### 请简单讲述为什么非UI线程不能更新View.

### 在哪些场景下需要使用SurfaceView?

### 介绍下自定义View的基本流程?以实现一个刻度表为例.

### View中setOnTouchListener钟的onTouch,onTouchEvent,onClick的执行顺序?

### 自定义控件这中滑动冲突该如何解决?

### ListView常用的优化策略

### 如何实现一个局部更新的ListView

### ListView中的ViewHolder为什么要设计成静态内部类？

###ListView跟RecyleView的区别?

###请例举Android中常用布局类型，并简述其用法以及排版效率



## Fragment系列

### 谈谈Fragment的用途以及为什么google要设计Fragment?

### 简述Fragment的生命周期

### Fragment重叠问题遇到过么?为什么?

### Fragment和父Activity通信方式有哪些?



## Service系列

### 启动Service的两种方式以及区别是什么?

Service存在两种启动方式:startService和bindService.通过startService启动的Service,其生命周期和调用者不相关;通过bindService启动的Service其生命和调用者一致.

![image-20180806163026754](http://pbj0kpudr.bkt.clouddn.com/blog/2018-08-06-083027.png)


 ### 谈谈Service的生命周期

先来看以**startService**方式启动Service时的生命周期.该种方式启动的Service会一直运行下去(不考虑系统杀死的情况),必须显式的调用Context.stopService或Service内部调用stopSelf来停止该Service.



- onCreate:startService时,如果对应的Service未处于运行状态,则会执行onCreate进行Service初始化操作.
- onStartCommand:startService时,如果Service已经处于运行状态,则会直接执行该方法;否则会先调用onCreate,然后再执行.该方法的返回值决定改Service被系统销毁之后的重建行为.
- onDestroy:Service被销毁时将会调用该方法.

再来看以**bindService**进行绑定的Service的生命周期.

- onCreaqte:bindService时如果Service未处于运行状态,同样会执行该方法进行Service创建和初始化操作.
- onBind:onCreate执行成功后,会继续执行onBind方法,需要返回相应的IBinder对象给调用方.
- onUnbind:调用方调用该方法来解绑Service对象,断开调用方和Service之间的绑定关系
- onDestroy:onUnbind执行后,onDestroy方法会被调用

### onStartCommand返回值解释

当Android系统内存吃紧时,可能会销毁当前正在运行的某些Service,将来在内存充足的时
候可以重建Service.其重建行为依赖于Service中`onStartCommand`方法的返回值:

- START_CONTINUATION_MASK
- START_STICKY_COMPATIBILITY:是对START_STICKY进行兼容的版本,不保证被杀后一定能重建
- START_STICKY:Service被杀死后,保留Service为开始状态,在系统内存充足后会重建该Service,并调用onStartCommand方法,不过此时Intent为null
- START_NOT_STICKY:Service被杀死后,在系统内存充足后也不会重建该Service
- START_TASK_REMOVED_COMPLETE
- START_FLAG_REDELIVERY:Service被杀死时会保存最后接受到的Intent,并在重建时调用onStartCommand,并将该Intent传入
- START_FLAG_RETRY

### IntentService的用途是什么?简述其工作原理.

### 如何提升Service优先级?

### Service和Activity通信的手段有哪些?

### 如何保证Service在后台不被kill?

- 提交进程的优先级,见底进程被杀死的概率.

  方法一 :监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。
  方法二：启动前台service。
  方法三：提升service优先级：
  在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

- 在进程被杀死后，进行拉活
  方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等
  方法二：双进程相互唤起,比较有效的方法是两个Service进行远程绑定.
  方法三：依靠系统唤起。
  方法四：onDestroy方法里重启service：service +broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service

- 根据终端厂家不同,接入相应的推送.如在小米手机接入小米推送,在华为手机接入华为推送

- 如果是做系统开发这,那就直接申请加入白名单.

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

### 1. Linux中常见的进程通信方式有哪些呢?Android呢?

### 2. 什么时AIDL?使用AIDL的基本流程是什么?


### 3. 如何实现跨进程回调?

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

###假设A类存在一个静态变量,在变量被赋值后,该应用进程被杀死,请问应用重启后该变量的状态?

### Android应用如何开进程?最多可以开多少个进程?

通过配置android:process即可实现多进程.基于Linux内核实现的Android系统并没有限制一个应用最多可以开多少个进程,这些参数都是可以配置的,可以通过`adb shell ulimit -a`查看:

```shell
time(cpu-seconds)    unlimited
file(blocks)         unlimited
coredump(blocks)     0
data(KiB)            unlimited
stack(KiB)           8192
lockedmem(KiB)       unlimited
nofiles(descriptors) 1024
processes            3619
sigpending           3619
msgqueue(bytes)      819200
maxnice              40
maxrtprio            0
resident-set(KiB)    unlimited
address-space(KiB)   unlimited
```



### 如何追踪某个对象的回收情况?





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

### 如何进行单元测试?



## 安全加固


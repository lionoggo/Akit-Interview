# Android系统原理

## Android进程分类及变化?

Android中将进程分为五种:

- 前台进程: 用户当前操作所必需的进程。通常在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们.具有以下情况的进程属于前台进程:
  1. 拥有用户正在交互的 Activity
  2. 拥有某个Service并且它绑定到用户正在交互的Activity或者有用前台Service
  3. 拥有正执行一个生命周期回调方法的Service或者正在执行`onReceive()`的BroadcastReceiver
- 可视进程:没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程.可见进程被视为是极其重要的进程,除非为了维持所有前台进程同时运行而必须终止,否则系统不会终止这些.具有以下情况的进程属于可见进程:
  1. 拥有不在前台,但仍然可见的Activity(即当前Activity的`onPause()`被调用)
  2. 拥有某个Service并且它绑定到上述仍然对用户可见的Activity
- 服务进程: 服务进程与用户所见内容没有直接关联,但是它们通常在执行一些用户关心的操作,如后台下载数据等.除非内存不足以维持所有前台进程和可见进程同时运行,否则系统会让服务进程保持运行状态.具有以下情况的进程属于服务进程:
  1. 通过 `startService()`启动的服务,并且不属于上述两个更高类别进程
- 后台进程: 此类进程对用户体验没有直接影响,系统可能随时终止它们以回收内存供前台进程,可见进程或服务进程使用.手机中通常会有很多后台进程在运行,这些进程会被保存在 LRU表中,以保证包含用户最近查看的Activity的进程最后一个被终止.具有以下情况的进程都属于后台进程:
  1. 含有对用户不可见的 Activity 的进程,即已调用 `onStop()`并且不属于服务进程
- 内容节点进程:只含有ContentProvider节点的进程,
- 空进程: 这种进程存在一目的是用作缓存,一遍再下次启动时能够减少所需的启动时间.系统在需要的时候会终止这些进程.具有以下情况的进程属于空进程:
  1. 不含有任何活动组件

## 简述Android 进程回收机制

Android采用LMK(即Low Memory Killer,所谓的低内存杀进程)机制来决定进程的回收,该机制是在Linux内核的OOM Killer(Out-Of-Memory Killer)机制上发展而来.在Android进程中,内核会为每个进程分配一个oom_adj值,用来表示进程的重要性,oom_adj的值越小,表示进程越重要,越不容易被杀掉回收.LMK会根据每个进程的oom_adj以及系统预定义不同等级进程的阀值来决定杀掉哪些进程.

在Android中通过以下方式可以查看进程的oom_adj值:

```shell
cat /proc/进程号/oom_adj
```

不同等级的进程所定义的阈值不同,可以通过以上方式查看:

```shell
cat /sys/module/lowmemorykiller/parameters/minfree
```

比如当前设备的情况如下:

```shell
// 前台进程,可视进程,服务进程,后台进程,内容提供节点进程,空进程
18432,23040,27648,32256,36864,46080
```

这6个数值分别代表android系统回收6种进程的阀值,其中数字单位是page(page是内存分配的单位,1page=4k),可以按照此公式将其转为M:`数值 * 4/1024`,即分别是:72M,90M,108M,126M,144M,180M.

当系统剩余内存位于阀值中的一个范围内时,如果一个进程的oom_adj值大于或等于阀值就会被杀死.比如当前系统内存在144M~180M之间时,就会按照空进程中oom_adj值的大小选择杀死哪些空进程.

## 简述Android进程优先级和oom_adj的映射关系?

在底层,Android采用oom_adj来描述进程的重要的性,那这oom_adj是如何和之前我们说的六种进程等级关联起来的?也就是前台进程的oom_adj是多少,后台又是多少呢?

在`ActivityManager.RunningAppProcessInfo`中我们可以看到有关进程重要性的定义:

```java
public static class RunningAppProcessInfo implements Parcelable {
     public static final int IMPORTANCE_FOREGROUND = 100;
		......
        public static final int IMPORTANCE_FOREGROUND_SERVICE = 125;
        @Deprecated
        public static final int IMPORTANCE_TOP_SLEEPING_PRE_28 = 150;
        public static final int IMPORTANCE_VISIBLE = 200;
        public static final int IMPORTANCE_PERCEPTIBLE_PRE_26 = 130;
        public static final int IMPORTANCE_PERCEPTIBLE = 230;
        public static final int IMPORTANCE_CANT_SAVE_STATE_PRE_26 = 170;
        public static final int IMPORTANCE_SERVICE = 300;
        public static final int IMPORTANCE_TOP_SLEEPING = 325;
        public static final int IMPORTANCE_CANT_SAVE_STATE = 350;
        public static final int IMPORTANCE_CACHED = 400;
        public static final int IMPORTANCE_BACKGROUND = IMPORTANCE_CACHED;
        @Deprecated
        public static final int IMPORTANCE_EMPTY = 500;
        public static final int IMPORTANCE_GONE = 1000;
        ......
}        
```

以上这些常量表示了Process的重要性等级而在ProcessList则定义了相关adj:

```java
public final class ProcessList {
	static final int INVALID_ADJ = -10000;
    static final int UNKNOWN_ADJ = 1001;
    static final int CACHED_APP_MAX_ADJ = 906;
    static final int CACHED_APP_MIN_ADJ = 900;
    static final int SERVICE_B_ADJ = 800;
    static final int PREVIOUS_APP_ADJ = 700;
    static final int HOME_APP_ADJ = 600;
    static final int SERVICE_ADJ = 500;
    static final int HEAVY_WEIGHT_APP_ADJ = 400;
    static final int BACKUP_APP_ADJ = 300;
    static final int PERCEPTIBLE_APP_ADJ = 200;
    static final int VISIBLE_APP_ADJ = 100;
    static final int VISIBLE_APP_LAYER_MAX = PERCEPTIBLE_APP_ADJ - VISIBLE_APP_ADJ - 1;
    static final int FOREGROUND_APP_ADJ = 0;
    static final int PERSISTENT_SERVICE_ADJ = -700;
    static final int PERSISTENT_PROC_ADJ = -800;
    static final int SYSTEM_ADJ = -900;
    static final int NATIVE_ADJ = -1000;
}
```

不同版本的定义的值可能不一样,比如早期SYSTEM_ADJ被定义为-16.

在AMS中,方法`procStateToImportance()`用于实现adj到重要性的映射:

```java
    // com.android.server.am.ActivityManagerService#procStateToImportance
    static int procStateToImportance(int procState, int memAdj,
            ActivityManager.RunningAppProcessInfo currApp,
            int clientTargetSdk) {
        int imp = ActivityManager.RunningAppProcessInfo.procStateToImportanceForTargetSdk(
                procState, clientTargetSdk);
        if (imp == ActivityManager.RunningAppProcessInfo.IMPORTANCE_BACKGROUND) {
            currApp.lru = memAdj;
        } else {
            currApp.lru = 0;
        }
        return imp;
    }
```

在上述方法中,最终的转换是通过`RunningAppProcessInfo.procStateToImportanceForTargetSdk()`来完成,具体的转换流程就不说了.

## 根据变化情况判断当前进程状态?

- 某个应用只有一个Activity,打开该应用点击home键,此时该进程是哪种进程?如果按back键呢?

  分别是后台进程和空进程

- 某个应用只有一个Activity,但该Activity启动后会开启一个不断循环的线程,点击home键,该进程是哪种进程?如果按返回键呢?

  分别是后台进程和空进程

- 某个应用只有一个接受输入的Activity,该Activity显示后,点击输入弹出输入法此时是什么进程?点击home后呢?

  分别是可见进程和后台进程

- 某个应用只有一个Activity,但该Activity在启动时会通过`bindService()`方式关联一个Service,在Activity销毁时解绑该Service,此时点击home键和back后,当前进程分别是哪种?

  分别是服务进程和空进程


## Dalvik VM和JVM架构的区别?

Dalvik VM并不遵循JVM的实现规范,因此准确的说它不是一个JVM.从设计上来讲,Dalvik VM和JVM主要有以下区别:

### 基于的架构不同

JVM基于操作栈结构,其执行指令都依赖于内部的操作栈.而Dalvik VM是基于寄存器实现,其执行指令和硬件体系相关.对于同样的操作而言,所需的Dalvik VM的指令数量要远远小于JVM.

### 执行的文件不同

和JVM执行编译后的.class文件不同,Dalvik VM执行的是.dex文件.所有java文件编译后生成.class文件经过dx工具处理后,会被压缩到一个.dex文件中,之后Dalvik VM会加载该.dex文件,读取和执行其中的指令.比如每个.class中都有自己的常量池结构,JVM在加载该.class后会为当前.class在内存中开辟常量池空间,但dex中所有原来.class常量池被压缩合并到了一处,在加载之后,对常量池的查找效率更高.

## Art相比Dalvik在执行方式有什么变化?

在Dalvik下,应用运行时会通过解释器将字节码转换为机器码,如果某段代码运行频繁,则会通过JIT(及时编译器)直接将其转换为机器码,这样就不用每次进行转换操作了,进而提高了效率.

在ART 下,应用第一次安装的时候,字节码就会被编译成机器码,使其成为真正的本地应用.我们将这个过程称之为预编译,即我们常说的AOT(Ahead-Of-Time).之后应用开始运行时效率就很高了.

## Art和Dalvik中堆内存的区别?

Dalvik VM的堆结构相对于JVM的堆结构有所区别.在Dalvik中,堆空间分为两部分:Zygote堆和Active堆.其中Zygote堆中存放了Zygote进程在启动过程中预加载和创建的各种对象,在Zygote堆中不会触发GC,所有进程都共享该区域,如系统资源,除此之外所有的对象,包括我们在代码中创建的实例,静态域和数组都是储存在Active堆中.

Art堆结构在Dalvik的基础上进行了更细致了划分,原来的Zygote堆又被分为了Image Space和Zygote Space;原来的Active堆则被分成了Allocation Space和Large Object Space.Image Space用来存放哪些需要预加载的系统类对象,而Large Object Space则用来分配一些大对象.Zygote Space和Allocation Space与Dalvik虚拟机垃圾收集机制中的Zygote堆和Active堆的作用是一样的.

Zygote Space和Image Space是所有进程都共享的区域,而Allocation Space则是每个进程独占的.Zygote进程开始只有1个Image Space和1个Zygote Space.在Zygote进程fork第一个子进程之前,Zygote Space会被一分为二:原来的已经被使用的那部分堆还叫Zygote Space,而未使用的那部分堆就叫Allocation Space.以后的对象都在Allocation Space上分配。

## GC原因

Art和Dalvik中GC的实现有所不同,因此其GC类型也有所差异,在Dalvik中GC主要有以下几种类型:

- GC_FOR_MALLOC: 应用程序创建对象,此时在堆上分配对象时内存不足触发的GC
- GC_CONCURRENT: 应用程序的堆内存使用率到一定程度时触发的并发GC来释放内存
- GC_EXPLICIT: 应用程序主动调用`System.gc()`,`VMRuntime.gc()`或收到SIGUSR1信号时触发的GC
- GC_BEFORE_OOM: 应用在准备抛OOM异常之前进行的最后努力而触发的GC
- GC_HPROF_DUMP_HEAP: 当请求创建 HPROF文件来分析堆内存时触发的GC
- GC_EXTERNAL_ALLOC: API 级别小于等于10时 ，外部分配内存导致的GC

在Dalvik虚拟机中,主要提供了两种GC的实现:标记-清理和标记-复制.在编译Dalvik阶段,通过指定WITH_COPYING_GC选项来决定具体要使用哪种GC算法.

在Art中GC主要由以下几种类型:

- Alloc: 当要分配内存的时候发现内存不够的情况下引起的GC,这个GC会发生在正在分配内存的线程
- Concurrent: 并发GC,不会使App的线程暂停,该GC是在后台线程运行的,并不会阻止内存分配
- Explicit: 显式的请求进行GC,如调用`System.gc()`.
- NativeAlloc: Native内存分配时,比如在为Bitmaps或RenderScript分配对象导致内存吃紧时,会触发GC

## GC日志

Dalvik中GC日志格式如下:

```she'l'l
D/dalvikvm: <GC_Reason> <Amount_freed>, <Heap_stats>, <External_memory_stats>, <Pause_time>
```

- GC_Reason:表示哪种情况下触发的GC操作,即GC原因
- Amount_freed: 本次GC回收的内存大小
- Heap_stats: 表示堆中空闲内存所占百分比,以及[已使用内存大小]/[堆总大小]
- External memory stats:外部内存统计数据,适应于API 10及以下表示外部分配的内存,以及[已分配内存大小]/[GC下限]
- Pause time:暂停时间,另外越大的堆会有更长的暂停时间.对于并发GC(GC类型为GC_CONCURRENT)时,该时间分两部分:一个出现在垃圾收集开始时,另一个出现在垃圾收集快要完成时

如下述GC日志:

```
D/dalvikvm( 9050): GC_CONCURRENT freed 2049K, 65% free 3571K/9991K, external 4703K/5261K, paused 2ms+2ms
```

1. 引发GC的原因是GC_CONCURRENT
2. 此次GC释放内存2049K
3. 当前堆空闲内存百分比为65%,已使用堆内存为3571K,堆总内存大小为9991K
4. 外部分配内存已经分配4703K,GC下限为5261K
5. GC暂停时间为2ms+2ms,总耗时共4ms.

Art中GC日志格式如下:

```java
I/art: <GC_Reason> <GC_Name> <Objects_freed>(<Size_freed>) AllocSpace Objects, <Large_objects_freed>(<Large_object_size_freed>) <Heap_stats> LOS objects, <Pause_time(s)>
```

- GC_Reason:表示哪种情况下触发的GC操作,即GC原因
- GC_Name:表示使用了哪种GC回收器,目前有Concurrent mark sweep,Concurrent partial mark sweep,Concurrent sticky mark sweep,Marksweep + semispace四种.
- Objects_freed: 本次GC从非大对象空间（non large object space）回收的对象数目
- Size freed:本次GC从非大对象空间回收的字节数
- Large objects freed:本次GC从大对象空间里回收的对象数目
- Large object size freed:本次GC从大对象空间里回收的字节数
- Heap stats:堆的空闲内存百分比和[已使用内存大小]/[堆总大小]
- Pause times:暂停时间,暂停时间与在GC运行中修改的对象引用的数量成比例.CMS只会在GC结束的时停顿一次,GC过渡会有一个长停顿,是GC时耗的主要因素

如下述GC日志:

```
I/art : Explicit concurrent mark sweep GC freed 104710(7MB) AllocSpace objects, 21(416KB) LOS objects, 33% free, 25MB/38MB, paused 1.230ms total 67.216ms
```

1. 引起GC原因是Explicit
2. 垃圾收集器为concurrent mark sweep,即CMS
3. 释放对象的数量为104710个,释放字节数为7MB;
4. 释放大对象的数量为21个,释放大对象字节数为416KB
5. 堆的空闲内存百分比为33%,已用内存为25MB,堆的总内存为38MB
6. GC暂停时长为1.230ms,GC总时长为67.216ms

> 和Dalvik相比,ART下GC的策略更完善,同时支持了更多的垃圾回收器,目前主要有以下几种:
>
> - Concurrent mark sweep (CMS): 整堆回收器,负责收集释放除Image Space外的所有空间
> - Concurrent partial mark sweep: 几乎是整回收器,负责收集除image和zygote外的所有空间
> - Concurrent sticky mark sweep: 分代垃圾收集器,只负责释放从上次GC到现在分配的对象,该GC比全堆和部分标记清除(mark sweep)执行得更频繁,因为它更快而且停顿更短
> - Marksweep + semispace: 非并发的GC,复制垃圾回收,用于堆转换堆碎片整理

## 事件分发机制

在用户在手指与屏幕接触过程中会产生一系列事件,事件由MotionEvent表示,它使用一个int类型的值来表示不同的事件,我们常见的事件如下:

- MotionEvent.ACTION_DOWN: 手指按下屏幕的瞬间
- MotionEvent.ACTION_MOVE:手指在屏幕上移动
- MotionEvent.ACTION_UP手指离开屏幕瞬间

整个事件分发机制其实很容易理解:先分发再消费.一个事件从外往内分发,也就是从最外层容器(Activity)开始不断的向内分发,直到事件被某个View消费掉.此外在Android还允许拦截分发.

在整个事件分发过程中,涉及到以下三个重要的函数:

- boolean dispatchTouchEvent(MotionEvent ex): 该方法负责事件的分发,即将事件由外往内传递.其返回值表示当前事件是否被分发成功:返回true表示事件被分发成功,此次事件分发终止;返回flase表示事件分发失败即当前View及子View均没有消费此次事件,将调用父View的`onTouchEvent()`.
- boolean onInterceptTouchEvent(MotionEvent ex): 该方法负责事件的拦截操作,用于ViewGroup.其返回值表示是否拦截,true表示拦截该事件,该事件不会继续向下分发,而是调用当前View自身的`onTouchEvent()`;返回false表示不拦截事件,而是继续调用子View的`dispatchTouchEvent()`将事件分发到子View中.
- boolean onTouchEvent(MotionEvent ex): 该方法负责消费事件,在`dispatchTouchEvent()`中调用,其返回值表示事件是否被消费:true表示事件被消费,本次事件终止;false表示事件没有被消费,将调用父View的`onTouchEvent()`来处理.

整个事件分发流程可以用一下伪代码表示:

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean consumed = false;
        if (onInterceptTouchEvent(ev)){
            consume = onTouchEvent(ev);
        }else{
            consume = child.dispatchTouchEvent(ev);
        }
        return consume;
    }
```

整个事件分发机制是基于递归思想,因此很多时候看源代码会有一种云里雾里的感觉.市面上有不少写该知识点的blog,个人感觉只需要多读几遍源代码,明白以上三个方法有什么用以及在哪里用,这样会比较容易.

## Handler原理

首先要明确整个Handler系统涉及两个层次:Java层和Native.单纯说Java层的Handler体系只能说是浮于表面.

在Java层,主要存在以下几个相关的类型

- Handler: 用于发送消息和处理消息
- MessageQueue: 存储消息的队列,在获取不到消息时会发生阻塞
- Looper : 用于循环遍历队列中的消息

在应用的入口函数ActivtyThread的`main()`方法中,会调用`Looper.prepareMainLooper()`为其创建Looper,并会通过`Looper.loop()`来使得它循环起来.

```java
android.app.ActivityThread#main
public static void main(String[] args) {
    ......
    Looper.prepareMainLooper();
    ......
    
    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    ......
    
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}

```

### Looper及MessageQueue对象创建

在Looper中,提供了两中方式来为当前线程Looper对象,一种是`Looper#prepareMainLooper()`,另一种是`Looper#prepare()`,前者是用来应用启动时自动调用,为主线程创建Looper对象,而后者是我们自己开始时常用的方式,两者的实现并无区别,`prepareMainLooper()`底层使用`prepare()`:

```java
public final class Looper {
    
    public static void prepare() {
        prepare(true);
    }

    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

    public static void prepareMainLooper() {
        // 为主线程创建Looper对象,最终仍然是通过prepare()实现
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
}
```

线程拥有自己的Looper 对象,该Looper对象由ThreadLocal对象管理.在线程内第一次调用`prepare()`时便会为其生成Looper对象,并添加到sThreadLocal中.后续每个线程取出的Looper对象,都是从sThreadLocal拿到的.Looper的存在可以很容易让我创建一个单线程串行消息处理模型.

在Looper的构造函数中会创建一个MessageQueue对象.

```java
public final class Looper {   
    
	private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
    
}
```

MessageQueue是典型的单链表队列.其每个元素Message中存在next字段指向下一个Message.之所以采用链接而非数组的原因是该场景是典型插入多的场景,采用链表效率更高.

在MessageQueue的构造函数中会调用`nativeInit()`来在Native层创建与之对应的NativeMessageQueue.同时NativeMessageQueue创建后期被存放在MessageQueue的mPrt中,以便MessageQueue在需要时能快速找到对应的NativeMessageQueue.

```java
public final class MessageQueue {       
	MessageQueue(boolean quitAllowed) {
        mQuitAllowed = quitAllowed;
        mPtr = nativeInit();
    }
}
```

在`android_os_MessageQueue_nativeInit()`中进行NativeMessageQueue创建.

```c++

static jlong android_os_MessageQueue_nativeInit(JNIEnv* env, jclass clazz) {
    NativeMessageQueue* nativeMessageQueue = new NativeMessageQueue();
    if (!nativeMessageQueue) {
        jniThrowRuntimeException(env, "Unable to allocate native queue");
        return 0;
    }

    nativeMessageQueue->incStrong(env);
    return reinterpret_cast<jlong>(nativeMessageQueue);
}
```

这个NativeMessageQueue作用和Java层的MessageQueue类似,并且它也需要一个Looper来不断的循环处理其中的消息.

```c++
NativeMessageQueue::NativeMessageQueue() :
        mPollEnv(NULL), mPollObj(NULL), mExceptionObj(NULL) {
    mLooper = Looper::getForThread();
    if (mLooper == NULL) {
        mLooper = new Looper(false);
        Looper::setForThread(mLooper);
    }
}
```

当然在NativeMessageQueue中创建的Looper也是Native层的.

```c++
Looper::Looper(bool allowNonCallbacks)
    : mAllowNonCallbacks(allowNonCallbacks),
      mSendingMessage(false),
      mPolling(false),
      mEpollRebuildRequired(false),
      mNextRequestSeq(0),
      mResponseIndex(0),
      mNextMessageUptime(LLONG_MAX) {
    // 通过eventfd创建mWakeEventFd,用于实现进程或线程之间的通知,EFD_NONBLOCK|EFD_CLOEXEC
    // 用来将其     设置成非阻塞,且在执行exec时自动关闭描述符
    mWakeEventFd.reset(eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC));
    LOG_ALWAYS_FATAL_IF(mWakeEventFd.get() < 0, "Could not make wake event fd: %s", strerror(errno));

    AutoMutex _l(mLock);
    rebuildEpollLocked();
}
```

在Native层Looper的创建中,涉及到整个Handler体系最核心的地方,首先通过`eventfd()`创建mWakeEventFd.eventfd类似于管道的概念,可以实现进程/线程间的事件通知,但它的效率比传统的管道性能更高.其函数原型如下:

```c++
#include <sys/eventfd.h>
int eventfd(unsigned int initval, int flags);
```

接下主要的逻辑是`rebuildEpollLocked()`:

```c++

void Looper::rebuildEpollLocked() {
    // Close old epoll instance if we have one.
    if (mEpollFd >= 0) {
#if DEBUG_CALLBACKS
        ALOGD("%p ~ rebuildEpollLocked - rebuilding epoll set", this);
#endif
        mEpollFd.reset();
    }

    // 通过epoll_create()创建epoll实例mEpollFd.epoll是一种高效的I/O复用机制,可以检测一个或多个
    // 文件的变化
    mEpollFd.reset(epoll_create(EPOLL_SIZE_HINT));
    LOG_ALWAYS_FATAL_IF(mEpollFd < 0, "Could not create epoll instance: %s", strerror(errno));

    struct epoll_event eventItem;
    memset(& eventItem, 0, sizeof(epoll_event)); // zero out unused members of data field union
    eventItem.events = EPOLLIN;
    eventItem.data.fd = mWakeEventFd.get();
    // 将之前打开的mWakeEventFd注册到epoll中,但mWakeEventFd被写入数据时,epoll会检测的到
    // EPOLL_CTL_ADD表示将fd注册到epoll中
    int result = epoll_ctl(mEpollFd.get(), EPOLL_CTL_ADD, mWakeEventFd.get(), &eventItem);
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not add wake event fd to epoll instance: %s",
                        strerror(errno));

    for (size_t i = 0; i < mRequests.size(); i++) {
        const Request& request = mRequests.valueAt(i);
        struct epoll_event eventItem;
        request.initEventItem(&eventItem);

        int epollResult = epoll_ctl(mEpollFd.get(), EPOLL_CTL_ADD, request.fd, &eventItem);
        if (epollResult < 0) {
            ALOGE("Error adding epoll events for fd %d while rebuilding epoll set: %s",
                  request.fd, strerror(errno));
        }
    }
}
```

上述过程最重要的就是eventfd和epoll的结合使用.通过eventfd来实现进程/线程键效果的消息机制,并将它注册到epoll中,在检测eventfd被写入数据时,epoll就会被唤醒,进而处理这些产生变化的fd.

### Looper获取待执行Message

到这里,关于Handler体系在Java层和Native层作用才说一般.接下来需要之道Looper是如何循环处理MessageQueue的.回到Java,从`Looper#loop()`可见端倪

```java
 public static void loop() {
        final Looper me = myLooper();
        final MessageQueue queue = me.mQueue;
     	.......
        for (;;) {
            // 调用MessageQueue的next()来获取下一个待处理的Message.在获取不到或者
            // Message还未到执行时间时,该方法可能阻塞
            Message msg = queue.next(); // might block
            if (msg == null) {

                return;
            }

            try {
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                
            }
			.......
        }
    }
```

在`loop()`主要在一个循环中不断的调用MessageQueue的`next()`来获取下一个待处理的Message.其中`next()`可能会阻塞.对我们而言需要了解是它是如何实现阻塞的.之前遇到有面试官说这是通过`Object.wait(time)`,兼职颠覆三观.

```java
    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                
                if (msg != null) {
                    // 当前Message还没到该执行的时间,需要计算要等待的时间,以便后面实现阻塞多久
                    if (now < msg.when) {
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // 获取到Message
                        .......    
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }
            }
			.......
        }
    }
```

上述过程主要是取出下一个待执行的Message,如果该Message还未运行时间,那么需要计算需要等待的时间,接下来调用底层方法`nativePollOnce()`,在`nativePollOnce()`会继续调用NativeMessageQueue的`pollOnce()`,在该方法中继续调用Native层Looper的`pollOnce()`,最终又转到Looper的`pollInner()`中:

```c++
int Looper::pollInner(int timeoutMillis) {
	// timeoutMillis即在Java层计算出当前Message到执行时还需要等待的时间
    ......
    struct epoll_event eventItems[EPOLL_MAX_EVENTS];
    // 调用epoll_wait()阻塞timeoutMillis时间,或者直到有人向其监听的mWakeEventFd写入数据
    // 该操作会被唤醒,从阻塞中状态解除,并将唤醒事件填充到eventItems数组中
    int eventCount = epoll_wait(mEpollFd.get(), eventItems, EPOLL_MAX_EVENTS, timeoutMillis);
	......
   
    mPolling = false;
	......
        
    for (int i = 0; i < eventCount; i++) {
        int fd = eventItems[i].data.fd;
        uint32_t epollEvents = eventItems[i].events;
        if (fd == mWakeEventFd.get()) {
            // EPOLLIN事件意味着有数据写入mWakeEventFd中,接下来通过awoken()读取出数据
            // 读取出的数据并没有什么作用.只是因为epoll这里采用水平触发模式,如果不读取出
            // 数据,epoll_wait()下一次就不会进入到等待状态
            if (epollEvents & EPOLLIN) {
                awoken();
            } else {
                .......
            }
        } else {
           ......
        }
    }
	.......

    return result;
}

```

最终限时等待操作由`epoll_wait()`完成,并且epoll在检测到mWakeEventFd中有数据被写入时,`epoll_wait()`就会从阻塞状态中唤醒,并读取出数据,但这数据没什么用.

### Handler发送和处理数据

到现在关于Handler体系中Looper获取Message的流程已经完成.通过Handler发送消息时,最终调用到`sendMessageAtTime()`

```java
    public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
```

接下来通过`enqueueMessage()`将Message添加到对应的MessageQueue中,最终入队方法时`MessageQueue#enqueueMessage()`:

```java
    boolean enqueueMessage(Message msg, long when) {
  		......
        synchronized (this) {
          
            msg.markInUse();
            msg.when = when;
            Message p = mMessages;
            boolean needWake;
            if (p == null || when == 0 || when < p.when) {
                // 如果当mMessages为空或者还新加入的msg执行时间比MMessage靠前,就将新进入的msg设为
                // MessageQueue中的头结点,并唤醒阻塞中的MessageQueue
                msg.next = p;
                mMessages = msg;
                needWake = mBlocked;
            } else {
             	// 主要是讲新加入的Message插入到MessageQueue中合适位置,主要是根据时间进行
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
                for (;;) {
                    prev = p;
                    p = p.next;
                    if (p == null || when < p.when) {
                        break;
                    }
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                msg.next = p; // invariant: p == prev.next
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```

在上述放中,主要做两件事:将新加入的Message插入到MessageQueue中以及通过调用本地nativeWake()唤醒处在阻塞状态的MessageQueue.之前说道阻塞是通过native层的`epoll_wait()`实现,那其唤醒操作同样也需要通过底层来实现.`nativeWake()`最终调用NativeMessageQueue的`wake()`,而该方法最终又调用了Native层的`wake()`:

```c++
void NativeMessageQueue::wake() {
    mLooper->wake();
}
```

```c++
void Looper::wake() {
    uint64_t inc = 1;
    ssize_t nWrite = TEMP_FAILURE_RETRY(write(mWakeEventFd.get(), &inc, sizeof(uint64_t)));
    if (nWrite != sizeof(uint64_t)) {
        if (errno != EAGAIN) {
            LOG_ALWAYS_FATAL("Could not write wake signal to fd %d: %s", mWakeEventFd.get(),
                             strerror(errno));
        }
    }
}
```

要想唤醒处在阻塞中的`epoll_wait()`非常简单.由于mWakeEventFd是通过`eventfd()`创建的用于进程/线程之间通知的fd,同时该fd被epoll监控,因此只需向mWakeEventFd写数据(比如此处写入了一个uint64_t类型数据),就可以唤醒处在阻塞状态的`epoll_wait()`,原来处在阻塞状态的`MessageQueue#next`也会从阻塞中唤醒,Looper就可以继续循环处理MessageQueue了.如果MessageQueue中没有要执行的Message或者需要执行Message还没到执行时间,最终又会进入到阻塞状态,知道下次mWakerEventFd被再次写入数据,即有人通过Handler发送消息.

## Activity与window之间的联系

## AMS相关

## PMS相关

## WMS相关

## Android系统启动流程

Android系统整体启动流程相对简单,但细节很多.从主干来看,其过程如下:

```
Boot Rom -> BootLoader -> Kernal -> init进程 -> Zygote进程 -> SystemServer进程 -> Home Launcher
```

1. 按下开机键,引导芯片从固化在Rom的预设代码开始执行,然后加载Bootloader到RAM
2. Bootloader又称为引导程序,先于操作系统内核运行的程序.主要用来检查RAM,初始化硬件参数,以及加载Kernal并运行
3. Kernal被加载后,首先从start_kernel方法开始执行,进行一系列的初始化.start_kernel函数执行到最后调用了reset_init函数进行后续的初始化.而 reset_init函数最主要的任务就是启动内核线程kernel_init.kernel_init函数将完成设备驱动程序的初始化,并调用init_post函数启动用户空间的init进程.到init_post函数为止,内核的初始化已经基本完成.
4. 当初始化内核之后,会启动init进程,init进程是所有进程的祖先进程.在Android中,init进程负责创建系统最关键的守护进程,比如我们熟知的zygote和servicemanager.前者会创建一个Dalvik虚拟机,它负责启动Java层的进程,后者是Binder通信的基础.此外,Android中的属性系统也有init进程负责.
5. Zygote进程是后续所有Java进程的父进程,它接受来此其他请求创建进程命令,在底层最终通过fork方式生成新进程.在在Zygote启动中,主要做两方面的事:1.在Native层创建并启动虚拟机,注册JNI函数,然后执行Java层的ZygoteInit.类的`main()`,2.在ZygoteInit的`main()`,主要是创建ZygoteServer,以及创建SystemServer进程.
6. SystemServer进程启动时,会创建ActivityThread,通过`createSystemContext()`创建系统上下文,以及初始化很多的系统服务,如AMS,并将这些系统服务添加到ServiceManager.
7. AMS启动完成之后,会调用`finishBooting()`来完成系统引导过程,同时发送开机广播,并向Zygote进程请求创建Launcher进程,来使得桌面显示出来

## Binder原理

## ServiceManager的作用,何时被注册的?

## APP启动流程

## Debug 跟Release的APK的区别

## 匿名Binder有什么用?

## Android的签名机制?V2签名有什么优点?

## Android系统在什么时候通知应用进行GC?

## 如何禁止动态加载代码?

## Bundle传数据为什么要序列化?

## 简述Measurespec这个类

## RecyclerView动画,缓存及数据绑定底层实现

## JNI 原理?

当在代码中调用`System.loadLibrary()`,在虚拟机底层会通过系统调用`dlopen()`动态加载链接库,这是Linux内核原生支持的功能.加载完成后可以通过`dlsym()`从符号表中查找符号对应的符号地址,该地址指向函数的地址,以供调用.除此之外,可以通过'dlclose()'来卸载该链接库.

## so文件加载流程?

## 简述View绘制流程?

![image-20181112003747633](https://i.imgur.com/JAUqCbs.png)

measure()方法,layout(),draw()三个方法主要存放了一些标识符,来判断每个View是否需要再重新测量,布局或者绘制,主要的绘制过程还是在`onMeasure()`,`onLayout()`,`onDraw()`这个三个方法中

- onMesarue(): 为整个View树计算实际的大小,即设置实际的高(对应属性:mMeasuredHeight)和宽(对应属性: mMeasureWidth),每个View的控件的实际宽高都是由父视图和本身视图决定的
- onLayout(): 为将整个根据子视图的大小以及布局参数将View树放到合适的位置上
- onDraw(): 开始绘制图像,绘制的流程如下:
  1. 首先绘制该View的背景
  2. 调用`onDraw()`方法绘制视图本身(每个View都需要重载该方法,ViewGroup不需要实现该方法),如果该View是ViewGroup,调用`dispatchDraw()`方法绘制子视图

## Android权限管理

## Android打包原理

所谓打包就是将工程变为apk的过程.对于Android APK而言,主要由两部分构成:代码文件和资源文件.因此Android的项目的打包过程也就基本分为对两类资源的处理过程.使用V1签名和V2的签名的的打包流程稍有不同,我们以V1签名为例来简述打包流程:

![image-20180807113012986](https://i.imgur.com/BjapIH3.png)

1. 通过AAPT(Android Studio3.0后默认使用AAPT2)工具进行资源(AndroidMainifest.xml,各种xml资源等)处理,该过程生成我们常见的R.java和被处理过测资源文件
2. 通过aidl工具处理aidl文件,生成相应的java文件
3. javac编译工项目源码,生成class文件
4. 通过dex工具处理所有的class文件,将class字节码转为Dalvik字节码,进行压缩等,最终生成Dex文件
5. 通过apkbuilder将资源文件,dex文件进行处理,生成未签名的APK文件
6. 使用Jarsigner工具对该APK进行
7. 使用zipalign对签名过的APK文件进行对齐操作

Android 7.0后新增了V2签名,其签名由apksigner工具完成,其大概流程非常类似上述:

![image-20180807113823444](https://i.imgur.com/YCcZbi0.png)

由于V2签名机制改变,apk的对齐被移到签名之前.即现在是先进行对齐操作然后才会使用apksigner进行签名.

## Android类加载器体系

Android中的虚拟机读取Dex文件,Dex本质就是对Class文件进一步压缩调整,优化,最终为每个API生成class.dex(没有开启多Dex前提下).目前Android主要有以下几种加载器:

![image-20180806152446643](https://i.imgur.com/UWh1Uoi.png)

其中我们常见的是:DexClassLoader和PathClassLoader.两者都是BaseDexClassLoader的子类.区别在于调用父类构造器时,DexClassLoader会传入optimizedDirectory参数.该参数指定的路径(该路径通常为应用的应用的私有目录,以避免注入攻击)用来缓存Dex优化后的文件.

PathClassLoader中该参数为NULL,此时就会使用默认的optimizedDirectory路径`/data/dalvik-cache`,换句话说,两者只是对BaseDexClassLoader传入的构造参数不同而已,从用途上来讲,PathClassLoader用来加载Android系统类和app应用,DexClassLoader用来动态加载包含class.dex的jar/apk文件.虽然我们都知道Dalvik不能识别jar,但在BaseClassLoader中会对jar/zip/apk/dex文件生成一个对应的dex文件,因此最终都是处理的Dex文件.

```java
public class DexClassLoader extends BaseDexClassLoader {
    //dexPath:被解压apk路径;optimizedDirectory:apk被解压后其中dex文件的存储路径
    //librarySearchPath:so库搜索路径;parent:父类加载器
    public DexClassLoader(String dexPath, String optimizedDirectory, String librarySearchPath, ClassLoader parent) {
        super((String)null, (File)null, (String)null, (ClassLoader)null);
        throw new RuntimeException("Stub!");
    }
}
```

```java
public class PathClassLoader extends BaseDexClassLoader {
    public PathClassLoader(String dexPath, ClassLoader parent) {
        super((String)null, (File)null, (String)null, (ClassLoader)null);
        throw new RuntimeException("Stub!");
    }

    public PathClassLoader(String dexPath, String librarySearchPath, ClassLoader parent) {
        super((String)null, (File)null, (String)null, (ClassLoader)null);
        throw new RuntimeException("Stub!");
    }
}
```

注意:

- Android应用启动时,默认父构造器时PathClassLoader,且其父加载器是BootClassLoader.BootClassLoader是PathClassLoader内部类(这和JVM不一样,在JVM中BootClassLoader由C++编写),无法被重写.
- InMemoryDexClassLoader是API 26新增的类加载器
- DelegateLastClassLoader是API 27后新增

## 简述Dalvik下BaseDexClassLoader加载dex的过程

Dex的加载是由BaseDexClassLoader完成的,在够BaseDexClassLoader的构造函数中会创建DexPathList的变量pathList其中在DexPathList的构造方法中会调用`DexPathList#makeDexElements()`,在`makeDexElements()`方法中会调用`loadDexFile()`来对.dex/.jar/.zip/.apk进行处理.

在`loadDexFile()`中,首先通过`optimizedPathFor()`来对optimizedDiretcory路径进行修正,之后通过DexFile的静态方法`loadDex()`正式进行Dex加载,加载完成后将返回一个DexFile对象.该方法内部其实就是调用DexFile的构造方法:

```java
    static DexFile loadDex(String sourcePathName, String outputPathName,
        int flags, ClassLoader loader, DexPathList.Element[] elements) throws IOException {
        return new DexFile(sourcePathName, outputPathName, flags, loader, elements);
    }

	private DexFile(String sourceName, String outputName, int flags, ClassLoader loader,
            DexPathList.Element[] elements) throws IOException {
       	.......
        mCookie = openDexFile(sourceName, outputName, flags, loader, elements);
    }

    private static Object openDexFile(String sourceName, String outputName, int flags,
            ClassLoader loader, DexPathList.Element[] elements) throws IOException {
     
        return openDexFileNative(new File(sourceName).getAbsolutePath(),
                                 (outputName == null)
                                     ? null
                                     : new File(outputName).getAbsolutePath(),
                                 flags,
                                 loader,
                                 elements);
     }

	private static native Object openDexFileNative(String sourceName, 
                                                   String outputName, 
                                                   int flags,
            ClassLoader loader, DexPathList.Element[] elements);

```

在DexFile的构造方法中最重要的是调用调用本地方法`openDexFileNative()`,来打开Dex文件.在`openDexFileNative()`中将分别对后缀为.dex/.zip/.jar/.apk进行处理:

- .dex文件通过`dvmRawDexFileOpen()`函数进行处理

  在`dvmRawDexFileOpen()`函数中,将会检验dex文件的标志,检验odex文件的缓存名称,然后将dex文件拷贝到odex文件中,并对该odex进行优化.最终调用`dvmDexFileOpenFromFd()`对优化后的odex文件进行映射,通过mprotect置为"只读"属性并将映射的内存结构保存在`DvmDex`结构中.

- .zip/.jar/.apk通过`dvmJarFileOpen()`函数进行处理

  在`dvmJarFileOpen()`中首先对文件进行映射,并将其结构保存在ZipArchive中,接下来尝试以文件名作为dex文件名来打开文件,如果打开失败,就调用`dexZipFindEntry()`函数在ZipArchive的名称hash表中找名为"class.dex"的文件,然后创建odex文件,之后的流程就和`dvmRawDexFileOpen()`一样.

上述过程完后后,最终得到DexFile可以认为是Dex优化后的odex在Java层的表示.整个加载过程中最关键的是Native层对Dex的加载及优化,细节较多,先于篇幅不做展开说明,简单总结下,如下所示

```shell
BaseDexClassLoader -> DexPathList#makeDexElements -> DexPathList#loadDexFile
-> DexFile#openDexFile -> DexFile#openDexFileNative
```

## 简述BaseDexClassLoader加载类过程

无论是PathClassLoader和是DexClassLoader,其类加载的流程都是在父类BaseDexClassLoader中实现.

```java
public class BaseDexClassLoader extends ClassLoader {
        private final DexPathList pathList;
 
        public BaseDexClassLoader(String dexPath, File optimizedDirectory,
            String librarySearchPath, ClassLoader parent, boolean isTrusted) {
        super(parent);
        this.pathList = new DexPathList(this, dexPath, librarySearchPath, null, isTrusted);
		......
    }
}
```

在BaseDexClassLoader存在重要的成员变量pathList.pathList是DexPathList类型的,其中它中包含一个DexPathList.Element类型的数组.每个Element代表加载的Dex及优化后Dex文件.

```java
 static class Element {
        // file 是dex的路径
        public final File file;
        public final ZipFile zipFile;
        // dexFile代表dex优化后odex文件
        public final DexFile dexFile;
        
        ......
 }
```

简单来说,DexPathList包含了所有被加载到内存中Dex文件.当需要查找类时,只需要遍历DexPathList中所有的DexFile,然后从DexFile中查找我们需要的类即可,如下:

```java
//  DexPathList#findClass
public Class<?> findClass(String name, List<Throwable> suppressed) {
      for (Element element : dexElements) {
          Class<?> clazz = element.findClass(name, definingContext, suppressed);
          if (clazz != null) {
              return clazz;
          }
      }

      if (dexElementsSuppressedExceptions != null) {
          suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
      }
      return null;
 }

// DexPathList.Element#findClass
public Class<?> findClass(String name, ClassLoader definingContext, List<Throwable> suppressed) {
    // 调用DexFile的loadClassBinaryName()进行查找
     return dexFile != null ? dexFile.loadClassBinaryName(name, definingContext, suppressed)
                    : null;
 }
```

最终类加载的操作转交给DexFile:

```java
public Class loadClassBinaryName(String name, ClassLoader loader, 
                                 List<Throwable> suppressed) {
      return defineClass(name, loader, mCookie, this, suppressed);
}

private static Class defineClass(String name, ClassLoader loader, Object cookie,
                                     DexFile dexFile, List<Throwable> suppressed) {
      Class result = null;
      try {
          // 由本地方法defineClassNative()实现最终加载操作
          result = defineClassNative(name, loader, cookie, dexFile);
       } catch (NoClassDefFoundError e) {
          if (suppressed != null) {
              suppressed.add(e);
          }
       } catch (ClassNotFoundException e) {
          if (suppressed != null) {
              suppressed.add(e);
          }
      }
      return result;
}
```

不难看出最终负责类加载的是DexFile的本地方法`defineClassNative()`.简单总结下:

```shell
BaseDexClassLoader#findClass -> DexPathList#findClass -> Element#findClass -> DexFile#loadClassBinaryName -> DexFile#defineClass -> DexFile#defineClassNative
```

## 简述MultiDex原理

MultiDex仍然是基于DexClassLoader实现的多Dex加载机制.应用在启动后会检查系统版本是否支持MultiDex,检查是否有其他dex需要被安装.如果需要被安装,就会从应用apk中解压出对应的dex,比如classes2.dex,classes3.dex等,并将其拷贝到`/data/data/code_cache/secondary-dexes/`中,然后通过反射的方式将该dex注入到PathClassLoader中的pathList中.

# Android数据结构与设计

### SparseArray实现原理?

SparseArray<E>是用于在Android平台上替代HashMap的数据结构,更准确的说使用去替代key为int类型value为Object类型时使用HashMap的场景.和HashMap相比,在存储数据量小的情况下,SparseArray占用更小的空间且能避免"装箱-拆箱"带来的性能损耗.此外由于它仅仅实现Cloneable接口,因此使用时不兼容Java中提供的Map接口.

在底层数据结构上,它采用了两个数组结构:mKeys和mValues.前者用来存储int类型的键,后者用来存储值.在搜索键时,采用典型的二分查找来实现.以其`put(key,value)`方法为例:

```java
    public void put(int key, E value) {
        // 以二分查找的方式从mKeys中检索key对应数组下标,需要注意二分查找的前提是数组有序
        int i = ContainerHelpers.binarySearch(mKeys, mSize, key);
        if (i >= 0) {
            // 如果该位置已经存在相同的Key,则直接覆盖mValues中对应位置的元素
            mValues[i] = value;
        } else {
            i = ~i;
			// 如果当前要插入位置的元素值为DELETED,直接覆盖即可
            if (i < mSize && mValues[i] == DELETED) {
                mKeys[i] = key;
                mValues[i] = value;
                return;
            }
			// 满容且需要gc时,先通过gc()回收失效的元素
            if (mGarbage && mSize >= mKeys.length) {
                gc();
                // 重新计算要插入的位置
                i = ~ContainerHelpers.binarySearch(mKeys, mSize, key);
            }
			// 插入数据,内部涉及到数组拷贝的过程
            mKeys = GrowingArrayUtils.insert(mKeys, mSize, i, key);
            mValues = GrowingArrayUtils.insert(mValues, mSize, i, value);
            mSize++;
        }
    }
```

在上述代码唯一需要注意的是此处的二分查找在查找失败的情况下不是返回-1:

```java
    static int binarySearch(int[] array, int size, int value) {
        int lo = 0;
        int hi = size - 1;

        while (lo <= hi) {
            final int mid = (lo + hi) >>> 1;
            final int midVal = array[mid];

            if (midVal < value) {
                lo = mid + 1;
            } else if (midVal > value) {
                hi = mid - 1;
            } else {
                return mid;  // value found
            }
        }
        return ~lo;  // value not present
    }
```

最终查找失败返回的~lo,即对lo取反.通过取反之后既能知道查找失败还是成功,又能知道需要插入的位置.比如最后返回值是-5,可以知道此次查找失败,元素需要被存放至数组下标为5的地方.这就是典型的通过取反来让一个数组传达更多的含义的常用做法.

和SparseArray类似的还有以下:

- SparseIntArray: 存储int-int结构
- SpareLongArray: 存储int-long结构
- SpareBooleanArray: 存储int-boolean结构

### ArrayMap的实现原理是什么?

ArrayMap同样Android提供用来在某些场景下替代HashMap的数据结构.和HashMap相比,在数据量较小的情况下,它更节省内存.其实现底层数据结构同样采用双数组实现:mHashes和mArray.前者用来存储Key的Hash值,后者用来存储Key和Value.计算出Key的哈希值之后,会通过二分查找的方式在mHashes中查找索引位置,并计算出Key和Value需要存在mArray数组中的位置:

```java
mHashes[index] = hash;
mArray[index<<1] = key;       // Key在mArray中的位置为2*index
mArray[(index<<1)+1] = value; // VALUE在mArray中的位置为2*index +1
```

比如如果index为0,那么Key和VALUE分别存放在mArray数组中的0和1;如果index为1,那么Key和VALUE分别存放在2和3处

![image-20181115154701038](https://i.imgur.com/rLTP06a.png)

### LruCache底层实现原理?

LruCache底层基于LinkHashMap实现,LinkedHashMap是一种能够保证插入顺序或者访问顺序的数据结构,其底层基于HashMap实现,但通过为每个元素加入了head和tail指针来维持顺序.

LinkedHashMap在构造时可用过指定boolean类型的参数来决定是插入顺序或是访问顺序来保存元素.借助它,我们可以很快的实现一个LruCache:

```java
public class LruCache extends LinkedHashMap {
    private int maxSize;

    public LruCache(int maxSize) {
        super(maxSize, 0.75f, true);
        this.maxSize = maxSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        if (size() > maxSize) {
            return true;
        }
        return false;
    }   
}
```

# 开源框架

## 插件与热修复

### 插件化原理

### 简述资源ID

在构建时,aapt工具会收集所有资源,并为它们分配资源ID.资源ID是占用32位,其规则如下0xPPTTNNNN.

- PP: 用来标记资源所属的apk,占用2字节.默认情况下系统资源PP值为0x01,应用程序PP值为0x7f.
- TT: 用来标记资源的类型,占用2字节,默认从1开始累加.常见的资源类型:attr，drawable，layout，dimen，string，style
- NNNN: 表示某种资源类型中资源的id,占用4字节,默认从0开始,依次累加.

TT和NNNN值由aapt任意分配 ,其规则如下:对于每个新类型,分配和使用下一个可用号码(从1开始);同样对于类型中的每个新名称分配并使用下一个可用编号(从1开始).比如aapt要处理以下资源:

```xml
layout/main.xml
drawable/icon.xml
layout/about.xml
```

1. 当aapt遇到第一个资源类型是layout时,为其分配TT值为1.遇到该类型下第一个布局文件main为其分配NNNN值为1,最终资源ID表示为0x7f010000.
2. 当aapt遇到第二个资源类型是drawable时,为其分配TT值为2.遇到该类型下第一个图片资源icon为其分配值为1,最终资源ID表示为0x7f02000.
3. 当aapt遇到第再次遇到资源类型是layout,由于之前已经存在该类型了,因此TT值仍为1.此时遇到该类型下第二个布局文件为其分配NNNN值为2,最终资源表示为0x07010001.

需要注意,aapt在分配ID过程不会尝试保证两次构建过程这些ID相同.每次资源改变时,资源都会被分配新的ID.在aapt为资源并分配了标识符后,aapt将为项目生成生成R.java文件,并生成一个名为resources.arsc二进制文件.

### 插件化过程如何解决资源ID重复问题?

随着项目复杂性增加难免遇到资源ID重复问题.在之前的某个项目曾经遇到主应用和第三方aar中某个资源冲突导致界面显示问题.在项目进行插件化后,解决资源冲突问题迫在眉睫.在了解了资源ID生成的规则之后,我们可以修改aapt的相关逻辑,实现对资源的固定分组,从而避免重复.

实现资源ID即可通过修改PP段也可以通过修改TT段来实现.比如在编译时,将宿主中layout的TT值固定为33,而插件中layout的TT值固定为03.

### 热修复原理

目前热修复的基本思路有两种,第一种是从类加载角度出发,对比修复前后的dex文件,生成 patch.dex.然后patch.dex更新有问题的dex文件.另一种则是基于Native Hook的思想,通过JNI 层实现对方法进行替换修复粒度为方法级别,可及时生效.

热修复框架对比如下:

| 项目                                                         | 修复粒度 | 原理                                                         |
| ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
| [Dexposed](https://github.com/alibaba/dexposed)              | 方法级别 | 基于 Xposed 实现的无侵入的运行时 AOP框架,不支持A             |
| [AndFix](https://github.com/alibaba/AndFix)                  | 方法级别 | 基于Native Hook 方式,及时修复,只能修复方法,无法添加类和字段,方法参数类型受限,部分机型不支持 |
| [Nuwa](https://github.com/jasonross/Nuwa)                    | 类级别   | 重启后生效,需要为每个类构造函数中添加一段代码,防止打上CLASS_ISPREVERIFIED,但由于兼容问题不建议使用,且作者已经放弃维护 |
| [HotFix](https://github.com/dodola/HotFix)                   | 类级别   | 基本类似Nuwa,作者已经放弃                                    |
| [RocooFix](https://github.com/dodola/RocooFix)               | 类级别   | HotFix升级版,相比之前兼容性更好                              |
| [Tinker_imitator](https://github.com/zzz40500/Tinker_imitator) | Dex级别  | 基于dex文件全量替换的实现原理相对于ClassLoader方式,在性能上有很大优势,重启后生效,但生成新的dex占用内存较大 |
|                                                              |          |                                                              |



## 组件化开发

## 网络框架

### Volley实现原理,简单画出其架构图.

### Volley的优缺点是什么?

### 能否利用Volley下载大文件或者加载大图?

### OkHttp特点是什么?简述其基本架构?

OKHttp是一款高效的HTTP客户端,支持连接同一地址的链接共享同一个socket,能够通过连接池来减小响应延迟,此外还支持透明的GZIP压缩,请求缓存等,其核心涉及路由,连接协议,拦截器,代理,安全性认证,连接池以及网络适配.其中拦截器主要是用于添加,移除或者转换请求或者回应的头部信息

OkHttp整体架构比较简单,但细节处理非常之多.了解OKHttp过程,要从整体把握脉络,借用一张图来简述:

![image-20181202153201028](https://i.imgur.com/VxEiIHy.png)

### OkHttp任务执行方式及设计?

OkHttp支持两种任务执行方式:同步和异步,分别由Dispatcher的`execute()`和`enqueue()`来实现,此外Dispatcher引入了三个任务队列用于实现对任务的管理和存储:

```java
public final class Dispatcher {
  // 就绪队列
  private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
  // 运行队列(异步)
  private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
  // 运行队列(同步)	
  private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
}
```

所有的任务都会交给Dispatcher内部的线程池去处理,它被定义为一个核心线程数为0,最大线程数不限,线程超时时间为60s,并采用SynchronousQueue作为任务队列的线程池.

```java
public final class Dispatcher {
	
  private ExecutorService executorService;

  public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
}    
```

### OkHttp两种拦截器的区别?

OkHttp中支持Application interceptors和Network Interceptors两种拦截器,两者最大的特点是工作时机不同:

![image-20181202155208375](https://i.imgur.com/hzgnef9.png)

对Application Interceptors而言:

- 不需要担心中间过程的响应,如重定向和重试
- 只会被调用一次,即使这个响应是从缓存中获取的
- 只关注最原始的请求,不关心OkHttp注入的头信息如: *If-None-Match*
- 可以中断调用过程,有权决定是否要执行`Chain.proceed()`
- 允许重试,可以多次调用`Chain.proceed()`

对Network Interceptors而言:

- 能够操作中间过程的响应,如重定向和重试.
- 当返回缓存时不被调用.
- 观察在网络上传输的数据变化,比如重定向
- 携带请求来访问连接

### Retrofit的特点是什么?简述其基本架构?

Retrofit是一套快速搭建网络请求的脚手架,底层依赖于OkHttp,其最大的特点是通过接口和注解能快速的实现网络请求接口.此外,Retrofit整个体系架构非常清晰和明确,可以说是设计模式优秀的示范:

![Retrofit](https://i.imgur.com/vFqj8Ma.png)

### Retrofit如何实现动态代理的?

在基于对接口进行代理的情况下,Retrofit采用JDK中提供的Proxy及InvocationHandler来实现动态代理,其过程基本如下:

```java
  public <T> T create(final Class<T> service) {
  	......
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
				......
          }
        });
  }
```

> 使用JDK提供的Proxy来实现动态代理时,其支持接口的形式,如果需要直接对类进行代理它就无能为力了,此时可以选用cglib.

### 你知道Retrofit采用了哪些设计模式?

Retrofit作为设计的最佳典范,其内部使用了多种设计模式来保证整个架构的清晰,具体有:

- 动态代理
- 适配器
- 门面模式
- 策略模式
- 方法工厂和抽象工厂

在之前架构图的基础上,稍作调整整理了如下图:

![Retrofit模式](https://i.imgur.com/bZiRw5Y.png)

## 下载框架

### 如何实现一个支持断点续传的客户端?

实现多线程断点续传需要解决两个问题:

- 对下载内容进行分段,并创建不同的线程分别下载对应段的内容到本地
- 继续下载的进度,以便在任务重启后继续从下载点进行下载

首先来看如何解决第一个问题?在请求下载该文件时,首先获取到下载文件的大小,并为之创建对应的RandomAccessFile.RandomAccessFile是Java中用来支持对文件进行随机读写的类.接下来根据下载文件的大小及线程数计算每个线程下载起始点及大小,然后每个线程在通过Http请求下载时在请求头中设置Range字段来指定要下载文件的起始点即可下线多线程分段下载.

接下来就是要如何实现断点续传.实现断点相对比较简单只需要继续每个线程的已经下载的字节数即可.在下次任务启动时重新计算下载起始位置即可.

## 图片加载框架

### 如何计算一张图片的大小?

在Android中一张图片所占的大小主要和以下几个因数有关:图片长度,图片宽度以及单位像素所占的字节数,计算公式如下:`图片宽度 * 图片高度 * 单位像素所占字节数 `,需要注意此处图片宽高是以像素为单位,其中单位像素所占的字节数和图像的色彩模式有关,目前我们常见的编码方式可分为以下几种:

| 图像色彩模式 | 描述                                           | 单位像素所占内存(单位字节) |
| ------------ | ---------------------------------------------- | -------------------------- |
| ALPHA_8      | 一个像素只有alpha值,没有RGB值,只占用8位        | 1                          |
| ARGB_4444    | 一个像素中透明及红绿蓝分别占用4位              | 2                          |
| ARGB_8888    | 一个像素中透明及红绿蓝分别占用8位              | 4                          |
| RGB_565      | 一个像素中红绿蓝分别占用5,6,5位,不存在透明通道 | 2                          |

举个例子,如果一张宽高都是100像素的图片,如果再用RGB_565进行编码,那么它在内存中所占用的大小为:`100*100*2`=20000字节.

### 图片加载框架的基本实现原理?

图片加载框架是一种特殊的网络框架,不过其专门服务于图片.凡是高效的图片框架都致力于解决以下两个问题:

- 减少图片从网络下载次数
- 提高加载速度

解决以上两个问题的基本方案是引入缓存方案.常见的缓存方案是三级缓存方案,即网络->硬盘->内存:当需要加载一张图片时,首先尝试从内存中读取,读取失败则尝试从硬盘读取,读取失败则从网络读取.从网络中读取成功的图片资源会被缓存在内存和硬盘中.

### Gide原理

对于Glide而言,其基本原理离不开以下几个要点:

- Glide生命周期同步是如何实现的?
- Glide对于图片缓存算法实现原理是什么?

对于生命周期同步问题,Glide采取了非常巧妙的做法,即为当前Activity创建了内部不可见的Fragment,通过监控Fragment的生命周期来实现管理.

对于图片缓存算法,Glide在三级缓存方案的基础上进行扩展,不仅仅考虑减少图片从网络加载次数,还兼顾减少硬盘加载到内存的次数.为此,Glide引入了活动缓存的概念,其基本原理是当前正在使用的图片以弱引用的方式被缓存在内寸中,以减少由于内存LRU空间限制导致图片资源频繁被回收而最终导致多次从硬盘加载的情况.对于活动缓存的回收不是仅仅依赖gc对弱引用的回收,而是对活动缓存中的每个图片资源都采用"引用计数"的算法进行管理,当图片被使用时,其计数器进行+1操作,否则进行-1操作,当引用计数器为0时,就可以将图片资源从活动缓存中删除,并将其移至基于LRU实现的内存缓存中.

### Glide缓存方案与传统的三级缓存方案相比有什么好处?

对于Glide的缓存方案而言,其在典型三级缓存方案的基础上引入了活动缓存的概念.当一张图片首次被加载时,对该资源的查找过程为:`活动缓存 -> LRU内存缓存 -> LRU硬盘缓存 -> 网络`;当图片成功从网络下载后,其写入缓存过程如下:`磁盘缓存 -> 活动缓存 `;一张图片从在使用到不使用时缓存的写入流程:`活动缓存 -> LRU内存缓存`.

尽管Glide缓存流程看起来比较简单,但实际上其提供了非常灵活的配置,允许我们根据自己的需求定制缓存策略,比如针对磁盘缓存,其提供了以下策略:

- DiskCacheStrategy.NONE: 不缓存图片
- DiskCacheStrategy.SOURCE: 只缓存原始图片 
- DiskCacheStrategy.RESULT: 只缓存结果图片,这也是默认的缓存策略)
- DiskCacheStrategy.ALL: 同时缓存原时图片和结果图片
> 什么是原图？什么是结果图？
>
> 当我们使用Glide去加载一张图片的时候,默认情况下Glide并不会将原始图片展示出来,而是会对原始图片进行压缩和转换,之后得到的图片成为结果图.如加载的图片分辨率为1000x1000,但是最终显示该图片的ImageView大小只有300x300,那么Glide就会对原始图片进行处理成我们需要的300x300的小图,并缓存这张小图.



### 缓存方案与LRU算法

无论是内存空间,还是硬盘空间,都是有大小限制的,这意味着图片不能无限制缓存.一张图片什么时候被缓存,什么时候需要从从缓存中删除,这就涉及到具体的缓存算法.缓存算法有很多,比如:

- LFU(Least Frequently Used):最近最少使用算法,即如果一个数据在最近一段时间内使用次数很少，那么在将来一段时间内被使用的可能性也很小.
- LRU(LeastRecently Used):最近最久未使用,即一个数据在最近一段时间没有被访问到,那么在将来它被访问的可能性也很小.也就是说,当限定的空间已存满数据时,应当把最久没有被访问到的数据淘汰.
- FIFO(First in First out): 先进先出,即一个数据最先进入缓存中,则应该最早淘汰掉.也就是说,当缓存满的时候,应当把最先进入缓存的数据给淘汰掉.

由于图片资源的使用情况,在Android的图片缓存算法基本都是基于LRU.针对典型的的三级缓存方案,内存/硬盘中分别使用LruCache和LruDiskCache.

### Glide加载长图与图片背景色

## 响应式框架

## 简述对RxJava框架的理解



## 注解框架

### 简述运行时注解和编译时注解

### BufferKnife框架原理

ButterKnife采用编译时注解,在编译阶段读取源代码,扫描源代码要关注的注解,并生成新的Java源代码.新生成的源代码会一同被编译为字节码文件.

### EventBus实现原理

### 路由框架

#### ARouter

## 内存/性能相关

### LeakCanary实现原理

LeakCanary是用来检测内存泄露的工具,其主要针对于Activity对象产生的内存泄露,当然我们也可以用它来检测其他对象.其实现原理主要涉及两个知识点:一是如何检测对象是否被正常回收,而是如何监控每个Activity的`onDestroy()`的执行以便来触发检测

对于检测对象是否被回收可以通过WeakReference+ReferenceQueue的方式来实现,而监控Activity生命周期则可以通过ActivityLifecycleCallbacks来实现.实际上LeakCanary也正是如此实现的,在检测到Activity的`onDestroy()`后,将当期Activity对象封装在KeyedWeakReference(KeyedWeakReference是基于WeakReference实现的),并关联到ReferenceQueue,然后根据ReferenceQueue中元素的情况来判断是否已经被回收.如果检测到没有回收,那么会主动发起一次GC,并再次检测,如果还没被回收则会dump相关的内存信息进行分析,具体分析原理可见[haha](https://github.com/square/haha).

## JSBridge相关

# Gradle相关

## Gradle的Flavor能否配置sourceset？

## 写过Gradle脚本么?简述Gradle生命周期

Gradle构建过程中,生命周期主要分为三个阶段:

- 初始化阶段: 负责判断有多少个模块参与构建,该阶段主要涉及settings.gradle.
- 配置阶段: 负责对初始化阶段创建的模块完成配置,比如添加Task,修改Task的行为等
- 执行阶段: 根据配置阶段生成的配置执行任务.在构建过程中,Gradle会发布一些事件通知,我们可以监听此类通知并额外做一些事情.

## 了解Gradle Transform么?

[Gradle Transform](http://tools.android.com/tech-docs/new-build-system/transform-api)是Android官方提供给开发者在项目构建阶段(即由class到dex转换期间)修改class文件的一套API,通常我们会用来实现字节码插桩,以及代码注入等工作.

# 应用优化

## 如何压缩APK的大小?

大凡APK,内部无非都离不开两部分:代码和资源.因此优化的角度也要从这两点触发.

- 从代码角度

  良好的编程习惯,进来不要存在重复代码,及时删废弃起代码,谨慎的使用第三方库.善用proguard,开启代码优化和压缩功能.如果有用到so库,仔细考虑要支持的平台架构,比如选择支持armabi.

- 从资源角度

  善用Lint工具,检测无效和重复的资源文件,对于不用的文件及时删除.在使用图片之前,考虑再次进行压缩.很多时候给到的UI资源是很大的,此外根据实际情况来使用哪种图片:png还是jpeg.善用.9图片.如有必要,可以考虑webp.从适配的角度,有选择性的提供hdpi,xhdpi,xxhdpi的图片资源,优先提供xhdpi的图片,其他酌情处理,多复用少添加.能通过代码实现或者变化的图片,就通过代码去做.

## 如何统计应用冷启动时间?

线下统计应用冷启动时间时除了使用高速相机外,还可以用以下命令:

```java
adb shell am start -w packagename/activity
```

```shell
Activity: com.lionoggo.test/.MainActivity
ThisTime: 278
TotalTime: 278
WaitTime: 296
Complete

```

- WaitTime: 表示返回从startActivity到应用第一帧完全显示这段时间. 就是总的耗时,包括前一个应用Activity pause的时间和新应用启动的时间
- ThisTime:表示一连串启动 Activity的最后一个 Activity的启动耗时
- TotalTime: 表示新应用启动的耗时,包括新进程的启动和Activity的启动,但不包括前一个应用Activity pause的耗时

线上统计冷启动时间时,主要需要确定好在哪里记录起始时间.通常在`Application#attachBaseContext()`记录开始,在`Activity#onWindowFocusChanged()`记录结束时间.

## 如何统计帧率?

帧率统计有三种方法,对于已经root的设备可以通过hook住Surfaceflinger中eglSwapBuffers函数;另外也可以通过使用Choreographer中的FrameCallback接口,该接口的`doFrame()`在Choreographer每次接受到VSync信号后会被触发.

当然可以使用以下两个adb命令:

- `adb shell dumpsys gfxinfo <package_name>` : gfxinfo会记录最近128帧的绘制时间,如果每帧不同阶段的时间累加和不超过16.6ms,就可以认为是流畅的.需要注意的是在界面静止不同,或应用内部没有刷新界面的情况下,其值输出为0
- `adb shell dumpsys SurfaceFlinger --latency <window_activity>` :实际记录了最近127帧的数据,静止不动时,输出的将是上一帧的时间.

## 性能优化如何分析systrace？












Android高级

## 系统原理

### 事件分发机制

### Handler原理

###Activity与window之间的联系

### AMS相关

### PMS相关

### WMS相关

### Android系统启动流程

### Binder原理

### ServiceManager的作用,何时被注册的?

### APP启动流程

### PathClassLoader与DexClassLoader的实现原理

### Debug 跟Release的APK的区别

### 匿名Binder有什么用?

###AIDL的实现原理?

### Android的签名机制?V2签名有什么优点?

### Android系统在什么时候通知应用进行GC?

###如何禁止动态加载代码?

### Bundle传数据为什么要序列化?

### 简述Measurespec这个类

### Linearlayou与Relativelayout的绘制原理

### RecyclerView动画,缓存及数据绑定底层实现

###so文件加载流程?

###View是如何被绘制到屏幕上的

### Android权限管理



### Android打包原理

所谓打包就是将工程变为apk的过程.对于Android APK而言,主要由两部分构成:代码文件和资源文件.因此Android的项目的打包过程也就基本分为对两类资源的处理过程.使用V1签名和V2的签名的的打包流程稍有不同,我们以V1签名为例来简述打包流程:

![image-20180807113012986](http://pbj0kpudr.bkt.clouddn.com/blog/2018-08-07-033013.png)

1. 通过AAPT(Android Studio3.0后默认使用AAPT2)工具进行资源(AndroidMainifest.xml,各种xml资源等)处理,该过程生成我们常见的R.java和被处理过测资源文件
2. 通过aidl工具处理aidl文件,生成相应的java文件
3. javac编译工项目源码,生成class文件
4. 通过dex工具处理所有的class文件,将class字节码转为Dalvik字节码,进行压缩等,最终生成Dex文件
5. 通过apkbuilder将资源文件,dex文件进行处理,生成未签名的APK文件
6. 使用Jarsigner工具对该APK进行
7. 使用zipalign对签名过的APK文件进行对齐操作



Android 7.0后新增了V2签名,其签名由apksigner工具完成,其大概流程非常类似上述:

![image-20180807113823444](http://pbj0kpudr.bkt.clouddn.com/blog/2018-08-07-033823.png)

由于V2签名机制改变,apk的对齐被移到签名之前.即现在是先进行对齐操作然后才会使用apksigner进行签名.



### Android类加载器

Android中的虚拟机读取Dex文件,Dex本质就是对Class文件进一步压缩调整,优化,最终为每个API生成class.dex(没有开启多Dex前提下).目前Android主要有以下几种加载器:

![image-20180806152446643](http://pbj0kpudr.bkt.clouddn.com/blog/2018-08-06-072446.png)

其中我们常见的是:DexClassLoader和PathClassLoader.两者都是BaseDexClassLoader的子类.区别在于调用父类构造器时,DexClassLoader会传入optimizedDirectory参数.该参数指定的路径用来缓存Dex的文件(该路径通常为应用的应用的私有目录,以避免注入攻击).PathClassLoader中该参数为NULL,此时就会使用默认的optimizedDirectory路径`/data/dalvik-cache`,换句话说,两者只是对BaseDexClassLoader传入的构造参数不同而已,从用途上来讲,PathClassLoader用来加载Android系统类和app应用,DexClassLoader用来动态加载包含class.dex的jar/apk文件.虽然我们都知道Dalvik不能识别jar,但在BaseClassLoader中会对jar/zip/apk/dex文件生成一个对应的dex文件,因此最终都是处理的Dex文件.

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

- Android应用启动时,默认父构造器时PathClassLoader,且其父加载器是BootClassLoader.BootClassLoader是PathClassLoader内部类,无法被重写.
- InMemoryDexClassLoader是API 26新增的类加载器
- DelegateLastClassLoader是API 27后新增

-------



## 数据结构与设计

### ArrayMap的实现原理是什么?

###LruCache底层实现原理?

### 如何定制一个高效的线程池?线程池中几个参数的定义.

### 插件化原理

### 热修复原理

### 组件化原理?

### 了解Room架构么?

### 如何实现一个支持断点续传的客户端?

### 单利模式如何修改数据?

-----

## 开源框架

### 网络框架

####Volley实现原理,简单画出其架构图.

####Volley的优缺点是什么?

####能否利用Volley下载大文件或者加载大图?

### 图片加载

#### Gide原理

### Glide加载原理

### 缓存方案与LRU算法

### Glide加载长图与图片背景色

### 数据库框架

### 响应式框架

### 注解框架

#### 如何实现一个findViewById注解

### 路由框架

#### ARouter

###内存检测

####LeakCanary实现原理

###其他

####JSBridge实现原理

## Gradle相关

### Gradle的Flavor能否配置sourceset？

### 写过Gradle脚本么?简述Gradle生命周期

##应用优化

### 如何压缩APK的大小?

### 有哪些打渠道包的方案?如何提高打包速度?

### 如何排查内存泄漏问题?

###没有给权限如何定位，特定机型定位失败，如何解决 

### 如何排查ANR问题?

### 现在需要遍历SD卡下所有的文件打印出后缀名为.txt文件名称，如何提高时间效率？










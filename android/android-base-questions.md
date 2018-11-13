# Android基础问题

##  Activity&View系列

### Android布局分类及绘制效率对比

FrameLayout,LinearLayout,AbsoluteLayout,RelativeLayout,,TableLayout以及ConstraintLayout

### 谈谈Activity的生命周期

![image-20181111193213312](https://i.imgur.com/OSxnLEC.png)

Activity实例是由系统自动创建,并在不同的状态期间回调相应的方法.一个最简单的完整的Activity生命周期会按照如下顺序回调：`onCreate -> onStart -> onResume -> onPause -> onStop -> onDestroy`.

- 启动Activity:`onCreate() -> onStart() -> onResume()`,Activity进入运行状态
- Activity退居后台: 当前Activity转到新的Activity界面或按Home键回到主屏:`onPause() -> onStop()`
- Activity返回前台:`onRestart() -> onStart() -> onResume()`,再次回到运行状态

### 介绍不同场景下Activity生命周期的变化过程

假设A Activity位于栈顶,此时用户操作,从A Activity跳转到B Activity.那么对AB来说.具体会回调哪些生命周期中的方法呢？回调方法的具体回调顺序又是怎么样的呢？

1. 当用户点击A中按钮来到B时,假设B全部遮挡住了A,将依次执行A:`onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop`.
2. 此时如果点击Back键,将依次执行`B:onPause -> A:onRestart -> A:onStart -> A:onResume -> B:onStop -> B:onDestroy`

至此,Activity栈中只有A.在Android中,有两个按键在影响Activity生命周期这块需要格外区分下,即Back键和Home键.

- 此时如果按下Back键,系统返回到桌面,并依次执行`A:onPause -> A:onStop -> A:onDestroy`
- 此时如果按下Home键(非长按),系统返回到桌面,并依次执行`A:onPause -> A:onStop`.由此可见Back键和Home键主要区别在于是否会执行onDestroy.

### 开启开发者选项中"不保留活动"对Activity生命周期有什么影响?

开启此设置项后,当A到B时,假设B全部遮挡住了A,将依次执行:

`A:onPause -> B:onCreate -> B:onStart -> B:onResume -> A:onStop -> A:onDestroy`

不难看出,A在系统原本的生命周期回调中增加了onDestroy,此即“用户离开后即销毁每个活动”的含义.需要注意的是,只要没有人为的调用A的`finish()`方法，虽然A执行了onDestroy,但Activity栈中依然保留有A，此时B处于栈顶.

那么在B中按Back键回到A时,将依次执行:

`B:onPause -> A:onCreate -> A:onStart -> A:onResume -> B:onStop -> B:onDestroy`

可以看出A从onCreate开始执行.

### Activity的`finish()`和`onDestory()`联系?

- 对于`finish()`方法:当调用此方法的时候系统只是将最上面的Activity移出了栈,并没有及时的调用`onDestory()`方法,其占用的资源也没有被及时释放.因为移出了栈,所以当你点击手机上面的“back”按键的时候,也不会再找到这个Activity

- 对于`onDestory()`方法:系统销毁了这个Activity的实例在内存中占据的空间.在Activity的生命周期中,onDestory()方法是他生命的最后一步,资源空间等就被回收了.当重新进入此Activity的时候,必须重新创建,执行onCreate()方法

简单来说就是:`finish()`方法用于结束一个Activity的生命周期,而`onDestory()`方法则是Activity的一个生命周期方法,其作用是在一个Activity对象被销毁之前,Android系统会调用该方法,用于释放此Activity之前所占用的资源.

### Activity销毁但Task如果没有销毁掉，当Activity重启时这个AsyncTask该如何解决？

当一个屏幕发生旋转时,当前Activity会被销毁和重建.在当Activity重启时,AsyncTask持有该Activity将失效,因此onPostExecute()不会起作用.若AsynTask正在执行,此时会报 view not attached to window manager 异常.

通常对于生命周期问题,一般的解决思路是将其进行同步,对于AsyncTask而言可以在Activity的`onDestory()`方法中调用`Asyntask.cancel()`方法.

### 内存不足时,系统会杀死后台的Activity,此时如何保存当前Activity的某些状态?如何恢复？

有A,B两个Activity，当从A进入B之后一段时间,可能系统会把A回收,这时候按back,执行的不是A的onRestart而是onCreate方法即A被重新创建一次,这A中的临时数据和状态可能就丢失了.在Activity中,提供了`onSaveInstanceState()/onRestoreInstanceState()`回调方法用于临时保存/恢复数据,其使用实例代码如下:

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if( savedInstanceState != null ){
            savedInstanceState.getString("anAnt");
        }
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        // 将需要的数据保存到outState中
        super.onSaveInstanceState(outState);
    }

	@Override
	public void onRestoreInstanceState(Bundle savedInstanceState) {
        super.onRestoreInstanceState(savedInstanceState);
		// 从savedInstanceState中取出数据
    }

}
```

### `onSaveInstanceState()`的工作原理是什么?

当某个Activity变得“容易”被系统销毁时,该Activity的`onSaveInstanceState()`就会被调用,除非该activity是被用户主动销毁的,例如当用户按BACK键的时候.

那Activity什么时候处在"容易"被销毁时呢?在以下几种情况中,Activity处在易被销毁的状态:

1. 按下HOME键后,该Activity进入后台,由于系统不知道我们按下HOME后要可能要运行多少其他的程序,也就不知道该Activity是否会被销毁,此时系统会调用`onSaveInstanceState()`
2. 长按HOME键,选择运行其他应用程序,原理同上
3. 按下电源按键(关闭屏幕显示)
4. 从Activity A中启动一个新的Activity时,由于系统可能会在某些时机回收掉A,因此系统会调用`onSaveInstanceState()`
5. 屏幕方向切换时,会导致Activity实例重建,因此系统需要调用`onSaveInstanceState()`以便在重建后的Activity显示之前的Activity状态.

简单来说,只要系统未经你许可而销毁了你的Activity是,系统一定会调用`onSaveInstanceState()`.对于该方法,需要注意一下几点:

- 只有你为布局中的每个View设置了id,且实现了其`onSaveInstanceState()`时,系统才会对这个UI的任何改变自动存储及恢复.
- 重写该方法时,最好需要显示调用`super.onSaveInstanceState()`,如果需要额外保存自己的UI数据,那么在进行额外操作.
- 不要滥用`onSaveInstanceState()`,需要谨记其目的是为了保存当前界面的瞬时状态.

### onRestoreInstanceState()的工作原理是什么?

`onRestoreInstanceState(`)被调用的前提是,Activity A“确实”被系统销毁了,而不是处在可能被销毁的状态下.比如当正在显示Activity A的时候,按下HOME键回到主界面,紧接着又返回到Activity A,这种情况下Activity A一般不会因为内存的原因被系统销毁,因此Activity A的`onRestoreInstanceState()`方法不会被执行.

`onRestoreInstanceState()`在`onStart()`后进行回调,此外`onRestoreInstanceState()`中的bundle参数也会传递到`onCreate(Bundle savedInstanceState)`方法中.

那在什么时候进行数据恢复呢?或者说恢复在这两个方法中进行恢复操作有什么不同?当在`onCreate()`中恢复数据时,需要对savedInstanceState进行判空操作,而在`onRestoreInstanceState(Bundle savedInstanceState)`进行数据操作时不需要对savedInstanceState判空.

### 简述Activity任务栈

为了对应用中的Activity进行管理,Google采用了栈结构(后进先出)来管理Activity,即所谓的任务栈.默认情况下,每启动一个应用,系统便会为其创建一个包名为名的任务栈.

默认情况下,一个Activity启动另一个Activity时,两个Activity是放置在同一个task中的,后者被压入前者所在的task栈.当用户按下后退键,后者从task被弹出,前者又显示在幕前,特别是启动其他应用中的Activity时,两个Activity对用户来说就好像是属于同一个应用;系统中task和task之间是互相独立的,当我们运行一个应用时,按下Home键回到主屏,启动另一个应用,这个过程中,之前的task被转移到后台,新的task被转移到前台,其根Activity也会显示到幕前.过了一会之后,在此按下Home键回到主屏,再选择之前的应用,之前的task会被转移到前台,系统仍然保留着task内的所有Activity实例,而那个新的task会被转移到后台,如果这时用户再做后退等动作,就是针对该task内部进行操作了.

处于栈顶的Activity为当前活动栈,处在焦点状态.当按下Back键或调用Activity的`finish()`时,栈内Activity会依次出栈,并调用其onDestory().当所有的Activity被销毁后,系统会回收该栈.

### 请介绍下Activity中的luancheMode

LancheMode即所谓的启动模式,系统会根据该模式来控制Activity打开时对任务栈的影响.该选项有以下四个值选项:standard,singleTop,singleTask,singleInstance.其含义如下:

1. standard:默认模式,每次启动都会创建一个新的activity对象,放到目标任务栈中
2. singleTop:判断当前的任务栈顶是否存在相同的activity对象,如果存在,则直接使用;如果不存在,那么创建新的activity对象放入栈中
3. singleTask:在任务栈中会判断是否存在相同的activity,如果存在,那么会清除该activity之上的其他activity对象显示;如果不存在,则会创建一个新的activity放入栈顶
4. singleIntance:会在一个新的任务栈中创建activity,并且该任务栈种只允许存在一个activity实例,其他调用该activity的组件会直接使用该任务栈种的activity对象

控制任务栈的方式有两种方式,一种是在manifest中配置android:launchMode="standard|singleInstance|single Task|singleTop"来控制Acivity任务栈,另一种则是在启动Activity的Intent中指定Intent Flags:

```java
Intent intent=new Intent();
intent.setClass(MainActivity.this, MainActivity2.class);
intent.addFlags(Intent. FLAG_ACTIVITY_CLEAR_TOP);
startActivity(intent);
```

Flags有很多中,常见的有:

- `Intent.FLAG_ACTIVITY_NEW_TASK`相当于singleTask
- `Intent. FLAG_ACTIVITY_CLEAR_TOP`相当于singleTop

### 请介绍下LaunchMode的使用场景

### 简单描述Activity,Window,View三者的关系

### 如何实现Activity窗口变暗效果,简单写一下代码?

### RemoteView应用及与普通View的区别?

### 简述Scroller原理

在Scroller进行滑动过程中,离不开以下三个方法:

- `Scroller#startScroll()`: 为滑动做了一些初始化准备,如起始坐标,滑动的距离和方向以及持续时间(有默认值),动画开始时间等
- `Scroller#computeScrollOffset()`: 根据当前已经消逝的时间来计算当前的坐标点.由于`startScroll()`设置了动画的时间,因此就可以根据已经消逝的时间计算当前时刻应该所处的位置,其返回结果表示滚动是否结束
- `View#computeScroll()`: View在绘制的过程中,会触发调用该方法

具体来说就是,首先通过Scroller的`startScroll()`方法来进行一些滚动的初始化设置,然后调用View的`invalidate()`或`postInvalidate()`来请求绘制View.在绘制View(即其draw()方法执行)时内部会触发执行`computeScroll()`方法.`computeScroll()`是个空方法,需要我们自己实现.在实现该方法时,需要我们调用`computeScrollOffset()`计算当前的坐标,并根据其返回结果来确定已经滚动完成.如果滚动没有完成就调用`scrollTo()`方法来继续滚动操作.`scrollTo()`虽然会重新绘制View,但仍然需要我们手动调用`invalidate()`或`postInvalidate()`来触发界面重绘.重新绘制View时又会触发`computeScroll()`,这样就进入循环计算当前位置和绘制的过程,直到最终滚动完成,最终就实现了在时间段内平滑滚动的效果.

Scroller典型用法如下所示:

```java

public class OwnView extends LinearLayout {  
    private Scroller mScroller;  
  
    public OwnView(Context context, AttributeSet attrs) {  
        super(context, attrs);  
        mScroller = new Scroller(context);  
    }  
  
    public void toPos(int fx, int fy) {  
        int dx = fx - mScroller.getFinalX();  
        int dy = fy - mScroller.getFinalY();  
        mScroller.startScroll(mScroller.getFinalX(), mScroller.getFinalY(), dx, dy);  
        invalidate();
    }  
  
    @Override  
    public void computeScroll() {  
        if (mScroller.computeScrollOffset()) {  
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());  
            postInvalidate();  
        }  
        super.computeScroll();  
    }  
} 
```







### 简述`requestLayout()`,`onLayout()`,`onDraw()`,`drawChild()`区别与联系

- `requestLayout()`方法 会导致调用`measure()`过程 和`layout()`过程 ,同时会根据标志位判断是否需要`onDraw()`
- `onLayout()`方法,如果该View是ViewGroup对象,需要实现该方法,对每个子视图进行布局
- 调用`onDraw()`方法绘制视图本身,每个View都需要重载该方法,ViewGroup通常不需要实现该方法
- drawChild()去重新回调每个子视图的draw()方法

### 如何解决TextView性能问题?

在有大量图文混排的情况下,使用原生的TextView会出现性能问题.要解决该问题需要了解TextView的实现原理.在TextView中,包含三种布局:

- StaticLayout:当文本为非单行文本,且非Spannable的时候,就会使用StaticLayout.其内部并不会监听span的变化,因此效率上会比DynamicLayout高,只需一次布局的创建即可,但其实内部也能显示SpannableString,只是不能在span变化之后重新进行布局而已
- BoringLayout:主要负责显示单行文本,并提供了isBoring方法来判断是否满足单行文本的条件
- DynamicLayout:多行可编辑复杂文本,当文本为Spannable的时候,TextView就会使用它来负责文本的显示,在内部设置了SpanWatcher,当检测到span改变的时候,会进行reflow,重新计算布局

其中StaticLayout适应于以下场景:

- 文中高频度大量textview刷新优化
- 一个textview显示大量的文本,如阅读app
- 在控件上画文本,比如一个ImageView中心画文本
- 一些排版效果,比如多行文本文字居中对齐等

### LinearLayout和RelativeLayout性能对比?

查阅这两者的`onMeasure(int widthMeasureSpec,int heightMeasureSpec)`实现代码,发现RelativeLayout会对子View进行2次measure,即调用子View 2次onMeasure;而对于LinearLayout如果不使用weight属性,LinearLayout会在当前方向上进行一次measure的过程;如果使用weight属性,LinearLayout会避开设置过weight属性的view做第一次measure,完了再对设置过weight属性的view做第二次measure.

同时,当RelativeLayout的子View高度和RelativeLayout不同,则会引发效率问题,如果子View非常复杂时,那这个问题会更加严重,因此在很多情况下尽量使用padding代替margin.

### 从哪几个角度进行布局优化?

布局优化的首要任务是减少嵌套层次,其次是延迟布局创建.在Android中,提供了三个标签帮助开发者更好进行布局优化:

- `<include>`: 将布局抽取出来,以实现复用
- `<merge>`: 作为`<include>`标签的一种辅助扩展来使用的,它的主要作用是为了防止在引用布局文件时产生多余的布局嵌套
- `<ViewStub>`:有些布局仅在特殊情况下出现,可以通过该标签实现在需要时加载布局

### Android中动画的分类?

Android中动画分为两大类:视图动画和属性动画.其中视图动画又分为逐帧动画和补间动画.属性动画是在Android 3.0之后提供的.

### 属性动画和补间动画的有什么区别?

如果我们只需要对View进行移动,缩放,旋转和淡入淡出操作,那么使用补间动画足够.但如果我们的需求超出了这四种对View的操作,那么补间动画就无能为力了,简单来说就是补间动画功能有限,这具体表现在以下:

- **无法作用于非View对象**: 首先补间动画只能够作用在View上的,简单点说我们可以对任何一个继承自View的组件进行动画操作,但是如果我们想要对一个非View的对象进行动画操作,补间动画就没办法了,举个例子,我们自定义的View时,在这个View当中有一个Point对象用于管理坐标,在`onDraw(`)方法中就是根据这个Point对象的坐标值来进行绘制的,此时我们如果我们可以对这个Point对象进行动画操作的话,那么整个自定义View的动画效果就有了.但是补间动画无法作用于该Point对象
- **动画有限**: 只能够实现移动,缩放,旋转和淡入淡出这四种动画操作.有时候我们希望可以对View的背景色进行动态地改变呢,就无法使用补间动画来实现了.
- **只是显示效果的变化**:这也是补间动画致命的缺陷,就是它只是改变了View的显示效果而已,而不会真正去改变View的属性.什么意思呢？比如说,屏幕的顶端有一个按钮,然后我们通过补间动画将它移动到了屏幕下端,此时点击这个按钮你会发现无法触发点击事件,而在原来位置点击却触发了点击事件.

### 在补间动画执行的过程中,调用其invalidate()方法会出现什么现象?为什么?

### `setX()`和`setTranslationX()`的区别?

`setX()`就是单纯的位置,包含了子View到父View的左边距(例如marginLeft),而`setTranslationX()`则是是View移动的距离

### SurfaceView与View的区别?

- View一般在onDraw方法里面绘图,而onDraw在UI主线程执行.onDraw默认只在View初始化的时候调用一遍,所以View不会自动刷新画面,一般需要调用`invalidate()`或者`postInvalidate()`来重新执行onDraw里面的代码实现刷新画面.UI主线程一般用来渲染组件,处理组件与用户之间的交互事件,比如说按钮的点击事件,文本框的输入事件.

- SurfaceView是一个专门用于做频繁绘制的View子类,内部使用双缓冲机制,它的绘制操作是在子线程中执行,所以频繁绘制不会阻塞线程,使用它去做一些需要频繁绘制和长时间绘制效果会高很多,而如果这种操作放入到View中去做的话就不合适了:首先View的刷新UI操作都是需要在UI线程中,也就是我们的主线程中,如果去执行一些需要长时间绘制,就会造成UI阻塞造成无法响应其他时间.

### 请简单讲述为什么非UI线程不能更新View.

对于大多数图形系统而言,目前基本上都是采用单线程UI模型,这样能够有效的减少多线程更新UI模型造成的锁问题,事实上在早期发展中曾经有基于多线程的UI模型,但最终都因为过于复杂且性能问题最终被淘汰.因此Android非UI线程不能更新View的本质原因是因为其设计就采用单线程UI模型.

### 在哪些场景下需要使用SurfaceView?

根据SurfaceView特性,它一般用在游戏,视频,摄影等一些复杂UI且需要高效的图像的显示的场景,这类的图像处理都需要开单独的线程来处理.

### 介绍下自定义View的基本流程?以实现一个刻度表为例.

### View中setOnTouchListener钟的onTouch,onTouchEvent,onClick的执行顺序?

onTouch优于onTouchEvent,onTouchEvent优于onClick

### 自定义控件这中滑动冲突该如何解决?

### ListView常用的优化策略

对于ListView的优化策略相对比较固定,一般从以下三个角度出发:

1. convertView复用:如子view不改变宽高,重用View可以减少重新分配缓存造成的内存频繁分配/回收压力
2. 使用ViewHolder:使用ViewHolder的原因是`findViewById()`方法耗时较大,在控件个数过多时会严重影响性能，通过使用ViewHolder可以减少控件的查找次数
3. 提高`getView()`效率:在该方法中避免创建大量对象,避免耗时操作
4. 控制异步任务创建频率:在滑动过程中控制好异步的创建,比如在快速滑动过程中,停止图片加载
5. 开启硬件加速:在很多情况下,开启硬件加速提升巨大

### ListView中的ViewHolder为什么要设计成静态内部类？

静态内部类和非静态内部类最大的区别在于非静态内部类会隐式持有外部类的引用.将ViewHolder设计成静态内部能够有效的避免内存泄露.

### ListView跟RecyleView的区别?

## Fragment系列

### 谈谈Fragment的用途以及为什么google要设计Fragment?

### 简述Fragment的生命周期

![image-20181111212552021](https://i.imgur.com/biO4S4B.png)

需要注意和Activity相比,主要增加了以下方法:

- onAttach(),onDetach()
- onCreateView(),onDestroyView()

### Fragment重叠问题遇到过么?为什么?

### Fragment和父Activity通信方式有哪些?



## Service系列

### Service分类

Android服务包括系统服务和应用服务,系统服务是有Android系统提供,并跟随系统启动的服务.根据服务所实现的方式,系统服务又分为Java服务和Native服务,其中Java服务是由Java代码编写而成,由SystemServer进程提供,而本地服务是由C/C++实现的服务,由Init进程在系统启动时启动的服务.我们常说的服务是指应用服务,即有开发者继承Service而实现的特定服务.

对于应用服务而言,根据其运行方式和使用分类,可以分成两类:

1. 按**运行方式**可以分为前台服务和后台服务.所谓的前台服务是指那些经常会被用户关注的服务因,在内存过低时它不会成为被杀的对象.在Android中,前台服务需要提供一个状态栏通知,并会置于“正在进行的”（“Ongoing”）组之下.只有在服务被终止或从前台移除之后,该通知才能被清除.比如对于音乐播放器,通常在状态栏显示当前正在播放的音乐.

2. 按**使用分类**可以分为本地服务和远程服务.本地服务通常用于应用同一进程内执行耗时的任务,而远程服务通常用于多进程之间.

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

IntentService是Service的子类,是一个异步的,会自动停止的服务类,有效解决了传统的Service中处理完耗时操作必须显式调用`stopService()`来停止并销毁Service实例.IntentService内部基于HandlerThread实现,HandlerThread是一个支持串行处理任务的线程.

### Service和Activity通信的手段有哪些?

实现Service和Activity通信的两种常见方式是广播和Binder.

### 说一说你知道绑定Service时常用的flag.

- Context.BIND_AUTO_CREATE: 请求绑定服务时,如果服务尚未创建,则先创建该服务.当系统内存不足需要先销毁组件来释放内存且该服务所在进程成为被销毁对象时,服务才被摧毁
- BIND_DEBUG_UNBIND: 用于调试场景中判断绑定的服务是否正确,但容易引起内存泄漏,一般情况下不建议使用
- Context.BIND_NOT_FOREGROUND: 系统将阻止该服务具有前台优先级,仅在后台运行

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

- 静态注册:在清单文件中注册,常见的有监听设备启动,常驻注册不会随程序生命周期改变 
- 动态注册:在代码中注册,随着程序的结束,也就停止接受广播了

需要注意有些广播只能通过动态方式注册,比如时间变化事件,屏幕亮灭事件,电量变更事件,因为这些事件触发频率通常很高,如果允许后台监听,会导致进程频繁创建和销毁,从而影响系统整体性能.

### 本地广播和全局广播有什么区别?

本地广播在本应用范围内传播,不用担心隐私数据泄露,不用担心别的应用伪造广播.相比全局广播,本地广播更高效.

### 同一个广播分别使用静态注册和动态注册时,谁先收的到?

当广播为普通广播时,无视接收器的优先级:

- 动态广播接收器优先于静态广播接收器
- 同优先级的动态广播接受器,先注册的优先于后注册的
- 同优先级的静态广播接受器,先扫描的优先于后扫描的

当广播为有序广播时,动态广播接收器和静态广播接受器会结合起来处理,排序之后最终确定广播接收器的顺序:

- 优先级的先接受
- 同优先级的,动态广播优先于静态广播接收器
- 同优先级的动态广播接收器,先注册的优先于后注册的
- 同优先级的静态广播接收器,先扫描的优先于后扫描的

### 静态广播和动态广播最终由谁来处理?

静态广播接收的处理器由PackageManagerService负责.当手机启动或者新安装了应用的时候,PackageManagerService会扫描手机中所有已安装的APP应用,将AndroidManifest.xml中有关注册广播的信息解析出来,存储至一个全局静态变量当中.

动态广播接收的处理器由ActivityManagerService负责.当APP的服务或者进程起来之后,执行到注册广播接收的代码逻辑时进行加载,最后会存储在一个另外的全局静态变量中.

### Android为什么要引入广播机制?

## ContentProvider系列

### ContentProvider实现数据共享过程简述?

当一个应用程序需要把自己的数据暴露给其他程序使用时,该就用程序就可通过提供ContentProvider来实现.其他应用程序就可通过ContentResolver来操作ContentProvider暴露的数据.需要注意, 一旦某个应用程序通过ContentProvider暴露了自己的数据操作接口,无论该应用程序是否启动,其他应用程序都可以通过该接口来操作该应用程序的内部数据,包括增加数据,删除数据,修改数据,查询数据等.

ContentProvider以某种Uri的形式对外提供数据,允许其他应用访问或修改数据.其他应用程序使用ContentResolver根据Uri去访问操作指定数据,使用步骤如下： 

1. 定义自己的ContentProvider,该类需要继承Android提供的ContentProvider基类 
2. 为自定义的ContentProvider类在AndroidManifest.xml进行配置`<ContentProvider>`标签,同时需要为它绑定一个Uri, 如 android:authorities=”com.myit.providers.MyProvider” /> ,此处authorities就相当于为该ContentProvider指定Uri. 其他应用程序就可以通过该Uri来访问MyProvider所暴露的数据了.
3. 接下来,就是使用ContentResolver操作数据了.Context提供了`getContentResolver()`来获取ContentResolver对象,然后调用对应的`insert()`,`delete() `,`update()`,`query()`即可.

### ContentProvider如何支持多线程操作?

除了ContentProvider的`onCreate()`运行在UI线程,其他操作都运行在子线程中.

### ContentProvider和SQL的联系与区别?

### ContextProvider启动时机?

当一个应用启动时,最终执行ActivityThread入口方法`main()`.在`main()`中会创建ActivityThread的实例并创建主线程的消息队列,然后在会调用ActivityThread的`attach()`,在该方法中通过远程调用的方式调用ActivityManagerService的`attachApplication()`将ApplicationThread对象提供给ActivityManagerService.

我们知道ApplicationThread是一个Binder对象,它的Binder接口是IApplicationThread,主要用于ActivityThread和ActivityManagerService之间的通讯.在ActivityManagerService的`attachApplication()`中,会收集所有的ContentProvider的信息,这些ContentProvide的信息会在接下来远程调用应用的ApplicationThread的`bindApplication()`中作为参数传递过来,最终调用到`handleBindApplication() `.

在`handleBindApplication()`方法中,ActivityThread首先会创建Application对象并调用其子类`attachBaseContext()`,然后通过`installContentProviders()`创建ContentProvider对象并调用其`onCreate()`,接着将Provider的信息保存到AMS中,完成上述操作后会继续调用Application的`onCreate()`,即:`Application#attachBaseContext() -> ContentProvider#onCreate() -> Application#onCreate()`

## Context系列

### 简述Android中Context的体系?

![image-20181111230945060](https://i.imgur.com/0CzrImC.png)

- Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper.
- 每个Activity和Service以及Application的Context都是一个新的ContextImpl对象
- 尽管Application,Activity,Service都有自己的ContextImpl,并且每个ContextImpl都有自己的mResources成员,但是由于它们的mResources成员都来自于唯一的ResourcesManager实例,所以它们看似不同的mResources其实都指向的是同一块内存.

在一个应用中,Context的数量=Activity的个数 + Service的个数 + 至少1个Application



### 你经常使用Context的哪些方法?

### Context可以手动创建么?在哪些情况下需要手动创建?

## 适配相关

### dp,dip,dpi,px,sp含义及换算公式

### 屏幕适配

### RTL布局适配

### mipmap文件夹和drawable文件加的区别

### 

# 数据存储系列

### 简单介绍你所知的数据存储方式

- 文件存储
- 数据库存储
- 网络存储
- SharedPreferences
- 使用ContentProvider

### 使用SharedPreferences,是用`commit()`还是`apply()`提交数据修改?

`commit()`在提交数据时,其修改会同步写入到磁盘中,并返回一个boolean类型的值.而`apply()`则只是提交了一个异步写入任务后就返回,该方法不会返回执行结果.

当不需要关注修改数据的结果时,优先考虑使用`apply()`而非`commit()`,这样在I/O性能较差的设备上,也可以较好的体验.

需要注意,要尽可能的减少`apply()`调用,原因是因为在每次apply()时,会新增写入任务到QueuedWork(QueuedWork是用来保存和执行SharedPreferences异步写入磁盘任务的串行工作队列).如果该工作队列中任务太多,可能会阻塞Activity中`onPause()`和`onStop()`,进而会阻塞UI线程的操作,如ActivityThread中以下两个方法所示:

```java
    private void handlePauseActivity(IBinder token, boolean finished,
            boolean userLeaving, int configChanges, boolean dontReport, int seq) {
       	......
            
        if (r != null) {
         	......
            // 等待QueuedWork中所有的写入任务执行完
            if (r.isPreHoneycomb()) {
                QueuedWork.waitToFinish();
            }
			......
        }
    }

    private void handleStopActivity(IBinder token, boolean show, int configChanges, int seq) {
        ......

		// 等待QueuedWork中所有的写入任务执行完
        if (!r.isPreHoneycomb()) {
            QueuedWork.waitToFinish();
        }
		......
    }
```



### 如何发布数据库文件到apk中?

将原数据库文件放置项目的` res/raw`中,在应用第一次启动的时候,将该数据文件拷贝到`/data/packagname/databases/`即可.


### SQLite支持事务么?如何有效的提供操作数据库的性能?

### 你对SQLite有优化经验么?如果让你来优化,你会从哪些角度进行?



# 网络相关

### 对WebView熟么?简单讲讲你在使用WebView中遇到过哪些坑?

### 你知道哪些WebView中的漏洞

### XML解析方式和对比?

### 你所知道的网络数据传输协议有哪些?比较一下

### Http工作原理及缺点?

### Http中Etag的用途是什么?

### 如何实现多线程下载框架?简述思路

### 对HTTPS了解么?用户以及原生HttpUrlConnection如何支持HTTPS

### 对网络请求有过优化方案么？有哪些思路？

移动端进行网络优化时一般需要从以下几个角度进行考量:

- 连接复用
- 请求合并
- 减少传输数据大小
- 更加网络情况决定内容显示策略

  ### 如何防止重复发送网络请求？


### 如何解决域名被运营商拦截问题？

### 网页被运行商篡改问题？

### 协议选择json,xml还是protobuf?

 XML是一种扩展标记语言 (Extensible Markup Language, XML) ,是一种结构性的标记语言,可以用来标记数据,定义数据类型,是一种允许用户对自己的标记语言进行定义的源语言. 而JSON只是一种轻量级的数据交换格式.

相比于JSON,XML具有更高的扩展性(毕竟XML是一种语言,而JSON知识一种数据协议而已),JSON能做的事情,XML都能做.但对于传输相同语义的数据时,JSON所占的带宽小于XML,传输速度更快.另外由于JSON数据设计简单,通常其解析速度更快.

Protobuf是一种序列化协议,能够对结构化数据进行序列化.被序列化的数据已二进制的格式进行传输.相比于JSON/XML,Protobuf生成序列化数据占用空间更小且更高效,除此之外,由于序列化的数据已二进制的方式进行传输,因此具有很高保密性,比如当你看到下述二进制字符串时,能立刻理解么?

```java
1001111011000001110111111001011001000001010011101110000001111101101101100110101100110111001101011100101000101010011101010100100011100110000111111
```

当前,对于移动端而言,多数采用的是JSON作为网络数据传输协议,如果追求更高的安全性和性能,那么可以采用Protobuf.

# 跨进程通信

### Linux中常见的进程通信方式有哪些呢?Android呢?

### AIDL的实现原理?

AIDL(Android Interface Definition Language)即我们所说的Android接口定义语言.我们知道Android中提供了Binder作为跨进程通信的机制,但直接基于Binder相关类进行开发的前提是需要开发者具有相关的领域知识,这对开发者并不友好.为了简化开发难度,Android定义了一种简单描述语言,开发者只需要根据规则来描述自己想做的事即可,这种语言就是我们说的AIDL.使用AIDL描述的信息,将在编译阶段自动被翻译为相关Binder类.

### 如何实现跨进程回调?

# NDK开发

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







## 调试与测试

### 如何调试Android应用程序？so调试呢？

### 如何进行压力测试？从数据库和网络两方介绍下

### 如何进行UI性能测试？如何试用Android Profiler排查？

### 如何进行内存测试？

### 如何进行单元测试?

# 常见问题分析

## ANR问题

### ANR类型?

要排查ANR首先要明确ANR发生的原因,其本质是在主线程进行耗时操作,一般有三种类型:

- KeyDispatchTimeout:主要是按键或触摸事件在特定时间内(5秒)无响应
- BoadcastTimeout:BroadcastReceiver在特定时间内(前台广播10秒,后台60秒)无法处理完成
- ServiceTimeout:Service在特定的时间内(前台Service 20秒,后台Service200秒)无法处理完成

### ANR产生的原因?

针对开发场景,ANR一般产生的原因一般有以下几种:

1. 主线程中进行网络访问
2. 主线程大量的数据读写
3. 主线程中进行数据库操作,如打开数据操作
4. 系统资源吃紧,比如CPU,导致主线程无法得到时间片而无法执行
5. .硬件操作（比如camera)
6. 调用thread的join()方法、sleep()方法、wait()方法或者等待线程锁的时候
7. service binder的数量达到上限
8. system server中发生WatchDog ANR
9. service忙导致超时无响应
10. 其他线程持有锁，导致主线程等待超时
11. 其它线程终止或崩溃导致主线程一直等待

### 如何避免ANR?

要解决ANR问题,需要从两方面下手:避免主线程做耗时操作及优化APP性能,减少资源占用.凡是涉及IO操作,如网络请求,文件读写,数据库操作的地方都要尽力避免发生在主线程.优化APP性能,能够从全局的角度减少对系统的占用,使得系统良好的工作.

### 如何排查ANR?

ANR发生时都会在log中输出错误信息,从log中可以获得ANR的类型,CPU的使用情况,CPU使用率情况.

对于开发阶段产生的ANR,首先可以通过跟踪日志输出及审读代码的方式进行排查,有条件的可以使用APM追踪应用运行状态.另外,系统会在data/anr目录下生成trace.txt来保存anr日志,除了以上两种方式外,还需要关注系统负载情况,有些ANR很可能是在系统资源极其吃紧的情况下造成的,比如CPU饥饿导致的ANR.

## 内存泄露

### 如何追踪某个对象的回收情况?

### 如何评估应用是否发生了内存泄漏？

### 内存泄漏产生的原因是什么？如何避免？

### 如何排查OOM么？OOM的产生的原因是什么？

### 如何制造一个OOM的场景？

### 如何避免OOM？




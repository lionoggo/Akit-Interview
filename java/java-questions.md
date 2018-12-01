# 基础概念

## 面向对象的三个特征

封装,继承,多态.这个应该是人人皆知.有时候也会加上抽象.

## 多态的好处

允许不同类对象对同一消息做出响应,即同一消息可以根据发送对象的不同而采用多种不同的行为方式(发送消息就是函数调用).主要有以下优点:

1. 可替换性:多态对已存在代码具有可替换性.
2. 可扩充性:增加新的子类不影响已经存在的类结构.
3. 接口性:多态是超类通过方法签名,向子类提供一个公共接口,由子类来完善或者重写它来实现的.
4. 灵活性:
5. 简化性:

## 虚拟机是如何实现多态的

动态绑定技术(dynamic binding),执行期间判断所引用对象的实际类型,根据实际类型调用对应的方法.如果你知道Hotspot中oop-klass模型的实现,对这个问题就了解比较深了.

## 接口的意义

接口的意义用三个词就可以概括:规范,扩展,回调.
## 抽象类的意义
抽象类的意义可以用三句话来概括:

 1. 为其他子类提供一个公共的类型
 2. 封装子类中重复定义的内容
 3. 定义抽象方法,子类虽然有不同的实现,但是定义时一致的

## 接口和抽象类的区别

|比较点|抽象类|接口|
|----|------|----|
|默认方法|抽象类可以有默认的方法实现|java 8之前,接口中不存在方法的实现|
|实现方式|子类使用extends关键字来继承抽象类.如果子类不是抽象类,子类需要提供抽象类中所声明方法的实现|子类使用implements来实现接口,需要提供接口中所有声明的实现.|
|构造器|抽象类中可以有构造器|接口中不能|
|和正常类区别|抽象类不能被实例化|接口则是完全不同的类型|
|访问修饰符|抽象方法可以有public,protected和default等修饰|接口默认是public,不能使用其他修饰符|
|多继承|一个子类只能存在一个父类|一个子类可以存在多个接口|
|添加新方法|抽象类中添加新方法,可以提供默认的实现,因此可以不修改子类现有的代码|如果往接口中添加新方法,则子类中需要实现该方法|
||||
## 父类的静态方法能否被子类重写?

不能.重写只适用于实例方法,不能用于静态方法,而子类当中含有和父类相同签名的静态方法,我们一般称之为隐藏.

## 什么是不可变对象?好处是什么?

不可变对象指对象一旦被创建,状态就不能再改变,任何修改都会创建一个新的对象,如 String、Integer及其它包装类.不可变对象最大的好处是线程安全.

## 静态变量和实例变量的区别?

静态变量存储在方法区,属于类所有.实例变量存储在堆当中,其引用存在当前线程栈.需要注意的是从JDK1.8开始用于实现方法区的PermSpace被MetaSpace取代了.


## 能否创建一个包含可变对象的不可变对象?
当然可以,比如`final Person[] persons = new Persion[]{}`.persons是不可变对象的引用,但其数组中的Person实例却是可变的.这种情况下需要特别谨慎,不要共享可变对象的引用.这种情况下,如果数据需要变化时,就返回原对象的一个拷贝.

## java 创建对象的几种方式

java中提供了以下四种创建对象的方式:

- new创建新对象
- 通过反射机制
- 采用clone机制
- 通过序列化机制

前两者都需要显式地调用构造方法.   对于clone机制,需要注意浅拷贝和深拷贝的区别,对于序列化机制需要明确其实现原理,在java中序列化可以通过实现Externalizable或者Serializable来实现.

## switch中能否使用string做参数?

在JDK 1.7之前,switch只能支持byte,short,char,int或者其对应的包装类以及Enum类型.从JDK 1.7之后switch开始支持String类型.但到目前为止,switch都不支持long类型.

## Object中有哪些公共方法?
`equals()`,`clone()`,`getClass()`,`notify(),notifyAll(),wait()`,`toString`

## java中`==`和`eqauls()`的区别?

- `==`是运算符,用于比较两个变量是否相等,对于基本类型而言比较的是变量的值,对于对象类型而言比较的是对象的地址.
- `equals()`是Object类的方法,用于比较两个对象内容是否相等.默认Object类的`equals()`实现如下:

```java
public class Object {
    ......
        
        public boolean equals(Object obj) {
        return (this == obj);
    }
    ......
}
```

不难看出此时`equals()`是比较两个对象的地址,此时直接`==`比较的的结果一样.对于可能用于集合存储中的对象元素而言,通常需要重写其`equals()`方法.

## `a==b`与`a.equals(b)`有什么区别

如果a 和b 都是对象,则 `a==b` 是比较两个对象内存地址,只有当 a 和 b 指向的是堆中的同一个对象才会返回 true.而 `a.equals(b)` 是进行内容比较,其比较结果取决于`equals()`具体实现.多数情况下,我们需要重写该方法,如String 类重写 `equals() `用于两个不同对象，但是包含的字母相同的比较:

```java
    public boolean equals(Object anObject) {
        if (this == anObject) {						// 同一个对象直接返回true
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {      		  // 按字符依次比较
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```



## Object中的`equals()`和`hashcode()`的联系

`hashCode()`是Object类的一个方法,返回一个哈希值.如果两个对象根据equal()方法比较相等,那么调用这两个对象中任意一个对象的hashCode()方法必须产生相同的哈希值;如果两个对象根据eqaul()方法比较不相等,那么产生的哈希值不一定相等(碰撞的情况下还是会相等的.)

## a.hashCode()有什么用?与a.equals(b)有什么关系
`hashCode()`方法是为对象产生整型的 hash 值,用作对象的唯一标识.它常用于基于 hash 的集合类,如 Hashtable,HashMap等等.根据 Java 规范,使用 `equal() `方法来判断两个相等的对象,必须具有相同的 hashcode.

将对象放入到集合中时,首先判断要放入对象的hashcode是否已经在集合中存在,不存在则直接放入集合.如果hashcode相等,然后通过`equal()`方法判断要放入对象与集合中的任意对象是否相等:如果`equal()`判断不相等,直接将该元素放入集合中,否则不放入.

## 有没有可能两个不相等的对象有相同的hashcode

有可能.在产生hash冲突时,两个不相等的对象就会有相同的 hashcode 值.当hash冲突产生时,一般有以下几种方式来处理:

- 拉链法:每个哈希表节点都有一个next指针,多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表进行存储.
- 开放定址法:一旦发生了冲突,就去寻找下一个空的散列地址,只要散列表足够大,空的散列地址总能找到,并将记录存入 
- 再哈希:又叫双哈希法,有多个不同的Hash函数.当发生冲突时,使用第二个,第三个….等哈希函数计算地址,直到无冲突.


## 可以在hashcode中使用随机数字吗?
不行,因为同一对象的 hashcode 值必须是相同的.

## & 和 &&的区别

基础的概念不能弄混:&是位操作,&&是逻辑运算符.需要记住逻辑运算符具有短路特性,而&不具备短路特性.来看看一下代码执行结果?
```java
public class Test{
    static String name;

    public static void main(String[] args){
        if(name!=null&userName.equals("")){
            System.out.println("ok");
        }else{
            System.out.println("erro");
        }
    }
}

```

上述代码将会抛出空指针异常.原因你懂得.

## 在.java文件内部可以有多少类(非内部类)?

在一个java文件中只能有一个public公共类,但是可以有多个default修饰的类.

## 如何正确的退出多层嵌套循环?

1. 使用标号和break;
2. 通过在外层循环中添加标识符

## 内部类有什么作用?

内部类可以有多个实例,每个实例都有自己的状态信息,并且与其他外围对象的信息相互独立.在单个外围类当中,可以让多个内部类以不同的方式实现同一接口,或者继承同一个类.创建内部类对象的时刻不依赖于外部类对象的创建.内部类并没有令人疑惑的”is-a”关系,它就像是一个独立的实体.此外,内部类提供了更好的封装,除了该外围类,其他类都不能访问.

## `final`,`finalize()`和`finally{}`的不同之处

三者没有任何相关性,遇到有问着问题的面试官就拖出去砍了吧.final是一个修饰符,用于修饰变量,方法和类.如果 final 修饰变量,意味着该变量的值在初始化后不能被改变.`finalize() `方法是在对象被回收之前调用的方法,给对象自己最后一个复活的机会.但是该方法由Finalizer线程调用,但调用时机无法保证.finally是一个关键字,与 try和catch一起用于异常的处理,`finally{}`一定会被执行,在此处我们通常用于资源关闭操作.

## `clone()`是哪个类的方法?

java.lang.Cloneable 是一个标示性接口,不包含任何方法.`clone ()`方法在 Object 类中定义的一个Native方法:

```java
protected native Object clone() throws CloneNotSupportedException;
```



## 深拷贝和浅拷贝的区别是什么?
- 浅拷贝:被复制对象的所有变量都含有与原来的对象相同的值,而所有的对其他对象的引用仍然指向原来的对象.换言之,浅拷贝仅仅复制所考虑的对象,而不复制它所引用的对象.

- 深拷贝:被复制对象的所有变量都含有与原来的对象相同的值.而那些引用其他对象的变量将指向被复制过的新对象.而不再是原有的那些被引用的对象.换言之.深拷贝把要复制的对象所引用的对象都复制了一遍.

  ![image-20181023180427459](https://i.imgur.com/mBrnqBT.png)

## static都有哪些用法?
所有的人都知道static关键字这两个基本的用法:静态变量和静态方法.也就是被static所修饰的变量/方法都属于类的静态资源,类实例所共享.

除了静态变量和静态方法之外,static也用于静态块,多用于初始化操作:
```java
public calss PreCache{
    static{
        //执行相关操作
    }
}
```
此外static也多用于修饰内部类,此时称之为静态内部类.

最后一种用法就是静态导包,即`import static`.import static是在JDK 1.5之后引入的新特性,可以用来指定导入某个类中的静态资源,并且不需要使用类名,可以直接使用资源名,比如:
```java
import static java.lang.Math.*;

public class Test{

    public static void main(String[] args){
        //System.out.println(Math.sin(20));传统做法
        System.out.println(sin(20));
    }
}
```

## final有哪些用法?

final也是很多面试喜欢问的地方,但我觉得这个问题很无聊,通常能回答下以下5点就不错了:

- 被final修饰的类不可以被继承
- 被final修饰的方法不可以被重写
- 被final修饰的变量不可以被改变.如果修饰引用,那么表示引用不可变,引用指向的内容可变.
- 被final修饰的方法,JVM会尝试将其内联,以提高运行效率
- 被final修饰的常量,在编译阶段会存入常量池中.

除此之外,编译器对final域要遵守的两个重排序规则更好:

- 在构造函数内对一个final域的写入,与随后把这个被构造对象的引用赋值给一个引用变量,这两个操作之间不能重排序
- 初次读一个包含final域的对象的引用,与随后初次读这个final域,这两个操作之间不能重排序.

# 数据类型相关

## java中int char,long各占多少字节?

这个问题其实很无聊.应该去问java中的各种数据类型在不同的平台运行时期所占位数一样么?

|类型|字节|
|-|-|
|short|2|
|int|4|
|long|8|
|float|4|
|double|8|
|char||
|||



## 64位的JVM当中,int的长度是多少?
Java中数据类型所占用的位数和平台无关,在 32 位和64位 的Java 虚拟机中,int 类型的长度都是占4字节.

## int和Integer的区别?

Integer是int的包装类型,在拆箱和装箱中,二者自动转换.int是基本类型,直接存数值;而integer是对象;用一个引用指向这个对象.由于Integer是一个对象,在JVM中对象需要一定的数据结构进行描述,相比int而言,其占用的内存更大一些.

## `String s = new String("abc")`创建了几个String对象?

2个.一个是字符串字面常数,在字符串常量池中;另一个是new出来的字符串对象,在堆中.

## 请问s1`==`s3是true还是false，s1`==`s4是false还是true?s1`==`s5呢？

```java
   String s1 = "abc";
   String s2 = "a";
   String s3 = s2 + "bc";
   String s4 = "a" + "bc";
   String s5 = s3.intern();
```

s1`==`s3返回false,s1`==`s4返回true,s1`==`s5返回true.

"abc"这个字符串常量值会直接方法字符串常量池中,s1是对其的引用.由于s2是个变量,编译器在编译期间无法确定该变量后续会不会改,因此无法直接将s3的值在编译器计算出来,因此s3是堆中"abc"的引用.因此s1!=s3.对于s4而言,其赋值号右边是常量表达式,因此可以在编译阶段直接被优化为"abc",由于"abc"已经在字符串常量池中存在,因此s4是对其的引用,此时也就意味s1和s4引用了常量池中的同一个"abc".所以s1`==`s4.String中的`intern()`会首先从字符串常量池中检索是否已经存在字面值为"abc"的对象,如果不存在则先将其添加到字符串常量池中,否则直接返回已存在字符串常量的引用.此处由于"abc"已经存在字符串常量池中了,因此s5和s1引用的是同一个字符串常量.



## 以下代码中,s5`==`s2返回值是什么?

```java
String s1="ab";
String s2="a"+"b";
String s3="a";
String s4="b";
String s5=s3+s4;
```

 返回false.在编译过程中,编译器会将s2直接优化为"ab",将其放置在常量池当中;而s5则是被创建在堆区,相当于s5=new String("ab");

## 你对String对象的intern()熟悉么?

 Stirng中的`intern()`是个Native方法,它会首先从常量池中查找是否存在该常量值的字符串,若不存在则先在常量池中创建,否则直接返回常量池已经存在的字符串的引用. 比如

```java
 String s1="aa";
 String s2=s1.intern();
 System.out.print(s1==s2);
```

上述代码将返回true.因为在"aa"会在编译阶段确定下来,并放置字符串常量池中,因此最终s1和s2引用的是同一个字符串常量对象.

## String,StringBuffer和StringBuilder区别?

String是字符串常量,final修饰;StringBuffer字符串变量(线程安全);StringBuilder 字符串变量(线程不安全).此外StringBuilder和StringBuffer实现原理一样,都是基于数组扩容来实现的.

## String和StringBuffer的区别?

String和StringBuffer主要区别是性能:String是不可变对象,每次对String类型进行操作都等同于产生了一个新的String对象,然后指向新的String对象.所以尽量不要对String进行大量的拼接操作,否则会产生很多临时对象,导致GC开始工作,影响系统性能.

StringBuffer是对象本身操作,而不是产生新的对象,因此在有大量拼接的情况下,我们建议使用StringBuffer(线程安全).

需要注意现在JVM会对String拼接做一定的优化,比如

```java
String s="This is only "+ "simple" +"test";
```

以上代码在编译阶段会直接被优化成会`String s="This is only simple test".

## StringBuffer和StringBuilder

StringBuffer和StringBuilder的实现原理一样,其父类都是AbstractStringBuilder.StringBuffer是线程安全的,StringBuilder是JDK 1.5新增的,其功能和StringBuffer类似,但是非线程安全.因此,在没有多线程问题的前提下,使用StringBuilder会取得更好的性能.

## 什么是编译器常量?使用它有什么风险?

公共静态不可变,即public static final修饰的变量就是我们所说的编译期常量.这里的public可选的.实际上这些变量在编译时会被替换掉,因为编译器明确的能推断出这些变量的值(如果你熟悉C++,那么这里就相当于宏替换).

编译器常量虽然能够提升性能,但是也存在一定问题:你使用了一个内部的或第三方库中的公有编译时常量,但是这个值后面被其他人改变了,但是你的客户端没有重新编译,这意味着你仍然在使用被修改之前的常量值.

## 3*0.1`==`0.3返回值是什么

false,因为有些浮点数不能完全精确的表示出来.

## java当中使用什么类型表示价格比较好?
如果不是特别关心内存和性能的话,使用BigDecimal.否则使用预定义精度的 double 类型.

## 如何将byte转为String

可以使用String接收 byte[] 参数的构造器来进行转换,注意要使用的正确的编码,否则会使用平台默认编码.这个编码可能跟原来的编码相同.也可能不同.


## 可以将int强转为byte类型么?会产生什么问题?
可以做强制转换,但是Java中int是32位的而byte是8 位的.如果强制转化int类型的高24位将会被丢弃,byte 类型的范围是从-128到128.

## a=a+b与a+=b有什么区别吗?

`+=`操作符会进行隐式自动类型转换,此处a+=b隐式的将加操作的结果类型强制转换为持有结果的类型,而a=a+b则不会自动进行类型转换.如：

```java
byte a = 127;
byte b = 127;
b = a + b; // 报编译错误:cannot convert from int to byte
b += a; 
```

##  以下代码是否有错,有的话怎么改？

```java
short s1= 1;
s1 = s1 + 1;
```

有错误.short类型在进行运算时会自动提升为int类型,也就是说`s1+1`的运算结果是int类型,而s1是short类型,此时编译器会报错.

## 以下代码是否有错,有的话怎么改？

```java
short s1= 1; 
s1 += 1; 
```

+=操作符会对右边的表达式结果强转匹配左边的数据类型,所以没错.



## 了解泛型么?简述泛型的上界和下界?

有时候希望传入的类型有一个指定的范围，从而可以进行一些特定的操作,这时候就需要通配符了?在Java中常见的通配符主要有以下几种:

- `<?>`: 无限制通配符
- `<? extends E>`: extends 关键字声明了类型的上界,表示参数化的类型可能是所指定的类型,或者是此类型的子类
- `<? super E>`: super关键字声明了类型的下界,表示参数化的类型可能是指定的类型,或者是此类型的父类

它们的目的都是为了使方法接口更为灵活,可以接受更为广泛的类型.

- `< ? extends E>`: 用于灵活**读取**，使得方法可以读取 E 或 E 的任意子类型的容器对象。
- `< ? super E>`: 用于灵活**写入或比较**,使得对象可以写入父类型的容器,使得父类型的比较方法可以应用于子类对象。

用简单的一句话来概括就是为了获得最大限度的灵活性,要在表示生产者或者消费者的输入参数上使用通配符,使用的规则就是:生产者有上限(读操作使用extends),消费者有下限(写操作使用super).

# 垃圾回收

## 简单的解释一下垃圾回收?

JVM中垃圾回收机制最基本的做法是分代回收.内存中的区域被划分成不同的世代,对象根据其存活的时间被保存在对应世代的区域中.一般的实现是划分成3个世代:年轻,年老和永久代.所有新生成的对象优先放在年轻代的(大对象可能被直接分配在老年代,作为一种分配担保机制),年轻代按照统计规律被分为三个区:一个Eden区，两个 Survivor区.在年轻代中经历了N次垃圾回收后仍然存活的对象,就会被放到年老代中.因此可以认为年老代中存放的都是一些生命周期较长的对象. 

方法区也被称为永久代,用于存储每一个java类的结构信息:比如运行时常量池,字段和方法数据,构造函数和普通方法的字节码内容以及类,实例,接口初始化时需要使用到的特殊方法等数据,根据虚拟机实现不同,GC可以选择对方法区进行回收也可以不回收.

对于不同的世代可以使用不同的垃圾回收算法。比如对由于年轻代存放的对象多是朝生夕死,因此可以采用标记-复制,而对于老年代则可以采用标记-整理/清除.

### Minor GC

发生在新生代的GC为Minor GC .在Minor GC时会将新生代中还存活着的对象复制进一个Survivor中,然后对Eden和另一个Survivor进行清理.所以,平常可用的新生代大小为Eden的大小+一个Survivor的大小.

### Major GC

在老年代中的GC则为Major GC.

### Full GC

通常是和Major GC等价的,针对整个新生代,老年代,元空间metaspace(java8以上版本取代perm gen)的全局范围的GC.

关于GC的类型,其实依赖于不同的垃圾回收器.可以具体查看相关垃圾回收器的实现.

### 新生代进入老年代

- 分配担保机制:当Minor GC时,新生代存活的对象大于Survivor的大小时,这时一个Survivor装不下它们,那么它们就会进入老年代.
- 如果设置了-XX：PretenureSizeThreshold5M 那么大于5M的对象就会直接就进入老年代.
- 在新生代的每一次Minor GC 都会给在新生代中的对象+1岁,默认到15岁时就会从新生代进入老年代,可以通过-XX：MaxTenuringThreshold来设置这个临界点

## 常见的垃圾回收算法有哪些?简述其原理.

垃圾回收从理论上非常容易理解,具体的方法有以下几种:
 1. 标记-清除
 2. 标记-复制
 3. 标记-整理
 4. 分代回收
 更详细的内容参见[深入理解垃圾回收算法](http://blog.csdn.net/dd864140130/article/details/50084471)

## 如何判断一个对象是否应该被回收?

这就是所谓的对象存活性判断,常用的方法有两种:

- 引用计数法

- 对象可达性分析

  由于引用计数法存在互相引用导致无法进行GC的问题,所以目前JVM虚拟机多使用对象可达性分析算法.

## 哪些对象可以做GC Root?

主要由以下四种:

- JVM方法栈中引用的对象
- 本地方法栈中引用的对象
- 方法区常量引用的对象
- 方法区类属性引用的对象


## 调用System.gc()会发生什么?
通知GC开始工作,但是GC真正开始的时间不确定.

## 了解java当中的四种引用类型?他们之间的区别是什么?

在java中主要有以下四种引用类型:强引用,软引用,弱引用,虚引用.不同的引用类型主要体现在GC上:

- 强引用:如果一个对象具有强引用,它就不会被垃圾回收器回收.即使当前内存空间不足,JVM也不会回收它.而是抛出 OutOfMemoryError 错误.使程序异常终止.如果想中断强引用和某个对象之间的关联.可以显式地将引用赋值为null,这样一来的话.JVM在合适的时间就会回收该对象.

- 软引用:在使用软引用时,如果内存的空间足够,软引用就能继续被使用而不会被垃圾回收器回收.只有在内存不足时,软引用才会被垃圾回收器回收.

- 弱引用:具有弱引用的对象拥有的生命周期更短暂.因为当 JVM 进行垃圾回收,一旦发现弱引用对象,无论当前内存空间是否充足,都会将弱引用回收.不过由于垃圾回收器是一个优先级较低的线程,所以并不一定能迅速发现弱引用对象.

- 虚引用:如果一个对象仅持有虚引用,那么它相当于没有引用,在任何时候都可能被垃圾回收器回收.

更多了解参见[深入对象引用](http://blog.csdn.net/dd864140130/article/details/49885811)

## WeakReference与SoftReference的区别?

这点在四种引用类型中已经做了解释,这里在重复一下.虽然WeakReference与SoftReference都有利于提高 GC和内存的效率,但是 WeakReference ,一旦失去最后一个强引用,就会被 GC 回收,而软引用虽然不能阻止被回收,但是可以延迟到 JVM 内存不足的时候.

## 为什么要有不同的引用类型

不像C语言,我们可以控制内存的申请和释放,在Java中有时候我们需要适当的控制对象被回收的时机,因此就诞生了不同的引用类型,可以说不同的引用类型实则是对GC回收时机不可控的妥协.

# 进程,线程相关

## 说说进程,线程之间的区别?

简而言之,进程是程序运行和资源分配的基本单位,一个程序至少有一个进程,一个进程至少有一个线程.进程在执行过程中拥有独立的内存单元,而多个线程共享内存资源,减少切换次数,从而效率更高.线程是进程的一个实体,是cpu调度和分派的基本单位,是比程序更小的能独立运行的基本单位.同一进程中的多个线程之间可以并发执行.在Linux中,进程也称为Task.

## 守护线程是什么?它和非守护线程有什么区别

程序运行完毕,jvm会等待非守护线程完成后关闭,但是jvm不会等待守护线程.守护线程最典型的例子就是GC线程.
## 什么是多线程上下文切换
多线程的上下文切换是指CPU控制权由一个已经正在运行的线程切换到另外一个就绪并等待获取CPU执行权的线程的过程.

## 创建两种线程的方式?他们有什么区别?
通过实现java.lang.Runnable或者通过扩展java.lang.Thread类.相比扩展Thread,实现Runnable接口可能更优.原因有二:

  1. Java不支持多继承.因此扩展Thread类就代表这个子类不能扩展其他类.而实现Runnable接口的类还可能扩展另一个类.
  2. 类可能只要求可执行即可,因此继承整个Thread类的开销过大.

## Callable和Runnable的区别是什么?

两者都能用来编写多线程,但实现Callable接口的任务线程能返回执行结果,而实现Runnable接口的任务线程不能返回结果.Callable通常需要和Future/FutureTask结合使用,用于获取异步计算结果.

## Thread类中的start()和run()方法有什么区别?

在`start(`)方法中最终要的是调用了Native方法`start0()`用来启动新创建的线程线程启动后会自动调用`run()`方法.如果我们直接调用其run()方法就和我们调用其他方法一样,不会在新的线程中执行.

##怎么检测一个线程是否持有对象锁

Thread类提供了一个Native方法`holdsLock(Object obj)`方法用于检测是否持有某个对象锁:当且仅当对象obj的锁被某线程持有的时候才会返回true.

```java
 public static native boolean holdsLock(Object obj);
```

## 线程阻塞有哪些原因?

阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。

|方法|说明|
|---|----|
|sleep()|sleep() 允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止|
|suspend() 和 resume()|两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。|
|yield() |yield() 使当前线程放弃当前已经分得的CPU 时间，但不使当前线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程|
|wait() 和 notify()|两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许 指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用.|

## wait(),notify()和suspend(),resume()之间的区别
初看起来它们与 suspend() 和 resume() 方法对没有什么分别，但是事实上它们是截然不同的。区别的核心在于，前面叙述的所有方法，阻塞时都不会释放占用的锁（如果占用了的话），而这一对方法则相反。上述的核心区别导致了一系列的细节上的区别。

首先，前面叙述的所有方法都隶属于 Thread 类，但是这一对却直接隶属于 Object 类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致从调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

其次，前面叙述的所有方法都可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。

wait() 和 notify() 方法的上述特性决定了它们经常和synchronized关键字一起使用，将它们和操作系统进程间通信机制作一个比较就会发现它们的相似性：synchronized方法或块提供了类似于操作系统原语的功能，它们的执行不会受到多线程机制的干扰，而这一对方法则相当于 block 和wakeup 原语（这一对方法均声明为 synchronized）。它们的结合使得我们可以实现操作系统上一系列精妙的进程间通信的算法（如信号量算法），并用于解决各种复杂的线程间通信问题。

关于 wait() 和 notify() 方法最后再说明两点：
第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。

第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。

谈到阻塞，就不能不谈一谈死锁，略一分析就能发现，suspend() 方法和不指定超时期限的 wait() 方法的调用都可能产生死锁。遗憾的是，Java 并不在语言级别上支持死锁的避免，我们在编程中必须小心地避免死锁。

以上我们对 Java 中实现线程阻塞的各种方法作了一番分析，我们重点分析了 wait() 和 notify() 方法，因为它们的功能最强大，使用也最灵活，但是这也导致了它们的效率较低，较容易出错。实际使用中我们应该灵活使用各种方法，以便更好地达到我们的目的。

## 产生死锁的条件
1.互斥条件：一个资源每次只能被一个进程使用。  
2.请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。  
3.不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。  
4.循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。


## 为什么wait()方法和notify()/notifyAll()方法要在同步块中被调用
这是JDK强制的，wait()方法和notify()/notifyAll()方法在调用前都必须先获得对象的锁

## wait()方法和notify()/notifyAll()方法在放弃对象监视器时有什么区别
wait()方法和notify()/notifyAll()方法在放弃对象监视器的时候的区别在于：wait()方法立即释放对象监视器，notify()/notifyAll()方法则会等待线程剩余代码执行完毕才会放弃对象监视器。

## wait()与sleep()的区别

 关于这两者已经在上面进行详细的说明,这里就做个概括好了:

 - sleep()来自Thread类，和wait()来自Object类.调用sleep()方法的过程中，线程不会释放对象锁。而 调用 wait 方法线程会释放对象锁
 - sleep()睡眠后不出让系统资源，wait让其他线程可以占用CPU
 - sleep(milliseconds)需要指定一个睡眠时间，时间一到会自动唤醒.而wait()需要配合notify()或者notifyAll()使用

## 为什么wait,nofity和nofityAll这些方法不放在Thread类当中
一个很明显的原因是JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。


##怎么唤醒一个阻塞的线程
如果线程是因为调用了wait()、sleep()或者join()方法而导致的阻塞，可以中断线程，并且通过抛出InterruptedException来唤醒它；如果线程遇到了IO阻塞，无能为力，因为IO是操作系统实现的，Java代码并没有办法直接接触到操作系统。

##什么是多线程的上下文切换
多线程的上下文切换是指CPU控制权由一个已经正在运行的线程切换到另外一个就绪并等待获取CPU执行权的线程的过程。


## synchronized和ReentrantLock的区别
synchronized是和if、else、for、while一样的关键字，ReentrantLock是类，这是二者的本质区别。既然ReentrantLock是类，那么它就提供了比synchronized更多更灵活的特性，可以被继承、可以有方法、可以有各种各样的类变量，ReentrantLock比synchronized的扩展性体现在几点上：
（1）ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁
（2）ReentrantLock可以获取各种锁的信息
（3）ReentrantLock可以灵活地实现多路通知
另外，二者的锁机制其实也是不一样的:ReentrantLock底层调用的是Unsafe的park方法加锁，synchronized操作的应该是对象头中mark word.

## FutureTask是什么
这个其实前面有提到过，FutureTask表示一个异步运算的任务。FutureTask里面可以传入一个Callable的具体实现类，可以对这个异步运算的任务的结果进行等待获取、判断是否已经完成、取消任务等操作。当然，由于FutureTask也是Runnable接口的实现类，所以FutureTask也可以放入线程池中。

## 一个线程如果出现了运行时异常怎么办?
如果这个异常没有被捕获的话，这个线程就停止执行了。另外重要的一点是：如果这个线程持有某个某个对象的监视器，那么这个对象监视器会被立即释放

## Java当中有哪几种锁
1. 自旋锁: 自旋锁在JDK1.6之后就默认开启了。基于之前的观察，共享数据的锁定状态只会持续很短的时间，为了这一小段时间而去挂起和恢复线程有点浪费，所以这里就做了一个处理，让后面请求锁的那个线程在稍等一会，但是不放弃处理器的执行时间，看看持有锁的线程能否快速释放。为了让线程等待，所以需要让线程执行一个忙循环也就是自旋操作。在jdk6之后，引入了自适应的自旋锁，也就是等待的时间不再固定了，而是由上一次在同一个锁上的自旋时间及锁的拥有者状态来决定

2. 偏向锁: 在JDK1.6之后引入的一项锁优化，目的是消除数据在无竞争情况下的同步原语。进一步提升程序的运行性能。 偏向锁就是偏心的偏，意思是这个锁会偏向第一个获得他的线程，如果接下来的执行过程中，改锁没有被其他线程获取，则持有偏向锁的线程将永远不需要再进行同步。偏向锁可以提高带有同步但无竞争的程序性能，也就是说他并不一定总是对程序运行有利，如果程序中大多数的锁都是被多个不同的线程访问，那偏向模式就是多余的，在具体问题具体分析的前提下，可以考虑是否使用偏向锁。

3. 轻量级锁: 为了减少获得锁和释放锁所带来的性能消耗，引入了“偏向锁”和“轻量级锁”，所以在Java SE1.6里锁一共有四种状态，无锁状态，偏向锁状态，轻量级锁状态和重量级锁状态，它会随着竞争情况逐渐升级。锁可以升级但不能降级，意味着偏向锁升级成轻量级锁后不能降级成偏向锁

## 如何在两个线程间共享数据
通过在线程之间共享对象就可以了，然后通过wait/notify/notifyAll、await/signal/signalAll进行唤起和等待，比方说阻塞队列BlockingQueue就是为线程之间共享数据而设计的

## 如何正确的使用wait()?使用if还是while?
wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：
```
 synchronized (obj) {
    while (condition does not hold)
      obj.wait(); // (Releases lock, and reacquires on wakeup)
      ... // Perform action appropriate to condition
 }
```


## 什么是线程局部变量ThreadLocal
线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。

## ThreadLoal的作用是什么?
简单说ThreadLocal就是一种以空间换时间的做法在每个Thread里面维护了一个ThreadLocal.ThreadLocalMap把数据进行隔离，数据不共享，自然就没有线程安全方面的问题了.

## 生产者消费者模型的作用是什么?
（1）通过平衡生产者的生产能力和消费者的消费能力来提升整个系统的运行效率，这是生产者消费者模型最重要的作用
（2）解耦，这是生产者消费者模型附带的作用，解耦意味着生产者和消费者之间的联系少，联系越少越可以独自发展而不需要收到相互的制约

## 写一个生产者-消费者队列
可以通过阻塞队列实现,也可以通过wait-notify来实现.

### 使用阻塞队列来实现

```java
//消费者
public class Producer implements Runnable{
    private final BlockingQueue<Integer> queue;

    public Producer(BlockingQueue q){
        this.queue=q;
    }

    @Override
    public void run() {
        try {
            while (true){
                Thread.sleep(1000);//模拟耗时
                queue.put(produce());
            }
        }catch (InterruptedException e){

        }
    }

    private int produce() {
        int n=new Random().nextInt(10000);
        System.out.println("Thread:" + Thread.currentThread().getId() + " produce:" + n);
        return n;
    }
}
//消费者
public class Consumer implements Runnable {
    private final BlockingQueue<Integer> queue;

    public Consumer(BlockingQueue q){
        this.queue=q;
    }

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(2000);//模拟耗时
                consume(queue.take());
            }catch (InterruptedException e){

            }

        }
    }

    private void consume(Integer n) {
        System.out.println("Thread:" + Thread.currentThread().getId() + " consume:" + n);

    }
}
//测试
public class Main {

    public static void main(String[] args) {
        BlockingQueue<Integer> queue=new ArrayBlockingQueue<Integer>(100);
        Producer p=new Producer(queue);
        Consumer c1=new Consumer(queue);
        Consumer c2=new Consumer(queue);

        new Thread(p).start();
        new Thread(c1).start();
        new Thread(c2).start();
    }
}

```
### 使用wait-notify来实现

该种方式应该最经典,这里就不做说明了

##如果你提交任务时,线程池队列已满,这时会发生什么

如果使用的LinkedBlockingQueue,也就是无界队列的话,继续添加任务到阻塞队列中等待执行.因为LinkedBlockingQueue可以近乎认为是一个无穷大的队列,可以无限存放任务;如果你使用的是有界队列比方说ArrayBlockingQueue的话,任务首先会被添加到ArrayBlockingQueue中,ArrayBlockingQueue满了,则会使用拒绝策略RejectedExecutionHandler处理满了的任务,默认是AbortPolicy.

## 有哪几种任务拒绝策略机制?

线程池的拒绝策略,是指当任务添加到线程池中被拒绝,而采取的处理措施.当任务添加到线程池中之所以被拒绝,可能是由于:线程池异常关闭或者任务数量超过线程池的最大限制.

当前线程池共存在以下四种策略:

- AbortPolicy:任务被拒绝时,将丢弃该任务,并抛出 RejectedExecutionException异常
- DiscardPolicy:任务被拒绝时,将丢弃该任务,但是不抛出异常
- DiscardOldestPolicy:任务被拒绝时,丢弃等待队列中前边的任务,然后将新任务添加到等待队列中
- CallerRunsPolicy:任务被拒绝时,由调用线程处理该任务 

##为什么要使用线程池?

避免频繁地创建和销毁线程,达到线程对象的重用.另外,使用线程池还可以根据项目灵活地控制并发的数目.

##java中用到的线程调度算法是什么?

抢占式调度算法.一个线程用完CPU之后,操作系统会根据线程优先级,线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行.

##Thread.sleep(0)的作用是什么?

由于Java采用**抢占式**的线程调度算法,因此可能会出现某条线程常常获取到CPU控制权的情况.为了让某些优先级比较低的线程也能获取到CPU控制权,可以使用Thread.sleep(0)手动触发一次操作系统分配时间片的操作,这也是平衡CPU控制权的一种操作.

##什么是CAS?

CAS全称为Compare and Swap,即"比较-替换".假设有三个操作数:内存值V,旧的预期值A,要修改的值B,当且仅当预期值A和内存值V相同时,才会将内存值修改为B并返回true,否则什么都不做并返回false.当然CAS一定要volatile变量配合,这样才能保证每次拿到的变量是主内存中最新的那个值,否则旧的预期值A对某条线程来说,永远是一个不会变的值A,只要某次CAS操作失败,永远都不可能成功.

##什么是乐观锁和悲观锁?

**乐观锁**:乐观锁认为竞争不总是会发生,因此它不需要持有锁,将"比较-替换"这两个动作作为一个原子操作尝试去修改内存中的变量,如果失败则表示发生冲突,那么就应该有相应的重试逻辑.

**悲观锁**:悲观锁认为竞争总是会发生,因此每次对某资源进行操作时,都会持有一个独占的锁,就像synchronized,不管三七二十一,直接上了锁就操作资源了.


## ConcurrentHashMap的并发度是什么(或者说分的段数)?
JDK 1.8之前ConcurrentHashMap的并发度就是segment的大小,默认为16,这意味着最多同时可以有16条线程操作ConcurrentHashMap,这也是ConcurrentHashMap对Hashtable的最大优势.

## 为什么使用ConcurrentHashMap?

HashMap线程不安全,而Hashtable是线程安全,但是它使用了synchronized进行方法同步,插入和读取数据都使用了synchronized.当插入数据的时候不能进行读取(相当于把整个Hashtable都锁住了,全表锁),当多线程并发的情况下,都要竞争同一把锁,导致效率极其低下.而在JDK1.5后为了改进Hashtable的痛点.ConcurrentHashMap应运而生.

## ConcurrentHashMap的工作原理?

ConcurrentHashMap在jdk 1.6和jdk 1.8实现原理是不同的.

### jdk 1.6及之前:

ConcurrentHashMap是线程安全的,但是与Hashtable相比,实现线程安全的方式不同.Hashtable通过对hash表结构进行锁定,是阻塞式的.当一个线程占有这个锁时,其他线程必须阻塞等待其释放锁.ConcurrentHashMap使用的是分段锁技术,将ConcurrentHashMap将锁一段一段的存储,然后给每一段数据配一把锁(segment),当一个线程占用一把锁(segment)访问其中一段数据的时候,其他段的数据也能被其它的线程访问,默认分配16个segment,默认比Hashtable效率提高16倍.

ConcurrentHashMap内部有一个Segment<K,V>数组,该Segment对象可以充当锁.Segment对象内部有一个HashEntry<K,V>数组,于是每个Segment可以守护若干个桶(HashEntry),每个桶又有可能是一个HashEntry连接起来的链表,存储发生碰撞的元素.

默认情况下会创建包含16个Segment对象的数组,每个数组有若干个桶.当我们进行put方法时,通过hash方法对key进行计算,得到hash值,找到对应的segment,然后对该segment进行加锁,然后调用segment的put方法进行存储操作,此时其他线程就不能访问当前的segment,但可以访问其他的segment对象,不会发生阻塞等待.

![image-20181023220331387](https://i.imgur.com/pQ0DGDQ.png)

### jdk 1.8
在jdk 8中,ConcurrentHashMap不再使用Segment分段锁,而是采用一种CAS算法来实现同步问题,但其底层还是"数组+链表->红黑树"的实现,和HashMap一样.在没有发生Hash冲突的情况下,通过CAS进行更新,否则通过synchronized锁定当前链表或红黑二叉树的首节点,这样只要hash不冲突,就不会产生并发,效率又提升N倍,简单来看一下其put的过程:

```java
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        // ConcurrentHashMap 不允许插入null键,HashMap允许插入一个null键
        if (key == null || value == null) throw new NullPointerException();
        // 计算key的hash值
        int hash = spread(key.hashCode());
        int binCount = 0;
        //for循环的作用:因为更新元素是使用CAS机制更新,需要不断的失败重试,直到成功为止
        for (Node<K,V>[] tab = table;;) {
            // f:链表或红黑二叉树头结点,向链表中添加元素时,需要synchronized获取f的锁
            Node<K,V> f; int n, i, fh;
            //判断Node[]数组是否初始化,没有则进行初始化操作
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // 通过hash定位Node[]数组的索引坐标,是否有Node节点,如果没有则使用CAS
            // 进行添加(链表的头结点),添加失败则进入下次循环
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // 检查到内部正在移动元素(Node[] 数组扩容)
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                // 锁住链表或红黑二叉树的头结点
                synchronized (f) {
                    // 判断f是否是链表的头结点
                    if (tabAt(tab, i) == f) {
                         // 如果fh>=0 是链表节点
                        if (fh >= 0) {
                            binCount = 1;
                            // 遍历链表所有节点
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                 // 如果节点存在,则更新value
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                // 不存在则在链表尾部添加新节点
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // TreeBin是红黑二叉树节点
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            // 添加树节点
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    // 如果链表长度已经达到临界值8就需要把链表转换为树结构
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 将当前ConcurrentHashMap的size数量+1
        addCount(1L, binCount);
        return null;
    }
```

总结一下,JDK8中的实现也是锁分离的思想,它把锁分的比segment(JDK1.6)更细一些.只要hash不冲突,就不会出现并发获得锁的情况.它首先使用无锁操作CAS插入头结点,如果插入失败,说明已经有别的线程插入头结点了,再次循环进行操作.如果头结点已经存在,则通过synchronized获得头结点锁,进行后续的操作.

## CyclicBarrier和CountDownLatch区别?

这两个类非常类似,都在java.util.concurrent下.都可以用来表示代码运行到某个点上,二者的区别在于:

 - CyclicBarrier的某个线程运行到某个点上之后,该线程即停止运行,直到所有的线程都到达了这个点,所有线程才重新运行;CountDownLatch则不是,某线程运行到某个点上之后,只是给某个数值-1而已,该线程继续运行
 - CyclicBarrier只能唤起一个任务,CountDownLatch可以唤起多个任务
 - CyclicBarrier可重用,CountDownLatch不可重用,计数值为0该CountDownLatch就不可再用了

## java中的`++`操作符线程安全么?

不是线程安全的操作.它涉及到多个指令,如读取变量值->增加->然后存储回内存,这个过程可能会出现多个线程交差.

## 你有哪些多线程开发良好的实践?

1. 给线程命名
2. 最小化同步范围
3. 优先使用volatile
4. 尽可能使用更高层次的并发工具而非wait和notify()来实现线程通信,如BlockingQueue,Semeaphore
5. 优先使用并发容器而非同步容器.
6. 考虑使用线程池

#关于volatile关键字

## 可以创建Volatile数组吗?
Java中可以创建volatile类型数组,不过只是一个指向数组的引用,而不是整个数组.如果改变引用指向的数组,将会受到volatile 的保护,但是如果多个线程同时改变数组的元素,volatile标示符就不能起到之前的保护作用了.

## volatile能使得一个非原子操作变成原子操作吗?
一个典型的例子是在类中有一个 long类型的成员变量.如果你知道该成员变量会被多个线程访问,最好是将其设置为 volatile.为什么?因为 Java中读取long类型变量不是原子的,需要分成两步,如果一个线程正在修改该 long变量的值,另一个线程可能只能看到该值的一半.

一种实践是用volatile修饰long和double变量,使其能按原子类型来读写.double和long都是64位宽,JVM对这两种类型的读是分为两部分的:第一次读取第一个 32 位，然后再读剩下的 32 位,这个过程不是原子的.但 Java中volatile型的long或double 变量的读写是原子的.

volatile另一个作用是提供内存屏障(memory barrier).简单的说,就是当你写一个volatile变量之前,Java内存模型会插入一个写屏障(write barrier),读一个 volatile 变量之前,会插入一个读屏障(read barrier).意思就是说,在你写一个volatile域时,能保证任何线程都能看到你写的值;同时在写之前,也能保证任何数值的更新对所有线程是可见的,因为内存屏障会将其他所有写的值更新到缓存.

## volatile类型变量提供什么保证?
volatile 主要有两方面的作用:

- 避免指令重排

- 可见性保证

例如JVM 或者JIT为了获得更好的性能会对语句重排序,但是volatile 类型变量即使在没有同步块的情况下赋值也不会与其他语句重排序.volatile提供 happens-before 的保证,确保一个线程的修改能对其他线程是可见的.某些情况下,volatile 还能提供原子性,如读 64 位数据类型,像 long和double都不是原子的(低32位和高32位),但volatile 类型的double和long 就是原子的.

----------
#关于集合

## Java中的集合及其继承关系
关于集合的体系是每个人都应该烂熟于心的,尤其是对我们经常使用的List,Map的原理更该如此.这里我们看这张图即可:
![image-20181024120943625](https://i.imgur.com/YlV4qvc.png)

更多内容可见[集合类总结](http://write.blog.csdn.net/postedit/40826423)

## poll()方法和remove()方法区别?
poll()和 remove()都是从队列中取出一个元素,但是 poll() 在获取元素失败的时候会返回空,但是 remove() 失败的时候会抛出异常.

## HashMap和HashTable的区别?

- HashMap支持Key和Value为null的情况,而Hashtable不允许,因为HashMap对null进行了特殊处理,将null的hashCode值固定为0,从而将其存放在哈希数组下标为0的位置.
- HashMap非线程安全,Hashtable是线程安全.但可以通过`Collections.synchronziedMap(new HashMap())`获得线程安全的HashMap.
- HashMap默认长度是16,扩容是原先的2倍;Hashtable默认长度是11,扩容是原先的2n+1
- HashMap继承AbstractMap；Hashtable继承了Dictionary 

## LinkedHashMap和HashMap的区别?

LinkedHashMap也是一个HashMap,通过维护一个额外的双向链表保证了迭代顺序,该迭代顺序可以是插入顺序,也可以是访问顺序.因此，根据链表中元素的顺序可以将LinkedHashMap分为:**保持插入顺序的LinkedHashMap** 和 **保持访问顺序的LinkedHashMap**,默认实现是按插入顺序排序的.

## LinkedHashMap和PriorityQueue的区别

PriorityQueue是一个优先级队列.保证最高或者最低优先级的的元素总是在队列头部.但是 LinkedHashMap维持的顺序是元素插入的顺序.当遍历一个PriorityQueue时,没有任何顺序保证,但是 LinkedHashMap 课保证遍历顺序是元素插入的顺序.

## WeakHashMap与HashMap的区别是什么?
WeakHashMap的工作与正常的 HashMap 类似.但是使用弱引用作为 key,意思就是当key对象没有任何引用时,key/value 将会被回收.

## TreeMap是实现原理
采用红黑树实现,具体实现自行查阅源码.


## 什么是ArrayMap?它和HashMap有什么区别?
ArrayMap是Android SDK中提供的,非Android开发者可以略过.
ArrayMap是用两个数组来模拟map,更少的内存占用空间,更高的效率.
具体参考这篇文章:[ArrayMap VS HashMap](http://lvable.com/?p=217%5D)


## HashMap的实现原理
HashMap是基于哈希表的Map接口的非同步实现,此实现提供所有可选的映射操作,并允许使用null值和null键.此类不保证映射的顺序,特别是它不保证该顺序恒久不变.
在java编程语言中,最基本的结构就是两种,一个是数组另外一个是模拟指针(引用),所有的数据结构都可以用这两个基本结构来构造的,HashMap也不例外.HashMap实际上是一个“链表散列”的数据结构,即数组和链表的结合体.

当我们往HashMap中put元素时,首先根据key的hashcode重新计算hash值,根据hash值得到这个元素在数组中的位置(下标),如果该数组在该位置上已经存放了其他元素,那么在这个位置上的元素将以链表的形式存放,新加入的放在链头,最先加入的放入链尾.如果数组中该位置没有元素,就直接将该元素放到数组的该位置上..

需要注意Jdk 1.8中对HashMap的实现做了优化,当链表中的节点数据超过8个之后,该链表会转为红黑树来提高查询效率,从原来的O(n)到O(logn).当树的节点数小于6个后会再转为链表结构.在`get()`操作时,通过对Hash值进行取模操作来计算哈希数组的下标,此时取到的元素类型是Node的话,则按链表的方式进行查找,如果是TreeNode的话,则按红黑树的方式查找.在进行`put()`操作时,如果哈希数组为空,则首先通过扩容的方式对其进行初始化,然后计算其要存放的位置.如果要插入的键值已经存在,则用新值替换旧值;否则,将其插入链表中,并根据链表的大小决定是否要转为红黑树.插入完成后,判断当前元素数量是否达到阀值,到达之后进行扩容操作.

## 简述HashMap在创建时是如何设置哈希数组的大小的?

HashMap在初始化时允许我们指定负载因子和初始化容量大小,以JDK 1.8为例,其最终调用构造方法如下:

```java
    public HashMap(int initialCapacity, float loadFactor) {
		......
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
```

默认情况下负载因子loadFactory为`DEFAULT_LOAD_FACTOR = 0.75f`,初始化容量initialCapacity为` DEFAULT_INITIAL_CAPACITY = 1 << 4`即16.其中HashMap要求哈希数组的大小必须2<sup>n</sup>,如果我们在初始时指定initialCapacity为7会是如何?其处理逻辑在`tableSizeFor()`中:

```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

该方法能返回大于输入参数且离该参数最近的2<sup>n</sup>的整数,比如如果如输入10,则返回16;如果输入7则返回8.

## 简述HashMap中哈希值计算方法?

以JDK 1.8源码为例,对于key为null的情况,其hash值固定为0,否则进行以下两步:

- 获取key对应的hashCode
- hashCode高位参与运算

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

和JDK 1.7之前不同,在JDK 1.8中优化了高位运算算法,即让hashCode()的高16位与低16位进行异或运算:`(h = k.hashCode()) ^ (h >>> 16)`,这样做的好处是当哈希数组比较小的情况,也能保证高位参与到hash值的计算过程:

![image-20181201145328121](https://i.imgur.com/Pg6VtbT.png)

在上图中,n代表哈希数组的长度,此时其大小为16.

## 简述HashMap中通过哈希值定位哈希数组位置的过程?

在计算出Hash值之后,需要通过它确定Key在哈希数组中的下标,常用的方式取模.以JDK 1.8红的HashTable为例,其计算过程如下:

```java
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }
		......
        // 计算hash    
        int hash = key.hashCode();
        // 通过取模运算计算index,这里通过hash & 0x7FFFFFFF是将符号位强制为0,
        // 以保证计算出的都是正整数
        int index = (hash & 0x7FFFFFFF) % tab.length;
  		......
    }
```

`0x7FFFFFFF` 是`0111 1111 1111 1111 1111 1111 1111 1111`,即除符号位外的所有1.将其与hash进行或操作,可以保证产生都是正整数,这样不至于出现类似`-1 & 10 = -1`的情况.

在HashMap中通常是对Hash值进行取模操作,但是和HashTable不同,HashMap在取模和扩容时做了优化.HashMap在创建或扩容时保证其哈希桶的大小始终是2<sup>n</sup>,此时可以用`h & (length -1)`运算等价于`(hash & 0x7FFFFFFF) % length`,以JDK 1.8 HashMap的代码为例:

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // i= (n - 1) & hash即计算哈希数组下标的过程
        if ((p = tab[i = (n - 1) & hash]) == null)
        ......    
    }
```

## 简单说说 JDK1.7 中 HashMap 什么情况下会发生扩容？如何扩容？

HashMap 中默认的负载因子为 0.75,默认情况下第一次扩容阀值是 12(16 * 0.75),故第一次存储第13个键值对时就会触发扩容机制变为原来数组长度的2倍,以后每次扩容都类似计算机制.

![image-20181024125053256](https://i.imgur.com/jDiNG6m.png)

JDK 1,8在1.7的基础上做了优化,不需要像JDK1.7的实现那样在扩容后重新计算hash,只需要看看原来的hash值新增的那个bit是1还是0就好了:是0的话索引没变,是1的话索引变成"原索引+oldCap",即扩容之前的索引值+扩容之前的容量.比如起始时,容量为16,元素计算后的索引值index为5,那么容量在由16扩容到32之后,其index= 5+16,即21,如下所示:

![image-20181024130306810](https://i.imgur.com/RCaliUr.png)

## LinkedHashMap的实现原理?

LinkedHashMap的实现原理和HashMap并没有本质区别,下图中除去红色的虚连接线,其结构就是典型的HashMap.

![image-20181024121753568](https://i.imgur.com/TqdtOHK.png)

和HashMap的不同的是,其Entry节点中,增加了两个指针:befor和after,用来维护双向链表结构.

![image-20181024122059677](https://i.imgur.com/5MHyA71.png)

## 如何利用LinkedHashMap实现LRU缓存?

LRU是Least Recently Used 的缩写,即"最近最少使用".简单的说就是缓存一定量的数据,当超过设定的阈值时就把一些过期的数据删除掉.比如我们缓存1000条数据,当数据小于1000时可以随意添加,当超过1000时就需要把新的数据添加进来,同时要把过期数据删除,以确保我们最大缓存1000条.那怎么确定删除哪条过期数据呢?采用LRU算法实现的话就是将最老的数据删掉.

LinkedHashMap自身已经实现了顺序存储,默认情况下是按照元素的添加顺序存储,也可以启用按照访问顺序存储:即最近读取的数据放在最前面,最早读取的数据放在最后面.然后它还有一个判断是否删除最老数据的方法,默认是返回false,即不删除数据,我们使用LinkedHashMap实现LRU缓存的方法就是对LinkedHashMap实现简单的扩展,以下是简单实现:

```java

class LRUCache<K,V> extends LinkedHashMap<K,V>{
	private int capacity;
	private static final long serialVersionUID = 1L;

	LRUCache(int capacity){
		super(16,0.75f,true);
		this.capacity=capacity;
	}
    
	@Override
	public boolean removeEldestEntry(Map.Entry<K, V> eldest){ 
		return size()>capacity;
	}  
}
```

## ArrayList和LinkedList的区别?

最明显的区别是 ArrayList底层的数据结构是数组,支持随机访问,而 LinkedList 的底层数据结构是**双向循环链表**，不支持随机访问.访问一个元素时，ArrayList 的时间复杂度是 O(1),而 LinkedList 是O(n).

## ArrayList和Array有什么区别?

1. Array可以容纳基本类型和对象而,ArrayList只能容纳对象。
2. Array是固定大小的，而ArrayList是可以动态扩容的.

## 了解CopyOnWriteArrayList么?

如果你还记得Linux中进程fork时的Copy-On-Write机制,那此时一定很轻松理解此类的设计.Copy-On-Write是一种用于程序设计中的优化策略.其基本思路是:一开始大家都在共享同一个内容,当某个人想要修改这个内容的时候,才会真正把内容Copy出去形成一个新的内容然后再改.从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器:CopyOnWriteArrayList和CopyOnWriteArraySet.

## 遍历ArrayList时如何正确移除一个元素

该问题的关键在于面试者使用的是 ArrayList的 remove() 还是 Iterator 的 remove()方法。

## LinkedList的是单向链表还是双向?

双向循环列表,具体实现自行查阅源码.

## ArrayList扩容算法?

具体可见ArrayList实现:

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

通过代码看出int newCapacity = oldCapacity + (oldCapacity >> 1),即容量每次扩增为原来的1.5倍.

## Vector扩容算法?

具体可见其实现:

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

```

默认情况下,capacityIncrement为0,此时newCapacity=oldCapacity+oldCapacity,即容量扩增为原来的2倍.如果已经设置了capacityIncrement,此时newCapacity=oldCapacity+capacityIncrement.

## ArrayList和HashMap默认大小?

在 JDK 7中,ArrayList 的默认大小是 10 个元素,HashMap 的默认大小是16个元素(必须是2的幂),这就是 Java 7 中 ArrayList 和 HashMap 类的代码片段

```java
// from ArrayList.java JDK 7
private static final int DEFAULT_CAPACITY = 10;
 
 //from HashMap.java JDK 7
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

## Comparator和Comparable的区别?

Comparable接口用于定义对象的自然顺序,而 comparator 通常用于定义用户定制的顺序.Comparable 总是只有一个,但是可以有多个 comparator 来定义对象的顺序.

## 如何实现集合排序?

你=可以使用有序集合,如TreeSet 或TreeMap,也可以使用有顺序的的集合,如 list,然后通过Collections.sort() 来排序.

## 如何打印数组内容?

你可以使用 Arrays.toString() 和 Arrays.deepToString() 方法来打印数组.由于数组没有实现 toString() 方法,所以如果将数组传递给 System.out.println() 方法,将无法打印出数组的内容，但是 Arrays.toString() 可以打印每个元素.

## 你了解Fail-Fast机制吗

Fail-Fast即我们常说的快速失败,更多内容参看[fail-fast机制](http://blog.csdn.net/chenssy/article/details/38151189)

## Fail-fast和Fail-safe有什么区别

Iterator的fail-fast属性与当前的集合共同起作用,因此它不会受到集合中任何改动的影响.Java.util包中的所有集合类都被设计为fail-fast的,而java.util.concurrent中的集合类都为fail-safe的.当检测到正在遍历的集合的结构被改变时,fail-fast迭代器抛出ConcurrentModificationException,而fail-safe迭代器从不抛出ConcurrentModificationException.

## 常见的阻塞队列有哪些?

- **ArrayBlockingQueue**
  ArrayBlockingQueue是一个有界的阻塞队列,其内部实现是将对象放到一个数组里.有界也就意味着,它不能够存储无限多数量的元素.它有一个同一时间能够存储元素数量的上限.我们可以在对其初始化的时候设定这个上限,但之后就无法对这个上限进行修改了.

- **LinkedBlockingQueue**

  LinkedBlockingQueue内部以一个链式结构(链接节点)对其元素进行存储.如果需要的话,这一链式结构可以选择一个上限.如果没有定义上限,将使用 Integer.MAX_VALUE 作为上限,此时可以认为是无界队列.

- **PriorityBlockingQueue**

  PriorityBlockingQueue是一个无界优先级队列.它使用了和类 java.util.PriorityQueue一样的排序规则.所有插入到 PriorityBlockingQueue的元素必须实现 java.lang.Comparable接口.因此该队列中元素的排序就取决于我们自己的 Comparable 实现.

- **SynchronousQueue**

  SynchronousQueue是一个特殊的队列,它的内部同时只能够容纳单个元素.如果该队列已有一元素的话,试图向队列中插入一个新元素的线程将会阻塞,直到另一个线程将该元素从队列中抽走.同样,如果该队列为空,试图向队列中抽取一个元素的线程将会阻塞,直到另一个线程向队列中插入了一条新的元素.很显然,这货并不像个队列,而是个中转站的存在.

- **DelayQueue**

  DelayQueue对元素进行持有直到一个特定的延迟到期.注入其中的元素必须实现 java.util.concurrent.Delayed 接口.

## 阻塞队列常用方法

BlockingQueue具有4组不同的方法用于插入,移除以及对队列中的元素进行检查,如果请求的操作不能得到立即执行的话，每个方法的表现也不同,具体如下:

![image-20181023223156834](https://i.imgur.com/pxNOprO.png)

- 抛异常:如果试图的操作无法立即执行,抛一个异常
- 特定值:如果试图的操作无法立即执行,返回一个特定的值(常常是 true / false)
- 阻塞:如果试图的操作无法立即执行,该方法调用将会发生阻塞,直到能够执行.
- 超时:如果试图的操作无法立即执行,该方法调用将会发生阻塞,直到能够执行,但等待时间不会超过给定值.返回一个特定值以告知该操作是否成功(典型的是true / false)。

需要注意我们无法向一个BlockingQueue 中插 null.如果你试图插入null,BlockingQueue将会抛出一个 NullPointerException.

## 知道BlockingDeque?

BlockingDeque是双端阻塞队列,和BlockingQueue是类似的,只不过BlockingDeque提供从任意一端插入或者获取元素的功能.

## 阻塞队列的原理是什么?

阻塞队列实现阻塞同步的方式很简单,使用的就是是lock锁的多条件(condition)阻塞控制.使用BlockingQueue封装了根据条件阻塞线程的过程.

## 非阻塞队列是什么?原理呢?

有时候需要使用线程安全的队列.实现一个线程安全的队列有两种方式:一种是使用阻塞算法,另一种是使用非阻塞算法.使用阻塞算法实现的队列成为阻塞队列,其内部可以用一个锁(入队和出队用同一把锁)或两个锁(入队和出队用不同的锁)等方式来实现.非阻塞的实现方式则可以使用循环CAS的方式来实现.

ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列,它采用先进先出的规则对节点进行排序,当我们添加一个元素的时候,它会添加到队列的尾部,当我们获取一个元素时,它会返回队列头部的元素,它采用了“wait－free”算法来实现,[非阻塞原理底层实现](https://my.oschina.net/u/2988360/blog/956533)

# 关于日期

## SimpleDateFormat是线程安全的吗?
不幸的是DateFormat 的所有实现,包括SimpleDateFormat都不是线程安全的,因此你不应该在多线程序中使用,除非是在对外线程安全的环境中使用,如 将SimpleDateFormat限制在ThreadLocal 中.我建议在处理时间/日期时选择使用joda-time库,当然我们也可以选择使用java8提供的新的时间处理库如LocalDate等.

## 如何格式化日期?

Java中,可以使用SimpleDateFormat 类或者joda-time库来格式日期.DateFormat 类允许你使用多种流行的格式来格式化日期.

# 关于异常

## 简单描述java异常体系

相比没有人不了解异常体系,关于异常体系的更多信息可以见:[白话异常机制](http://blog.csdn.net/dd864140130/article/details/42504189)

## 什么是异常链
详情直接参见[白话异常机制](http://blog.csdn.net/dd864140130/article/details/42504189),不做解释了.

## throw和throws的区别
throw用于主动抛出java.lang.Throwable类的一个实例化对象,意思是说你可以通过关键字 throw抛出一个 Error 或者 一个Exception,如：`throw new IllegalArgumentException(“size must be multiple of 2″)`,
而throws 的作用是作为方法声明和签名的一部分,方法被抛出相应的异常以便调用者能处理.Java 中,任何未处理的受检查异常强制在 throws 子句中声明.

# 关于序列化

## Java 中Serializable 与 Externalizable 的区别

Serializable 接口是一个序列化Java 类的接口,以便于它们可以在网络上传输或者可以将它们的状态保存在磁盘上,是 JVM 内嵌的默认序列化方式,成本高,脆弱而且不安全.Externalizable 允许你控制整个序列化过程,指定特定的二进制格式,增加安全机制.

## 反射与注解

### 简单述说实现注解的流程?

### 动态注解和编译时注解有什么区别?

### 反射的使用场景?为什么说反射耗性能?

# JVM

## 简单解释一下类加载器

有关类加载器一般会问你四种类加载器的应用场景以及双亲委派模型,更多的内容参看[深入理解JVM加载器](http://blog.csdn.net/dd864140130/article/details/49817357)

## 简述堆和栈的区别
VM中堆和栈属于不同的内存区域,使用目的也不同.栈常用于保存方法帧和局部变量.而对象总是在堆上分配.栈通常都比堆小,也不会在多个线程之间共享,而堆被整个 JVM 的所有线程共享.

## JVM内存分配流程

![image-20181023212348861](https://i.imgur.com/v7HRCCv.png)

#其他

## java当中采用的是大端还是小端?

大端模式,具体可见[JVM值之大端模式](https://blog.csdn.net/dd864140130/article/details/82427033)

## XML解析的几种方式和特点
DOM,SAX,PULL三种解析方式:

 - DOM:消耗内存：先把xml文档都读到内存中，然后再用DOM API来访问树形结构，并获取数据。这个写起来很简单，但是很消耗内存。要是数据过大，手机不够牛逼，可能手机直接死机
 - SAX:解析效率高，占用内存少，基于事件驱动的：更加简单地说就是对文档进行顺序扫描，当扫描到文档(document)开始与结束、元素(element)开始与结束、文档(document)结束等地方时通知事件处理函数，由事件处理函数做相应动作，然后继续同样的扫描，直至文档结束。
 - PULL:与 SAX 类似，也是基于事件驱动，我们可以调用它的next（）方法，来获取下一个解析事件（就是开始文档，结束文档，开始标签，结束标签），当处于某个元素时可以调用XmlPullParser的getAttributte()方法来获取属性的值，也可调用它的nextText()获取本节点的值。

## JDK 1.7特性

然 JDK 1.7 不像 JDK 5 和 8 一样的大版本，但是，还是有很多新的特性，如 try-with-resource 语句，这样你在使用流或者资源的时候，就不需要手动关闭，Java 会自动关闭。Fork-Join 池某种程度上实现 Java 版的 Map-reduce。允许 Switch 中有 String 变量和文本。菱形操作符(\<\>)用于类型推断，不再需要在变量声明的右边申明泛型，因此可以写出可读写更强、更简洁的代码
## JDK 1.8特性
java 8 在 Java 历史上是一个开创新的版本，下面 JDK 8 中 5 个主要的特性：
Lambda 表达式，允许像对象一样传递匿名函数
Stream API，充分利用现代多核 CPU，可以写出很简洁的代码
Date 与 Time API，最终，有一个稳定、简单的日期和时间库可供你使用
扩展方法，现在，接口中可以有静态、默认方法。
重复注解，现在你可以将相同的注解在同一类型上使用多次

## JDK 1.9特性

- 引入了模块化机制.
- 加入JShell.可以控制台启动 jshell ,并直接启动输入和执行Java 代码.
- 添加了一些集合类的工厂方法,比如现在可以`Set.of(1,2)`来直接创建Set对象.
- 更好用的Stream API.
- 接口中支持私有方法了

## Maven和ANT有什么区别?
虽然两者都是构建工具，都用于创建 Java 应用，但是 Maven 做的事情更多，在基于“约定优于配置”的概念下，提供标准的Java 项目结构，同时能为应用自动管理依赖（应用中所依赖的 JAR 文件.

## JDBC最佳实践

 - 优先使用批量操作来插入和更新数据
 - 使用PreparedStatement来避免SQL漏洞
 - 使用数据连接池
 - 通过列名来获取结果集

## IO操作最佳实践
1. 使用有缓冲的IO类,不要单独读取字节或字符
2. 使用NIO和NIO 2或者AIO,而非BIO
3. 在finally中关闭流
4. 使用内存映射文件获取更快的IO
















































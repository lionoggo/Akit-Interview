# SOLID原则

面向对象设计需要遵循SOLID原则,即:

- S: 单一原则
- O: 开闭原则
- L: 里氏替换原则
- I: 接口隔离原则
- D: 依赖反转原则

# 设计模式

## 单利模式



### 以下单利模式有什么问题?

```java
public class Singleton{
    private static Singleton instance;
    private Singleton() {};
    public static Singleton getInstance() {
        if (instance==null) {
            synchronized(Singleton.class) {
                if (instance==null)
                    instance=new Singleton();
            }
        }
        return instance;
    }
}
```

这种典型的单利写法在个别情况下,仍然不能保证只产生一个实例,但是可以使用volidate修饰instance来修正上述问题:

```java
public class Singleton{
    private static volatile Singleton instance;
    private Singleton() {};
    public static Singleton getInstance() {
        if (instance==null) {
            synchronized(Singleton.class) {
                if (instance==null)
                    instance=new Singleton();
            }
        }
        return instance;
    }
}
```

在实际开发中,最佳安全和保守的方式仍然是饿汉式的写法,即静态内部类的方式.


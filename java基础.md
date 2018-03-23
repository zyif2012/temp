---
typora-copy-images-to: img
---

## java 集合

##### 1. Java Queue 中的方法区别

offer，add区别：如果队列已经满了，add 方法会抛出异常，而 offer 方法只是返回 false。

poll，remove区别： poll() 方法在用空集合调用时不是抛出异常，只是返回 null ，而 remove 会出异常。

peek、element区别：element() 和 peek() 用于在队列的头部查询元素。与 remove() 方法类似，在队列为空时， element() 抛出一个异常，而peek() 返回 null。

##### 2. Java 技巧

- Java的外部类为什么不能使用private、protected进行修饰 

> 对于顶级类(外部类)来说，只有两种修饰符：public 和 default 。protected表示的是受保护的，也就是说不能被外部的类重新实例化和调用，那么这个类就成了没用的文件了，所以不能被定义为此类型。
>
> 内部类可以被四种修饰符修饰。
>
> protected 修饰的变量可以在子类中访问, 子类的对象在非子类不同包的类中不能直接访问该变量。

- 接口为什么只有常量

> 接口的所有方法都是 public ，其本身也是被 public 修饰，其变量都是 public static final ，就是常量。这是一种设计原则，
>
> 抽象的概念是将不可变的东西提取出来封装到一起，将可变的东西放到实现中去，接口的设计理念既然是高层的抽象，那么就应该定义为不可变的东西，如果接口中定义了变量，就说明带了可变的成分，就不是高层抽象了。体现的是OCP（对修改关闭，对扩展开放）的设计原则。

- 内部类和静态内部类、静态变量与静态代码块什么时候初始化

> 静态变量与静态代码块在类初始化的同时被初始化，而静态内部类是第一次被调用时才初始化（如果其内有静态变量，则在此时初始化，内部类是在实例被调用时初始化。静态变量，静态方法，静态块等都是类级别的属性，而不是单纯的对象属性。他们在类第一次被使用时被加载


例子

``` java
public class OuterClass {
    public static long OUTER_DATE = System.currentTimeMillis();
    static {
        System.out.println("外部类静态块加载时间：" + System.currentTimeMillis());
    }
    public OuterClass() {
        timeElapsed();
        System.out.println("外部类构造函数时间：" + System.currentTimeMillis());
    }
    static class InnerStaticClass {
        public static long INNER_STATIC_DATE = System.currentTimeMillis();
    }
    class InnerClass {
        public long INNER_DATE = 0;
        public InnerClass() {
            timeElapsed();
            INNER_DATE = System.currentTimeMillis();
        }
    }
    public static void main(String[] args) {
        System.out.println("静态内部类加载时间："+InnerStaticClass.INNER_STATIC_DATE);
        OuterClass outer = new OuterClass();
        System.out.println("外部类静态变量加载时间：" + outer.OUTER_DATE);
        System.out.println("非静态内部类加载时间"+outer.new InnerClass().INNER_DATE);
    }
    //单纯的为了耗时，来扩大时间差异
    private void timeElapsed() {
        for (int i = 0; i < 10000000; i++) {
            int a = new Random(100).nextInt(), b = new Random(100).nextInt();
            a = a + b;
        }
    }
}
```

静态内部类实现线程安全的单例模式。这个实现思路中最主要的一点就是利用类中静态变量的唯一性，饿汉式也是利用了这一点，线程安全。

```java 
public class SingleTon {
    // 利用静态内部类特性实现外部类的单例
    private static class SingleTonBuilder {
        private static SingleTon singleTon = new SingleTon();
    }
    // 私有化构造函数
    private SingleTon() {

    }
    public static SingleTon getInstance() {
        return SingleTonBuilder.singleTon;
    }
    public static void main(String[] args) {
        SingleTon instance = getInstance();
    }
}
```



## 多线程基础

#### 1.  线程概述

##### 1.1 线程状态

初始化、运行、冻结、死亡。运行状态通过 sleep 和 wait 可以到达冻结状态

一个线程实例不能启动两次，即继承自 Thread 的只能启动一次，而Runnable 实现的一个实例可以传入多个创建的线程中，该实例的资源可以被创建的线程共享。

Runnable接口

> 实现该接口，覆写 fun 方法，是另一种创建线程的方式。其实它只是一个任务接口，并不是一个线程，它的出现是为了线程和任务执行的逻辑分离。
>
> 继承了 Thread 父类就没有办法去继承其他类，而实现了 Runnable 接口可以继承其他类。

##### 1.2 策略模式

策略接口：主要是规范或者让结构程序知道如何进行调用

```java
interface CalcStrategy{
    int calc(int x, int y);
}
```

 程序的结构，里面约束了整个程序的框架和执行的大概流程，但并未涉及到业务层面的东西，只是将一个数据如何流入如何流出做了规范，只是提供了一个默认的逻辑实现。

```java
class Calculator{
    private int x = 0;
    private int y = 0;
    private CalcStrategy strategy = null;

    public Calculator(int x, int y) {
        this.x = x;
        this.y = y;
    }
    public Calculator(int x, int y, CalcStrategy strategy) {
        this(x, y);
        this.strategy = strategy;
    }
    public int calc(int x, int y) {
        return x + y;
    }
    /**
     * 只需关注接口，并且将接口用到的入参传递进去即可，并不关心到底具体是要如何进行业务封装
     * @return
     */
    public int result() {
        if (null != strategy) {
            return strategy.calc(x, y);
        }
        return calc(x, y);
    }
}
```

实现了不同的策略，就可以改变功能，传入的不同对象就像 Runnable 传入 Thread 中

```java
class AddStrategy implements CalcStrategy{
    public int calc(int x, int y) {
        return x + y;
    }
}
class SubStrategy implements CalcStrategy{
    public int calc(int x, int y) {
        return x - y;
    }
}
public class StrategyTest{
    public static void main(String[]args) {
//没有任何策略时的结果
        Calculator c = new Calculator(30, 24);
        System.out.println(c.result());

//传入减法策略的结果
        Calculator c1 = new Calculator(10, 30, new SubStrategy());
        System.out.println(c1.result());

//看到这里就可以看到策略模式强大了，算法可以随意设置，系统的结构并不会发生任何变化
        Calculator c2 = new Calculator(30, 40, new CalcStrategy() {
            public int calc(int x, int y) {
                return ((x + 10) - (y * 3)) / 2;
            }
        });
        System.out.println(c2.result());
    }
}
```


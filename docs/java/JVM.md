# JVM

Activate the cover feature by setting `coverpage` to **true**. See [coverpage configuration](configuration.md#coverpage).

![](https://s2.ax1x.com/2020/01/30/1lQ9Fe.md.png#id=zworr&originHeight=381&originWidth=680&originalType=binary&ratio=1&status=done&style=none)

## Java内存结构

### 1.程序计数器(Program Counter Register): 寄存器

- 作用: 记住下一条指令的执行地址
- 特点: 线程私有,不存在内存溢出问题



### 2.虚拟机栈(JVM Stacks): 线程运行需要的内存空间,先进后出

- 每个栈由多个栈帧组成,对应着每次方法调用时占用的内存
- 每个线程只能有一个活动栈帧(栈顶部的栈帧,正在执行的方法),对应着当前正在执行的那个方法



```java
public class StacksDemos {
    public static void main(String[] args) {
        method1();
    }

    public static void method1(){
        method2();
    }
    public static void method2(){
        System.out.println("method2");
    }

}
```


- 垃圾回收不涉及栈内存
- 栈内存的大小设定: -Xss1024k ,总内存一定,栈内存设定越大,线程数量越少;栈内存大可以支持更多次的方法调用
- 局部变量不会有线程安全问题: 如果方法内局部变量没有逃离方法的作用范围,则是线程安全的,否则需要考虑线程安全问题
- 栈内存溢出:java.lang.StackOverflowError
    - 方法一直调用,只进栈不出栈,导致栈内存溢出



```java
    public class StackOverFlow {
        private static int counter;
        public static void main(String[] args) {
            try{
                m1();
            }catch (Throwable e){
                e.printStackTrace();
                System.out.println(counter);
            }

        }

        public static void m1(){

            counter++;
            m1();

        }
    }
```


      - 栈帧过大,直接超过栈内存导致栈内存溢出



### 3.本地方法栈(Native Method Stacks):
给本地方法的运行提供内存空间
### 4.堆(Heap): 通过new关键字创建对象都会使用堆内存

- 线程共享
- 有垃圾回收机制
- 堆内存溢出:java.lang.OutOfMemoryError: Java heap space



```java
    public class OutOfMemory {
        public static void main(String[] args) {
            int i=0;
            List<String> list=new ArrayList<>();
            String a="love";
            try{
                while (true){
                    a=a+a;
                    list.add(a);
                    i++;
                }

            }catch (Throwable t){
                t.printStackTrace();
                System.out.println(i);
            }
        }
    }
```


- 修改堆内存大小:-Xmx2m
- 堆内存诊断
    - jps工具:查看当前系统中有哪些java进程
    - jmap:工具:查看堆内存占用情况; jmap -heap 进程id
    - jconsole工具:图形界面的,多功能的监测工具,可以连续监测



### 5.方法区(Method Area)

- java.lang.OutOfMemoryError: Compressed class space



```java
    import jdk.internal.org.objectweb.asm.ClassWriter;
    import jdk.internal.org.objectweb.asm.Opcodes;

    /**
     + 演示元空间的内存溢出
     + -XX:MaxMetaspaceSize=8m 这里也是一个问题,设置成8m启动不了,10m才能正常启动
     + 想演示java.lang.OutOfMemoryError: Metaspace但是出现的确是java.lang.OutOfMemoryError: Compressed class space的错误,不知道为什么
     */
    public class MetaSpace extends ClassLoader{
        public static void main(String[] args) {
            int j=0;

            try {
                MetaSpace metaSpace = new MetaSpace();
                for (int i = 0; i < 10000; i++, j++) {
                    ClassWriter cw = new ClassWriter(0);

                    cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
                    byte[] code = cw.toByteArray();
                    //执行类的加载
                    metaSpace.defineClass("Class" + i, code, 0, code.length);

                }
            } finally {
                System.out.println(j);
            }
        }
    }
```


- java.lang.OutOfMemoryError: Metaspace
- java.lang.OutOfMemoryError: PermGen space
- 设置元空间的大小(jdk1.8之后):-XX:MaxMetaspaceSize=8m
- 设置永久代内存大小(jdk1.8之前):-XX:MaxPermSize=8m
- 常量池



```java
    public class ConstantPool {
        public static void main(String[] args) {
            System.out.println("luopeiwen");
        }
    }
    //-------------------------------------------------分割线---------------------------------------
    D:\IdeaProjects\java-basic\out\production\jvm\com\baidu\methodArea>javap -v ConstantPool.class
    Classfile /D:/IdeaProjects/java-basic/out/production/jvm/com/baidu/methodArea/ConstantPool.class
      Last modified 2020-1-31; size 579 bytes
      MD5 checksum 33892d27aae36066bb2addf1aa66b1b8
      Compiled from "ConstantPool.java"
    public class com.baidu.methodArea.ConstantPool
      minor version: 0
      major version: 52
      flags: ACC_PUBLIC, ACC_SUPER
    Constant pool:
       #1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
       #2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
       #3 = String             #23            // luopeiwen
       #4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
       #5 = Class              #26            // com/baidu/methodArea/ConstantPool
       #6 = Class              #27            // java/lang/Object
       #7 = Utf8               <init>
       #8 = Utf8               ()V
       #9 = Utf8               Code
      #10 = Utf8               LineNumberTable
      #11 = Utf8               LocalVariableTable
      #12 = Utf8               this
      #13 = Utf8               Lcom/baidu/methodArea/ConstantPool;
      #14 = Utf8               main
      #15 = Utf8               ([Ljava/lang/String;)V
      #16 = Utf8               args
      #17 = Utf8               [Ljava/lang/String;
      #18 = Utf8               SourceFile
      #19 = Utf8               ConstantPool.java
      #20 = NameAndType        #7:#8          // "<init>":()V
      #21 = Class              #28            // java/lang/System
      #22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
      #23 = Utf8               luopeiwen
      #24 = Class              #31            // java/io/PrintStream
      #25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
      #26 = Utf8               com/baidu/methodArea/ConstantPool
      #27 = Utf8               java/lang/Object
      #28 = Utf8               java/lang/System
      #29 = Utf8               out
      #30 = Utf8               Ljava/io/PrintStream;
      #31 = Utf8               java/io/PrintStream
      #32 = Utf8               println
      #33 = Utf8               (Ljava/lang/String;)V
    {
      public com.baidu.methodArea.ConstantPool();
        descriptor: ()V
        flags: ACC_PUBLIC
        Code:
          stack=1, locals=1, args_size=1
             0: aload_0
             1: invokespecial #1                  // Method java/lang/Object."<init>":()V
             4: return
          LineNumberTable:
            line 3: 0
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0       5     0  this   Lcom/baidu/methodArea/ConstantPool;

      public static void main(java.lang.String[]);
        descriptor: ([Ljava/lang/String;)V
        flags: ACC_PUBLIC, ACC_STATIC
        Code:
          stack=2, locals=1, args_size=1
             0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
             3: ldc           #3                  // String luopeiwen
             5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
             8: return
          LineNumberTable:
            line 5: 0
            line 6: 8
          LocalVariableTable:
            Start  Length  Slot  Name   Signature
                0       9     0  args   [Ljava/lang/String;
    }
    SourceFile: "ConstantPool.java"
```


- Stringtable:new String("a");在串池中,但是String s=new String("a")+new String("b");的s是由两个对象拼接而成,

存在于堆中,不在串池中
- 常量池中的字符串仅是符号,第一次用到时才变为对象,
- 利用串池的机制,来避免重复创建字符串对象
- 字符串变量拼接的原理是StringBuilder(1.8)
- 字符串常量拼接的原理是编译器优化
- 可以使用intern方法,主动将串池中还没有的额字符串对象放入串池
- 1.8将这个字符串对象尝试放入串池,如果有则不会放入,如果没有则放入串池,会把串池中的对象返回



## 五种引用:
### 1.强引用:

      - 只有所有GC Roots对象都不通过**_强引用_**引用该对象,该对象才能被垃圾回收



### 2.软引用(softReference):

      - 仅有软引用引用该对象时,在垃圾回收后,内存仍不足时再次触发垃圾回收,回收软引用对象,
      - 可以配合引用队列来释放软引用自身



### 3.弱引用(weakReference)

      - 仅有弱引用引用该对象是,在垃圾回收时,无论是否充足,都会回收弱引用对象,
      - 可以配合引用队列来释放弱引用自身



```java
/**
 + 弱引用示例
 + -Xmx20m -XX:-PrintGCDetails -verbose:gc
 */
public class WRDemo1 {
    private static final int  _4MB=4*1024*1024;

    public static void main(String[] args) {
        weakRef();
    }
    //当产生第四个对象时,内存还够放一个对象,没有移除第三个,把第四个对象产生后内存不够,移除第四个
    //只有当full gc时才会不管内存够不够,都会移除弱引用
    private static void weakRef(){
        List<WeakReference<byte[]>> list=new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            WeakReference<byte[]> reference=new WeakReference<>(new byte[_4MB]);
            list.add(reference);
            for (WeakReference<byte[]> w : list) {
                System.out.print(w.get()+" ");
            }
            System.out.println();
        }

    }
}
//---------------------分割线-下面是打印结果------------------------
[B@4554617c 
[B@4554617c [B@74a14482 
[B@4554617c [B@74a14482 [B@1540e19d 
[GC (Allocation Failure)  14229K->13028K(19968K), 0.0007915 secs]
[B@4554617c [B@74a14482 [B@1540e19d [B@677327b6 
[GC (Allocation Failure)  17236K->13068K(19968K), 0.0004885 secs]
[B@4554617c [B@74a14482 [B@1540e19d null [B@14ae5a5 
[GC (Allocation Failure)  17276K->13060K(19968K), 0.0004925 secs]
[B@4554617c [B@74a14482 [B@1540e19d null null [B@7f31245a 
[GC (Allocation Failure)  17267K->13076K(19968K), 0.0006208 secs]
[B@4554617c [B@74a14482 [B@1540e19d null null null [B@6d6f6e28 
[GC (Allocation Failure)  17282K->13100K(19968K), 0.0003957 secs]
[B@4554617c [B@74a14482 [B@1540e19d null null null null [B@135fbaa4 
[GC (Allocation Failure)  17419K->13132K(18944K), 0.0007010 secs]
[B@4554617c [B@74a14482 [B@1540e19d null null null null null [B@45ee12a7 
[GC (Allocation Failure)  17317K->13116K(19456K), 0.0004451 secs]
[Full GC (Ergonomics)  13116K->644K(14336K), 0.0052044 secs]
null null null null null null null null null [B@330bedb4
```


### 4.虚引用(PhantomReference)

      - 必须配合引用队列使用,主要配合ByteBuffer使用,被引用对象回收时,会将虚应用入队,由Reference Handler线程

调用虚引用相关方法释放直接内存



### 5.终结器引用(FinalReference)

      - 无需手动编码,但其内部配合引用队列使用,在垃圾回收时,终结器引用入队(被引用对象暂时没有被回收),再由Finalizer线程

通过终结器引用找到被引用对象并调用它的finalize方法,第二次GC时才能回收被引用对象



## 回收算法

- 标记清楚算法:优点:快;缺点:内存碎片
- 标记整理算法:优点:没有内存碎片;缺点:慢
- 复制算法:优点:无内存碎片;缺点:需要双倍的内存空间
- 回收机制:

![](https://s2.ax1x.com/2020/02/02/1YeU6s.md.png#id=kklIR&originHeight=203&originWidth=680&originalType=binary&ratio=1&status=done&style=none)
- 对象首先分配在伊甸园区域
- 新生代空间不足时,触发minor gc,伊甸园和from存活的对象使用copy赋值到to区,存活的对象年龄呢加1并且交换from to
- minor gc会触发stop the world,暂停其他用户线程,等垃圾回收结束,用户线程才恢复运行
- 当对象寿命超过阈值时,会晋升至老年代,最大寿命是15(4bit)
- 当老年代空间不足,先尝试进行minor gc,如果之后空间仍不足,那么触发full gc,stw的时间更长



## JVM相关参数
| 含义 | 参数 |
| --- | --- |
| 堆初始大小 | -Xms |
| 堆最大大小 | -Xmx或-XX:MaxHeapSize=size |
| 新生代大小 | -Xmn或(-XX:NewSize=size -XX:MaxNewSize=size) |
| 幸存区比例(动态) | -XX:InitialSurvivorRatio=ratio和-XX:+UserAdaptiveSizePolicy |
| 幸存区比例 | -XX:SurvivorRatio=ratio |
| 晋升阈值 | -XX:MaxTenuringThreshold=threshold |
| 晋升详情 | -XX:+PrintTenuringDistribution |
| GC详情 | -XX:+PrintGCDetail -verbose:gc |
| FullGC前MinorGC | -XX:+ScavengeBeforeFullGC |

1. 串行:-XX:+UseSerialGC=Serial+SerialOld

![](https://s2.ax1x.com/2020/02/02/1YJ0BD.md.png#id=vuOv6&originHeight=233&originWidth=680&originalType=binary&ratio=1&status=done&style=none)
1. 吞吐量优先:-XX:+Use



## JMM(Java Memory Model)Java内存模型

> 这个模块通过分析问题代码,引入多线程下原子性,可见性,有序性的概念,并分析原因和介绍解决方法


### 原子性

分析一段代码:

```java
public class SyncDemo02 {

    static int a;
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            for (int i = 0; i < 5000; i++) {
                a++;
            }
        });

        Thread t2=new Thread(()->{
            for (int i = 0; i < 5000; i++) {
                a--;
            }

        });
        t1.start();
        t2.start();

        t1.join();
        t2.join();
        
        System.out.println(a);
    }
}
```

上面结果可能是正数,负数或零,因为Java中对静态变量的自增自减并不是原子操作

对于a++而言(a是静态变量),则实际会产生如下的JVM字节码指令:

```
getstatic  a //获取静态变量a的值
iconst_1     //准备常量1  (当int取值-1~5时，JVM采用iconst指令将常量压入操作数栈中)
iadd         //加法
putstatic  a //将修改后的值存入静态变量a
```

而对应a--类似:

```
getstatic  a
iconst_1
isub          //加法
putstatic   a
```

若是顺序执行则指令执行顺序为:

```
getstatic  a 
iconst_1     
iadd         
putstatic 
getstatic  a
iconst_1
isub
putstatic   a
```

而多线程下则上面几行指令则可能会交错运行,多个线程获取时间片,轮流执行

上面代码要想保证最后输出结果是0,则可更改为如下代码:

```java
public class SyncDemo02 {
    static int a;
    static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            for (int i = 0; i < 5000; i++) {
                synchronized(object){
                a++;
              }
            }
        });
        //注意synchronized()括号中的对象要是同一个,不然也不起作用
        //即当一个线程对这个对象加锁后,没释放之前,另一个线程遇到这个对象是加锁状态,则会等待其释放,此线程阻塞
        Thread t2=new Thread(()->{
            for (int i = 0; i < 5000; i++) {
              synchronized(object){
                a--;
              }
            }
        });
        t1.start();
        t2.start();

        t1.join();
        t2.join();
        
        System.out.println(a);
    }
}
```

以上做法可以保证a++和a--的四个指令在一起执行,即防止指令交错执行.

对加锁的我的理解是:需要加锁的代码像一个房间,没有锁之前谁都可以进入房间做一些事情,加了synchronized()就像一个人进入房间后,

从里面反锁,在房间内办完了事,打开门,其他人进入后再次从里面锁门,做事情.

当每次加锁会执行monitorenter 对应每次解锁会执行monitorexit ,所以上面代码每个线程都会执行5000的加锁解锁,此操作是十分消耗资源的,

故可优化如下:

```java
public class SyncDemo02 {
    static int a;
    static Object object=new Object();
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
          //锁的粒度大一些,减少加锁解锁的操作,减少不必要的消耗
          synchronized(object){
            for (int i = 0; i < 5000; i++) {
              a++;
            }
          }
        });
        Thread t2=new Thread(()->{
          synchronized(object){
            for (int i = 0; i < 5000; i++) {
              a--;
            }
          }
        });
        t1.start();
        t2.start();

        t1.join();
        t2.join();
        
        System.out.println(a);
    }
}
```

### 可见性

---

变量的内存可见性:

```java
public class volatileDemo01 {
    static boolean flag=true;
    static int a=0;
    public static void main(String[] args) throws InterruptedException {
        Thread t1=new Thread(()->{
            while (flag){
                a++;
                //System.out.println("循环中"+a);// 比较消耗资源,和让线程睡眠效果差不多
            }
        });
        t1.start();

        Thread.sleep(1000);
        flag=false;
        System.out.println("主线程已修改flag");
    }
}
```

运行以上程序,控制台会打印:"主线程已修改flag",但是程序并未停下

原因是:虽然主线程已经修改了flag的值并且已经更新到主存中,但是线程t1由于循环多次调用flag,触发JIT(及时编译)优化,会使变量flag加载到t1线程的工作内存中,每次都是从工作内存中读取,工作内存中flag一直是true,导致进入死循环;

可以给flag使用volatile修饰:`volatile static boolean flag=true;`

> JIT(及时编译)优化:当虚拟机发现某个方法或代码块的运行特别频繁时，就会把这些代码认定为“热点代码”,

变量会被当前线程的工作内存中,读取更快


---

Volatile(易变关键字):

1. 可用来修饰成员变量和静态成员变量,可避免线程从自己的工作缓存中查找变量的值,必须到主存中获取它的值,

线程操作volatile修饰的变量都是直接操作主存,仅仅保证可见性,不能保证原子性
1. 适用场景:一个线程写,多个线程读
1. synchronized既可以保证代码块的原子性,也可以保证代码块内变量的可见性,但是属于重量级操作,性能相对较低

### 有序性

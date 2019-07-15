---
title: Java杂记之static&threadlocal 
date: 2018-07-30 22:11:00
tags: Java
comments: true
---
>闲来无事，写篇文章，如果有幸被看到了，希望你能当做下酒菜。
>本文思路不是特别严谨，主要是每个人对于Java知识点的理解都是有偏颇不断修正的，所以不太理解的就会绕些弯子，如果你看的费劲，敬请谅解。

## 1 Integer,String
先说一些和本文无关的事情，简称前菜，对于经验比较丰富的java程序员来说这些都是毛毛雨。
```
@Test
public void testInteger(){
    Integer a = new Integer(123);
    Integer b = new Integer(123);
    //false
    Assert.assertTrue(a==b);
}
```
这个testcase的结果是false，那么这个呢
```
@Test
public void testInteger(){
    Integer a = 123;
    Integer b = 123;
    //true
    Assert.assertTrue(a==b);
}
```
早就听说Integer内部有cache,-128到127的int值都会放在cache里面。所以a==b可以理解，因为值相等，都是同一个cache的引用，所以上述结果为true。这样描述可以大体让人接受，可是仔细想想，上面代码和我说的有半毛线关系吗？于是我赶紧去翻了下Integer的源码，现在截取两段
```
/**
     * Cache to support the object identity semantics of autoboxing for values between
     * -128 and 127 (inclusive) as required by JLS.
     *
     * The cache is initialized on first usage.  The size of the cache
     * may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
     * During VM initialization, java.lang.Integer.IntegerCache.high property
     * may be set and saved in the private system properties in the
     * sun.misc.VM class.
     */

    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
这是一个静态内部类，带着一个静态代码块，大致意思是从vmproperty中读一下high的值，默认是127，不默认的情况是high,然后一个cache数据就这样生成了，
那么这个和上面的==又有啥关系呢？这时候会想哪里用到这个cache,往下正好有个方法用到了，看下valueOf方法
```
/**
     * Returns an {@code Integer} instance representing the specified
     * {@code int} value.  If a new {@code Integer} instance is not
     * required, this method should generally be used in preference to
     * the constructor {@link #Integer(int)}, as this method is likely
     * to yield significantly better space and time performance by
     * caching frequently requested values.
     *
     * This method will always cache values in the range -128 to 127,
     * inclusive, and may cache other values outside of this range.
     *
     * @param  i an {@code int} value.
     * @return an {@code Integer} instance representing {@code i}.
     * @since  1.5
     */
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
很明确，如果进入了这个方法，先看看有没有cache，都得话就取cache,没有再new一个新的Integer.
那么和之前的==又有啥关系呢。。。这句话问的有点多，不卖关子了，其实Integer a = 123;这句话有个装箱的过程，这个过程jvm会调用Integer.valueOf，前面的内容一下子串联起来了，是不是豁然开朗了。。啥？没感觉，确实对与大佬来说这点还是有些基础了。怎么知道jvm调用了哪个方法呢，反编译，看字节码，反正我是看不懂，但是可以友情提示下对于Integer,自动拆箱调用了Integer.intValue,自动装箱调用了Integer.ValueOf.

String其实和说起来和Integer有那么点不同那么看看这个case
```
@Test
public void testString(){
    String a = "abc";
    String b = "abc";
    //true
    Assert.assertTrue(a==b);
 }
```
这个是true的原因，一句话概括->字符串常量池。jdk1.7之前，常量池是在方法区中的，jdk1.7以后移到了堆中，简答来说，如果常量池没有的字符串，会现在常量池里面创建一份，然后将其引用返回给String对象，以后遇到则返回该常量池的引用，所以a b 是同一个常量池对象的引用。
```
/**
     * Returns a canonical representation for the string object.
     * <p>
     * A pool of strings, initially empty, is maintained privately by the
     * class {@code String}.
     * <p>
     * When the intern method is invoked, if the pool already contains a
     * string equal to this {@code String} object as determined by
     * the {@link #equals(Object)} method, then the string from the pool is
     * returned. Otherwise, this {@code String} object is added to the
     * pool and a reference to this {@code String} object is returned.
     * <p>
     * It follows that for any two strings {@code s} and {@code t},
     * {@code s.intern() == t.intern()} is {@code true}
     * if and only if {@code s.equals(t)} is {@code true}.
     * <p>
     * All literal strings and string-valued constant expressions are
     * interned. String literals are defined in section 3.10.5 of the
     * <cite>The Java&trade; Language Specification</cite>.
     *
     * @return  a string that has the same contents as this string, but is
     *          guaranteed to be from a pool of unique strings.
     */
    public native String intern();
```
intern是个native方法，使用起来也是比较方便的
```
s1 = new String("bbb").intern();
s2 = "bbb";
```
另外简单说一下String的设计思想。下面这段话是网上摘抄的
```
Java中的String被设计成不可变的，出于以下几点考虑：

1. 字符串常量池的需要。字符串常量池的诞生是为了提升效率和减少内存分配。可以说我们编程有百分之八十的时间在处理字符串，而处理的字符串中有很大概率会出现重复的情况。正因为String的不可变性，常量池很容易被管理和优化。

2. 安全性考虑。正因为使用字符串的场景如此之多，所以设计成不可变可以有效的防止字符串被有意或者无意的篡改。从java源码中String的设计中我们不难发现，该类被final修饰，同时所有的属性都被final修饰，在源码中也未暴露任何成员变量的修改方法。（当然如果我们想，通过反射或者Unsafe直接操作内存的手段也可以实现对所谓不可变String的修改）。

3. 作为HashMap、HashTable等hash型数据key的必要。因为不可变的设计，jvm底层很容易在缓存String对象的时候缓存其hashcode，这样在执行效率上会大大提升。
```
## 2 static关键字
static 用了很多次了，静态嘛顾名思义，放在变量前面就是静态变量，放在代码块前面，就是静态代码块，静态的意思就是固定的，在类加载的一开始就准备好了，类加载之初，初始化static parameters,static block.，静态变量放在哪里，方法区。代码中不管实例化了多少对象，这些变量都是共用的。那么他们线程安全吗，显然是不，让我们用代码来说明这一点
```
public class NormalCount implements Runnable {

    @Getter
    private static int count = 0;

    @Override
    public void run() {
        Double d = Math.random() * 1000;
        try {
            Thread.sleep(d.longValue());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        count++;
        System.out.println(Thread.currentThread().getName() + ": " + count);
    }

}
```
testCase如下,这种写法是先创建一个类然后并发的调用count++
```
@Test
public void testNormalnew() throws InterruptedException {
    ExecutorService es = Executors.newFixedThreadPool(100);
    for (int i = 0; i < 100; i++) {
        es.execute(new NormalCount());
    }
    es.shutdown();
    while (!es.isTerminated()) {
        Thread.sleep(3000);
    }
    Assert.assertEquals(100,NormalCount.getCount());
}
```
这里又想插句题外话了，不好的testcase真的会给你带来误解，如果testCase写的不行，最后可能告诉你错误的结论，最后给理解带来偏差，可以看到在NormalCount这个类里面count++前面我用了一个随机数去让线程睡眠，为什么要这样，如果没有这一步，testcase很可能不会报错，注意看我是创建了一个拥有100个线程的线程池，如果说for循环中的顺序去执行，每个count++的速度和时间都差不多，那么每个线程都是顺序执行的，实际上没有并发的场景。而误以为这是并发但是实际上是线程安全的，这就会带给你错误的结论。
那么问题来了，听说volatile可以解决变量多线程之间的可见性，那意思就是刚刚的变量前面加个volatile这个testCase就ok了吗,试试后发现不行，不管是static的volatile我们创建多个类去并发访问，还是not static的volatile我们用多个线程去访问一个类，都是没有通过testCase。这是为啥呢，其实还是对volatile有误解。
volatile可以让变量在多线程之间可见，可以防止jvm优化的指令重排，这段我简单说说，就是volatile变量每次使用前都会刷新(CAS机制)，但是因为count++这个操作是非原子性的，所以拿到变量的时候还是最新的，但是可能操作过程中已经是过期的数据了，最后会让数据不一致。所以用volatile最好是原子性的操作比如对boolean的判断，没有其他操作，它是完全ok的
上述代码如果想线程安全应该如何修改？有以下方法：
1 ++方法用synchronized加锁,效率会比较低
2 int 变量换成atomicInteger,这个类是线程安全的
好了关于static 就说这么多.

## 3 ThreadLocal分析
Threadlocal是用来干嘛的，没有用过的人会对她敬而远之，只有用过的人才知道没有那么高冷。threadlocal简单来说，是用来存放线程独有的变量，线程之间是不共用的(?如果引用了static变量那就....)。你可能会问了，那方法里面局部变量不也行吗，确实局部变量是线程独享的，但是我们想要在线程中多个参数里方便的传递一个变量，局部变量就做不到了。Threadlocal就是这么来的
```

This class provides thread-local variables.  These variables differ from
their normal counterparts in that each thread that accesses one (via its
{@code get} or {@code set} method) has its own, independently initialized
copy of the variable.  {@code ThreadLocal} instances are typically private
static fields in classes that wish to associate state with a thread (e.g.,
a user ID or Transaction ID).

<p>For example, the class below generates unique identifiers local to each
thread.
A thread's id is assigned the first time it invokes {@code ThreadId.get()}
and remains unchanged on subsequent calls.
<pre>
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadId {
// Atomic integer containing the next thread ID to be assigned
private static final AtomicInteger nextId = new AtomicInteger(0);

// Thread local variable containing each thread's ID
private static final ThreadLocal<Integer> threadId =
  new ThreadLocal<Integer>() {
      @Override 
      protected Integer initialValue() {
          return nextId.getAndIncrement();
  	}
};

// Returns the current thread's unique ID, assigning it if necessary
public static int get() {
  return threadId.get();
}
}
</pre>
<p>Each thread holds an implicit reference to its copy of a thread-local
variable as long as the thread is alive and the {@code ThreadLocal}
instance is accessible; after a thread goes away, all of its copies of
thread-local instances are subject to garbage collection (unless other
references to these copies exist).
```
来看看Java之父Josh Bloch怎么说，其实说的还是挺清楚的，threadlocal是一个线程独有的变量，通常声明为private static 是为了保证这个变量和线程具有同样的生命周期，因为内部使用了弱引用，所以如果没有别的变量引用threadlocal会被gc清理。至于线程安不安全，不能保证哦...注释里面的这个例子其实很经典了，为每个线程创建一个threadId,这个id用atomicInteger生成，保证并发情况下的唯一性。看下Threadlocal的源码分析很多了这里就不贴了，简单来说，每个Thread里面都有一个变量叫做threadlocalMap这个map的实现是一个继承了弱引用的entry，map里面的key是threadlocal对象，value是对象的值。Threadlocal的get方法就是通过Thread获得threadlocalmap,然后以threadlocal对象本身为键去求的value.看下我的例子
```
public class ThreadCount implements Runnable {

    private static ThreadLocal<Integer> localcount = ThreadLocal.withInitial(()->0);

    public int getLocalCount() {
        localcount.set(localcount.get()+1);
        return localcount.get();
    }

    @Override
    public void run() {
        for(int i =0;i<3;i++){
            Double d = Math.random() * 1000;
            try {
                Thread.sleep(d.longValue());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+": "+getLocalCount());
        }
    }
}
```
TestCase如下
```
@Test
public void testThreadLocals() throws InterruptedException {
    ThreadCount count = new ThreadCount();
    ExecutorService es = Executors.newFixedThreadPool(100);
    for(int i=0;i<100;i++){
        es.execute(count);
    }
    es.shutdown();
    while(!es.isTerminated()){
        Thread.sleep(3000);
    }
}
```
从打印的结果看，每个线程都打印了123,
```
pool-2-thread-81: 1
pool-2-thread-42: 1
pool-2-thread-6: 1
pool-2-thread-14: 2
pool-2-thread-80: 1
pool-2-thread-73: 2
pool-2-thread-25: 2
pool-2-thread-93: 1
pool-2-thread-15: 1
pool-2-thread-55: 1
```
这里我把线程池的默认线程个数改为10，结果如何呢？
```
pool-2-thread-5: 9
pool-2-thread-10: 9
pool-2-thread-1: 9
pool-2-thread-5: 10
pool-2-thread-7: 7
pool-2-thread-8: 11
pool-2-thread-9: 10
pool-2-thread-6: 8
```
居然有的线程的值到了9，10 这怎么可能，其实这个是对线程池不够了解，线程池会对已有的线程进行复用，所以如果一个线程用完以后是count的值3，但是被别的线程复用以后就会大于3了，这其实是threadlocal使用中需要注意的，如果用到了线程池，那用完threadlocal记得还原。
除了线程池的这个坑，threadlocal是不是可以说是线程安全的呢？其实讲道理这个问题是概念的混淆，threadlocal只是一个在threadlocalmap里面存放了key,value,如果这个value是个线程不安全的值，比如说是个static变量，所有线程都可以修改，那threadlocal就是不安全的。

## 4 总结
jvm内存模型，加载机制，各种关键字，final,static,synchronized,volatile这些都是java工程师的必备武器，还是好好修炼吧 


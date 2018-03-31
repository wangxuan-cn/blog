---
title: Spring Cloud中Hystrix线程隔离导致ThreadLocal数据丢失
date: 2018-03-30 09:26:28
tags:
    - 多线程
    - ThreadLocal
categories:
    - 多线程
    - ThreadLocal
---
在Spring Cloud中我们用Hystrix来实现断路器，Zuul中默认是用信号量（Hystrix默认是线程）来进行隔离的，我们可以通过配置使用线程方式隔离。  

在使用线程隔离的时候，有个问题是必须要解决的，那就是在某些业务场景下通过ThreadLocal来在线程里传递数据，用信号量是没问题的，从请求进来，但后续的流程都是通一个线程。  

当隔离模式为线程时，Hystrix会将请求放入Hystrix的线程池中去执行，这个时候某个请求就有A线程变成B线程了，ThreadLocal必然消失了。  

下面我们通过一个简单的列子来模拟下这个流程：  
```
public class CustomThreadLocal {
  static ThreadLocal<String> threadLocal = new ThreadLocal<>();
  public static void main(String[] args) {
    new Thread(new Runnable() {
      @Override
      public void run() {
        CustomThreadLocal.threadLocal.set("猿天地");
        new Service().call();
      }
    }).start();
  }
}

class Service {
  public void call() {
    System.out.println("Service:" + Thread.currentThread().getName());
    System.out.println("Service:" + CustomThreadLocal.threadLocal.get());
    new Dao().call();
  }
}

class Dao {
  public void call() {
    System.out.println("==========================");
    System.out.println("Dao:" + Thread.currentThread().getName());
    System.out.println("Dao:" + CustomThreadLocal.threadLocal.get());
  }
}
```
我们在主类中定义了一个ThreadLocal用来传递数据，然后起了一个线程，在线程中调用Service中的call方法，并且往Threadlocal中设置了一个值，在Service中获取ThreadLocal中的值，然后再调用Dao中的call方法，也是获取ThreadLocal中的值，我们运行下看效果：  
```
Service:Thread-0
Service:猿天地
==========================
Dao:Thread-0
Dao:猿天地
```
可以看到整个流程都是在同一个线程中执行的，也正确的获取到了ThreadLocal中的值，这种情况是没有问题的。

接下来我们改造下程序，进行线程切换，将调用Dao中的call重启一个线程执行：  
```
public class CustomThreadLocal {
  static ThreadLocal<String> threadLocal = new ThreadLocal<>();
  public static void main(String[] args) {
    new Thread(new Runnable() {
      @Override
      public void run() {
        CustomThreadLocal.threadLocal.set("猿天地");
        new Service().call();
      }
    }).start();
  }
}

class Service {
  public void call() {
    System.out.println("Service:" + Thread.currentThread().getName());
    System.out.println("Service:" + CustomThreadLocal.threadLocal.get());
    //new Dao().call();
    new Thread(new Runnable() {
      @Override
      public void run() {
        new Dao().call();
      }
    });
  }
}

class Dao {
  public void call() {
    System.out.println("==========================");
    System.out.println("Dao:" + Thread.currentThread().getName());
    System.out.println("Dao:" + CustomThreadLocal.threadLocal.get());
  }
}
```
再次运行，看效果：  
```
Service:Thread-0
Service:猿天地
==========================
Dao:Thread-1
Dao:null
```
可以看到这次的请求是由2个线程共同完成的，在Service中还是可以拿到ThreadLocal的值，到了Dao中就拿不到了，因为线程已经切换了，这就是开始讲的ThreadLocal的数据会丢失的问题。

那么怎么解决这个问题呢，其实也很简单，只需要改一行代码即可:  
`static ThreadLocal<String> threadLocal = new InheritableThreadLocal<>();`  
将ThreadLocal改成InheritableThreadLocal，我们看下改造之后的效果：  
```
Service:Thread-0
Service:猿天地
==========================
Dao:Thread-1
Dao:猿天地
```
值可以正常拿到，InheritableThreadLocal就是为了解决这种线程切换导致ThreadLocal拿不到值的问题而产生的。

要理解InheritableThreadLocal的原理，得先理解ThreadLocal的原理，我们稍微简单的来介绍下ThreadLocal的原理：
- 每个线程都有一个 ThreadLocalMap 类型的 threadLocals 属性，ThreadLocalMap 类相当于一个Map，key 是 ThreadLocal 本身，value 就是我们设置的值。  
```
public class Thread implements Runnable {     
    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```
- 当我们通过 threadLocal.set("猿天地"); 的时候，就是在这个线程中的 threadLocals 属性中放入一个键值对，key 是 当前线程，value 就是你设置的值猿天地。
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
- 当我们通过 threadlocal.get() 方法的时候，就是根据当前线程作为key来获取这个线程设置的值。
```
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
通过上面的介绍我们可以了解到threadlocal能够传递数据是用Thread.currentThread()当前线程来获取，也就是只要在相同的线程中就可以获取到前方设置进去的值。

如果在threadlocal设置完值之后，下步的操作重新创建了一个线程，这个时候Thread.currentThread()就已经变了，那么肯定是拿不到之前设置的值。具体的问题复现可以参考上面我的代码。

那为什么InheritableThreadLocal就可以呢？

InheritableThreadLocal这个类继承了ThreadLocal，重写了3个方法，在当前线程上创建一个新的线程实例Thread时，会把这些线程变量从当前线程传递给新的线程实例。  
```
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```
通过上面的代码我们可以看到InheritableThreadLocal 重写了childValue, getMap,createMap三个方法，当我们往里面set值的时候，值保存到了inheritableThreadLocals里面，而不是之前的threadLocals。

关键的点来了，为什么当创建新的线程池，可以获取到上个线程里的threadLocal中的值呢？原因就是在新创建线程的时候，会把之前线程的inheritableThreadLocals赋值给新线程的inheritableThreadLocals，通过这种方式实现了数据的传递。

源码最开始在Thread的init方法中，如下：
```
if(parent.inheritableThreadLocals != null)
  this.inheritableThreadLocals = ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```
createInheritedMap如下：
```
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap){       
  return new ThreadLocalMap(parentMap);
}
```
赋值代码：
```
private ThreadLocalMap(ThreadLocalMap parentMap) {
    Entry[] parentTable = parentMap.table;
    int len = parentTable.length;
    setThreshold(len);
    table = new Entry[len];
    for (int j = 0; j < len; j++) {
        Entry e = parentTable[j];
        if (e != null) {
            @SuppressWarnings("unchecked")
            ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
            if (key != null) {
                Object value = key.childValue(e.value);
                Entry c = new Entry(key, value);
                int h = key.threadLocalHashCode & (len - 1);
                while (table[h] != null)
                    h = nextIndex(h, len);
                table[h] = c;
                size++;
            }
        }
    }
}
```
到此为止，通过inheritableThreadLocals我们可以在父线程创建子线程的时候将Local中的值传递给子线程，这个特性已经能够满足大部分的需求了，但是还有一个很严重的问题是如果是在线程复用的情况下就会出问题，比如线程池中去使用inheritableThreadLocals 进行传值，因为inheritableThreadLocals 只是会再新创建线程的时候进行传值，线程复用并不会做这个操作，那么要解决这个问题就得自己去扩展线程类，实现这个功能。

不要忘记我们是做Java的哈，开源的世界有你需要的任何东西，下面我给大家推荐一个实现好了的Java库，是阿里开源的transmittable-thread-local。  
GitHub地址：https://github.com/alibaba/transmittable-thread-local 

主要功能就是解决在使用线程池等会缓存线程的组件情况下，提供ThreadLocal值的传递功能，解决异步执行时上下文传递的问题。

JDK的InheritableThreadLocal类可以完成父线程到子线程的值传递。但对于使用线程池等会缓存线程的组件的情况，线程由线程池创建好，并且线程是缓存起来反复使用的；这时父子线程关系的ThreadLocal值传递已经没有意义，应用需要的实际上是把 任务提交给线程池时的ThreadLocal值传递到任务执行时。

transmittable-thread-local使用方式分为三种，修饰Runnable和Callable，修饰线程池，Java Agent来修饰JDK线程池实现类

接下来给大家演示下线程池的修饰方式，首先来一个非正常的案例，代码如下：
```
public class CustomThreadLocal {
  static ThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
  static ExecutorService pool = Executors.newFixedThreadPool(2);
  public static void main(String[] args) {
    for (int i = 0; i < 100; i++) {
      int j = i;
      pool.execute(new Thread(new Runnable() {
        @Override
        public void run() {
          CustomThreadLocal.threadLocal.set("猿天地" + j);
          new Service().call();
        }
      }));
    }
  }
}

class Service {
  public void call() {
    CustomThreadLocal.pool.execute(new Runnable() {
      @Override
      public void run() {
        new Dao().call();
      }
    });
  }
}

class Dao {
  public void call() {
    System.out.println("Dao:" + CustomThreadLocal.threadLocal.get());
  }
}
```
运行上面的代码出现的结果是不正确的，输出结果如下：
```
Dao:猿天地99
Dao:猿天地99
Dao:猿天地99
Dao:猿天地99
Dao:猿天地99
Dao:猿天地99
Dao:猿天地99
```
正确的应该是从1到100，由于线程的复用，值被替换掉了才会出现不正确的结果

接下来使用transmittable-thread-local来改造有问题的代码，添加transmittable-thread-local的Maven依赖：
```
<dependency>   
  <groupId>com.alibaba</groupId>
  <artifactId>transmittable-thread-local</artifactId>
  <version>2.2.0</version>
</dependency>
```
只需要修改2个地方,修饰线程池和替换InheritableThreadLocal：
```
static TransmittableThreadLocal<String> threadLocal = new TransmittableThreadLocal<>();
static ExecutorService pool = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(2));
```
正确的结果如下：
```
Dao:猿天地93
Dao:猿天地94
Dao:猿天地95
Dao:猿天地96
Dao:猿天地97
Dao:猿天地98
Dao:猿天地99
```
到这里我们就已经可以完美的解决线程中，线程池中ThreadLocal数据的传递了，各位看官又疑惑了，标题不是讲的Spring Cloud中如何解决这个问题么，我也是在Zuul中发现这个问题的，解决方案已经告诉大家了，至于怎么解决Zuul中的这个问题就需要大家自己去思考了，后面有时间我再分享给大家。

本文作者：尹吉欢
原文链接：https://mp.weixin.qq.com/s/NxBLhGo6MzClGpQP9WXctA
版权归作者所有，转载请注明出处

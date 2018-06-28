---
title: Android中的内存泄露
date: 2016-11-22 15:25:54
tags:
---
#### Android中的常见内存泄露
Android应用程序本身系统分配的内存有限，一旦发生泄漏，程序就会变得卡顿，直至**OOM**崩溃。简单分析下常见的错误
##### 非静态内部类会持有外部类的一个隐式引用
  Activity是如何泄漏的，只要**非静态的匿名类对象**没有被回收，Activity就不会被回收，Activity所关联的资源和视图都不会被回收，发生比较严重的`内存泄漏`。
看这个问题之前，我们先回顾下java的**内部类**
<!--more-->
``` java
class Outter{
    public class Inner {}
    public void foo(Inner c){
        System.out.println(c);
    }
}
public class Main {
    public static void main(String[] args)throws Exception{
        Outter o1 = new Outter();
        Outter o2 = new Outter();
        Outter.Inner i1 = o1.new Inner();
        Outter.Inner i2 = o2.new Inner();
        o1.foo(i2);
    }
}
```
**非静态的内部类会持有外部类的一个隐式引用**，上面在Outter类内部定义了Inner类，在后边main里创建了两个Inner实例，注意创建内部类的时候
> Outter.Inner i1 = o1.new Inner();

**注意：**在用new**创建内部类**时，前边必须限定外部对象(内部类实例必须要访问到外部对象引用)：o1；如果是在 Outter类内部这个外部引用可以省略，它**默认会用传递外部this引用**。

``` java
class Outter {
 
    public class Inner{}
 
    public void test() {
        new Inner(); // 相当于this.new Inner(); 也可以写为Outter.this.new Inner();
    }
} 
```

换成我们的Activity

``` java
public class MainActivity extends Activity {
 
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    exampleOne();
  }
 
  private void exampleOne() {
    newThread() {//匿名内部类，非静态的匿名类会持有外部类的一个隐式引用
      @Override
      publicvoid run() {
        while(true) {
          SystemClock.sleep(1000);
        }
      }
    }.start();
  }
}
```
这个匿名的Thread类会**持有外部的Activity**的对象，从而造成`内存泄露`。
要解决MainActivity的内存泄漏问题，只需把非静态的Thread匿名类定义成静态的内部类就行了（静态的内部类不会持有外部类的一个隐式引用）

``` java
public class MainActivity extends Activity {
  private MyThread mThread;
 
  @Override
  protectedvoid onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    exampleThree();
  }
 
  privatevoid exampleThree() {
    mThread = new MyThread();
    mThread.start();
  }
 
  private static class MyThread extends Thread {
    private boolean mRunning = false;
 
    @Override
    publicvoid run() {
      mRunning = true;
      while(mRunning) {
        SystemClock.sleep(1000);
      }
    }
 
    publicvoid close() {
      mRunning = false;
    }
  }
 
  @Override
  protectedvoid onDestroy() {
    super.onDestroy();
    if(mThread!=null)
        mThread.close();
  }
}
```
这样当Activity结束销毁时，MyThread不会持有外部Activity的引用也就不会内存泄露。在onDestroy()方法中结束了新创建的线程，保证了thread不会发生泄漏。

#### 其他常见内存泄露方式
- 数据库Cursor没有关闭
- 调用registerReceiver后未调用解绑（unregisterReceiver）
- 流对象InputStream/OutputStream未关闭
- Bitmap使用后未调用recycle()
- activity使用静态成员
- 等等。。。。

泄露的方式很多，就不一一列举了。怎么去避免，去发现才是关键。下面简单说明下如何发现内存泄露

###### 1.使用开源框架LeakCanary
LeakCanary的优点，使用简单，展示效果好。下面介绍如何使用
在 `build.gradle` 中加入引用

```gradle
 dependencies {
   debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
   releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
 }
```
在 Application 中注册：
``` java
public class App extends Application {

  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}
```
 这样，在 debug的时候，如果检测到某个 activity 有内存泄露，LeakCanary 就是自动地显示一个通知。
 更多介绍请看[LeakCanary 中文使用说明][1]
###### 2.使用Android Studio自带工具
打开Android Studio，编译代码，在模拟器或者真机上运行App，然后点击![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161148890.png)，在Android Monitor下点击Monitor对应的Tab，进入如下界面
![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161234794.png)
在Memory一栏中，可以观察不同时间App内存的动态使用情况，**点击![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161348283.png)**可以手动触发GC，**点击![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161402738.png)**可以进入HPROF Viewer界面，查看Java的Heap，如下图
![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161426494.png)
`Reference Tree`代表指向该实例的引用，可以从这里面查看内存泄漏的原因，`Shallow Size`指的是该对象本身占用内存的大小，`Retained Size`代表该对象被释放后，垃圾回收器能回收的内存总和。
下面以一个`栗子`说明，`Memory Monitor`里面观察App的内存使用曲线，突然发现，纳尼！！！怎么内存使用越来越大了，这就很有可能是发生内存泄漏了，然后点击手动进行GC，再点击观看JavaHeap，**点击Analyzer Task**，Android Monitor就可以为我们自动分析泄漏的Activity啦，分析出来如下图所示
![mark](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/161633995.png)
点击直接定位到该单例中的代码，发现代码中出现了
``` java
public static VideoTagHelper getInstance(Context context) {
        if (tagHelper == null) {
            tagHelper = new VideoTagHelper();
        }
        tagHelper.context = context;
        return tagHelper;
    }
```
我们可以看到持有该Activity的单例对象,导致了内存泄露。直接修改为context.getApplicationContext()即可。这样就和Activity没关系了

###### 3.使用MAT分析
使用`Memory Monitor`导出hprof文件， 这个文件在Android Studio中的Captrues这个目录中可以看到。
![@hprof文件目录| center](http://ogzf36bsb.bkt.clouddn.com/blog/20161122/171011086.png)







[1]: https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/
 

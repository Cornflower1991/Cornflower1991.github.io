---
title: Xposed模块
date: 2016-11-21 17:18:51
tags: [xposed,xposed模块,hook]
---


IXposedMod接口定义了Xposed模块。但用户不能直接实现这个接口(具体原因参见XposedBridge的main方法)，只能实现作者定义好的四个接口，每个接口用于特定的用途：
```java
//用于修改app的资源文件
public interface IXposedHookInitPackageResources extends IXposedMod{}

//用于hook应用的代码
public interface IXposedHookLoadPackage extends IXposedMod{}
//在Zygote初始化时执行
public interface IXposedHookZygoteInit extends IXposedMod {}

/**用于hook基于java的命令行工具(例如pm)。它需要app_process 55版本，还需要在XposedInstaller的data目录下创建一个特殊文件(conf/enable_for_tools)*/
public interface IXposedHookCmdInit extends IXposedMod{}
```
<!--more-->
![@类加载过程 |center](http://ogzf36bsb.bkt.clouddn.com/blog/20161121/172513110.png)
从上图可以发现Xposed Module的类加载器与要运行Apk的类加载器都继承自PathClassLoader，但它们之间相互独立，每个ClassLoader只用于加载某一特定的apk(dex)，所以在Xposed module中如果要加载apk中的类需要用原应用的ApkClassLoader进行加载。

### Hook过程
- **Find:** 通过特定的类加载器加载要hook的类，通过反射找到被hook的成员。
- **Hook:** 被hook成员调用前后执行特定的回调方法。

工具类XposedHelpers提供了一些工具方法来简化find过程；XposedBridge的hook*方法用于处理hook并执行回调。

#### find
| XposedHelpers静态方法| - |
| :--------:  |:----: |
| findClass | 使用classLoader加载class |
| findField*| 通过反射查找类的数据成员并设置可访问性(setAccessible(true)) |
| findMethod*| 通过反射查找类的成员函数并设置可访问性  |
| findConstructor*| 通过反射查找类的构造函数并设置可访问性 |
| setStatic*|通过反射设置类静态变量的值 |
| set*|  通过反射设置对象数据成员的值 |
| findAndHook*| 查找并hook  |

#### hook
XC_MethodHook中定义了回调方法：
> beforeHookedMethod(MethodHookParam param)：
> 被hook方法调用前执行，调用param.setResult可以跳过被hook的方法
> 
> afterHookedMethod(MethodHookParam param) ：
>  被hook方法调用后执行，调用param.setResult更改被hook方法的执行结果。

XC_MethodReplacement继承自XC_MethodHook，通过在beforeHookedMethod中调用param.setResult**实现了方法的替换**。
``` java

/**以beforeHookedMethod为例，如何修改方法返回值*/
@Override
protected final void beforeHookedMethod(MethodHookParam param) throws Throwable {
    try {
		 //对hook的方法修改后的结果
        Object result = replaceHookedMethod(param);
        //设置修改后的结果为原方法（被Hook的函数）的结果返回
        param.setResult(result);
    } catch (Throwable t) {
        param.setThrowable(t);
    }
}
```
#### 最后
当然还有一些Xposed模版的操作，拷贝jar包，在AndroidManifest.xml中声明相应的mata标签之类的就不一一叙述了。
[Xposed](http://repo.xposed.info/module/de.robv.android.xposed.installer)
---
layout: post
title: Activity的启动模式
date: 2017-04-10
tags: android    
---
*adb shell dumpsys activity 输入这个命令可以得到一个清晰的 Task 视图*

### Activity的LaunchMode
**standard从不复用**  
每次创建一个新的实例，而不论是否存在。谁启动了这个实例，它就运行在谁的栈中。比如:  
1. 我们使用ApplicationContext启动stardard模式的activity时会报错，这是因为非activity类型的context并没有任务栈，解决办法是，指定flag_activity_new_task标志位。  
2. 假如需要在 service 在后台中做一些耗时操作，当它完成时, 你需要从此 service 中跳转进入一个 Activity 中，(Service 是 Context 一种扩展, 它含有 startActivity方法)但是当你调用service.startActivity(intent)时会报错，原因同上。    

**singleTop栈顶复用**
如果一个启动模式为 SingleTop 的 activity 实例在目标栈顶，intent 启动该 activity 时系统将通过 onNewIntent 的方法将 intent 传递给已有的那个实例而不会新创建一个的实例,这个activity的onCreate,onStart不会被调用，因为它并没有发生改变。   

**singleTask栈内复用**   
taskAffinity指定了此activity所在的栈名，若不指定，默认为应用的包名。此属性和singleTask或者allowTaskReparenting配对使用，在其它情况下无意义。任务栈分前台任务栈和后台任务栈，后台任务栈中activity处于暂停状态，后台任务栈可以切换到前台。  

寻找一个与aclass的affinity相同的task(即将被启动的activiy，简称为aclass)  
* yes         
  如果aclass的实例已经存在,则将此实例带到栈顶，并调用onNewIntent；若不存在，则新建；  
* no   
  新建一个以aclass的affinity命名的task，并新建实例加入此task
 
taskAffinity与allowTaskReparenting的配对：应用AA启动了，应用BB的activity b，此时b存在于AA的任务栈中，但他们的affinity是不同的，毕竟包名不同。接下来，若应用BB启动了，BB会创建自己栈，此时系统发现与b的affinity最配的栈是BB，于是就把b从AA的任务栈移到BB

**singleInstance单例**  
此种模式的activity只能单独的位于一个任务栈中。  
A -> B（singleInstance） -> C   
由于 B 需要一个只能容纳它的 Task , 所以 C 会被加上一个 FLAG_ACTIVITY_NEW_TASK 标识。所以 C(default) 变成了 C(singleTask) 。  
task1:ac  ---  task2:b

### 设置启动模式
1. 在manifest中通过launchMode指定；
2. 通过intent.addFlags()指定；   
通过代码的设置的优先级，高于manifest；

### Activity的Flags
* flag_activity_new_task:与manifest中指定singleTask效果相同；
* flag_activity_single_top:同manifest中的singleTop
* flag_activity_clear_top:顾名思义，将此activity带到栈顶。若activity是standard模式，1.出现其上所有activity；2.由于standard绝不复用，所以也出栈此activity，再新建个实例加入栈中；
* flag_activity_exclude_from_recents:等于与manifest中的android:excludeFromRecents="true",不在回退历史中出现；

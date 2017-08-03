---
layout: post
title: Activity的生命周期
date: 2017-04-09
tags: android    
---

### 用户参与下的生命周期
onCreate：Activity正在被创建，可以做一些初始化工作,如setContentView  
onRestart：onStop -> onRestart -> onStart  
onStart：可见，没有出现在前台  
onResume：可见，显示到前台  
onPause：可见，退出前台（离开栈顶）  
onStop：不可见   
onDestory：Activity即将被销毁    

onCreate和onDestroy是配对的，标识着activity的创建和销毁，并只可能调用一次；   
onStart和onStop是配对的，标识activity是否可见；   
onResume和onPause是配对的，标识activity是否在前台；   

栈顶的activity需要先onPause后，新的activity才会被启动   
```  
a: onPuase  
b: onCreate  
b: onStart  
b: onResume  
a: onStop  
```

很多人说弹出Dialog时只会调用onPause()而不会调用onStop()，可是自己做了下实验，此时既不会调用onPause()也不会调用onStop()。我觉得原因是Dialog并不会使当前activity离开栈顶。所以需要activity的参与，当activity的theme如下时，才会发生上面的情况：    
android:theme="@android:style/Theme.Translucent"   
android:theme="@android:style/Theme.Dialog"   

### 异常情况下的生命周期
**异常情况有2种：**  
1. 资源相关的系统配置发生改变导致activity被杀死并重建，比如横竖屏会使用不同的图片；   
2. 内存不足导致低优先级的activity被杀死；   
Activity优先级：  
a.前台activity   
b.可见但非前台activity   
c.后台activity，比如已经onStop的
如果一个进程没有在4大组件中执行，将很快被系统杀死。一些后台工作可以放在service中从而保证一定的优先级   
**生命周期**   
异常销毁时，activity会被销毁，onPause,onStop,onDestroy均会被调用，但会在onStop之前调用onSaveInstanceState（和onPause没什么关系，可能之前，也可能之后）。重建时，会在onCreate与onRestoreInstanceState（onStart之后）中恢复状态。
保存数据时，使用了委托的思想，activity调用onSaveInstanceState时，会委托Window，window委托decorView，然后再一一通知子view去调用自己的onSaveInstanceState保存数据。
恢复数据时，onCreate需要判断bundle是否为空，onRestoreInstanceState不用   

###configChanges
系统配置发生改变后，禁止重建
```
<activity
   android:configChanges="orientation|screenSize" />
```
keyboardHidden键盘可访问性发生的改变   
locale 设备的本地位置发生了改变，一般指切换了系统语言    
screenSize 屏幕尺寸改变，如旋转屏幕，在minSdk，targetSdk有大于13的需要加

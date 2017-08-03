---
layout: post
title: IntentFilter的匹配规则
date: 2017-04-12
tags: android    
---

一个activity中，可以有多个intent-filter，任何一组匹配成功即可通过。IntentFilter由action,category,data组成。
### action的匹配规则
action是一个字符串，我们可以使用系统预定义的，也可以使用自己的。   
匹配规则是：  
1. intent中必须至少有一个action。若有多个action，可以随意挑一个；    
2. 选中的action，需要和过滤规则中的某个相同；    

### category的匹配规则   
category是一个字符串，我们可以使用系统预定义的，也可以使用自己的。   
匹配规则是：    
1. intent中必须至少有一个category（包括隐式的）；
2. intent中的category必须全部和过滤规则匹配；
3. 系统在调用startActivity/startActivtiyForResult时，会默认为intent加上android.intent.category.DEFAULT，也因为如此，所以如果我们希望自己的某个activity支持隐式启动时，必须在intent-filter中加入这条；   

### data的匹配规则
若intent-filter中定义了data,那么intent中也必须定义一个与之相匹配的data。filter中有多个data，我们只需匹配一个就行。data由mimeType和uri组成：  
**mimeType:** image/jpeg, video/*等，支持通配符   
**uri:**
```
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]    
```

scheme:若intent-filter中定义了data，但没有定义scheme，则系统会给予content 和 file作为默认值，所以intent中的uri部分的scheme也必须是content或者file。，如；
intent.setDataAndType(Uri.parse("file://abc"),"image/png");   
注意：要指定完整的data必须调用setDataAndType，setData,setType会互相把对方的值置空。    

```
Intent intent = new Intent("action");  
intent.addCategory("category");  
intent.setDataAndType(Uri.parse("file://abc"),"text/plain");
startActivity(intent);
```

### 额外判断
当我们通过隐式的方式启动一个activity时，先判断下是否有满足条件的activity存在。
```
PackageManager:
packageManager.queryIntentActivities(Intent i, int flags)
返回所有匹配的activity信息，没有返回空
packageManager.resolveActivity(Intent i, int flags)
返回最佳匹配的信息，没有返回空

flags我们要使用PackageManager.MATCH_DEFAULT_ONLY，这个flag的含义是，仅匹配在intent-filter中声明了android.intent.category.DEFAULT的activity。他的意义在于只要上述方法不返回null，就一定可以启动成功
```

### Main-Launcher
```
<intent-filter>
   <action android:name="android.intent.action.MAIN" />
   <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
android.intent.action.MAIN决定应用程序最先启动的Activity 。android.intent.category.LAUNCHER决定应用程序是否显示在程序列表里，这里可以不用加android.intent.category.DEFAULT。且他俩必须配合使用。
```

```
<activity android:name="a">
     <intent-filter>
          <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity android:name="b">
         <intent-filter>
            <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.LAUNCHER" />
         </intent-filter>
</activity>
如果是这样的话，就会出现两个图标在列表里。
```


---
layout: post
title: "Activity 的生命周期和启动模式"
subtitle: 'Activity 的生命周期全面解析'
author: "Song"
header-style: text
tags:
  - android
  - activity
  - lifecycle
---

## Activity 的生命周期全面解析

### 典型情况下的生命周期分析
**Activity 生命周期的切换过程**

![https://developer.android.google.cn/guide/components/images/activity_lifecycle.png](https://developer.android.google.cn/guide/components/images/activity_lifecycle.png)

- onCreate：表示 Activity 正在被创建，且是 Activity 的第一个生命周期方法。可做一些初始化操作，如调用 setContentView 加载页面布局资源、初始化 Activity 所需数据。
- onRestart：表示 Activity 正在重新启动，当 Activity 从不可见重新变为可见状态时，onRestart 就会被调用。如从 Home 键返回桌面或跳转到其他页面，再回到当前 Activity。
- onStart：表示 Activity 正在被启动，已经可见，但是未出现在前台，还无法和用户交互。可以理解为 Activity 已经显示出来，但还是看不到。
- onResume：表示 Activity 已经可见，且出现在前台并开始活动。和 onStart 相比，都已经可见，但是 onStart 时候，Activity 还在后台，onResume 时候 Activity 才显示到前台。
- onPause：表示 Activity 正在停止，但还是可见，只是不在前台，正常情况下，紧接着 onStop 被调用。特殊情况下，快速的回到当前 Activity，onResume 被调用，一般用户操作很难出现这种情况。onPause 中可以做保存数据，停止动画操作，但是不能太耗时，因为会影响到新的 Activity 显示，onPause 必须先执行完，新的 Activity onResume 才会执行。
- onStop：表示 Activity 即将停止，已经不可见，可以做稍微重量级的回收操作，同样不能太耗时。
- onDestory：表示 Activity 即将被销毁，我们可以做一些回收工作和最终的资源释放。

#### 启动->回到桌面->回来->回退键
```
==============跳转================
E/OneActivity: onCreate
E/OneActivity: onStart
E/OneActivity: onResume
==============Home键================
E/OneActivity: onPause
E/OneActivity: onStop
E/OneActivity: onSaveInstanceState
==============回来================
E/OneActivity: onRestart
E/OneActivity: onStart
E/OneActivity: onResume
==============回退键================
E/OneActivity: onPause
E/OneActivity: onStop
E/OneActivity: onDestroy
```

> onCreate 和 onDestory 是配对的（因为内存应用被杀死除外），表示 Activity 的创建和销毁，并且只可能有一次调用。从 Activity 是否可见来说，onStart 和 onStop 是配对，跟随用户操作，这两个方法会被调用多次。从 Activity 是否在前台来说，onResume 和  onPause 是配对的，跟随户操作，这两个方法会被调用多次。

#### 启动->跳转->回退键
```
==============启动================
E/OneActivity: onCreate
E/OneActivity: onStart
E/OneActivity: onResume
==============跳转================
E/OneActivity: onPause
E/TwoActivity: onCreate
E/TwoActivity: onStart
E/TwoActivity: onResume
E/OneActivity: onStop
E/OneActivity: onSaveInstanceState
==============回退键================
E/TwoActivity: onPause
E/OneActivity: onRestart
E/OneActivity: onStart
E/OneActivity: onResume
E/TwoActivity: onStop
E/TwoActivity: onDestroy
```
> 1. One 页面跳转到 Two 页面，先执行 One 的 onPause 后，再执行 Two 的 onCreate。分析源码可知，只有当 One 的 onPause 执行，才会创建新的 Activity，所以 onPause 周期不能做耗时操作。
> 2. 当 Two 页面执行到 onResume 周期，One 页面才会执行 onStop，因为 onStop 周期是完全不可见状态才会执行。只有当 Two 执行到 onResume 可见且在前台，完全遮挡 One 页面，才会导致 One 的 onStop 执行。
> 3. 当 Two 在回退栈栈顶，Two Pop 出回退栈，紧跟着执行 Two 的 onPause，才会执行 One 相关周期。且只有当 One 页面执行到 onResume，Two 才会执行 onStop 和 onDestroy，同样的因为 onStop 方法在页面不可见才会执行。

#### 跳转到 DialogActivity->回退键
```
==============跳转================
E/TwoActivity: onPause
E/DialogTestActivity: onCreate
E/DialogTestActivity: onStart
E/DialogTestActivity: onResume
==============回退键================
E/DialogTestActivity: onPause
E/TwoActivity: onResume
E/DialogTestActivity: onStop
E/DialogTestActivity: onDestroy
```
> 1. 当跳转到  DialogTestActivity 首先执行 Two 的 onPause 才会创建新的 Activity。
> 2. 因为 DialogTestActivity 非全屏，导致 Two 页面还是可见，但不在前台，故只执行 onPause，却不会执行 onStop。
> 3. 当 DialogTestActivity 回退键，紧接着执行其  onPause，且 Two 回到前台，执行其 onResume。这时 DialogTestActivity 已经完全不可见，再执行其 onStop 和 onDestroy。

#### 跳转到 DialogActivity->Home 键
```
==============跳转================
E/TwoActivity: onPause
E/DialogTestActivity: onCreate
E/DialogTestActivity: onStart
E/DialogTestActivity: onResume
==============Home键================
E/DialogTestActivity: onPause
E/TwoActivity: onStop
E/TwoActivity: onSaveInstanceState
E/DialogTestActivity: onStop
E/DialogTestActivity: onSaveInstanceState
```
> 1. DialogTestActivity 在前台执行 Home 键，紧接执行其 onPause 周期，这时 DialogTestActivity 和 TwoActivity 都不在前台，且 TwoActivity 先入栈，故先执行 TwoActivity 的 onStop 周期，再执行 DialogTestActivity 的 onStop 周期。

### 异常情况下的生命周期分析
**横竖屏切换生命周期变化**
```
E/TwoActivity: onPause
E/TwoActivity: onStop
E/TwoActivity: onSaveInstanceState
E/TwoActivity: onDestroy
E/TwoActivity: onCreate
E/TwoActivity: onStart
E/TwoActivity: onRestoreInstanceState
E/TwoActivity: onResume
```
先销毁旧的 Activity，在创建新的 Activity。我们发现 onSaveInstanceState 执行在 onStop 之后，onRestoreInstanceState 执行在 onResume 之前。
> 1. onRestoreInstanceState 只有在异常情况下才会执行，如横竖屏切换，因为内存不足杀死 Activity。
> 2. onRestoreInstanceState 回调保存的 Bundle 数据，且在 onResume 之前执行，在展示在前台前恢复保存的数据。而 onSaveInstanceState 在 onStop 后执行，是因为当前页面已经不在前台，是报错数据的好时机。

#### View 实现了 onSaveInstanceState 和 onRestoreInstanceState 方法
因为所有的 ViewGroup 继承自 View，相当于使用了委托思想，上层委托下层，所以所有 View 基本都实现了这两个方法。如 TextView、EditText、ListView 横竖屏切换内容还是被保存。
> 当系统内存不足时，系统会按照优先级杀死对应 Activity 的进程，并后续通过 onSaveInstanceState 和 onRestoreInstanceState 存储和恢复数据。如果一个进程没有四大组件，那么这个进程很容易被杀死，因此后台工作不应该脱离四大组件而独立运行。

#### `configChanges` 配置
针对情况在 `AndroidManifest.xml` 配置 `configChanges` 可使 Activity 不重新被创建。如 `android:configChanges="locale|screenSize|orientation|keyboardHidden"`

| 项目               | 含义                                                         |
| ------------------ | ------------------------------------------------------------ |
| mcc                | SIM卡唯一标识IMSI（国际移动用户识别码）中的国家代码，有三位数字组成，中国为460。此项 识别mcc代码发生改变。 |
| mnc                | SIM卡唯一标识IMSI（国际移动用户标识码）中的运营商代码，有两位数字组成，中国移动TD系 统为00，中国联通为01，中国电信为03。此标识mnc发生改变。 |
| locale             | 设备的本地位置发生改变，一般只切换了系统语言。               |
| touchscreen        | 触摸屏发生了改变，这个很费解，正常情况下无法发生，可以忽略它。 |
| keyboard           | 键盘类型发生了改变，比如用户使用了外插键盘。                 |
| keyboardHidden     | 键盘的可访问性发生了变化，比如用户调出了键盘。               |
| navigation         | 系统导航方式发生了改变，比如用户采用了轨迹球导航，这个有点费解，很难发生，可以忽略它。 |
| screenLayout       | 屏幕布局发生了改变，很可能是用户激活了另一个设备显示。       |
| fontScale          | 系统字体缩放比例发生了改变，比如用户使用了一个新字号。       |
| uiMode             | 用户界面模式发生了改变，比如是否开启了夜间模式。             |
| orientation        | 屏幕方向发生了改变，这个是最常用的，比如旋转了手机屏幕。     |
| screenSize         | 当屏幕的尺寸信息发生了改变，当旋转设备屏幕时，屏幕尺寸会发生变化，这个选项比较特殊，它 和编译选项有关，当编译选项中的minSdkVersion和targetSdkVersion低于13时，此选项不会导致 Activity重启，否则会导致Activity重启（API13新添加）。 |
| smallwstScreenSize | 设备的物理屏幕尺寸发生改变，这个项目和屏幕方向没关系，仅仅表示在实际的物理屏幕的尺寸 改变的时候发生没比如用户切换到了外部的显示设备，这个选项和screenSize一样，当编译选项中的 minSdkVersion和targetSdkVersion低于13时，此选项不会导致Activity重启，都则会导致Activity 重启（API13新添加）。 |
| layoutDirection    | 当布局发生变化，这个属性用的比较少，正常情况下无需修改布局的layoutDirection属性（API17新添加）。 |

在对应 Activity 配置 `android:configChanges="screenSize|orientation|keyboardHidden"`，且重载 `onConfigurationChanged` 方法，横竖屏切换变化

```java
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);
    Log.e(TAG, "onConfigurationChanged： " + newConfig.orientation);
}
```
```
E/OneActivity: onConfigurationChanged： 2
E/OneActivity: onConfigurationChanged： 1
```

## Activity 的启动模式

### standard
标准模式，每启动一个 Activity 都会创建一个新的实例，不管这个实例是否已经存在。一个任务栈可以有多个实例，每个实例也可以属于不同的任务栈。谁启动了这个 Activity，那么这个 Activity 就运行在启动它的 Activity 所在栈中。

### singleTop：
栈顶复用模式，如果新的 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建，同时 onNewIntent 方法会被回调，但是 onCreate、onStart 方法不会被回调。

### singleTask
栈内复用模式，这是一种单实例模式。只要 Activity 在栈中存在，那么多次启动此 Activity 都不会创建新的实例，和 singleTop 一样，系统也会回调其 onNewIntent 方法。

- 比如当前任务栈 S1情况为 ABC，若 Activity D 以 singleTask 模式启动，其所需要任务栈为 S2，由于 S2 和 D 实例均不存在，所以系统会先创建任务栈 S2，然后创建 D 的实例并将其入栈到 S2
- 另一种情况，若 D 所需任务栈为 S1，其他情况如上所示，那么由于 S1 已经存在，所以系统会直接创建 D 的实例并且将其入栈到 S1
- 若 D 所需任务栈为 S1，且当前任务栈为 ADBC，根据栈内复用原则，此时 D 不用重新创建，系统会把 D 切换到栈顶并调用其 onNewIntent 方法。最终 S1 中情况为 AD。

情况 1，未指定 taskAffinity（任务相关性，标识任务栈名称，默认为应用包名） 或者 allowTaskReparenting，在当前任务栈查询，如果存在对应activity，则回调onNewIntent，且销毁对应activity以上页面
情况2，指定 taskAffinity 或者 allowTaskReparenting，查找对应任务栈，如果查找失败，则创建对应任务栈，并且入栈

在 Activity 中配置示例
```xml
<activity
	android:name=".lifecycle.TwoActivity"
	android:taskAffinity="com.song.android.single"
	android:launchMode="singleTask" />
```

### singleInstance
单实例模式，是一种加强的 singleTask 模式，加强的点是具有此种模式的 Activity 只能单独的位于一个任务栈中。如 Activity A 启动后在一个单独的任务栈，由于栈内复用性，后续的请求都不会创建新的 Activity，除非这个任务栈被销毁。
假设当前有前台任务栈 S1，其中存在 AB 页面，启动模式均为 standard。另外有后台任务栈 S2，其中存在页面 CD，启动模式均为 singleTask，此时任务栈为 （CD）（AB）。此时启动页面 C（指定任务栈为 S2，这时页面为 （AB）（CC，回退键后变为 （AB），此时 S2 任务栈被销毁。

情况1，A-->B(singleInstance)-->C,那么A和C在一个任务栈，B在单独任务栈，回退键，C回退，A回退，B回退，说明A和C任务栈在前台，B任务栈在后台
情况2，子线程中或者 ApplicationContext 也可以启动 Activity，但是必须开启新的任务栈，也就是设置 NEW_TASK 的 Flag（因为 standard 模式必须为在启动他的 Activity 中入栈，但 ApplicationContext 没有任务栈实例）

### 常用标记位
#### FLAG_ACTIVITY_NEW_TASK
为 Activity 指定 singleTask 启动模式，其效果和 XML 中配置一样

#### FLAG_ACTIVITY_SINGLE_TOP
为 Activity 指定 singleTop 启动模式，其效果和 XML 中配置一样

#### FLAG_ACTIVITY_CLEAR_TOP
当它启动时候，所有位于它上面的 Activity 都要出栈，若配合 FLAG_ACTIVITY_SINGLE_TOP 使用，则只调用 onNewIntent。若 Activity 模式为 standard 则重新创建新的实例入栈。

#### FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
等同 `android:excludeFromRecents="true"` 属性，这个标记的 Activity 不会出现在历史的 Activity 列表中。

## IntentFilter 的匹配规则
Activity 启动分为显式调用和隐式调用，其中显式调用需要明确的指定被启动对象的组件信息，包括包名和类名，而隐式调用则不需要明确指定组件信息。原则上讲一个 Intent 不能既是显式调用也是隐式调用，若二者共存则以显示调用为准。

```xml
<activity
    android:name=".lifecycle.OneActivity"
    android:configChanges="screenSize|orientation|keyboardHidden"
    android:launchMode="standard" >
    <intent-filter>
        <action android:name="com.song.test" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:host="topic"
            android:scheme="com.app.demo" />
    </intent-filter>
</activity>
```
只有一个 Intent 同时匹配 action、category、data 类别，才算完全匹配，只有完全匹配才能成功启动目标 Activity。一个 Activity 可以有多个 intent-filter，一个 Intent 只要能匹配任何一组 intent-filter 即可成功启动对应的 Activity。

### action
- 其是一个字符串，且区分大小写，必须和字符串完全相同才算匹配成功，action 可以是自定义也可以是系统预定义的。
- 一个 Intent 若为显式调用，则必须有且只有一个 action。（只有设置 `setAction`，说明只能存在一个 action）
- 一个过滤规则可以存在多个 action，Intent 只需要匹配其中一个 action 即可。action 的匹配要求 Intent 中的 action 存在且必须和过滤规则中的其中一个 action 相同。

### category
- 其是一个字符串，且区分大小写，必须和字符串完全相同才算匹配成功，action 可以是自定义也可以是系统预定义的
- Intent 中如果出现 category，则不管有几个 category，每个 category 必须是过滤规则中已经定义了的。
- Intent 中可以没有 category，但是 startActivity 和 startActivityForResult 会默认为 Intent 加上 category `android.intent.category.DEFAULT`，但是同时必须在 intent-filter 指定 `intent-filter` 这个 category。（设置 category 是 addCategory，说明可以设置多个 category）

### data
- 如果过滤规则中定义了 data，那么 Intent 中必须也要定义可匹配 data。
- 如需要 Intent 定义 data，则 Intent 只能存在一个 data。（setData 方法表示 Intent 只能有一个或者没有 data）。
- intent-filter 中可以定义多个 data，Intent 只需要和匹配规则中一个相同即可。
- data 由 mimeType 和 URI 组成，其中 mimeType 表示媒体类型。URI 由 `<scheme>://<host>:<port>/[<path>]|[<pathPrefix>]|[<pathPattern>]` 组成，如 `content://com.song.test:80/etc`。
- 其中 scheme 表示 URI 模式，如 http、content、file 等。scheme 必须指定，否则 URI 无效。
- host 表示 URI 主机名，如 `www.baidu.com`。host 必须指定，否则 URI 无效。
- port 表示 URI 端口，如 80。仅当 URI 指定了 scheme 和 host，port 才有意义。
- path、pathPrefix、pathPattern 表示路径，其中 path 为完整路径，pathPattern 为包含通配符路径，pathPrefix 为路径前缀。
- scheme 的默认值为 file 和 content，若要指定完整的 data 需要调用 setDataAndType，否则调用 setData、setType 会相互覆盖设置的值。

### IntentFilter 其他特性
- IntentFilter 的匹配规则对 BroadcastReceiver 和 Service 也有效。但是 Service 推荐使用显式调用。
- 判断是否匹配正确 Activity 有 PackageManager.queryIntentActivities 返回所有匹配的 Activity，或者 PackageManager.resolveActivity 返回匹配的 Activity。其中 flag 设置为 `PackageManager.MATCH_DEFAULT_ONLY`，只匹配 IntentFilter 中设置了 `android.intent.category.DEFAULT` 的 Activity。

### 示例
```java
// 拨打电话：
Intent intent=new Intent(); 
intent.setAction(Intent.ACTION_CALL);  
//intent.setAction("android.intent.action.CALL");  //以下各项皆如此，都有两种写法。
intent.setData(Uri.parse("tel:1320010001"));
startActivity(intent);

// 调用拨号面板：
Intent intent=new Intent();
intent.setAction(Intent.ACTION_DIAL); 
intent.setData(Uri.parse("tel:1320010001"));
startActivity(intent); 

// 调用拨号面板：
Intent intent=new Intent();
intent.setAction(Intent.ACTION_VIEW); 
intent.setData(Uri.parse("tel:1320010001"));
startActivity(intent); 

// 利用Uri打开浏览器、打开地图等：
Uri uri = Uri.parse("http://www.google.com"); //浏览器 
Uri uri=Uri.parse("geo:39.899533,116.036476"); //打开地图定位 
Intent intent = new Intent(); 
intent.setAction(Intent.ACTION_VIEW);
intent.setData(uri);
startActivity(intent); 

// 播放视频：
Intent intent = new Intent(); 
Uri uri = Uri.parse("file:///sdcard/media.mp4"); 
intent.setAction(Intent.ACTION_VIEW);
intent.setDataAndType(uri, "video/*"); 
startActivity(intent);

//发送短信息 
Uri uri = Uri.parse("smsto:13200100001"); 
Intent  intent  = new Intent(); 
intent.setAction(Intent.  ACTION_SENDTO );
intent.setData(uri);
intent.putExtra("sms_body", "信息内容..."); 
startActivity( intent ); 
```
# Android BroadcastReceiver 总结

`BroadcastReceiver` (广播接收器), 是 Android 四大组件之一. 可以**监听/接收**应用自身/应用之间/安卓系统发出的广播消息，并做出响应.

## 应用场景

* 不同组件间的通信, 应用内或者不同应用之间
* 多线程通信
* 与 Android 系统在特定情况下的通信, 如电话呼入时、网络状态切换时

## 使用流程

1. 自定义广播接收器
2. 注册广播
3. 发送广播
4. 回调接收器的 `onReceive()` 方法

### 自定义广播接收者 BroadcastReceiver

继承 BroadcastReceivre 基类, 复写抽象方法 `onReceive()` 方法

> 默认情况下, 广播接收器运行在 UI 线程, 因此, **onReceive() 方法不能执行耗时操作**, 否则将导致 ANR

```
public class mBroadcastReceiver extends BroadcastReceiver {

    // 复写 onReceive() 方法
    // 接收到相应广播后，会自动回调该方法
    @Override
    public void onReceive(Context context, Intent intent) {
    // 写入接收广播后的操作, 许多情况下会涉及与其他组件之间的交互, 如发送通知, 启动服务等
    }

}
```

### 注册接收器

注册的方式分:

1. **静态注册**: 在 AndroidManifest.xml 里通过 <receive> 标签声明
2. **动态注册**: 在代码中创建并通过 `Context.registerReceiver()` 注册接收器

静态注册是一种常驻广播, 不受任何组件生命周期影响, 应用程序关闭后, 如果有广播来, 程序依旧会被调用. 而动态注册用于特定时刻监听广播, 受组件生命周期影响, 组件销毁前必须停止接收广播.

#### 静态注册

```
<receiver android:directBootAware=["true" | "false"]
          android:enabled=["true" | "false"]
          android:exported=["true" | "false"]
          android:icon="drawable resource"
          android:label="string resource"
          android:name="string"
          android:permission="string"
          android:process="string" >
    <!-- 指定接收器接收的广播类型 -->      
    <intent-filter>
        <!-- 这是一个监听网络变化的系统广播 -->
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
```

* android:directBootAware: 能否在解锁屏幕之前的 Direct Boot Mode 下运行. 7.0 加入, 默认 false.
* android:enabled: 能否能实例化, 默认 true.
* android:exported: 能否接收其他 App 发出的广播, 默认值是由接收器中有无 intent-filter 决定的, 如果有intent-filter, 默认值为 true, 否则为 false.
* android:name: 继承 BroadcastReceiver 子类的类名
* android:permission: 发送者发送广播需要的权限, 如果没有设置, 那么将直接使用 <application> 的权限
* android:process: 接收器运行的进程的名称, 默认为 app 的本身的进程, 与包名相同. 我们也可以自己指定, 如果名称是 : 开头, 那么一个私有进程将被创建给接收器, 只能本应用内部使用. 如果没加, 则是全局进程, 可以被不同应用共享.

#### 动态注册

```
@Override
protected void onStart() {
    super.onStart();

    mBroadcastReceiver = new BroadcastReceiver();

    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

    registerReceiver(mBroadcastReceiver, intentFilter);
}

// 注册广播后, 要在相应位置销毁接收
@Override
protected void onStop() {
    super.onStop();
    unregisterReceiver(mBroadcastReceiver);
}
```

**推荐在 onStart 注册, 在 onStop 注销是因为 onStop 在应用销毁前一定会被执行, 保证广播一定会被注销, 从而防止内存泄露**. 不在 onCreate / onDestory 注册注销是因为当系统因为内存不足(优先级更高的应用需要内存)要回收 Activity 占用的资源时, Activity在执行完 onStop 方法后就会被销毁, onDestory 不一定保证会被执行. 当 onStop 回调时, activiy 已经不在前台了, 也就不需要再接收广播来提醒用户某个事件.

总的来说, 遵循以下几点(对 Service 也是同理):
* 如果只需要在 Activity 可见时与服务交互，则应在 onStart / onStop 期间注册注销.
* 如果希望 Activity 在后台停止运行状态下仍可接收广播，则可在 onCreate / onDestroy 期间注册注销.
* 通常情况下，切勿在 onResume / onPause 期间注册注销定，因为每一次生命周期转换都会发生这些回调，应该使发生在这些转换期间的处理保持在最低水平.

### 广播的类型

发送的广播大体可以分为 2 类:

1. Normal broadcasts 普通广播也叫无序广播, 会异步的发送给所有的 Receiver, 接收到广播的顺序是不确定的, 有可能是同时.
2. Ordered broadcasts 有序广播, 广播会先发送给优先级高 (android:priority)的 Receiver, 而且这个 Receiver 有权修改广播内容或者是决定是继续发送到下一个 Receiver 或者是直接终止广播.

我们还可以进一步将广播细分为 5 类:

1. Normal Broadcast - 普通广播
2. System Broadcast - 系统广播
3. Ordered Broadcast - 有序广播
4. Sticky Broadcast - 粘性广播
5. Local Broadcast - App应用内广播

### Normal Broadcast

即开发者自己定义 intent 发送广播：

```
Intent intent = new Intent();

// 对应 BroadcastReceiver 中 intentFilter 的 action
intent.setAction(BROADCAST_ACTION);

// 发送
sendBroadcast(intent);
```

若注册了的广播接收器中 的 intentFilter 的 action 与上述匹配, 则会接收此广播并回调 `onReceive()`:


```
<receiver 
    // 广播接收器类是 MyBroadcastReceiver
    android:name=".MyBroadcastReceiver" >
    // 接收 BROADCAST_ACTION 类型的广播
    <intent-filter>
        <action android:name="BROADCAST_ACTION" />
    </intent-filter>
</receiver>
```

### System Broadcast

Android 中内置了多个系统广播, 只要涉及到手机的基本操作, 如开机, 网络状态变化等, Android系统广播action如下:


系统操作 | action
---|---
监听网络变化 | android.net.conn.CONNECTIVITY_CHANGE
关闭或打开飞行模式 | Intent.ACTION_AIRPLANE_MODE_CHANGED
充电时或电量发生变化| Intent.ACTION_BATTERY_CHANGED
电池电量低 | Intent.ACTION_BATTERY_LOW
电池电量充足, 即从电量低变化到饱满时会发出广播  | Intent.ACTION_BATTERY_OKAY
系统启动完成后(仅广播一次) | Intent.ACTION_BOOT_COMPLETED
按下照相时的拍照按键(硬件按键)时 | Intent.ACTION_CAMERA_BUTTON
屏幕锁屏 | Intent.ACTION_CLOSE_SYSTEM_DIALOGS
设备当前设置被改变时(界面语言、设备方向等) | Intent.ACTION_CONFIGURATION_CHANGED
插入耳机时 | Intent.ACTION_HEADSET_PLUG
未正确移除SD卡但已取出来时(正确移除方法:设置--SD卡和设备内存--卸载SD卡)  | Intent.ACTION_MEDIA_BAD_REMOVAL
插入外部储存装置（如SD卡）| Intent.ACTION_MEDIA_CHECKING
成功安装APK | Intent.ACTION_PACKAGE_ADDED
成功删除APK | Intent.ACTION_PACKAGE_REMOVED
重启设备 | Intent.ACTION_REBOOT
屏幕被关闭 | Intent.ACTION_SCREEN_OFF
屏幕被打开 | Intent.ACTION_SCREEN_ON
关闭系统时 | Intent.ACTION_SHUTDOWN
重启设备 | Intent.ACTION_REBOOT

> 当使用系统广播时，只需要在注册广播接收器时在 intent-fliter 中定义相关的 action 即可, 并不需要手动发送, 当系统有相关操作时会自动进行系统广播.

### Ordered Broadcast

广播接收器按照先后顺序接收广播, 这里的有序是针对广播接收者而言的. 

接受器接收广播的顺序规则:

1. 同时面向静态和动态注册的广播接受器
2. 按照 Priority 属性值从大到小排序
3. Priority 属性相同者, 动态注册的广播优先

用途:

先接收的广播接收者可以对广播进行截断, 即后接收的广播接收者不再接收到此广播.
先接收的广播接收者可以对广播进行修改, 那么后接收的广播接收者将接收到被修改后的广播.

具体使用上与普通广播非常类似，差别仅在于广播的发送方式：


```
sendOrderedBroadcast(intent);
```

### Local Broadcast

由于 Android 中的广播可以跨应用直接通信 (exported对于有 intent-filter 情况下默认值为 true), 这样就可能出现下列问题:

* 其他应用针对性发出与当前应用 intent-filter 相匹配的广播, 由此导致当前应用不断接收广播并处理
* 其他应用注册与当前应用一致的 intent-filter 用于接收广播, 获取广播具体信息, 可能会出现安全问题

Local Broadcast 就是广播的发送者和接收者都同属于一个应用. 更安全也更高效. 使用 LocalBroadcastManager 就可以直接实现.

LocalBroadcastManager 方式发送的应用内广播, **只能通过 LocalBroadcastManager 动态注册, 不能静态注册**

```
BroadcastReceiver broadcastReceiver = new BroadcastReceiver();
IntentFilter intentFilter = new IntentFilter(); 

// 实例化 LocalBroadcastManager
LocalBroadcastManager localBroadcastManager = LocalBroadcastManager.getInstance(this);

// 设置接收广播的类型 
intentFilter.addAction(BROADCAST_ACTION);

// 调用 LocalBroadcastManager registerReceiver() 动态注册 
localBroadcastManager.registerReceiver(broadcastReceiver, intentFilter);

// 发送应用内广播
Intent intent = new Intent();
intent.setAction(BROADCAST_ACTION);
localBroadcastManager.sendBroadcast(intent);

// 注销内广播接收器
localBroadcastManager.unregisterReceiver(broadcastReceiver);
```

除此之外, 总共有三种方法控制谁可以接收你的广播:

1. 发送时指定 permission.
2. 在 Android 4.0 及以上, 指通过`intent.setPackage(packageName)`指定包名来发送广播.
3. 使用 LocalBroadcastManager.

对应的, 也有三种方法控制接收哪些广播

1. 注册广播时将 exported 属性设置为 false, 使得非本应用内部发出的此广播不被接收.
2. 注册广播时, 增设相应 permission, 用于权限验证.
3. 使用 LocalBroadcastManager.

### Sticky Broadcast
 
Sticky Broadcast 粘性广播在发送后就一直存在于系统的消息容器里面, 等待对应的处理器去处理, 如果暂时没有处理器处理这个消息则一直在消息容器里面处于等待状态, 粘性广播的Receiver如果被销毁, 那么下次重建时会自动接收到消息数据
但该方式目前已经 deprecated.

### 参考

* [Android - When to Register/Unregister Broadcast Receivers created in an activity?
](https://stackoverflow.com/questions/7887169/android-when-to-register-unregister-broadcast-receivers-created-in-an-activity)
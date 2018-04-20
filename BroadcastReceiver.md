# BroadcastReceiver

`BroadcastReceiver` (广播接收器), 是 Android 四大组件之一. 可以**监听/接收** 应用自身/应用之间/安卓系统发出的广播消息，并做出响应.

### 应用场景

* 不同组件间的通信, 可以应用内, 也可以不同应用之间
* 多线程通信
* 与 Android 系统在特定情况下的通信, 如电话呼入时、网络可用时

### 使用流程

1. 自定义广播接收器
2. 注册广播
3. 发送广播
4. 回调接收器的 `onReceive()` 方法

#### 自定义广播接收者 BroadcastReceiver

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

#### 注册接收器

注册的方式分:

1. **静态注册**: 在 AndroidManifest.xml 里通过 <receive> 标签声明
2. **动态注册**: 在代码中创建并通过 `Context.registerReceiver()` 注册接收器

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

```
@Override
protected void onResume(){
    super.onResume();

    mBroadcastReceiver = new BroadcastReceiver();

    IntentFilter intentFilter = new IntentFilter();
    intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE);

    registerReceiver(mBroadcastReceiver, intentFilter);
 }


// 注册广播后, 要在相应位置销毁广播
@Override
protected void onPause() {
    super.onPause();
    unregisterReceiver(mBroadcastReceiver);
}
```

推荐在 onResume 注册, 在 onPause 注销是因为 onPause 在应用销毁前一定会被执行, 从而保证广播一定会被注销, 从而防止内存泄露. 不在 onCreate / onDestory 或 onStart / onStop 注册注销是因为当系统因为内存不足(优先级更高的应用需要内存)要回收 Activity 占用的资源时, Activity在执行完 onPause 方法后就会被销毁, onStop, onDestory 也就不会执行.
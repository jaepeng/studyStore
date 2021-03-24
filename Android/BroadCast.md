# 广播(BoradCast)

## 标准广播(无序广播)

+ 完全异步执行的广播类型,广播发出之后,所有的广播接收器几乎都会在同一时间接收到这条广播消息,没有先后顺序可言

+ 无法被截断

## 有序广播

+ 同步执行的广播,同一时刻只能有一个广播接收器收到该条广播.当这个广播接收器中的逻辑执行完毕后,广播才会继续传递.
+ 有先后顺序,高优先级的广播接收器就可以先收到广播消息,并且前面的广播还可以截断正在传递的广播

## 例子:

### 动态注册:

在代码中进行注册,这是现在主流推行的方式.

+ 可以自由地控制和注销,灵活性方面有很大优势.
+ 必须在程序启动之后才能接收到广播

```java
//检测网络变换
//动态注册广播的方式
package com.example.broadcasttest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Bundle;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    private IntentFilter mIntentFilter;
    private NetWorkChangeReceiver mNetWorkChangeReceiver;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mIntentFilter=new IntentFilter();
        Toast.makeText(this, "onCreate", Toast.LENGTH_SHORT).show();
      //需要监听什么广播,就在这里添加什么广播
        mIntentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");//网络变化的Action
        mNetWorkChangeReceiver=new NetWorkChangeReceiver();
        registerReceiver(mNetWorkChangeReceiver,mIntentFilter);
    }

    class NetWorkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            ConnectivityManager connectivityManager= (ConnectivityManager) getSystemService(Context.CONNECTIVITY_SERVICE);
          //需要配置网络权限
            NetworkInfo networkInfo=connectivityManager.getActiveNetworkInfo();
            if (networkInfo!=null&& networkInfo.isAvailable()){
                Toast.makeText(context, "网络可用", Toast.LENGTH_SHORT).show();
            }else{
                Toast.makeText(context, "is error", Toast.LENGTH_SHORT).show();
            }

        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
      //记得在onDestory中销毁
        unregisterReceiver(mNetWorkChangeReceiver);
    }
}
```

### 静态注册:

+ 可以让程序未启动的情况下就能接收到广播

+ 去新建一个BoradCast Receiver

  + ![image-20201204134241359](https://gitee.com/pengjae/pic/raw/master/img/20201204134248.png)

  + Exported:代表是否允许这个广播接收器接受本程序意外的广播

  + Enabled:是否启用这个广播接收器

  + ```java
    1. 先添加权限
      <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    2. 添加意图过滤器
    <receiver
        android:name=".BootCompleteReceiver"
        android:enabled="true"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
        </intent-filter>
    </receiver>
    ```

+ 注意：

  + 在Android8.0之后，已经不建议使用静态广播注册了。如果非要用的话

  + ```java
    button.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent intent=new
                    Intent("com.example.broadcasttest.MyBroadCast");//自定义的过滤器内容。
            //8.0之后，不建议使用静态注册方式发送广播，如果真的需要的话，则添加以下信息
          	//上面两种都可以给将广播传递给其他app
            //该方式适用：给其他应用的广播接收者发送消息（指定应用的包名、指定类的全类名）
            //向其他应用发送广播
              intent.setComponent(new ComponentName("com.example.broadcasttest", "com.example.broadcasttest.MyBroadcastReceiver.java"));//目标包名(1)
              intent.setClassName("包名", "包名.MyBroadcastReceiver");//(2)
            //向自己发送广播
            intent.setClassName(MainActivity.this,"com.example.broadcasttest.MyBroadcastReceiver");//(3)
            sendBroadcast(intent);
        }
    });
    ```

  + 衍生：一个intent一次只能干一件事情。不能重复设置，上面三种包名都只能设置一次。



### 发送有序广播

sendOrederBroadcast（intent，null）；

根据优先级来进行接受。

## 广播的优先级

[Android 广播优先级研究_weixin_34326558的博客-CSDN博客_android 广播优先级](https://blog.csdn.net/weixin_34326558/article/details/92066148)

### 动态广播优先级

+ 无序广播无序，无优先级
+ 有序广播串行，有优先级

### 静态广播优先级

+ 无序广播无优先级
+ 有序广播串行执行，同等优先级按照app安装的先后执行，因为静态广播注册到系统中，由PMS管理

### 静态广播和动态广播的优先级

+ 无序广播：动态广播优先发送
+ 有序广播：动态广播和静态广播按照优先级合并之后发送，优先级相同，则动态广播先接受。




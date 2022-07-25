# chapters5

## 5.1 广播机制 

​	每个应用程序都可以对自己感兴趣的广播进行注册，这样该程序就只会接收到自己感兴趣的内容，这些广播可能是来自于系统的，也可能是来自于其他用于程序的。

广播接受器（Broadcast Receiver）

### 	标准广播 

​	完全异步执行的广播，广播发出后，所有的接收器都会在同一时刻接收到这条广播信息，它们之间没有任何先后顺序可言。这种广播的效率比较高，但同时意味着它是无法被截断的

### 	有序广播 

​	同步执行的广播，广播发出后，同一时刻只会有一个广播接收器能够收到这条广播，当这个广播接收器的逻辑执行完毕后，广播才会继续传递。这个广播接收器有先后顺序，优先级高的广播接收器可以先收到广播信息，并且前面的广播接收器还可以截断正在传递的广播，这样后面的广播接收器就无法收到

## 5.2 接受系统广播

### 5.2.1 动态注册监听网络变化

​	动态注册：代码中注册 

​	缺点：程序启动后才能注册

```
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

    private IntentFilter intentFilter;

    private NetworkChangeReceiver networkChangeReceiver;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //过滤器
        intentFilter =new IntentFilter();
        //当网络状态发生变化时，系统发出的正是一条android.net.conn.CONNECTIVITY_CHANGE 广播
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        networkChangeReceiver =new NetworkChangeReceiver();
        //调用registerReceiver 进行注册 从而实现动态监听网络变化的功能
        registerReceiver(networkChangeReceiver,intentFilter);
    }
    //动态监听的广播接收器一定要取消注册
    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(networkChangeReceiver);
    }

    class NetworkChangeReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            //调用getSystemService（）得到ConnectivityManager的实例，系统服务类，用于管理网络链接
            ConnectivityManager connectivityManager=(ConnectivityManager)
            getSystemService(Context.CONNECTIVITY_SERVICE);
            //获取NetworkInfo 实例
            NetworkInfo networkInfo=connectivityManager.getActiveNetworkInfo();
            if(networkInfo!=null&&networkInfo.isAvailable()){
                Toast.makeText(context, "network is available", Toast.LENGTH_SHORT).show();
            }
            else {
                Toast.makeText(context, "network is unavailable", Toast.LENGTH_SHORT).show();
            }

        }
    }
}
```

```
需要施加权限
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

### 5.2.2 静态注册

静态注册：AndroidManifest中注册  程序未启动就可收到广播 

静态注册非系统广播，不能收到广播

​	 1 新建一个广播接收器

```
package com.example.broadcasttest;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class BootCompleteReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_SHORT).show();
    }
}
```

​	2 AndroidManifest注册 在<receiver>标签内注册

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.broadcasttest">

    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BroadcastTest"
        tools:targetApi="31">
        <receiver
            android:name=".BootCompleteReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

## 5.3 发送自定义广播

### 5.3.1 发送标准广播

​	使用静态注册时，需要在发送广播前，需要使用 setComponent() 方法指定将要作用的应用包名和类名

```
package com.example.broadcasttest;

import androidx.appcompat.app.AppCompatActivity;

import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.IntentFilter;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {




    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button=(Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new
                        Intent("com.example.broadcasttest.MY_BROADCAST");
              //在发送广播前，需要使用 setComponent() 方法指定将要作用的应用包名和类名，
               intent.setComponent(new ComponentName("com.example.broadcasttest",
                        "com.example.broadcasttest.MyBroadcastReceiver"));
                sendBroadcast(intent);
            }
        });
    }
```

### 5.3.2 发送有序广播

​	广播是一种可以跨进程的通信方式

```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button=(Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new
                        Intent("com.example.broadcasttest.MY_BROADCAST");
              //在发送广播前，需要使用 setComponent() 方法指定将要作用的应用包名和类名，
               intent.setComponent(new ComponentName("com.example.broadcasttest2",
                        "com.example.broadcasttest2.AnotherBroadcastReceiver"));
               //sendOrderedBroadcast()发送有序广播
                sendOrderedBroadcast(intent,null);
            }
        });
    }

```

```
设置优先级
        <receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            //设置优先级
            <intent-filter android:priority="100">
            <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
            </intent-filter>
        </receiver>
```

```
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "received in MyBroadcastReceiver",
                Toast.LENGTH_SHORT).show();
        //截断这个广播
        abortBroadcast();
    }
}

```

## 5.4 本地广播

 	本地广播只能在应用程序内部进行传递，广播接收器也只能接受来自本应用程序发出的广播

​	localBroadcastManager

```
public class MainActivity extends AppCompatActivity {


    private IntentFilter intentFilter;
    private LocalReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ////调用LocalBroadcastManager.getInstance 获取实例
        localBroadcastManager =LocalBroadcastManager.getInstance(this);//获取实例
        Button button=(Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               //调用
               Intent intent=new
                        Intent("com.example.broadcasttest.LOCAL_BROADCAST");
                        //localBroadcastManager.sendBroadcast(intent) 发送实例
                localBroadcastManager.sendBroadcast(intent);
            }
        });
        intentFilter=new IntentFilter();
        intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST");
        localReceiver =new LocalReceiver();
        localBroadcastManager.registerReceiver(localReceiver,intentFilter);//注册本地监听器
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        localBroadcastManager.unregisterReceiver(localReceiver);
    }
    class LocalReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            Toast.makeText(context, "received local broadcast",
                    Toast.LENGTH_SHORT).show();
        }
    }
}
```

## 5.5 强制下线功能

​	1 强制关闭代码以及新的父类

```
public class ActivityCollector {
    public static List<Activity>activities=new ArrayList<>();

    public static void addActivity(Activity activity) {
        activities.add(activity);
    }
    public static void removeActivity(Activity activity){
        activities.remove(activity);
    }
    public static void finishAll(){
        for(Activity activity:activities){
            if(!activity.isFinishing()){
                activity.finish();
            }
        }
        activities.clear();
    }
}

```

```
public class BaseActivity extends AppCompatActivity {
    private ForceOfflineReceiver receiver;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityCollector.addActivity(this);

    }
```

​	2 创建登录界面的活动，LoginActivity  ，布局为activity_login.xml

```
/*
	一个登录布局，最外层是一个纵向的LinearLayout,里面有3行直接子元素。第一行是个横向LineaLayout，用于输入账号；第二次是个横向的LineaLayout，用于输入密码；第三个是登录按钮
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:orientation="horizontal">
        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:textSize="18sp"
            android:text="Account"/>
        <EditText
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:id="@+id/account"
            android:layout_weight="1"
            android:layout_gravity="center_vertical"/>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:orientation="horizontal">
        <TextView
            android:layout_width="90dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center_vertical"
            android:textSize="18sp"
            android:text="Password:"/>
        <EditText
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:id="@+id/password"
            android:layout_weight="1"
            android:layout_gravity="center_vertical"
            android:inputType="textPassword"/>
    </LinearLayout>
        <Button
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:id="@+id/login"
            android:text="Login"/>

</LinearLayout>
```

​	3 更改LoginActivity 代码

```
public class LoginActivity extends BaseActivity {
    private EditText accountEdit;
    private EditText passwordEdit;
    private Button login;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        accountEdit=(EditText) findViewById(R.id.account);
        passwordEdit=(EditText) findViewById(R.id.password);
        login=(Button) findViewById(R.id.login);
        login.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String account= accountEdit.getText().toString();
                String password= passwordEdit.getText().toString();
                //如果账号是admin 密码是123456 登录成功
                if(account.equals("admin")&&password.equals("123456")){
                    Intent intent =new Intent(LoginActivity.this,MainActivity.class);
                    startActivity(intent);
                    finish();
                }else {
                    Toast.makeText(LoginActivity.this,
                            "account or password is invaild", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }
}
```

​	4 在activity_main 中加入 按钮 触发强制下线功能

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/force_offline"
        android:text="Send force offline broadcast"/>

</LinearLayout>
```

​	5 修改MainActivity代码

```
/*
	在按钮的点击事件里发送了一条广播，广播的值为com.example.broadcastbestpractice.FORCE_OFFLINE，通知用户强制下线。
*/
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button forceoffline=(Button) findViewById(R.id.force_offline);
        forceoffline.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent=new Intent("com.example.broadcastbestpractice.FORCE_OFFLINE");
                sendBroadcast(intent);
            }
        });
    }
}
```

​	6 在BaseActivity中动态注册一个广播接收器，作为父类，其余活动均继承它，也继承了广播接收器

```
public class BaseActivity extends AppCompatActivity {
    private ForceOfflineReceiver receiver;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ActivityCollector.addActivity(this);

    }
//注册动态广播器
    @Override
    protected void onResume() {
        super.onResume();
        IntentFilter intentFilter=new IntentFilter();
        intentFilter.addAction("com.example.broadcastbestpractice.FORCE_OFFLINE");
        receiver=new ForceOfflineReceiver();
        registerReceiver(receiver,intentFilter);
    }
    //取消注册动态广播器

    @Override
    protected void onPause() {
        super.onPause();
        if(receiver!=null){
            unregisterReceiver(receiver);
            receiver=null;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        ActivityCollector.removeActivity(this);
    }
    
    class ForceOfflineReceiver extends BroadcastReceiver{
        @Override
        public void onReceive(Context context, Intent intent) {
            //AlertDialog.Builder 创建一个对话框
            AlertDialog.Builder builder =new AlertDialog.Builder(context);
            builder.setTitle("Warning");
            builder.setMessage("You are forced to be offline.Please try to login again. ");
            // 对话框不可取消
            builder.setCancelable(false);
            //为对话框注册确定按钮
            builder.setPositiveButton("ok", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    ActivityCollector.finishAll();
                    Intent i=new Intent(context,LoginActivity.class);
                    context.startActivity(i);
                }
            });
            builder.show();
        }
    }
}

```

​	7	修改主活动为LoginActivity

```
        <activity
            android:name=".LoginActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

## 5.6 GIT

### 	 创建代码仓库

```
git init
```

#### 	提交代码

```
git add  (可以提交代码 或者整个文件夹)
git add . 一次性添加所有文件
git commit -m "描述提交信息"
```


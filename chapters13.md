#  chapters13

## 13.1 全局获取Context方法 

```
Android 提供了一个Application 类，每当程序启动的时候，系统会自动将这个类进行初始化。因此我们可以定制一个自己的Application类，以便管理程序内一些全局的状态信息，比如全局Context
```

```
public class MyApplication extends Application {
    private static Context context;

    @Override
    public void onCreate() {
        super.onCreate();
        //得到一个应用程序级别的Context
        context = getApplicationContext();
    }
    
    public static Context getContext(){
        return context;
    }
}
```

```
接下来需要告诉系统，程序启动的时候初始化MyApplication 类，需要在AndroidManifest的Application标签中进行指定
```

```
android:name="com.example.toolbest.MyApplication"
```

```
随后只需调用该类的方法，就可在项目的任何地方使用Context
MyApplication.getContext()
```

## 13.2 使用Intent 传递对象 

### 13.2.1 Serializable 传递方式

```
 	序列化，表示将一个对象转换成可存储或可传输的状态，序列化后的对象可以在网络上进行传输，也可以存储到本地上。只需要一个类区实现Serializable 这个接口就可以 
```

```
public class Person implements Serializable {
    
    private String name;
    
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

```
	这里让Person 类实现了Serializable 接口，这样所有的Person对象都是可序列化的了 
```

```
   活动中的写法也很简单 
   Person person =new Person();
   person.setName ("Tom");
   person.setAge(20);
   Intent intent=new Intent(FirstActivity.this,SecondActivity.class);
   intent.putExtra("person_data",person);
   startActivity(intent);
```

```
	随后获取这个传输对象 主要调用getSerializableExtra 方法获取参数传递的序列化对象 
```

```
	Person person=(Person) getIntent().getSerializableExtra("person_data");
```

### 13.2.2 Parcelable 

```
	其实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这样也就实现传递对象的功能了。
```

```
public class Person implements Parcelable {

    private String name;

    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        //写出name与age
        dest.writeString(name);
        dest.writeInt(age);
    }
    
    public static final Parcelable.Creator<Person>CREATOR=new Creator<Person>() {
        @Override
        public Person createFromParcel(Parcel source) {
            Person person=new Person();
            //读取name与age
            person.name=source.readString();
            person.age=source.readInt();
            return person;
        }
        
        

        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };
}
```

```
 	必须重写describeContents 和 writeToParcel 两个方法 
 	在Person类中提供一个CREATOR 常量，创建一个Parcelable.Creator<Person> 接口的一个实现，将泛型指定为Person。随后重写createFromParcel和Person[] newArray 。
 	createFromParcel方法需要读取刚刚写入的name age字段，并创建一个Person对象进行返回 。
```

```
	第一个活动写法与Serializable 类似
	第二个活动写法改为：
		Person person=(Person) getIntent().getParcelableExtra("person_data");
```

#### 对比

```
Serializable 比Parcelable 效率地，通常建议采用Parcelable实现Intent传递对象的功能
```

## 13.3 定制自己的日志工具

```

public class LogUtil {
    public static final int VERBOSE=1;
    
    public static final int DEBUG=2;
    
    public static final int INFO=3;

    public static final int WARN=4;

    public static final int ERROR=5;

    public static final int NOTHING=6;

    public static final int lever=VERBOSE;
    
    public static void v(String tag, String msg){
        if(lever<=VERBOSE) {
            Log.v(tag,msg);
        }
    }

    public static void d(String tag, String msg){
        if(lever<=DEBUG) {
            Log.d(tag,msg);
        }
    }

    public static void i(String tag, String msg){
        if(lever<=INFO) {
            Log.i(tag,msg);
        }
    }

    public static void w(String tag, String msg){
        if(lever<=WARN) {
            Log.w(tag,msg);
        }
    }

    public static void e(String tag, String msg){
        if(lever<=ERROR) {
            Log.e(tag,msg);
        }
    }
}

```

```
随后直接使用即可
 	LogUtil.d("TAG","debug log")
```

## 13.4 调试Android 程序 

## 13.5 创建定时任务 

### 13.5.1 Alarm机制 

```
	借助AlarmManager 类实现 
	AlarmManager manager=(AlarmManager) getSystemService(Context.ALARM_SERVICE);
```

```
	随后调用AlarmManager 的set()方法设置一个定时任务，比如想要设定一个任务在10s后执行，可写出
	long triggerAtTime= SystemClock.elapsedRealtime()+10*1000;
	manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,triggerAtTime,pendingIntent);
	set 方法三个参数
		第一个参数 整形参数 ，用于指定AlarmManager的工作类型  有四个值可选
	ELAPSED_REALTIME 表示让定时任务的触发时间从系统开机开始算起，不会唤醒CPU
	ELAPSED_REALTIME_WAKEUP 定时任务的出发时间从系统开机开始算起，但会唤醒CPU
	RTC 表示定时任务的触发时间从1970.1.1.0点开始计算，但不会唤醒CPU
	RTC_WAKEUP 表示定时任务的触发时间从1970.1.1.0点开始计算，但会唤醒CPU
	使用SystemClock.elapsedRealtime()方法可以获取系统开机至今所经历时间的毫秒数
	使用System.currentTimeMills()可以获得从1970.1.1.0点至今所经历的毫秒数 
	第二个参数 定时任务触发的时间，以毫秒为单位 
	第三个参数是一个 PendingIntent 一般会调用getService 或者getBroadcast()获取一个能够执行服务或广播的PendingIntent 。这样定时任务被触发后，服务的onStartCommand()或广播接收器的onReceive（）方法就可以执行
		
```

#### 长时间在后台定时运行的服务

```
import android.app.AlarmManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.os.SystemClock;

import androidx.annotation.Nullable;

import java.security.Provider;

public class LongRunningService extends Service {
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                //执行具体的逻辑
            }
        }).start();
        AlarmManager manager=(AlarmManager) getSystemService(ALARM_SERVICE);
        int anHour= 60*60*1000;
        long triggerAtTime= SystemClock.elapsedRealtime()+anHour;
        Intent i=new Intent(this,LongRunningService.class);
        PendingIntent pi=PendingIntent.getService(this,0,i,0);
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,triggerAtTime,pi);
        return super.onStartCommand(intent, flags, startId);
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
    
}

```

```
	启动定时服务时调用如下：
	Intent intent =new Intent(context,LongRunningService.class);
	context.startService(intent);
```

### 13.5.2 Doze模式

```
	用户的设备是Android 6.0以上系统，该设备未插接电源，处于静止状态，且屏幕关闭了一段时间后，就会进入到Doze模式，在Doze模式下，系统会对CPU 网络
	Alarm 等活动进行限制，延长电池的使用寿命 
```

```
	系统不会长时间处于Doze模式，会间歇性的退出Doze模式一小段时间，这段时间，应用完成同步操作。
```

## 13.6 多窗口模式编程

### 13.6.1 进入多窗口模式 

### 13.6.2 多窗口模式下的生命周期 

```
 	多窗口模式并不会改变获得原有的生命周期，只是会将用户最进交互过的那个活动设置为运行状态，而将多窗口模式下另外一个可见的活动设置为暂停状态，如果此时用户又去和暂停的活动进行交互，那么该活动就成为运行状态，之前处于运行状态下的活动变成暂停。
```

```
	针对进入多窗口模式时活动会被重新创建，可在AndroidManifest的活动标签中 进行修改
	
	android:configChanges="orientation|keyboardHidden|screenSize|
	screenLayout"
```

### 13.6.3 禁用多窗口模式 

```
	可在AndroidManifest的活动或应用标签中 进行修改
	android:resizeableActivity="false" 
	
	设置主活动仅支持竖屏
	android:screenOrientation="portrait"
```

## 13.7 Lambad表达式

```
	本质上一种匿名方法，没有方法名，也没有修饰符和返回值类型 
```

### 更改配置

```
 defaultConfig {
...
        jackOptions.enabled=true

    }
        compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
```

### 编写代码

```
new Thread(()->{
	//处理逻辑
}).start();
```

```
	单反只有一个待实现方法的接口，都可以使用Lambda表达式的写法，
```

```
	通常实现匿名接口如下：
	Runnable runnable =new Runnable(){
		@Override 
		public void run(){
			//添加具体实现
		}
	}；
```

```
	可改写为
	Runnable runnable=()->{
		//添加具体逻辑
	};
```

#### 自定义接口并进行实现

```

public interface MyListener {
    String doSomething (String a,int b);
}
```

```
        MyListener listener=(String a,int b)->{
            String result=a+b;
            return result;
        };
```

```
	也可简化写为：
	        MyListener listener=( a, b)->{
            String result=a+b;
            return result;
        };
```

#### 点击事件更改

```
      fab.setOnClickListener((v)->{
          //处理逻辑  
        });
```

```
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                
            }
        
```


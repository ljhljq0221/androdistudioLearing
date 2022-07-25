# chapters2

## 2.1活动

​	活动是一种包含用户界面的组件，主要用于与用户交互

### 2.1.1创建和加载布局

```
android:id="@+id/button_1" 
	android:id用于给当前元素定义一个唯一的标识符
	XML文件中引用一个id @id/id_name
	xml文件中定义一个id @id/id_name
android:layout_width="matdh_parent"
	android:layout_width 用于指定当前元素的宽度
	match_parent 表示当前元素宽度与父类元素宽度一致
android:layout_height="wrap_content"
	android:layout_height 指定当前元素高度
	wrap_content 表示当前元素高度只有刚好包含里面内容就行
android:text="Button 1" 
	指定元素显示的文字内容
setContentView（）给当前的活动加载一个布局 
	布局引用格式 R.layout.layout_name
```

### 2.1.2 在AndroidManifest注册活动

​	所有活动需要在AndroidManifest中注册才能生效。

​	活动的声明注册要放在<application>标签内，这里是通过<activity>标签对活动进行注册的。

```
android:name=".activity_name" 表示活动名称
android:label="app_name" 指定活动中标题栏的内容，标题栏显示在活动最顶部
```

​	注册了活动后，需要为程序配置主活动，否则程序运行，不知道哪一个是主活动，主活动配置利用<intent-filter>标签

```
<activity
       android:name=".FirstActivity"
        android:label="This is FirstActivity"
        android:exported="true" >
        <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
 </activity>
 android:exported 
 	是否支持其他应用调用当前组件
 	在Activity中该属性用来标示：当前Activity是否可以被另一个Application的组件启动：true允许被启动；false不允许被启动。
 	默认值：如果包含有intent-filter 默认值为true; 没有intent-filter默认值为false。
 	
```

### 2.1.3 在活动中使用Toast

​	Toast是Android系统提供的一种提醒方式，在程序中使用它可将一些短小的信息提供给用户，这些信息过一段时间就会消失。

```
        Button button1=(Button) findViewById(R.id.button_1);
        button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(FirstActivity.this,"You clicked Button 1",
                        Toast.LENGTH_SHORT).show();
            }
        });
   	findViewById() 获取在布局文件中定义的元素 这里利用R.id.button_1 来得到按钮的实例
   	findViewById()返回的是一个View对象，向下转型转成Button对象
   	得到按钮的实例后，调用setOnClickListener()为按钮注册一个监听器，点击按钮就可以执行监听器的onClick()方法，弹出Toast的功能当然是要在onClick()方法编写了。
   	Toast弹出是通过makeText()创建出一个Toast对象，然后用show()将其显示出来就可以
   	makeText需要传入3个参数
   		第一个是 context 上下文
   		Android应用模型是基于组件的应用设计模式，组件的运行要有一个完整的Android工程环境。在这个环境下，Activity、Service等系统组件才能正常工作，而这些组件不能采用普通的java对象创建方式，new一下是不能创建实例的，而是要有它们各自的上下文环境，也就是Context.Context是维持android各组件能够正常工作的一个核心功能类。
   		为当前对象在程序中所处的一个环境，一个与系统交互的过程
   		由于活动本身是一个context对象，这里直接传入acvivity_name.this
   		第二个参数是Toast显示的文本内容，第三个是显示的时长
   		

```

### 2.1.4 在活动中使用Menu

```
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/add_item"
        android:title="Add"/>
    <item
    	android:id="@+id/remove_item"
        android:title="Remove"/>
</menu>
<item>标签用来创建具体的某一个菜单项，
	通过android:id给这个菜单项指定一个唯一的标识符，
	通过android:title 给这个菜单项指定一个名称
```

```
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main,menu);
        return true;
    }
    通过getMenuInflater（）方法能够得到MenuInflater对象，在调用他的inflate（）方法就可以对当前活动创建菜单了。
    inflate（）方法接受两个参数，第一个用于指定我们通过哪一个资源文件创建菜单，第二个用于指定我们的菜单项将添加到哪一个Menu对象中。这里直接使用onCreateOptionMenu()方法传入menu参数，然后将这个方法返回True，表示允许创建的菜单显示出来，如果返回了false，创建的菜单将无法显示。
    		
```

```
    定义菜单响应时间，可以添加或删除菜单 通过调用item.getItemId（）来判断我们点击的是哪一个菜单项
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.add_item:
                Toast.makeText(this,"You clicked Add",Toast.LENGTH_SHORT).show();
                break;
            case R.id.remove_item:
                Toast.makeText(this,"You clicked Remove",Toast.LENGTH_SHORT).show();
                break;
            default:    
        }
        return true;
    }

```

## 2.2 使用Intent在活动在穿梭

### 2.2.1 使用显示intent

​	intent是android程序各组件之间进行交互的一种重要方式，他不仅可以指明当前组件想要执行的动作，还可以在不同组件中传递数据。intent一般可用于启动活动，启动服务以及发送广播等场景。

```
intent(Context packageContext,Class<?> cls)
	第一个参数Context 要求提供启动活动的上下文
	第二个参数Class指定想要启动的目标活动。
startActivity(intent) 方法专门用于启动活动，会接受一个Intent参数
	button1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
               Intent intent=new Intent(FirstActivity.this,SecondActivity.class);
                startActivity(intent);
            }
     首先构建一个intent，传入FirstActivity.this作为上下文，传入SecondActivity.class作为目标活动。即在FirstActivity这个活动的基础上打开SecondActivity，通过startActivity（）来执行intent.
```

### 2.2.2 使用隐式intent

​	隐式intent不明确指出我们想要启动哪一个活动，而制定了一些列的更为抽象的action 和category信息，交由系统判断分析这个Intent，并帮我们找出合适的活动去启动。

```
        <activity
            android:name=".SecondActivity"
            android:exported="true" >
            <intent-filter>
                <action android:name="com.example.activitytest.ACTION_START"/>
                <category android:name="android.intent.category.DEFAULT"/>
            </intent-filter>
        </activity>
     在<activity>标签中，我们指明了当前活动可以响com.example.activitytest.ACTION_START这个action，而<category>这个标签包含了一些附加信息，精确的指明了当前活动能够响应的Intent还可能带有的category。只有<action >和<category>中内容同时能够匹配上Intent中指定的action和category是，这个活动才能响应.
     android.intent.category.DEFAULT 是一种默认的category，调用startActivity（）会自动将其加入到intent.
```

```
每个Intent 只能指定一个action，但能指定多个category
	可利用Intent.addCategory（""） 添加一个category
	Intent增加的category 需要在活动的<intent-filter>标签中可以响应，否则程序报错
```

### 2.2.3 更多隐式intent的用法

```
 // 3 利用系统浏览器打开网页 隐式应用
          Intent intent=new Intent(Intent.ACTION_VIEW);
          intent.setData(Uri.parse("http://www.baidu.com"));
          startActivity(intent);
     这里指定了intent的action 为Intent.ACTION_VIEW，是android系统内置动作，常量值为android.intent.action.VIEW,随后通过Uri.parse()方法，将一个网址字符解析成一个Uri对象，在调用intent的setData()方法将这个Uri对象传递进去
```

### 2.2.4 向下一个活动传输数据

```
利用 intent.putExra（）方法进行传输
在第一个活动中
	int data= 12;	
	Intent intent=new Intent(FirstActivity.this,ThirdActivity.class);
	intent.putExtra("extra_data",data);
	startActivity(intent);
第三个活动中
        Intent intent=getIntent();
        int data=intent.getIntExtra("extra_data",0);
        System.out.println(data);
     在下一个活动中，利用getIntExtra（）获取数据 
     如果是字符串 就用gerStringExtra(0)
```

### 2.2.5 向上一个活动传输数据

```
导入依赖库    implementation 'androidx.activity:activity:1.4.0'
利用 Activityresult API 完成对上一个活动的数据传输
在主活动中写入 
    ActivityResultLauncher<Intent> myResult=registerForActivityResult(
            new ActivityResultContracts.StartActivityForResult(),
            new ActivityResultCallback<ActivityResult>() {
                @Override
                public void onActivityResult(ActivityResult result) {
                    if(result.getResultCode()== RESULT_OK){
                        Intent intent=result.getData();
                        String res=intent.getStringExtra("res");
                        Log.d("FirstActivity",res);

                    }
                }
            }

    );
    利用API构建CONtract协议
    在onclick（）方法中写入
         Intent intent= new Intent(FirstActivity.this,SecondActivity.class);
         String data="hello SecondActivity";
         intent.putExtra("extra_data",data);
         intent.putExtra("data2","你要返回给我吗");
         myResult.launch(intent);
    在第二个活动中写入
        Intent intent=getIntent();
        String data=intent.getStringExtra("extra_data");
        String data1=intent.getStringExtra("data2");
        Log.d("SecondActivity",data);
        Log.d("SecondActivity",data1);
       Button button2=(Button) findViewById(R.id.button_2);
        button2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                    intent.putExtra("res","hell0,FirstActivity");
                    setResult(Activity.RESULT_OK,intent);
                    finish();
            }
        });
```

## 2.3 活动的生命周期

### 2.3.1 返回栈

​	Android是利用认为管理活动的，一个任务就是一组存放在栈里的活动的集合，这个栈被称作返回栈。启动了一个新的活动，会在返回栈中入栈，处于栈顶位置，按下Back()键或者finish()方法销毁一个活动时，栈顶的活动出栈。

### 2.4.2 活动状态

​	共有四种状态

1 运行状态 

​	活动位于返回栈的栈顶时，运行状态

2 暂停状态

​	活动不处于栈顶位置，但仍然可见，就处于暂停状态

3 停止状态

​	活动不在处于栈顶位置，并且完全不可见的时候，进入了停止状态，当其他地方需要内存的时候，停止状态的活动可能会被系统回收。

4 销毁状态

​	当一个活动从返回栈中移除后就变成了销毁状态

### 2.3.3 活动的生存期 

```
onCreate() 在活动的第一次被创建的时候调用。在这个方法中完成活动的初始化操作，例如加载布局，绑定事	件等。
onStart() 在活动由不可见变为可见的时候调用
onResume() 在活动准备好与用户界面交互时调用。活动处于栈顶，并处于运行状态
onPause（）在系统准备去启动或恢复另一个活动的时候调用。在这个方法中将一些消耗CPU的资源释放掉，会保存一些关键数据。
onStop() 在活动完全不可见的时候调用。如果启动的新活动是一个对话框式的，那么会启动onPause（），而onStop不会被执行
onDestroy（） 将活动进行销毁
onRestart() 在活动有停止状态变为运行状态之前调用
	完整生存期 活动在onCreate（）和onDestory之间的所经历的就是完整生存期
	可见生存期 活动在onStart() 和onStop（）之间所经历的就是可见生存期 活动对用户始终可见，即				使不能交互，合理管理对用户可见的资源
	前台生存期 活动在onResume（）和onPause（）之间经历的就是前台生存期 在前台生存期活动总是可				见的
```

### 2.3.4 体验活动的生命周期

### 2.3.5 活动被回收了怎么办？

​	回收了数据如何回调

```
onSaveInstanceState() 可以保证活动被回收之前一定会被调用
					  参数Bundle 可用putString等方法保存 第一个时键 第二个时值
	 @Override
    public void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        String tempData="SM";
        outState.putString("data_ket",tempData);
    }
   保存后，可在onCreate（）方法中取出值 代码如下
       protected void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
      	  	Log.d(TAG,"onCreate");
        	setContentView(R.layout.activity_main);
        	if(savedInstanceState!=null){
            	String tempData=savedInstanceState.getString("data_ket");
           	 	Log.d(TAG, tempData);
       		 }
       }
```

## 2.4 活动的启动模式

​	4种启动模式：standard singlTop singleTask singleInstance 

​	启动模式可在ActivityManifest,xml的《activity》标签中 用android:launchMode选择

### 2.4.1 standard

​	默认的启动模式，不进行设定的话，默认这种。

​	对于standard模式活动，系统不在乎活动是否已经在返回栈中存在，每次启动都会创建该活动的一个实例。

### 2.4.2 singleTop

​	在启动活动时，如果发现返回栈的栈顶已经是该活动，则认为可直接使用它，不需要重建实例。但如果活动并不在栈顶，那么还会创建新的实例

### 2.4.3 singleTask

​	可以很好的解决重复创建栈顶的活动，每次启动该活动时系统首先会在返回栈中检查是否存在该活动的实例，如果发现已经存在直接使用，并把这个活动之上的所有活动统统出栈，如果没有发现就会创建出一个新的实例

### 2.4.4 singleInstance

​	指定一个单独的返回栈来管理活动，不管那个应用程序来访问这个活动，都共同使用一个返回栈，解决了共享活动实例的问题

```
android:theme="@style/Theme.AppCompat.Dialog" 用于给当前活动指定主题 
											 选定对话框主题
```

 

## 2.5 活动的最佳实践

### 2.5.1 知晓当前是哪一个活动

​	重新构建一个类，作为其他活动的父类

```
package com.example.activitytest;

import android.os.Bundle;
import android.util.Log;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("BaseActivity",getClass().getSimpleName());
    }
}
```

### 2.5.2 随时退出程序

```
package com.example.activitytest;

import android.app.Activity;

import java.util.ArrayList;
import java.util.List;

public class ActivityCollector {
    public static List<Activity>activities=new ArrayList<>();
    public static void addActivity(Activity activity) {
        activities.add(activity);
    }
    public static void removeActivity(Activity activity){
        activities.remove(activity);
    }
    public static void finisAll(){
        for(Activity activity:activities){
            if(!activity.isFinishing()){
                activity.finish();
            }
        }
        activities.clear();
    }
}

```

### 2.5.3 启动活动的最佳写法

```
在活动中写一个方法，可进行intent的构建以及参数的传递
public class SecondActivity extends BaseActivity {
    public static void actionStart(Context context,String data1,String data2){
        Intent intent =new Intent(context,SecondActivity.class);
        intent.putExtra("para_num1",data1);
        intent.putExtra("para_num2",data2);
        context.startActivity(intent);
    }
   在主活动中调用这个方法
   SecondActivity.actionStart(FirstActivity.this,"data1","data2");
```


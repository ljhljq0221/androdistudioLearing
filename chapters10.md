# chapters10

## 10.1 服务 

​	Service是Android 中实现程序后台运行的解决方案，非常适合去执行那些不需要和用户交互而且还要求长期运行的任务 。服务的运行不依赖于任何用户界面，即使程序切换到后台，或则用户打开了另一个应用程序，服务仍然能够正常运行

​	服务不是运行在一个独立的进程当中的，而是依赖于创建服务时所在的应用程序进程，当某个应用程序进程被杀死时，所有依赖与该进程的服务也会停止运行。需要服务的内部手动创建子线程，并在这里执行具体任务。

## 10.2  多线程编程

### 10.2.1  线程的基本用法 

​	线程时进程当中的一条执行程序 

​	进程是资源分配的单位 ，线程是CPU调度的单位 

​	新建一个类继承 Thread 然后重写父类的run方法，在里面编写耗时逻辑即可 

```
public class MyThread extends Thread{
    @Override
    public void run() {
        super.run();
    }
}

```

​	 	启动方法只需要new出Mythread 的实例 调用 start方法就可以了

```
new MyThread().start();
```

​	通常利用实现Runnable 接口的方式 来定义一个线程

```
public class MyThread implements Runnable{
    @Override
    public void run() {
        //具体逻辑
    }
}
```

   这样启动线程也需要进行改进

```
MyThread myThread =new MyThread();
new Thread(myThread).start();
```

​	如下更常见

```
new Thread(new Runnable(){
	    @Override
    public void run() {
        //具体逻辑
    }
}).start();
```

### 10.2.2 在子线程中更新UI

​	更新应用程序的UI元素，必须在主线程中进行，否则回出现bug，如下所示

```
   android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
```

```
    private TextView text;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        text=(TextView) findViewById(R.id.text);
        Button changeText =(Button) findViewById(R.id.change_text);
        changeText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()){
                    case R.id.change_text:
                        new Thread(new Runnable() {
                            @Override
                            public void run() {
                                text.setText("Nice to meet you");
                            }
                        }).start();
                        break;
                    default:
                        break;
                }
            }
        });
    }
}
```

​	有时候需要根据子线程处理的结果更新UI控件，这需要借助到异步消息处理机制 ，修改代码

```
public class MainActivity extends AppCompatActivity {
    public static final int UPDATE_TEXT=1;
    private TextView text;
    private Handler handler=new Handler(){
        @Override
        public void handleMessage(@NonNull Message msg) {
            switch (msg.what){
                case UPDATE_TEXT:
                    //这这里可进行UI操作
                    text.setText("Nice to meet you");
                    break;
                default:
                    break;
            }
        }
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        text=(TextView) findViewById(R.id.text);
        Button changeText =(Button) findViewById(R.id.change_text);
        changeText.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()){
                    case R.id.change_text:
                        new Thread(new Runnable() {
                            @Override
                            public void run() {
                                Message message =new Message();
                                message.what=UPDATE_TEXT;
                                handler.sendMessage(message);//将message对象发出去 
                            }
                        }).start();
                        break;
                    default:
                        break;
                }
            }
        });
    }
}
```

​	但在API30中，Handle的无参构造函数被弃用，但还可以用其他的方法进行构造Handler对象

```
使用 Executor
明确指定Looper 。
使用Looper.getMainLooper()定位并使用主线程的Looper
如果又想在其他线程，又想要不出bug，请使用new Handler(Looper.myLooper(), callback) 或者 new Handler(Looper.myLooper()）两个构造方法。
```

```
private final Handler handler = new Handler(Looper.getMainLooper());
```

```
Handler() 此构造函数已弃用。在 Handler 构造期间隐式选择 Looper 会导致操作无声地丢失（如果 Handler 不期待新任务并退出）、崩溃（如果有时在没有 Looper 活动的线程上创建处理程序）或竞争条件，处理程序关联的线程不是作者预期的。相反，使用 Executor 或 显式指定 Looper，使用 Looper#getMainLooper {link android.view.View#getHandler} 或类似方法。如果为了兼容性需要隐式线程本地行为，请使用 new Handler(Looper.myLooper())。

```

### 10.2.3 解析异步消息处理机制

#### 	1 message 

​	message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间交换数据。上一小节中使用到了Message的what字段，此外还可用arg1和arg2字段来携带一些整形数据，使用obj字段携带一个Object对象

#### 	2 Handle

​	主要用于处理和发送信息，发送信息用Handler.sendMessage()方法，发出的消息会传递到handleMessage（）方法中

#### 	3 MessageQueue

​	消息队列，用于存放所有通过Handler发送的消息，这部分消息会一直存在于消息队列中，等待被处理，每个线程只会有一个MessageQueue对象

#### 	4 Looper

​	每个线程中消息队列的MessageQueue的管家，调用Looper的loop（）方法后，就会进入到一个无限循环中，每当发现消息对象中存在一条信息，就会将其取出，传递到hanleMessage方法中 

### 10.2.4 使用AsyncTask

​	抽象类 指定3个泛型参数

```
Params 在执行AsyncTask是需要传入的参数，可用于在后台任务中使用 
Progress 后台任务执行完毕时，如果需要在界面上显示当前进度，使用这里指定的			泛型作为进度单位
Result 当任务执行完毕后，需要对结果进行返回，需要指定这里的泛型作为返回值class DownloadTask extends AsyncTask<Void,Integer,boolean>{
    
}
```

```
class DownloadTask extends AsyncTask<Void,Integer,Boolean>{
        //第一个参数为Void,表示执行AsyncTask期间不需要传入参数给后台
        //第二个参数为 Integer 表示使用整形数据作为进度显示单位
        //第三个参数为 Boolean  表示使用布尔型数据反馈执行结果

        //在后台任务开始执行之前调用，用于进行界面上的初始化操作，如在界面上显示一个进度条
        @Override
        protected void onPreExecute() {
            progressDialog.show();
        }

        // 该方法在子线程中运行，我们应该在这里去处理所有的耗时任务，任务完成就可通过return
        //将任务的执行结果返回
        //如果AsyncTask的第三个参数是 Void，可以不返回任务执行结果
        //这个方法不可以进行UI操作，如果需要更新UI，可以调用publishProgress(Progress)完成
        @Override
        protected boolean doInBackground(Void... voids) {
            try{
                while(true){
                    int downloadPercent=doDownload;
                    publishProgress(downloadPercent);
                    if(downloadPercent>=100) break;
                }
            }catch (Exception e){
                return false;
            }
            return true;
        }

        /*
            当在后台中调用了publishProgress(Progress) 方法后，该方法很快被调用，方法中的参数就是
                后台任务中传递过来的。在这个方法中可对UI进行操作，利用参数的数组进行更新UI
         */

        @Override
        protected void onProgressUpdate(Integer... values) {
            //更新下载进度
            progressDialog.setMessage("Downloadede "+ values[0]+"%");
                    
        }

        /*
            后台执行完毕并通过retuen返回时，这个方法会被调用。返回的数据作为参数传递到该方法
            可利用返回的数据进行更新UI操作
         */

        @Override
        protected void onPostExecute(Boolean result) {
            progressDialog.dismiss();//关闭对话框
            //在这里提示下载进度 
            if(result ){
                Toast.makeText(context,"Download Succeed",Toast.LENGTH_SHORT).show();
            }
            else {
                Toast.makeText(context,"Download failed",Toast.LENGTH_SHORT).show();
            }
        }
    }

```

 启动上述任务 

```
new DownloadTask.execute();
```

#### 问题 API30中弃用了 AsyncTask

  	替代方法 Executors

```

val executor = Executors.newSingleThreadExecutor()
val handler = Handler(Looper.getMainLooper())
executor.execute {
    /*
    * Your task will be executed here
    * you can execute anything here that
    * you cannot execute in UI thread
    * for example a network operation
    * This is a background thread and you cannot
    * access view elements here
    *
    * its like doInBackground()
    * */
    handler.post {
        /*
        * You can perform any operation that
        * requires UI Thread here.
        *
        * its like onPostExecute()
        * */
    }


```

```
    ExecutorService executorService = Executors.newScheduledThreadPool(4);
        executorService.execute(new Runnable() {
        @Override
        public void run() {

            //调用发送到UI线程中 
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    //这里是在UI线程中执行的 使用这里是可以更新UI控件的
                }
            });
        }
    });

```

## 10.3 服务的基本用法 

### 10.3.2 启动和停止 

```
public class MyService extends Service {
    private static String TAG= "MyService ";
    public MyService() {

    }
//onCreate() 服务第一次创建的时候调用
    @Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate: exexuted");
    }

    //每次服务启动的时候调用
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand:  exexuted");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy:  exexuted");
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

```
public class MainActivity extends AppCompatActivity  implements View.OnClickListener{

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button startService =(Button) findViewById(R.id.start_service);
        Button stopService=(Button) findViewById(R.id.stop_service);
        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.start_service:
                Intent startIntent =new Intent(this,MyService.class);
                startService(startIntent);
                break;
            case R.id.stop_service:
                Intent stopIntent =new Intent(this,MyService.class);
                stopService(stopIntent);
                break;
            default:
                break;
        }
    }
}
```

### 10.3.3 活动和服务之间进行通信 

​	借助于onBind( ) 实现通信 

​	例如 ：希望在服务中提供一个下载功能，活动中可以决定合适可以下载，以及随时查看下载进度 。实现这个功能的思路时创建一个Binder对象来对下载功能进行管理

```
public class MyService extends Service {
    private static String TAG= "MyService ";
    private Downlo adBinder mBinder=new DownloadBinder();
    class  DownloadBinder extends Binder{
        public void startDownload(){
            Log.d(TAG, "startDownload: executed");
        }
        public int getProgress(){
            Log.d(TAG, "getProgress: executed");
            return 0;
        }
    }
...
    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return mBinder;
    }
}
```

```
public class MainActivity extends AppCompatActivity  implements View.OnClickListener{

    private MyService.DownloadBinder downloadBinder;
    private ServiceConnection connection=new ServiceConnection() {
       // 活动与服务绑定的时候调用
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder=(MyService.DownloadBinder) service;
            downloadBinder.startDownload();
            downloadBinder.getProgress();
        }
        //活动与服务连接断开的时候调用
        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button startService =(Button) findViewById(R.id.start_service);
        Button stopService=(Button) findViewById(R.id.stop_service);
        startService.setOnClickListener(this);
        stopService.setOnClickListener(this);
        Button bindService =(Button) findViewById(R.id.bind_service);
        Button unbindService =(Button) findViewById(R.id.unbind_service);
        bindService.setOnClickListener(this);
        bindService.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.start_service:
                Intent startIntent =new Intent(this,MyService.class);
                startService(startIntent);
                break;
            case R.id.stop_service:
                Intent stopIntent =new Intent(this,MyService.class);
                stopService(stopIntent);
                break;
            case R.id.bind_service:
                Intent bindIntent=new Intent(this,MyService.class);
                // bindService 三个参数 第一个是 Intent 对象
                //                     第二个是 ServiceConnection实例
                //                     第三个是 标志位 BIND_AUTO_CREATE
                //                      表示在活动和服务进行绑定后自动创建服务
                bindService(bindIntent,connection,BIND_AUTO_CREATE);//绑定服务
                break;
            case R.id.unbind_service:
                unbindService(connection);//解除服务
                break;
            default:
                break;
        }
    }
}
```

## 10.4 服务的生命周期 

​	

```
//第一次创建服务的时候调用
@Override
    public void onCreate() {
        super.onCreate();
        Log.d(TAG, "onCreate: exexuted");
    }

    //每次服务启动的时候调用
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d(TAG, "onStartCommand:  exexuted");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.d(TAG, "onDestroy:  exexuted");
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return mBinder;
    }
```

​	还可用利用Context的bindService 获取一个服务的持久链接 。一个服务只要被启动或者绑定了以后，就一直处于运行状态 ，必须让以上两着同时不满足才可以销毁服务 

## 10.5 服务的更多技巧

### 10.5.1 前台服务 

  	希望服务一直保持运行，不会由于系统内存不足的原因导致被回收，可以考虑使用前台服务。前台服务与普通服务的区别在于有一个一直运行的图标在系统的状态栏显示，下拉后就可看到更加详细的信息。

#### BUG

```
: Permission Denial: startForeground from pid=5962, uid=10129 requires android.permission.FOREGROUND_SERVICE
这是由于Android在8.0开始限制后台服务，启动后台服务需要设置通知栏，使服务变成前台服务。同时在Android 9.0开始注册权限android.permission.FOREGROUND_SERVICE，并授权权限。
```

```
Bad notification for startForeground
Android 8.0开始Notification 需要指定一个channel，当target大于26时，这个channel需要进行系统注册，否则会crash，crash信息如下
```

```
 Notification notification=null;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel = new NotificationChannel("xxx", "xxx", NotificationManager.IMPORTANCE_LOW);
            NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            Intent intent=new Intent(this,MainActivity.class);
            PendingIntent pi=PendingIntent.getActivity(this,0,intent,0);
            if (manager == null)
                return;
            manager.createNotificationChannel(channel);
            notification = new NotificationCompat.Builder(this, "xxx")
                    .setAutoCancel(true)
                    .setCategory(Notification.CATEGORY_SERVICE).setOngoing(true)
                    .setPriority(NotificationManager.IMPORTANCE_LOW)
                    . setContentTitle("THis is content title")
                    .setContentText("this is content text")
                    .setWhen(System.currentTimeMillis())
                    .setSmallIcon(R.mipmap.ic_launcher)
                    .setLargeIcon(BitmapFactory.decodeResource(getResources(),
                            R.mipmap.ic_launcher))
                    .setContentIntent(pi)
                    .build();
        }
        startForeground(1,notification);
```

```
 Intent intent=new Intent(this,MainActivity.class);
            PendingIntent pi=PendingIntent.getActivity(this,0,intent,0);
            NotificationCompat.Builder notification;
            NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
                NotificationChannel channel = new NotificationChannel("xxx", "xxx", NotificationManager.IMPORTANCE_LOW);
                channel.enableLights(true);
                channel.setShowBadge(true);
                channel.setLockscreenVisibility(Notification.VISIBILITY_PUBLIC);
                manager.createNotificationChannel(channel);
                notification = new NotificationCompat.Builder(DownloadService.this).setChannelId("xxx");
            }else {
                notification =new NotificationCompat.Builder(DownloadService.this);
            }

                notification.setSmallIcon(R.mipmap.ic_launcher).setLargeIcon(BitmapFactory.decodeResource(getResources(),
                        R.mipmap.ic_launcher)).setContentIntent(pi).setContentTitle(title);
                if(progress>=0){
                    //当Progress 大于或等于0的时候，显示下载进度
                    notification.setContentText(progress+"%");
                    notification.setProgress(100,progress,false);
            }
            return notification.build();
```



### 10.5.2 使用IntentService

 服务中的代码默认运行在主线程中 ，如果在服务里处理一些耗时的逻辑，很容易出现ANR现象。我们应该在服务的每个具体方法里开启一个子线程，然后在这里去处理那些耗时的逻辑 。因此一个比较标准的服务为

```
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
       new Thread(new Runnable() {
           @Override
           public void run() {
               //
           }
       }).start();
        Log.d(TAG, "onStartCommand:  exexuted");
        return super.onStartCommand(intent, flags, startId);
    }

```

​	服务在执行完毕后自动停止

```
    public int onStartCommand(Intent intent, int flags, int startId) {
       new Thread(new Runnable() {
           @Override
           public void run() {
               //处理具体逻辑
               stopSelf();
           }
       }).start();
        Log.d(TAG, "onStartCommand:  exexuted");
        return super.onStartCommand(intent, flags, startId);
    }
```

 IntentService 类 异步的 自动停止的类

## 10.6 完整版的下载实例 

### 	1 添加依赖库

```
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
```

### 	2 设定回调接口 ，用于对下载过程中的各种状态进行监听和回调

```
public interface DownloadListener {
   // 通知当前下载进度 
    void onProgress(int progress);
    // 通知下载成功
    void Success();
    //通知下载失败
    void onFailed();
    // 通知下载暂停
    void onPaused();
    //下载取消 
    void onCanceled();

}
```

### 	3 编写下载功能 

​	新建一个DownloadTask继承自AsyncTask

```
import android.os.AsyncTask;
import android.os.Environment;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.io.RandomAccessFile;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

/*
    第一个泛型参数为String  需要传入一个字符串给后台任务
    第二个参数为Integer 表示使用整形数据作为进度显示单位
    第三个参数为 Integer 使用整形数据来反馈执行结果
 */
public class DownloadTask extends AsyncTask<String,Integer,Integer> {
    public static final int TYPE_SUCCESS=0;
    public static final int TYPE_FAILED=1;
    public static final int TYPE_PAUSED=2;
    public static final int TYPE_CANCELED=3;

    private DownloadListener listener;
    private boolean isCanceled=false;
    private boolean isPaused=false;
    private int lastProgress;

    public DownloadTask(DownloadListener listener){
        this.listener=listener;
    }

    @Override
    protected Integer doInBackground(String... params) {
        InputStream is=null;
        RandomAccessFile savedFile=null;
        File file= null;
        try {
            long downloadedLength=0;//记录已下载的文件长度
            String downloadUri=params[0];// 获取下载地址
            String fileName=downloadUri.substring(downloadUri.lastIndexOf("/"));
            //指定文件下载地址 
            String directory= Environment.getExternalStoragePublicDirectory
                    (Environment.DIRECTORY_DOWNLOADS).getPath();
            file = new File(directory+fileName);
            if(file.exists()){
                downloadedLength=file.length();
            }
            long contentLength =getContentLength(downloadUri);
            if(contentLength==0) {
                return TYPE_FAILED;
            }else  if(contentLength==downloadedLength){
                //已下载字节和文件总字节相等，下载完成
                return TYPE_SUCCESS;
            }
            OkHttpClient client =new OkHttpClient();
            Request request= new Request.Builder()
                    //断电下载，指定从哪个字节开始下载
                    .addHeader("RANGE","bytes="+downloadedLength+"-")
                    .url(downloadUri)
                    .build();
            Response response=client.newCall(request).execute();
            if(response!=null) {
                is= response.body().byteStream();
                savedFile =new RandomAccessFile(file,"rw");
                savedFile.seek(downloadedLength);//跳过已经下载的字节
                byte[]b=new byte[1024];
                int total=0;
                int len;
                while((len=is.read(b))!=-1){
                    if(isCanceled){
                        return TYPE_CANCELED;
                    }else if(isPaused) {
                        return TYPE_PAUSED;
                    }else {
                        total+=len;
                        savedFile.write(b,0,len);
                        //计算已下载的百分比
                        int progress =(int)((total+downloadedLength)*100/contentLength);
                        publishProgress(progress);
                    }
                }
                response.body().close();
                return  TYPE_SUCCESS;
            }

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            try{
                if(is!=null){
                    is.close();
                }
                if (savedFile!=null){
                    savedFile.close();
                }
                if(isCanceled&&file!=null){
                    file.delete();
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return TYPE_FAILED;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        int progress=values[0];
        if(progress>lastProgress){
            listener.onProgress(progress);
            lastProgress=progress;
        }
    }

    @Override
    protected void onPostExecute(Integer status) {
        switch (status){
            case TYPE_SUCCESS:
                listener.onSuccess();
                break;
            case TYPE_FAILED:
                listener.onFailed();
            case TYPE_PAUSED:
                listener.onPaused();
            case TYPE_CANCELED:
                listener.onCanceled();
                break;
            default:
                break;
        }
    }

    public void pauseDownload(){
        isPaused=true;
    }
    public void cancelDownload(){
        isCanceled=true;
    }

    private long getContentLength(String downloadUri) throws IOException {
        OkHttpClient client=new OkHttpClient();
        Request request=new Request.Builder()
                .url(downloadUri)
                .build();
        Response response=client.newCall(request).execute();
        if(response!=null&& response.isSuccessful()){
            long contentLength =response.body().contentLength();
            response.body().close();
            return contentLength;
        }
        return 0;
    }
}

```

### 	4 服务

```
package com.example.servicebestpractice;

import android.app.Notification;
import android.app.NotificationChannel;
import android.app.NotificationManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Context;
import android.content.Intent;
import android.graphics.BitmapFactory;
import android.os.Binder;
import android.os.Build;
import android.os.Environment;
import android.os.IBinder;
import android.widget.Toast;

import androidx.core.app.NotificationCompat;

import java.io.File;

public class DownloadService extends Service {
    private DownloadTask downloadTask;
    private String downloadUri;
    private DownloadListener listener =new DownloadListener() {
        @Override
        public void onProgress(int progress) {
            getNotificationManager().notify(1,
                    getNotification("Downloading...",progress) );
        }

        @Override
        public void onSuccess() {
            downloadTask=null;
            //下载成功时将前台服务关闭，并创建一个下载成功的通知
            stopForeground(true);
            getNotificationManager().notify(1,
                    getNotification("Download Success",-1));
            Toast.makeText(DownloadService.this, "Download Success", Toast.LENGTH_SHORT).show();

        }

        @Override
        public void onFailed() {
            downloadTask=null;
            //下载成功时将前台服务关闭，并创建一个下载失败的通知
            stopForeground(true);
            getNotificationManager().notify(1,
                    getNotification("Download Failed",-1));
            Toast.makeText(DownloadService.this, "Download Failed", Toast.LENGTH_SHORT).show();

        }

        @Override
        public void onPaused() {
            downloadTask=null;
            Toast.makeText(DownloadService.this, "Paused", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onCanceled() {
            downloadTask=null;
            stopForeground(true);
            Toast.makeText(DownloadService.this, "Canceled", Toast.LENGTH_SHORT).show();
        }
    };

    private DownloadBinder mBinder=new DownloadBinder();

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return mBinder;
    }
    class DownloadBinder extends Binder {
        public void startDownload(String url) {
            if (downloadTask == null) {
                downloadUri = url;
                downloadTask = new DownloadTask(listener);
                downloadTask.execute(downloadUri);
                startForeground(1, getNotification("Downloading", 0));
                Toast.makeText(DownloadService.this, "Downloading", Toast.LENGTH_SHORT).show();
            }
        }

        public void pauseDownload() {
            if (downloadTask != null) {
                downloadTask.pauseDownload();
            }
        }

        public void cancelDownload() {
            if (downloadTask != null) {
                downloadTask.cancelDownload();
            }
            if (downloadUri != null) {
                //取消下载时需将文件删除，并关闭通知
                String fileName = downloadUri.substring(downloadUri.lastIndexOf("/"));
                String directory = Environment.getExternalStoragePublicDirectory
                        (Environment.DIRECTORY_DOWNLOADS).getPath();
                File file = new File(directory + fileName);
                if (file.exists()) {
                    file.delete();
                }
                getNotificationManager().cancel(1);
                stopForeground(true);
                Toast.makeText(DownloadService.this, "Canceled", Toast.LENGTH_SHORT).show();
            }
        }
    }

        private NotificationManager getNotificationManager(){
            return (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        }
        private Notification getNotification(String title,int progress){

                Intent intent=new Intent(this,MainActivity.class);
                PendingIntent pi=PendingIntent.getActivity(this,0,intent,0);
                NotificationCompat.Builder builder= new NotificationCompat.Builder(this);
                builder.setSmallIcon(R.mipmap.ic_launcher);
                builder.setLargeIcon(BitmapFactory.decodeResource(getResources(),
                                R.mipmap.ic_launcher));
                builder.setContentIntent(pi);
                builder.setContentTitle(title);
                if(progress>=0){
                    //当Progress 大于或等于0的时候，显示下载进度
                    builder.setContentText(progress+"%");
                    builder.setProgress(100,progress,false);
                }
                return builder.build();


        }
}
```

### 5 活动

```
package com.example.servicebestpractice;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.ComponentName;
import android.content.Intent;
import android.content.ServiceConnection;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.os.IBinder;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity implements View.OnClickListener {

    private DownloadService.DownloadBinder downloadBinder;

    private ServiceConnection connection=new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder =(DownloadService.DownloadBinder) service;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button startDownload=(Button) findViewById(R.id.start_download);
        Button pauseDownload=(Button) findViewById(R.id.pause_download);
        Button cancelDownload=(Button) findViewById(R.id.cancel_download);
        startDownload.setOnClickListener(this);
        pauseDownload.setOnClickListener(this);
        cancelDownload.setOnClickListener(this);
        Intent intent =new Intent(this,DownloadService.class);
        startService(intent);//启动服务
        bindService(intent,connection,BIND_AUTO_CREATE);//绑定服务
        if(ContextCompat.checkSelfPermission(MainActivity.this,
                Manifest.permission.WRITE_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(MainActivity.this,new String[]
                    {Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
        }
    }

    @Override
    public void onClick(View v) {
        if(downloadBinder==null) return;
        switch (v.getId()){
            case R.id.start_download:
                String uri="https://raw.githubusercontent.com/guolindev/eclipse" +
                        "/master/eclipse-inst-win64.exe";
                downloadBinder.startDownload(uri);
                break;
            case R.id.pause_download:
                downloadBinder.pauseDownload();
                break;
            case R.id.cancel_download:
                downloadBinder.cancelDownload();
                break;
            default:
                break;
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                    Toast.makeText(this, "权限拒绝无法访问", Toast.LENGTH_SHORT).show();
                    finish();
                }
                break;
            default:
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(connection);
    }
}
```

### 6 BUG

```
 Caused by: java.net.ConnectException: failed to connect to localhost/127.0.0.1 (port 443) after 10000ms: isConnected failed: ECONNREFUSED (Connection refused)
 	 安卓模拟器把 127.0.0.1/localhost 当做模拟器本身，没有访问电脑。
 	 在模拟器上用10.0.2.2访问你的电脑本机
```


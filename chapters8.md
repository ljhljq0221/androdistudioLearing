# chapters8

## 8.1 通知使用

### 8.2.1 通知的基本用法 

```
	首先需要一个NotificationManager 来对通知进行管理 可以通过调用 Context的getSystemSrevice方法得到。getSystemSrevice接受一个字符串参数用于确定获取系统的哪个服务，传入Context.NOTIFICATION_SERVICE 
	NotificationManager manager=(NotificationManager)getSystemSrevice(Context.NOTIFICATION_SERVICE);
	接下来需要一个Builder构造器创建NotificationManager对象，使用NotificationCompat 构造器创建Notification 对象
	Notification notification =new NotificationCompat.Builder(Context).builder();
	上述为一个空的Notification 对象，可以在最终的build()方法之前连缀任意多的设置方法来创建一个丰富的Notification 对象
	Notification notification = new NotificationCompat.Builder(context).setContentTitle("This is content title").setContentText("This is contenttext").setWhen(System.currrentTimeMills()).
setSmallIcon(R.drawable.small_icon).setLargeIcon(BitmapFactoty.decodeResource(getResource(),R.drawable.large_icon)).builde();
	上述消息内容设置完毕后，只需要调用NotificationManager的notify()方法就可以让通知显示出来了。 
		第一个参数 是id 
		第二个参数时 Notification对象
		manager.notify(1,notification)
```

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button sendNotice=(Button) findViewById(R.id.send_notice);
        sendNotice.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()){
                    case R.id.send_notice:
                        NotificationManager manager=(NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                        Notification notification=new Notification.Builder(MainActivity.this).
                            setContentTitle("This is content title").
                            setContentText("This is content text").setWhen(System.currentTimeMillis()).
                            setSmallIcon(R.mipmap.ic_launcher).
                            setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher)).build();
                        manager.notify(1,notification);
                        break;
                    default:
                        break;
                }
            }
        });
    }
}
```

```
PendingIntent 
Intent 更加倾向于立即执行某个动作 
PendingIntent  更加倾向于在某个合适的时机去执行某个动作 
可根据需求选择使用getActivity() getBroadcast()
getService() 
	第一个参数 Content 
	第二个参数 通常传入0
	第三个参数 Intent 对象 通过这个对象构建出 PendIngIntent的意图
	第四个参数用于确定 PendIngIntent的 行为，通常为0
	
```

```
                              switch (v.getId()){
                    case R.id.send_notice:
                        Intent intent= new Intent(MainActivity.this,NotificationActivity.class);
                        PendingIntent pi=PendingIntent.getActivity(MainActivity.this,0,intent,0);
                        NotificationManager manager=(NotificationManager) getSystemService(NOTIFICATION_SERVICE);
                        Notification notification=new Notification.Builder(MainActivity.this).
                            setContentTitle("This is content title").
                            setContentText("This is content text").setWhen(System.currentTimeMillis()).
                            setSmallIcon(R.mipmap.ic_launcher).
                            setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                                .setContentIntent(pi).build();
                        manager.notify(1,notification);
                        break;
                    default:
                        break;
```



#### Bug 

```
 java.net.SocketTimeoutException: failed to connect to play.googleapis.com/172.217.163.42 (port 443) 网络问题 pc与手机不在一个网络
```

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button sendNotice=(Button) findViewById(R.id.send_notice);
        sendNotice.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()){
                    case R.id.send_notice:
                        Intent intent = new Intent(MainActivity.this,NotificationActivity.class);
                        PendingIntent pendingIntent = PendingIntent.getActivity(MainActivity.this,0,intent,0);

                        Notification notification=new Notification.Builder(MainActivity.this).
                                setContentTitle("This is content title").
                                setContentText("This is content text").setWhen(System.currentTimeMillis()).
                                setSmallIcon(R.mipmap.ic_launcher).
                                setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                                .setContentIntent(pendingIntent)
                                .build();
                        manager.notify(1,notification);
                }
            }
        });
    }
}
```

#### 取消通知

```
1 .setAutoCancel(true)
```

```
2 NotificationManager manager =(NotificationMananger) getSystemService(NOTIFICATION_SERVICE);
manager.cancel(1);
```

### 8.2.2 进阶技巧

```
setSound()方法 可以在通知发出的时候播放一段音频，接受一个Uri参数，指定音频文件的时候还需要获			的音频文件对应的URI
.setVibrate 通知到来的时候手机振动,用于设置手机静止和振动的时长，以毫秒为单位，下标为0表示手机静止的时长，下标为1表示手机振动的时长。
想让手机振动需要声明权限
	.setSound(Uri.fromFile(new File("/Music/郭静_心墙.lrc")))
	 <uses-permission android:name="android.permission.VIBRATE"/>
.setLights 展示LED灯闪烁 第一个参数 LED的颜色  第二个参数 LED等亮起的时长
						第三个参数 LED暗去的时长 以毫秒为单位
						.setLights(Color.GREEN,1000,1000).build();

```

```
如果不想搞这么多复杂设置 ，可以全部选用通知的默认效果，根据当前环境决定播放什么
.setDefaults(NotificationCompat.DEFAULT_ALL).build();
```

### 8.2.3 高级技巧

```
.setStyle() 允许我们构建出富文本的通知内容，不光有文字和图标，还可以包含更多东西 接受一个
		NotificationCompat.Stytle 参数，这个参数可以构建具体的富文本信息
	如果需要传入一个长文本，那么在setStyle()方法中，创建一个
Notification.BigTextStyle()对象，并调用他的bigText的方法将文字传入
	setStyle(new Notification.BigTextStyle().bigText
                   ("我带哦对弄u哦啊的切克闹阿斯顿你Joan查克拉卡洛斯积分 阿松第app"))
    如果需要传入一副图片，调用Notification.BigPictureStyle()对象，这个对象适用于设置方法图片的，然后调用bigPicture将图片传入。这里提放置了一张图片，通过(BitmapFactory.decodeResource（）方法将图片解析成Bitmap对象，在传入bigPicture 就好了
    .setStyle(new Notification.BigPictureStyle().bigPicture                           (BitmapFactory.decodeResource(getResources(),R.drawable.big_image)))
  setPriority() 用于设置通知的重要程度 接受一个整形参数用于设置这条通知的重要程度
  	PRORITY_DEFAULT 默认的重要程度 
  	PRORITY_MIN 最低的重要程度
  	PRORITY_LOW 较低的重要程度 
  	PRORITY_HIGH 较高的重要程度 
  	PRORITY_MAX 最高的重要程度 
```

#### 模拟器不能运行问题

​	下述代码模拟器真机都可进行

```
通道设置 
	public class MainActivity extends AppCompatActivity {
    private Button GetNotification;
    private static final int ID = 1;
    private static final String CHANNELID ="1";
    private static final String CHANNELNAME = "channel1";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        GetNotification = (Button) findViewById(R.id.GetNotification);
        GetNotification.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                NotificationManager manager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
               // manager.cancel(1);
                //安卓8.0以上弹出通知需要添加渠道NotificationChannel
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
                    //创建渠道
                    /**
                     * importance:用于表示渠道的重要程度。这可以控制发布到此频道的中断通知的方式。
                     * 有以下6种重要性，是NotificationManager的静态常量，依次递增:
                     * IMPORTANCE_UNSPECIFIED（值为-1）意味着用户没有表达重要性的价值。此值用于保留偏好设置，不应与实际通知关联。
                     * IMPORTANCE_NONE（值为0）不重要的通知：不会在阴影中显示。
                     * IMPORTANCE_MIN（值为1）最低通知重要性：只显示在阴影下，低于折叠。这不应该与Service.startForeground一起使用，因为前台服务应该是用户关心的事情，所以它没有语义意义来将其通知标记为最低重要性。如果您从Android版本O开始执行此操作，系统将显示有关您的应用在后台运行的更高优先级通知。
                     * IMPORTANCE_LOW（值为2）低通知重要性：无处不在，但不侵入视觉。
                     * IMPORTANCE_DEFAULT （值为3）：默认通知重要性：随处显示，产生噪音，但不会在视觉上侵入。
                     * IMPORTANCE_HIGH（值为4）更高的通知重要性：随处显示，造成噪音和窥视。可以使用全屏的Intent。
                     */
                    NotificationChannel channel = new NotificationChannel(CHANNELID,CHANNELNAME,NotificationManager.IMPORTANCE_HIGH);
                    manager.createNotificationChannel(channel);//开启渠道
                    Intent intent = new Intent(MainActivity.this,notification.class);
                    PendingIntent pendingIntent = PendingIntent.getActivity(MainActivity.this,0,intent,0);
                    NotificationCompat.Builder builder = new NotificationCompat.Builder(MainActivity.this,CHANNELID);
                    builder .setContentTitle("Title")//通知标题
                            .setContentText("ContentText")//通知内容
                            .setWhen(System.currentTimeMillis())//通知显示时间
                            .setContentIntent(pendingIntent)
                            .setSmallIcon(R.drawable.smile)
                            .setAutoCancel(true)//点击通知取消
                            //.setSound()
                            //第一个参数为手机静止时间，第二个参数为手机震动时间，周而复始
                            .setVibrate(new long[] {0,1000,1000,1000})//手机震动
                            //第一个参数为LED等颜色，第二个参数为亮的时长，第三个参数为灭的时长
                            .setLights(Color.BLUE,1000,1000)
                            /**表示通知的重要程度
                             * RIORITY_DEFAULT
                             * RIORITY_MIN
                             * RIORITY_LOW
                             * RIORITY_HIGE
                             * RIORITY_MAX
                            **/
                            .setPriority(NotificationCompat.PRIORITY_MAX)
                            .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.smile)).build();
                    manager.notify(1,builder.build());
                } else{
                Notification notification = new NotificationCompat.Builder(MainActivity.this)
                        .setContentTitle("Title")
                        .setContentText("ContentText")
                        .setWhen(System.currentTimeMillis())
                        .setSmallIcon(R.drawable.smile)
                        .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.drawable.smile))
                        .build();
                manager.notify(1,notification);
            }
       }
        });
    }
}

```

## 8.3 调用摄像头

### 	8.3.1 拍照

```
// 适用于安卓10以下版本
public class MainActivity extends AppCompatActivity {
    public static final int TAKE_PHOTO=1;
    private ImageView picture;
    private Uri imageUri;

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    try {
                        //把拍摄的照片取出来
                            Bitmap bitmap = null;
                            bitmap = BitmapFactory.decodeStream
                                    (getContentResolver().openInputStream(imageUri));
                            picture.setImageBitmap(bitmap);
                        }  catch (FileNotFoundException e) {
                            e.printStackTrace();
                    }
                }
                break;
            default:
                break;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button takePhoto=(Button) findViewById(R.id.take_photo);
        picture=(ImageView) findViewById(R.id.picture);
        takePhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //创建File 对象，用于存储拍照后的图片 将其存放在手机SD卡的应用关联缓存目录下
                File outputImage=new File(getExternalCacheDir(),"output_image.jpg");
                try {
                    if(outputImage.exists()){
                        outputImage.delete();
                    }
                    outputImage.createNewFile();
                }catch (IOException e){
                    e.printStackTrace();
                }
                if(Build.VERSION.SDK_INT>=24){
                    imageUri= FileProvider.getUriForFile(MainActivity.this,
                            "com.example.cameraalbumtest.fileprovider",outputImage);
                }else {
                    imageUri=Uri.fromFile(outputImage);
                }
                //启动相机程序
                Intent intent=new Intent("android.media.action.IMAGE_CAPTURE");
                intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri);
                startActivityForResult(intent,TAKE_PHOTO);
            }
        });

    }
}
```

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:orientation="vertical">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/take_photo"
        android:text="Take photo"/>
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/picture"
        android:layout_gravity="center_vertical"/>
</LinearLayout>
```

```
<?xml version="1.0" encoding="utf-8" ?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path
        name="my_images"
        path="Pictures"/>
</paths>
```

### 8.3.2 从相册中选择照片

​	1 加入从相册选择照片的逻辑

```
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case TAKE_PHOTO:
                if (resultCode == RESULT_OK) {
                    try {
                        //把拍摄的照片取出来
                            Bitmap bitmap = null;
                            //调用BitmapFactory.decodeStream方法将 output_image.jpg 解析成Bitmap图像，然后在让其显示出来
                            bitmap = BitmapFactory.decodeStream
                                    (getContentResolver().openInputStream(imageUri));
                            picture.setImageBitmap(bitmap);
                        }  catch (FileNotFoundException e) {
                            e.printStackTrace();
                    }
                }
                break;
            case CHOOSE_PHOTO:
                handleImageOnKitKat(data);
            default:
                break;
        }
    }
    @TargetApi(19)
    //对封装过的Uri 进行解析
    private void handleImageOnKitKat(Intent data){
        String imagePath=null;
        Uri uri=data.getData();
        if(DocumentsContract.isDocumentUri(this,uri)){
            //如果时document 类型的Uri，则通过document id处理
            String docId=DocumentsContract.getDocumentId(uri);
            Log.d("MainActivity",docId);
            if("com.android.providers.media.documents".equals(uri.getAuthority())){
                String id =docId.split(";")[1];//解析出数字格式的id
                
                String selection=MediaStore.Images.Media._ID+"="+id;
                imagePath= getImagePath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,selection);
            }else if("com.android.providers.downloads.documents".equals(uri.getAuthority())){
                Uri contentUri= ContentUris.withAppendedId(Uri.parse(
                        "content://downloads//public_downloads"),Long.valueOf(docId));
                System.out.println(contentUri);
                imagePath=getImagePath(contentUri,null);
            }
        }else if("content".equalsIgnoreCase(uri.getScheme())){
            //如果时content类型的Uri,则使用普通方式处理
            imagePath=getImagePath(uri,null);
        }else if("file".equalsIgnoreCase(uri.getScheme())){
            //如果是file类型的Uri，直接获取图片路径即可
            imagePath=uri.getPath();
        }
        displayImage(imagePath);
    }

    @SuppressLint("Range")
    private String getImagePath(Uri uri, String selection){
        String path=null;
        //通过uri和 selection 来获取真实图片路径
        Cursor cursor=getContentResolver().query(uri,null,selection,null,null,null);
        if(cursor!=null){
            if(cursor.moveToFirst()){
                path=cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
            }
            cursor.close();
        }
        return  path;
  }

  private void displayImage(String imagePath){
        if(imagePath!=null) {
            Bitmap bitmap=BitmapFactory.decodeFile(imagePath);
            picture.setImageBitmap(bitmap);
        }else{
            Toast.makeText(this, "Failed to gei image", Toast.LENGTH_SHORT).show();
        }
  }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button takePhoto=(Button) findViewById(R.id.take_photo);
        picture=(ImageView) findViewById(R.id.picture);
        takePhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //创建File 对象，用于存储拍照后的图片 将其存放在手机SD卡的应用关联缓存目录下
                File outputImage=new File(getExternalCacheDir(),"output_image.jpg");
                try {
                    if(outputImage.exists()){
                        outputImage.delete();
                    }
                    outputImage.createNewFile();
                }catch (IOException e){
                    e.printStackTrace();
                }
                if(Build.VERSION.SDK_INT>=24){
                    imageUri= FileProvider.getUriForFile(MainActivity.this,
                            "com.example.cameraalbumtest.fileprovider",outputImage);
                }else {
                    imageUri=Uri.fromFile(outputImage);
                }
                //启动相机程序
                Intent intent=new Intent("android.media.action.IMAGE_CAPTURE");
                intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri);
                startActivityForResult(intent,TAKE_PHOTO);
            }
        });
        Button chooseFromAlbum=(Button) findViewById(R.id.choose_from_album);
        chooseFromAlbum.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //先进行权限申请，运行时权限申请 WRITE_EXTERNAL_STORAGE
                if(ContextCompat.checkSelfPermission(MainActivity.this,
                        Manifest.permission.WRITE_EXTERNAL_STORAGE)!=
                        PackageManager.PERMISSION_GRANTED){
                    ActivityCompat.requestPermissions(MainActivity.this,
                            new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
                }else{
                    openAlbum();
                }
            }
        });
    }
    //
    public void openAlbum(){
        Intent intent=new Intent("android.intent.action.GET_CONTENT");
        intent.setType("image/*");
        startActivityForResult(intent,CHOOSE_PHOTO);//打开相册
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length >= 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    openAlbum();
                } else {
                    Toast.makeText(this, "You denied the permission", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
                break;
        }
    }
}


```



#### bug

```
活动之间通信异常 
Caused by: java.lang.ArrayIndexOutOfBoundsException: length=1; index=1

数组越界
java.lang.ArrayIndexOutOfBoundsException: length=1; index=1
 id 分解错误 
  String id =docId.split(";")[1];//解析出数字格式的id 
  这句话出来问题 索引为1 数组也为1
 解决 
  String id =docId.split(":")[1];//解析出数字格式的id
   符号打错 
```

## 8.4 播放多媒体文件 

### 8.4.1  播放音频

``` 
setDataSource() 设置要播放的音频文件位置
prepare（） 在开始播放器之前调用这个方法完成准备工作 
start 开始或继续播放音频 
pause 暂停播放
reset 将MediaPlayer对象重置到刚刚创建的状态 
seekTo 从指定的位置开始播放音频 
stop 停止播放音频，调用这个方法后的对象无法在播放音频
relea 释放掉与Mediaplayer 对象相关的资源
isPlaying 判断当前MediaPlayer是否正在播放 
getDuration 获取载入音频文件的时长 
```

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private MediaPlayer mediaPlayer=new MediaPlayer();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button play=(Button) findViewById(R.id.play);
        Button pause=(Button) findViewById(R.id.pause);
        Button stop=(Button) findViewById(R.id.stop);
        play.setOnClickListener(this);
        pause.setOnClickListener(this);
        stop.setOnClickListener(this);
        //动态权限申请 
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.
                WRITE_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(MainActivity.this,new String[]{
                    Manifest.permission.WRITE_EXTERNAL_STORAGE
            },1);

        }else {
            initMediaPlayer();//初始化MediaPlay
        }
    }
    private void initMediaPlayer(){
        try {
        //需要提前在根目录下放一个music.mp3的文件 才能使用 
        File file= new File(Environment.getExternalStorageDirectory(),"music.mp3");
            mediaPlayer.setDataSource(file.getPath());//指定音频文件路径
            mediaPlayer.prepare();//让MediaPlayer到准备状态
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    initMediaPlayer();
                } else {
                    Toast.makeText(this, "拒绝权限无法使用应用程序", Toast.LENGTH_SHORT).show();
                    finish();
                }
                break;
            default:
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.play:
                if(!mediaPlayer.isPlaying()){
                    mediaPlayer.start();
                }
                break;
            case R.id.pause:
                if(mediaPlayer.isPlaying()){
                    mediaPlayer.pause();
                }
                break;
            case R.id.stop:
                if(mediaPlayer.isPlaying()){
                    mediaPlayer.reset();
                    initMediaPlayer();
                }
            break;
            default:
                break;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(mediaPlayer!=null){
            mediaPlayer.stop();
            mediaPlayer.release();
        }
    }
}
```

```
声明用到的权限
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

### 8.4.2 播放视频

```
VideoVide 类
setVideoPath 设置要播放的视频文件位置 
start  开始或者继承
pause 暂停 
resume 视频从头播放 
seekTo 从指定位置播放 
isPlaying 当前是否正在播放
getDuration 获取载入的视频文件时长
```

```
package com.example.playvideotest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.os.Environment;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;
import android.widget.VideoView;

import java.io.File;

public class MainActivity extends AppCompatActivity implements View.OnClickListener{
    private VideoView videoView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button play=(Button) findViewById(R.id.play);
        Button pause=(Button) findViewById(R.id.pause);
        Button replay=(Button) findViewById(R.id.replay);
        play.setOnClickListener(this);
        pause.setOnClickListener(this);
        replay.setOnClickListener(this);
        //动态权限申请
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest.permission.
                WRITE_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(MainActivity.this,new String[]{
                    Manifest.permission.WRITE_EXTERNAL_STORAGE
            },1);

        }else {
            initvideoPath();//初始化videoPath
        }
    }
    private void initvideoPath(){
        File file=new File(Environment.getExternalStorageDirectory(),"movie.mp4");
        videoView.setVideoPath(file.getPath());
    }
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    initvideoPath();
                } else {
                    Toast.makeText(this, "拒绝权限无法使用应用程序", Toast.LENGTH_SHORT).show();
                    finish();
                }
                break;
            default:
        }
    }
    @Override
    public void onClick(View v) {
        switch (v.getId()){
            case R.id.play:
                if(!videoView.isPlaying()){
                    videoView.start();
                }
                break;
            case R.id.pause:
                if(videoView.isPlaying()){
                    videoView.pause();
                }
                break;
            case R.id.replay:
                if(videoView.isPlaying()){
                    videoView.resume();
                }
                break;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if(videoView!=null){
            videoView.suspend();
        }
    }
}
}
```

```
xxxxxxxxxx 声明用到的权限<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

#### bug

```
Bug
 Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'void android.widget.VideoView.setVideoPath(java.lang.String)' on a null object reference
         at com.example.playvideotest.MainActivity.initvideoPath(MainActivity.java:47)
        at com.example.playvideotest.MainActivity.onCreate(MainActivity.java:41) 
       没有引用这个控件的实例 所以是空引用
```


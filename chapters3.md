# chapters3

## 3.1 如何编写程序界面 

## 3.2 常用控件的使用

### 3.2.1 TextView 

​	主要用于在界面上显示一段文本信息

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="This is a TextView"
        android:gravity="center" //指定对齐方式 top bottom left right center
        android:textSize="24sp"//指定文字大小，android使用sp作为单位
        android:textColor="#00ff00"//指定文字颜色
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</LinearLayout>
```

### 3.2.2 Button

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"

    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/text_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="This is a TextView"
        android:gravity="center"
        android:textSize="24sp"
        android:textColor="#00ff00" />
    <Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Button"/>//系统会对Button的所有英文字母自动进行大写转换
        android:textAllCaps="false"// 禁用上述特性

</LinearLayout>
```

### 3.2.3 EditText

​	允许用户在控件里输入和编辑内容，并可以在程序中对这些内容进行处理

```
    <EditText
        android:id="@+id/edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Type something here"//指定了一段提示性的文本
        android:maxLines="2"/>// 指定了EditText的最大行数为两行 
	EditText可以和Button混合使用
	点击以下按钮，会把输入的东西弹窗出来
	public class MainActivity extends AppCompatActivity {
    private EditText editText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button=(Button) findViewById(R.id.button);
        editText=(EditText) findViewById(R.id.edit_text);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                switch (view.getId()){
                    case R.id.button:
                        String inputText=editText.getText().toString();
                         Toast.makeText(MainActivity.this, inputText,
                            Toast.LENGTH_SHORT).show();
                            break;
                    default:
                             break;
                }

            }
        });
    }
}
```

### 3.2.4 ImageView

​	用于显示图片，图片放在drawable开头下的目录

```
    <ImageView
        android:id="@+id/image_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/img_1"/>// 用于指定图片 
     还可在代码里进行修改
     private ImageView imageView;
     imageView=(ImageView) findViewById(R.id.image_view);
     imageView.setImageResource(R.drawable.img_2);
     通过setImageResource（）方法更改显示的图片
```

### 3.2.5 ProgressBar

 用于在界面上显示一个进度条，表示程序正在加载一些数据。

​	所有的Android 控件都具备可见属性

```
android:visbility  有三个可选值 
 					visible  可见的 默认值
 					invisible 不可见的 但仍然占据着原来的位置和大小，控件成透明了
 					gone 控件不可见，并且不占用任何屏幕空间
		还可通过代码设置
			setVisibility() 可传入View.VISIBLE View.INVISIBLE View.GONE
			private ProgressBar progressBar;
			progressBar=(ProgressBar) findViewById(R.id.progress_bar);
			      case R.id.button:
                        if(progressBar.getVisibility()==View.GONE){
                            progressBar.setVisibility(View.VISIBLE);
                        }else {
                            progressBar.setVisibility(View.GONE);
                        }
                        break;
                    default:
                        break;
   <ProgressBar
        android:id="@+id/progress_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        style="?android:attr/progressBarStyleHorizontal" 将其设置成水平进度条
        android:max="100"/> 进度条最大值100
        // 代码每点一次BUTTON 进度条加10
         case R.id.button:
                   int progress=progressBar.getProgress();
                   progress=progress+10;
                   progressBar.setProgress(progress);
                        break;
                    default:
                        break;
```

### 3.2.6 AlertDialog

​	 当前界面弹出一个对话框，这个对话框置顶于所有元素之上，能够屏蔽嗲篇其他控件的交互能力。一般用于提示重要信息以及警告信息

```
     public void onClick(View view) {
           switch (view.getId()){
              case R.id.button:
               AlertDialog.Builder dialog=
                          new AlertDialog.Builder(MainActivity.this);
                  dialog.setTitle("This is a Dialog");
                  dialog.setMessage("Something important:");
                  dialog.setCancelable(false);//是否可用Back键关闭对话框
                  dialog.setPositiveButton("OK", new DialogInterface.
                     OnClickListener() {
                       @Override
                  public void onClick(DialogInterface dialogInterface, int i) {
                             
                            }
                        });
                 dialog.setNegativeButton("cancel", new DialogInterface.
                   OnClickListener() {
                    @Override
                   public void onClick(DialogInterface dialogInterface, int i) {
                                
                            }
                        });
                        dialog.show();
                        break;
                    default:
                        break;
                }
           }
       首先通过 AlertDialog.Builder（）创建出一个AlertDialog实例，然后为这个对话框设置标题，内容，可用Back键关闭对话框等属性，随后调用setPositiveButton()为对话框设置确定按钮的点击事件，调用setNegativeButton()方法设置取消按钮的点击事件，最后调show()方法将对话框显示出来。
```

### 3.2.7 ProgressDialog

​	弹出一个对话框，并显示一个进度条，表示当前操作耗时，让用户耐心等待。也是直接修改MainActivity中的代码

```
                    case R.id.button:
                        ProgressDialog progressDialog =new ProgressDialog
                                (MainActivity.this);
                        progressDialog.setTitle("This is ProgessDialog");
                        progressDialog.setMessage("Loading");
                        progressDialog.setCancelable(true);
                        progressDialog.show();
                        break;
                }
```

## 3.3 4种基本布局

### 3.3.1 线性布局 LinerLayout

​	将其包含的所有控件在线性方向上依次排列。

```
android:orientation="" 用于指定排列方向时"vetrical" 垂直 还是"horizontal" 水平
	如果不指定值得花，默认排列方向水平
android:gravity 用于指定文字在控件中的对齐方式
android:layout_gravity 用于指定控件在布局中的对齐方式
				 但只有当排列方式时水平时，垂直方向的对齐方式才会生效 通过水平方向对齐一样
android:layout_weight 允许我们使用比例的方式指定控件的大小

    <EditText
        android:id="@+id/intput_message"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:hint="Type something"/>
    <Button

        android:id="@+id/button2"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:text="Send" />
    dp用于指定控件大小，间距的计量范围。
    android:layout_weight="1" 表示EditText和Button将在水平方向平分宽度
    
```

### 3.3.2 相对布局 RelativeLayout

​	可以通过相对定位让控件出现在布局的任何位置

```
<RelativeLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button

        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:text="Button1" /> 左上
    <Button

        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_alignParentTop="true"
        android:text="Button2" />右上
    <Button

        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Button3" />中间
    <Button

        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentLeft="true"
        android:text="Button4" />左下 
    <Button

        android:id="@+id/button45"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_alignParentRight="true"
        android:text="Button5" />右下


</RelativeLayout>
```

```
<RelativeLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button

        android:id="@+id/button3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="Button3" />
    <Button

        android:id="@+id/button1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@+id/button3" //一个控件位于另一个控件上方
        android:layout_toLeftOf="@id/button3"//一个控件位于另一个控件左方
        android:text="Button1" />
    <Button

        android:id="@+id/button2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@+id/button3"
        android:layout_toRightOf="@id/button3"//一个控件位于另一个控件右方
        android:text="Button2" />
    <Button

        android:id="@+id/button4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button3"//一个控件位于另一个控件下方
        android:layout_toLeftOf="@id/button3"
        android:text="Button4" />
    <Button

        android:id="@+id/button45"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/button3"
        android:layout_toRightOf="@id/button3"
        android:text="Button5" />


</RelativeLayout>
```

```
android:layout_alignLeft="@id/" //一个控件的左边缘和另一个控件的左边缘对齐
android:layout_alignRight="@id/" //一个控件的左边缘和另一个控件的右边缘对齐
```

### 3.3.3 帧布局 	FrameLayout

​	所有控件默认摆放在布局的左上角，会出现叠加

```

```

### 3.3.4 百分比布局 

​	可直接指定控件在布局中所占的百分比，主要为相对布局和帧布局进行功能扩展

​	将百分比布局定义在了support 库中

#### 	PercentFrameLayout 

#### 	PercentReleativeLayout 

```
<androidx.percentlayout.widget.PercentFrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">


    <Button
        android:id="@+id/btn1"
        android:layout_gravity="left|top"
        android:text="btn1"
        android:textAllCaps="false"
        app:layout_heightPercent="50%"//各按钮的高度为布局的50%
        app:layout_widthPercent="50%" />//各按钮的宽度为布局的50%

    <Button
        android:id="@+id/btn2"
        android:layout_gravity="right|top"
        android:text="btn2"
        android:textAllCaps="false"
        app:layout_heightPercent="50%"
        app:layout_widthPercent="50%" />
    <Button
        android:id="@+id/btn3"
        android:layout_gravity="left|bottom"
        android:text="btn3"
        android:textAllCaps="false"
        app:layout_heightPercent="50%"
        app:layout_widthPercent="50%" />

    <Button
        android:id="@+id/btn4"
        android:text="btn4"
        android:layout_gravity="right|bottom"
        app:layout_widthPercent="50%"
        app:layout_heightPercent="50%"/>


</androidx.percentlayout.widget.PercentFrameLayout>

```

## 3.4 自定义控件 

### 3.4.1引入布局

​	

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/title_bg">//用来给控件设置背景
    <Button
        android:id="@+id/title_back"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"//控件的偏移距离
        android:background="@drawable/back_bg"
        android:text="Back"
        android:textColor="#fff"/>
    <TextView
        android:id="@+id/title_text"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_weight="1"
        android:gravity="center"
        android:text="Title Text"
        android:textColor="#fff"
        android:textSize="24sp"/>
    <Button
        android:id="@+id/title_edit"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_margin="5dp"
        android:background="@drawable/edit_bg"
        android:text="Edit"
        android:textColor="#fff"/>
        

</LinearLayout>
```

### 3.4.2 自定义控件

```
package com.example.uicustonviews;

import android.app.Activity;
import android.content.Context;
import android.util.AttributeSet;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.Button;
import android.widget.LinearLayout;
import android.widget.Toast;

public class TitleLayout extends LinearLayout {
    // 重写了LinearLayout 带有两个参数的构造函数 ，在布局中引入TitleLayout就会调用这个构造函数

    public TitleLayout(Context context, AttributeSet attrs){
        super(context,attrs);
        LayoutInflater.from(context).inflate(R.layout.title,this);
        Button titleBack=(Button) findViewById(R.id.title_back);
        Button titleExit=(Button) findViewById(R.id.title_edit);
        titleBack.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                ((Activity)getContext()).finish();
            }
        });
        titleExit.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View view) {
                Toast.makeText(getContext(), "You clicked Edit button",
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
}

```

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"

    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <com.example.uicustonviews.TitleLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</LinearLayout>
```

## 3.5 ListView

### 3.5.1 简单用法

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

```
ArrayAdapter<> 适配器  通过泛型指定适配的数据类型，在构造函数中要把要适配的数据传入
package com.example.listview;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;

public class MainActivity extends AppCompatActivity {
    private String[] data = {"Apple", "Banana", "Orange", "Watermelon",
            "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango",
            "Apple", "Banana", "Orange", "Watermelon",
            "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango",};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ArrayAdapter<String>adapter=new ArrayAdapter<String>(
                MainActivity.this, android.R.layout.simple_list_item_1,data);
        ListView listView =(ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);
    }
}
ArrayAdapter<String> （）第一个参数 当前上下文 
						第二个参数 ListView子项布局的ID
						第三个参数适配的数据 
setAdapter（） 将构建好的适配器对象传递进去
```

### 3.5.2 定制ListView的界面

  1 定义实体类，作为ListView适配器的适配类型

```
package com.example.listview;

public class Fruit {
    private  String name;//表示水果名字 
    private  int imageId;//imageId 表示对应的图片资源
    public Fruit(String name,int imageId){
        this.name = name;
        this.imageId=imageId;
    }
    public String getName(){
        return name;
    }
    public int getImageId(){
        return imageId;
    }
}
```

 2 为ListView的子项指定一个自定义的布局,新建 fruit_item.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">
    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

3 创建一个自定义的适配器，继承ArrayAdapter,泛型指定为Fruit，新建适配器FruitAdapter

```
package com.example.listview;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;

import java.util.List;

public class FruitAdapter extends ArrayAdapter<Fruit> {
    private int resourceId;
    public FruitAdapter(Context context, int testViewResourcedId,
                        List<Fruit>objects){
        super(context,testViewResourcedId,objects);
        resourceId=testViewResourcedId;
    }//重写了构造函数，用于将上下文，ListView子项布局的id和数据都传递进来 

    // 重写了 getView()方法，首先通过getItem（）获得当前项的Fruit实例，然后使用LayoutInflater来为这个子项加载我们传入的布局 
    LayoutInflater的inflate（）接受三个参数 
			resource 要加载的xml资源id

			root   attachToRoot为true时，表示View的父容器
				attachToRoot为false时，单纯地返回一系列View的布局参数对象
        		attachToRoot  是否将View添加到root父容器中

            如果是false，root只用来生成正确的关于xml中父容器的关于布局参数的子类

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        Fruit fruit=getItem(position);//获取当前项的Fruit实例
        View view= LayoutInflater.from(getContext()).inflate(resourceId,parent,false);
       //调用View的findViewById 获取到 ImageView和TextView的实例，并分别调用他们的setImageResource和setText设置显示的图片和文字，最后将布局返回。完成自定义的适配器
       ImageView fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
        TextView fruitName=(TextView)  view.findViewById(R.id.fruit_name);
        fruitImage.setImageResource(fruit.getImageId());//设置显示的图片
        fruitName.setText(fruit.getName());//设置显示的文字
        return view;
    }
}

```

 4 在MainActivity中添加一个initFruits（）方法，用于初始化所有的水果数据 。在onCreate中创建FruitAdapter对象，并将FruitAdapter作为适配器传递给Listview

```
package com.example.listview;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    private List <Fruit> fruitList=new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initFruits();
        FruitAdapter adapter=new FruitAdapter(MainActivity.this,R.layout.fruit_item,fruitList);

        /*ArrayAdapter<String>adapter=new ArrayAdapter<String>(
                MainActivity.this, android.R.layout.simple_list_item_1,data);*/
        ListView listView =(ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);//搭建好的适配器对象传递进去，ListView与数据之间的关联就做好了
    }
    private void initFruits(){
        for(int i=0;i<4;i++){
            Fruit apple= new Fruit("apple",R.drawable.apple_pic);
            fruitList.add(apple);
            Fruit banana= new Fruit("banana",R.drawable.banana_pic);
            fruitList.add(banana);
            Fruit orange= new Fruit("orange",R.drawable.orange_pic);
            fruitList.add(orange);
            Fruit watermelon= new Fruit("watermelon",R.drawable.watermelon_pic);
            fruitList.add(watermelon);
            Fruit pear= new Fruit("pear",R.drawable.pear_pic);
            fruitList.add(pear);
            Fruit grape= new Fruit("grape",R.drawable.grape_pic);
            fruitList.add(grape);
            Fruit pineapple= new Fruit("pineapple",R.drawable.pineapple_pic);
            fruitList.add(pineapple);
            Fruit strawberry= new Fruit("strawberry",R.drawable.strawberry_pic);
            fruitList.add(strawberry);
            Fruit cherry= new Fruit("cherry",R.drawable.cherry_pic);
            fruitList.add(cherry);
            Fruit mango= new Fruit("mango",R.drawable.mango_pic);
            fruitList.add(mango);
        }
    }
}
```

### 3.5.3 提升ListView的运行效率

```
public View getView(int position,
					@Nullable View convertView, //之前加载的布局进行缓存，以方便后续使用
					@NonNull ViewGroup parent) 
```

​	修改FruitAdapter代码

```
    @Override
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        Fruit fruit=getItem(position);//获取当前项的Fruit实例
        View view;
        if(convertView==null){
            view=LayoutInflater.from(getContext()).
            inflate(resourceId,parent,false);//加载布局 
        }else {
            view=convertView;
        }
       //调用View的findViewById 获取到 ImageView和TextView的实例，并分别调用他们的setImageResource和setText设置显示的图片和文字，最后将布局返回。完成自定义的适配器
       ImageView fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
        TextView fruitName=(TextView)  view.findViewById(R.id.fruit_name);
        fruitImage.setImageResource(fruit.getImageId());//设置显示的图片
        fruitName.setText(fruit.getName());//设置显示的文字
        return view;
    }
}

```

​	上述代码不会重复加载布局，但每次用getView()方法还会调用View的findViewById()方法获取一次控件的实例。可以借助一个ViewHolder对这部分性能进行优化 

```
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        Fruit fruit=getItem(position);//获取当前项的Fruit实例
        View view;
        ViewHolder viewHolder;
        if(convertView==null){
            view=LayoutInflater.from(getContext()).
                    inflate(resourceId,parent,false);
             //如果当前实例不存在，加载布局,将控件的实例存放在viewholder中，
             //调用view.setTag（）方法，将viewholder对象存储在view中
            viewHolder =new ViewHolder() ;
            viewHolder.fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
            viewHolder.fruitName=(TextView) view.findViewById(R.id.fruit_name);
            view.setTag(viewHolder);
        }else {
        //如果当前实例存在，之前调用convertView，不重现加载布局
        //调用 view.getTag()取出将viewholder对象取出，、
        //避免了每次都通过View.findViewById()获取控件实例
            view=convertView;
            viewHolder=(ViewHolder)  view.getTag();
        }
        ImageView fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
        TextView fruitName=(TextView)  view.findViewById(R.id.fruit_name);
        fruitImage.setImageResource(fruit.getImageId());
        fruitName.setText(fruit.getName());
        return view;
    }
    //创建一个 ViewHolder类，用于保存控件实例
    class ViewHolder{
        ImageView fruitImage;
        TextView fruitName;
    }
```

### 3.5.4 ListView的点击事件

​	在MainAciivity中增加点击事件代码

```
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initFruits();
        FruitAdapter adapter=new FruitAdapter(MainActivity.
                this,R.layout.fruit_item,fruitList);
        ListView listView =(ListView) findViewById(R.id.list_view);
        listView.setAdapter(adapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent,
                                    View view, int position, long id) {
                Fruit fruit =fruitList.get(position);//获取用户点击的是哪一个子项
                Toast.makeText(MainActivity.this, fruit.getName(),
                        Toast.LENGTH_SHORT).show();
            }
        });
    }
```

## 3.6 RecycleView

### 3.6.1 基本用法

```
build.grade文件中导入依赖库
    implementation 'androidx.recyclerview:recyclerview:1.1.0'
XML文件中加入控件 
	<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recycle_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

</LinearLayout>
```

​	创建适配器FruitAdapter，继承Recycle.Adapter，并将泛型指定为FruitAdapter.ViewHolder

```
package com.example.recyclerviewtest;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {
    private List<Fruit>mFruitList;
    //ViewHolder 自定义的类 继承自RecyclerView.ViewHolder
    public class ViewHolder extends RecyclerView.ViewHolder{
        ImageView fruitImage;
        TextView fruitName;
        //构造函数中传入一个View参数，这个参数通常为RecycleView子项的最外参数
        // 利用View.findViewById获取布局中的ImageView和TextView实例
        public ViewHolder(View view){
            super(view);
            fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
            fruitName=(TextView) view.findViewById(R.id.fruit_name);
        }
    }
    //FruitAdapter 构造函数，传入一个List<Fruit>参数，并赋值给全局变量mFruitList
    public FruitAdapter(List<Fruit>fruitList) {
        mFruitList=fruitList;
    }

    @NonNull
    //重写onCreateViewHolder（）方法 用于创建ViewHolder实例，加载的布局传入，返回一个ViewHolder实例
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view= LayoutInflater.from(parent.getContext())
                .inflate(R.layout.fruit_item,parent,false);//加载布局
        ViewHolder holder=new ViewHolder(view);
        return holder;
    }
    // onBindViewHolder 方法对RecyclerView的子项进行赋值，通过mFruitList.get(position)获取当前滚动的Fruit实例
    // 随后将数据设置给ImageView和TextView
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Fruit fruit=mFruitList.get(position);
        holder.fruitImage.setImageResource(fruit.getImageId());
        holder.fruitName.setText(fruit.getName());
    }
    //getItemCount 获取RecyclerView一共有多少子项，直接返回数据长度即可
    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}

```

​	修改MainActivity中代码

```
package com.example.recyclerviewtest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.os.Bundle;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    private List<Fruit> fruitList=new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initFruits();
        RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycle_view);
        //LayoutManager 用于指定RecyclerView的布局方式，这里使用LinearLayoutManager 线性布局
        //和ListView类似
        LinearLayoutManager layoutManager=new LinearLayoutManager(this);
        recyclerView.setLayoutManager(layoutManager);
        FruitAdapter adapter=new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);
    }
    private void initFruits() {
        for (int i = 0; i < 4; i++) {
            Fruit apple = new Fruit("apple", R.drawable.apple_pic);
            fruitList.add(apple);
            Fruit banana = new Fruit("banana", R.drawable.banana_pic);
            fruitList.add(banana);
            Fruit orange = new Fruit("orange", R.drawable.orange_pic);
            fruitList.add(orange);
            Fruit watermelon = new Fruit("watermelon", R.drawable.watermelon_pic);
            fruitList.add(watermelon);
            Fruit pear = new Fruit("pear", R.drawable.pear_pic);
            fruitList.add(pear);
            Fruit grape = new Fruit("grape", R.drawable.grape_pic);
            fruitList.add(grape);
            Fruit pineapple = new Fruit("pineapple", R.drawable.pineapple_pic);
            fruitList.add(pineapple);
            Fruit strawberry = new Fruit("strawberry", R.drawable.strawberry_pic);
            fruitList.add(strawberry);
            Fruit cherry = new Fruit("cherry", R.drawable.cherry_pic);
            fruitList.add(cherry);
            Fruit mango = new Fruit("mango", R.drawable.mango_pic);
            fruitList.add(mango);
        }
    }

}
```

### 3.6.2 实现横向滚动和瀑布流布局

#### LinearLayoutManager  横向滚动	 

1 修改fruit_item.xml 将水平排列改为垂直排列

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="100dp"//设定布局宽度为 100dp 
    android:layout_height="wrap_content">
    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"/>
    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"//布局水平居中
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

​	2 修改MainActivity中的代码

```
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       initFruits();
       RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycle_view);
       //LayoutManager 用于指定RecyclerView的布局方式，这里使用LinearLayoutManager 线性布局
       //和ListView类似
       LinearLayoutManager layoutManager=new LinearLayoutManager(this);
       //调用LinearLayoutManager.setOrientation（）设置布局排列方向，默认纵向，
       //LinearLayoutManager.HORIZONTAL 表示横向排列
       layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
       recyclerView.setLayoutManager(layoutManager);
       FruitAdapter adapter=new FruitAdapter(fruitList);
       recyclerView.setAdapter(adapter);
   }
```

#### StaggeredGridLayoutManager 瀑布流布局 

​	1 修改xml代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    //宽度改为match_parent 瀑布流布局的宽度应该是根据布局的列数自动适配的，不是一个固定值
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    // 每个子项之间
    android:layout_margin="5dp">
    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"/>
    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

​	 2 修改MainActivity代码

​			瀑布流布局需要各个子项高度不一致才能看出明显的效果

```
   //瀑布流布局
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       initFruits();
       RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycle_view);
       //LayoutManager 用于指定RecyclerView的布局方式，
       //创建了一个StaggeredGridLayoutManager实例，
       // 构造函数会接受两个参数，第一个指定布局的列数，第二各指定布局的排列方向
       //                                   StaggeredGridLayoutManager.VERTICAL 布局纵向排列
       StaggeredGridLayoutManager layoutManager=new
               StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL);
       recyclerView.setLayoutManager(layoutManager);
       FruitAdapter adapter=new FruitAdapter(fruitList);
       recyclerView.setAdapter(adapter);
   }
    private void initFruits() {
        for (int i = 0; i < 4; i++) {
            Fruit apple = new Fruit(getRandomLengthName("apple"), R.drawable.apple_pic);
            fruitList.add(apple);
            Fruit banana = new Fruit(getRandomLengthName("banana"), R.drawable.banana_pic);
            fruitList.add(banana);
            Fruit orange = new Fruit(getRandomLengthName("orange"), R.drawable.orange_pic);
            fruitList.add(orange);
            Fruit watermelon = new Fruit(getRandomLengthName("watermelon"), R.drawable.watermelon_pic);
            fruitList.add(watermelon);
            Fruit pear = new Fruit(getRandomLengthName("pear"), R.drawable.pear_pic);
            fruitList.add(pear);
            Fruit grape = new Fruit(getRandomLengthName("grape"), R.drawable.grape_pic);
            fruitList.add(grape);
            Fruit pineapple = new Fruit(getRandomLengthName("pineapple"), R.drawable.pineapple_pic);
            fruitList.add(pineapple);
            Fruit strawberry = new Fruit(getRandomLengthName("strawberry"), R.drawable.strawberry_pic);
            fruitList.add(strawberry);
            Fruit cherry = new Fruit(getRandomLengthName("cherry"), R.drawable.cherry_pic);
            fruitList.add(cherry);
            Fruit mango = new Fruit(getRandomLengthName("mango"), R.drawable.mango_pic);
            fruitList.add(mango);
        }
    }
    private String getRandomLengthName(String name){
        Random random =new Random();
        int length=random.nextInt(20)+1;
        StringBuilder builder =new StringBuilder();
        for(int i=0;i<length;i++){
            builder.append(name);
        }
        return builder.toString();
```

#### GridLayoutManager 网格布局 

 	1 修改xml布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="50dp"
    android:layout_height="wrap_content"
    android:layout_margin="5dp">
    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"/>
    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:layout_marginLeft="10dp"/>

</LinearLayout>
```

​	2 MainActivity代码修改

```
  @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
       initFruits();
       RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycle_view);
       //LayoutManager 用于指定RecyclerView的布局方式，
       //创建了一个GridLayoutManager实例，
       // 构造函数会接受两个参数，第一个上下文，第二各指定布局的列数
       //                                  StaggeredGridLayoutManager.VERTICAL 布局纵向排列
       GridLayoutManager layoutManager=new
               GridLayoutManager(MainActivity.this,4);
       layoutManager.setOrientation(RecyclerView.HORIZONTAL);//设置横向移动
       recyclerView.setLayoutManager(layoutManager);
       FruitAdapter adapter=new FruitAdapter(fruitList);
       recyclerView.setAdapter(adapter);
   }
```

### 3.6.3 点击事件

​	recyclerView没有提供监听器，需要我们给子项具体的View注册点击事件

​	 1 修改适配器FruitAdapter中的代码

```
package com.example.recyclerviewtest;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;

import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {
    private List<Fruit>mFruitList;
    //ViewHolder 自定义的类 继承自RecyclerView.ViewHolder
    // 在ViewHolder中添加了fruitView变量包窜子项最外层布局的实例
    //随后在 onCreateViewHolder注册实例
    public class ViewHolder extends RecyclerView.ViewHolder{
        View fruitView;
        ImageView fruitImage;
        TextView fruitName;
        //构造函数中传入一个View参数，这个参数通常为RecycleView子项的最外参数
        // 利用View.findViewById获取布局中的ImageView和TextView实例
        public ViewHolder(View view){
            super(view);
            fruitView=view;
            fruitImage=(ImageView) view.findViewById(R.id.fruit_image);
            fruitName=(TextView) view.findViewById(R.id.fruit_name);
        }
    }
    //FruitAdapter 构造函数，传入一个List<Fruit>参数，并赋值给全局变量mFruitList
    public FruitAdapter(List<Fruit>fruitList) {
        mFruitList=fruitList;
    }

    @NonNull
    //重写onCreateViewHolder（）方法 用于创建ViewHolder实例，加载的布局传入，返回一个ViewHolder实例
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view= LayoutInflater.from(parent.getContext())
                .inflate(R.layout.fruit_item,parent,false);//加载布局
        //随后在 onCreateViewHolder注册点击事件
       final ViewHolder holder=new ViewHolder(view);
       holder.fruitView.setOnClickListener(new View.OnClickListener() {
           @Override
           public void onClick(View v) {
               int position =holder.getAdapterPosition();
               Fruit fruit= mFruitList.get(position);
               Toast.makeText(v.getContext(), "you clicked view "+fruit.getName(),
                       Toast.LENGTH_SHORT).show();
           }
       });
       holder.fruitImage.setOnClickListener(new View.OnClickListener() {
           @Override
           public void onClick(View v) {
               int position =holder.getAdapterPosition();
               Fruit fruit= mFruitList.get(position);
               Toast.makeText(v.getContext(), "you clicked image "+fruit.getName(),
                       Toast.LENGTH_SHORT).show();
           }
       });
        return holder;
    }
    // onBindViewHolder 方法对RecyclerView的子项进行赋值，通过mFruitList.get(position)获取当前滚动的Fruit实例
    // 随后将数据设置给ImageView和TextView
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Fruit fruit=mFruitList.get(position);
        holder.fruitImage.setImageResource(fruit.getImageId());
        holder.fruitName.setText(fruit.getName());
    }
    //getItemCount 获取RecyclerView一共有多少子项，直接返回数据长度即可
    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}

```

## 3.7编写界面的事件

### 3.7.1 制作Nine-Patch图片

​	在Android Studio中对任意图片点击右键，Create 9-Patch file就可创建

### 3.7.2 编写精美的聊天界面

​	1 修改acitivity.xml

​			设置成线性布局，加入recyclerview控件，并设置一个EditText输入消息，Button用于发送消息

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#d8e0e8">
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/input_text"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Type something here"
            android:maxLines="2"/>
        <Button
            android:id="@+id/send"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Send"/>
    </LinearLayout>


</LinearLayout>
```

​	2 新建Msg类  content表示消息的内容，type表示消息的类型，接受还是发送

```
package com.example.uibestpractice;

public class Msg {
   //这是一条收到的消息
    public static final int TYPE_RECEIVED=0;
    //这是一条发出的消息
    public static final int TYPE_SENT=1;
    //消息的内容
    private String content;
    //消息的类型
    private int type;
    public Msg(String content,int type){
        this.content=content;
        this.type=type;
    }

    public String getContent() {
        return content;
    }

    public int getType() {
        return type;
    }
}

```

​	3 新建自定义控件 msg_item.xml

​	收到的消息左对齐，发出的信息右对齐

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp">
    <LinearLayout
        android:id="@+id/left_layout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="left"
        android:background="@drawable/message_left">
        <TextView
           android:id="@+id/left_msg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="10dp"
            android:textColor="#fff"/>
    </LinearLayout>
    <LinearLayout
        android:id="@+id/right_layout"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:background="@drawable/message_right">
        <TextView
            android:id="@+id/right_msg"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:layout_margin="10dp" />
    </LinearLayout>
</LinearLayout>
```

 	 4 编写适配器 MsgAdapter 指定Msg作为泛型 ，继承RecyclerView.Adapter

```
package com.example.uibestpractice;

import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import java.util.List;

public class MsgAdapter extends RecyclerView.Adapter<MsgAdapter.ViewHolder> {
    private List<Msg> mMsgList;
    public class ViewHolder extends RecyclerView.ViewHolder{
        LinearLayout leftLayout;
        LinearLayout rightLayout;
        TextView leftMsg;
        TextView rightMsg;
        public ViewHolder(View view){
            super(view);
            leftLayout=(LinearLayout) view.findViewById(R.id.left_layout);
            rightLayout=(LinearLayout) view.findViewById(R.id.right_layout);
            leftMsg=(TextView) view.findViewById(R.id.left_msg);
            rightMsg=(TextView) view.findViewById(R.id.right_msg);
        }
    }
    public MsgAdapter(List<Msg> msgList){
        mMsgList=msgList;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view= LayoutInflater.from(parent.getContext())
                .inflate(R.layout.msg_item,parent,false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Msg msg=mMsgList.get(position);
        if(msg.getType()==Msg.TYPE_RECEIVED){
            //如果是收到的消息，则显示左侧的消息布局，隐藏右侧的
            holder.leftLayout.setVisibility(View.VISIBLE);
            holder.rightLayout.setVisibility(View.GONE);
            holder.leftMsg.setText(msg.getContent());
        }
        else if(msg.getType()==Msg.TYPE_SENT){
            //是发送的信息，显示右侧的，隐藏左侧的
            holder.rightLayout.setVisibility(View.VISIBLE);
            holder.leftLayout.setVisibility(View.GONE);
            holder.rightMsg.setText(msg.getContent());
        }
    }

    @Override
    public int getItemCount() {
        return mMsgList.size();
    }
}

```

​	5 编写MainActivity 

​	

```
package com.example.uibestpractice;

import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.LinearLayout;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    private List<Msg> msgList=new ArrayList<>();
    private EditText inputText;
    private Button send;
    private RecyclerView msgRecyclerView;
    private MsgAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //初始化聊天界面
        initMsgs();
        inputText=(EditText) findViewById(R.id.input_text);
        send=(Button) findViewById(R.id.send);
        msgRecyclerView=(RecyclerView) findViewById(R.id.recycler_view);
        LinearLayoutManager layoutManager =new LinearLayoutManager(this);
        msgRecyclerView.setLayoutManager(layoutManager);
        adapter=new MsgAdapter(msgList);
        msgRecyclerView.setAdapter(adapter);
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //获取EditText内容，即输入内容。
                String content= inputText.getText().toString();
                //内容不为空，新建一个Msg对象，添加到msgList中
                if(!"".equals(content)){
                    Msg msg=new Msg(content,Msg.TYPE_SENT);
                    msgList.add(msg);
                    //当有信息时，刷新RecyclerView中的显示
                    // adapter.notifyItemInserted（） 通知有新的数据插入
                    adapter.notifyItemInserted(msgList.size()-1);
                    //将RecyclerView定位到最后一行
                    //RecyclerView.scrollToPosition（） 将数据定位到最后一行
                    msgRecyclerView.scrollToPosition(msgList.size()-1);
                    //清空输入框的内容
                    inputText.setText("");//清空输入框的内容

                }
            }
        });
    }
    //舒适化数据用于显示
    private void initMsgs(){
        Msg msg1=new Msg("Hello guy",Msg.TYPE_RECEIVED);
        msgList.add(msg1);
        Msg msg2=new Msg("Hello, who is that",Msg.TYPE_SENT);
        msgList.add(msg2);
        Msg msg3=new Msg("This is Tom.Nice talking to you. ",Msg.TYPE_RECEIVED);
        msgList.add(msg3);
    }
}
```


# chapters 12

## 12.1 Material Design

## 12.2 Toolbar

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    
    <android.widget.Toolbar
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:id="@+id/toobal"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
        

</FrameLayout>
```

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        setSupportActionBar(toolbar);
    }

    private void setSupportActionBar(Toolbar toolbar) {
    }
    public boolean onCreateOptionsMenu(Menu menu){
        getMenuInflater().inflate(R.menu.toolbal,menu);
        return  true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You Clicked Delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.setting:
                Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show();
                break;
            default:

        }
        return true;
    }
}
```

## 12.3 滑动菜单 

### 12.3.1 DrawerLayout 

 一个布局 允许放入两个直接子控件 ，第一个子控件主屏幕显示内容，第二个子控件时滑动菜单显示内容  第二个子控件需要指定 android:layout_gravity="start" 告诉我们滑动窗口在屏幕左边还是右边 



```
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawer_layout"
    tools:context=".MainActivity">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <android.widget.Toolbar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/toobal"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
    </FrameLayout>
    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:text="This is menu"
        android:textSize="30sp"
        android:background="#FFF"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

```
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        setSupportActionBar(toolbar);
        mDrawerLayout=(DrawerLayout) findViewById(R.id.drawer_layout);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null) {
            // setDisplayHomeAsUpEnabled 将导航按钮显示出来
            actionBar.setDisplayHomeAsUpEnabled(true);
            // setHomeAsUpIndicator 导航按钮图标
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
        }
    }

    private void setSupportActionBar(Toolbar toolbar) {
    }
    public boolean onCreateOptionsMenu(Menu menu){
        getMenuInflater().inflate(R.menu.toolbal,menu);
        return  true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You Clicked Delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.setting:
                Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show();
                break;
                // HomeAsUp 按钮的id永远是 android.R.id.home
            case  android.R.id.home:
                // openDrawer 将滑动菜单显示出来
                mDrawerLayout.openDrawer(GravityCompat.START);
            default:

        }
        return true;
    }
}
```

### 12.3.2 NavigationView 

#### 依赖库

```
    def nav_version = "2.1.0"
    implementation "androidx.navigation:navigation-fragment:$nav_version"
    implementation "androidx.navigation:navigation-ui:$nav_version"
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
    implementation 'de.hdodenhof:circleimageview:3.0.0'
```

####  menu 用来在NavigationView 中显示具体菜单项 

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
/*
	group 代表组 
	single 代表组中的菜单只能单选 
*/
<group android:checkableBehavior="single">

    <item android:title="Call"
        android:id="@+id/nav_call"
        android:icon="@drawable/nav_call"
        />
    <item android:title="Friends"
        android:id="@+id/nav_friends"
        android:icon="@drawable/nav_friends"
        />
    <item android:title="Location"
        android:id="@+id/nav_location"
        android:icon="@drawable/nav_location"
        />
    <item android:title="Mail"
        android:id="@+id/nav_mail"
        android:icon="@drawable/nav_mail"
        />
    <item android:title="Task"
        android:id="@+id/nav_task"
        android:icon="@drawable/nav_task"
        />
</group>
</menu>
```

#### headerLayout 显示头部布局 

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="?attr/colorPrimary">
    //CircleImageView 是一个将图片圆形化的控件 
    <de.hdodenhof.circleimageview.CircleImageView
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:id="@+id/icon_image"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/mail"
        android:layout_alignParentBottom="true"
        android:text ="tonygreendev@gmail.com"
        android:textColor="#FFF"
        android:textSize="14sp"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/username"
        android:layout_above="@+id/mail"
        android:text ="Tony Green"
        android:textColor="#FFF"
        android:textSize="14sp"/>

</RelativeLayout>
```

修改主布局

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawer_layout"
    tools:context=".MainActivity">

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <android.widget.Toolbar
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:id="@+id/toobal"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
    </FrameLayout>
    <com.google.android.material.navigation.NavigationView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/nav_view"
        android:layout_gravity="start"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

####  修改菜单项的点击事件 

## 12.4 悬浮按钮和可交互提示 

​	立面设计 

### 12.4.1 FloatingActionButton 

```
        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/fab"
            //放置在右下角 
            android:layout_gravity="bottom|end"
            //  控件四周留点边距 
            android:layout_margin="16dp"
            android:src="@drawable/ic_done"
            // 悬浮高度
            app:elevation="8dp"
            />
```

```

```

```

fab.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Toast.makeText(MainActivity.this, "FAB clicked", Toast.LENGTH_SHORT).show();
    }
});
```

### 12.4.2 Snackbar

```
与Toast区别
Toast  告诉用户现在发生了什么，用户只能被动接受，无法更改 
Snackbar 允许在提示中加入一个可交互按钮，用户点击时执行一些额外逻辑操作
```

```
        FloatingActionButton fab=(FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //调用Snackbar.make 方法创建一个Snackebar 对象
                Snackbar.make(v,"Data deleted",Snackbar.LENGTH_SHORT)
                        //调用setAction 设置一个动作
                        .setAction("Undo", new View.OnClickListener() {
                            @Override
                            public void onClick(View v) {
                                Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
                            }
                        }).show();
            }
        });
```

### 12.4.3 CoordinatorLayout

​	可以监听其所有子控件的各种事件，然后自动帮助我们做出最为合理的响应 

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawer_layout"
    tools:context=".MainActivity">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <android.widget.Toolbar
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:id="@+id/toobal"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/fab"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:src="@drawable/ic_done"
            app:elevation="8dp"/>
    </androidx.coordinatorlayout.widget.CoordinatorLayout>
    <com.google.android.material.navigation.NavigationView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/nav_view"
        android:layout_gravity="start"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

## 12.5 卡片式布局 

### 12.5.1 CardView 

#### 依赖库

```
implementation 'androidx.cardview:cardview:1.0.0'
```

```
implementation 'androidx.recyclerview:recyclerview:1.1.0'
```

```
implementation 'com.github.bumptech.glide:glide:4.4.0'
```

​	glide 图片加载库 可以用于加载本地图片 网络图片 视频

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="1000dp"
        android:orientation="vertical">
        <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="1000dp"
        //指定图片的缩放模式
        android:scaleType="centerCrop"/>
         <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_margin="5dp"
             android:textSize="16sp"/>
    </LinearLayout>
</androidx.cardview.widget.CardView>
```

#### 为RecyclerView 指定适配器

```
package com.example.materialtest;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.cardview.widget.CardView;
import androidx.recyclerview.widget.RecyclerView;

import com.bumptech.glide.Glide;

import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {
    private  Context mContext;
    private List<Fruit>mFruitList;
    //ViewHolder 自定义的类 继承自RecyclerView.ViewHolder
    // 在ViewHolder中添加了fruitView变量包窜子项最外层布局的实例
    //随后在 onCreateViewHolder注册实例
    static class ViewHolder extends RecyclerView.ViewHolder{
        CardView cardView;
        ImageView fruitImage;
        TextView fruitName;
        //构造函数中传入一个View参数，这个参数通常为RecycleView子项的最外参数
        // 利用View.findViewById获取布局中的ImageView和TextView实例
        public ViewHolder(View view){
            super(view);
            cardView=(CardView) view;
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
        if(mContext==null){
            mContext=parent.getContext();
        }
        View view= LayoutInflater.from(mContext)
                .inflate(R.layout.fruit_item,parent,false);//加载布局
        return new ViewHolder(view);
    }
    // onBindViewHolder 方法对RecyclerView的子项进行赋值，通过mFruitList.get(position)获取当前滚动的Fruit实例
    // 随后将数据设置给ImageView和TextView
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Fruit fruit=mFruitList.get(position);
        holder.fruitName.setText(fruit.getName());
        
  //   Glide.with 传入一个参数，调用load 传入一个URL 地址    Glide.with(mContext).load(fruit.getImageId()).into(holder.fruitImage);
    }
    //getItemCount 获取RecyclerView一共有多少子项，直接返回数据长度即可
    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}
 
```

#### 修改活动代码

```
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;
    private Fruit[] fruits={new Fruit("Apple",R.drawable.apple),new Fruit("Banana",R.drawable.banana),
            new Fruit("Orange",R.drawable.orange),new Fruit("Watermelon",R.drawable.watermelon),
            new Fruit("Pear",R.drawable.pear),
            new Fruit("Grape",R.drawable.grape),new Fruit("Pineapple",R.drawable.pineapple),
            new Fruit("Strawberry",R.drawable.strawberry),
            new Fruit("Cherry",R.drawable.cherry), new Fruit("Mango",R.drawable.mango)};
    private List<Fruit> fruitList=new ArrayList<>();

    private FruitAdapter adapter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        setSupportActionBar(toolbar);
        mDrawerLayout=(DrawerLayout) findViewById(R.id.drawer_layout);
        NavigationView navView=(NavigationView) findViewById(R.id.nav_view);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null) {
            // setDisplayHomeAsUpEnabled 将导航按钮显示出来
            actionBar.setDisplayHomeAsUpEnabled(true);
            // setHomeAsUpIndicator 导航按钮图标
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
        }
       //setCheckedItem 将Call菜单设定为默认选中
        navView.setCheckedItem(R.id.nav_call);
        // 设置监听器 用户点击后 会回调到 onNavigationItemSelected
        navView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                mDrawerLayout.closeDrawers();
                return true;
            }
        });
        FloatingActionButton fab=(FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //调用Snackbar.make 方法创建一个Snackebar 对象
                Snackbar.make(v,"Data deleted",Snackbar.LENGTH_SHORT)
                        //调用setAction 设置一个动作
                        .setAction("Undo", new View.OnClickListener() {
                            @Override
                            public void onClick(View v) {
                                Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
                            }
                        }).show();
            }
        });

        initFruits();
        RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycler_view);
        GridLayoutManager layoutManager=new GridLayoutManager(this,2);
        recyclerView.setLayoutManager(layoutManager);
        adapter=new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);
    }

    private void initFruits() {
        fruitList.clear();
        for(int i=0;i<50;i++){
            Random random=new Random();
            int index=random.nextInt(fruits.length);
            fruitList.add(fruits[index]);
        }
    }

```



#### bug

```
Duplicate class android.support.v4.app.INotificationSideChannel found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.app.INotificationSideChannel$Stub found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.app.INotificationSideChannel$Stub$Proxy found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.IResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.IResultReceiver$Stub found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.IResultReceiver$Stub$Proxy found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.ResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.ResultReceiver$1 found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
Duplicate class android.support.v4.os.ResultReceiver$MyResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)

```

```
 
# 需要在gradle.properties中添加下面两行代码
# 这是因为混合支持库。通过添加这些行选择androidX作为您的支持库
 
android.useAndroidX=true   //一般这个项目中已经存在
android.enableJetifier=true

```

### 12.5.2 AppBarLayout 

  垂直方向的LinearLayout 做了很多滚动事件的封装 

可以用来recyclerview挡住Toolbar 

```
app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
避免recyclerview挡住Toolbar 
```

```
app:layout_scrollFlags="scroll|enterAlways|snap"
```

## 12.6 下拉刷新

SwipeRefreshLayout 

#### 依赖

```
 implementation "androidx.swiperefreshlayout:swiperefreshlayout:1.0.0"
```

```
 <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/swipe_refresh"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

                <androidx.recyclerview.widget.RecyclerView
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:id="@+id/recycler_view"
                    app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
        </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

```
        swipeRefresh=(SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        // 进度条颜色
        swipeRefresh.setColorSchemeColors(R.color.purple_500);
        // 设置一个下拉刷新的监听器，当触发了下拉刷新功能的时候回调refreshFruits();
        swipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                refreshFruits();
            }
        });
    }

    private void refreshFruits(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        initFruits();
                        adapter.notifyDataSetChanged();
                        swipeRefresh.setRefreshing(false);
                    }
                });
            }
        }).start();
    }
```

#### 水果刷新 完整代码

```
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;
    private Fruit[] fruits={new Fruit("Apple",R.drawable.apple),new Fruit("Banana",R.drawable.banana),
            new Fruit("Orange",R.drawable.orange),new Fruit("Watermelon",R.drawable.watermelon),
            new Fruit("Pear",R.drawable.pear),
            new Fruit("Grape",R.drawable.grape),new Fruit("Pineapple",R.drawable.pineapple),
            new Fruit("Strawberry",R.drawable.strawberry),
            new Fruit("Cherry",R.drawable.cherry), new Fruit("Mango",R.drawable.mango)};
    private List<Fruit> fruitList=new ArrayList<>();

    private FruitAdapter adapter;

    private SwipeRefreshLayout swipeRefresh;
    @SuppressLint("ResourceAsColor")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        setSupportActionBar(toolbar);
        mDrawerLayout=(DrawerLayout) findViewById(R.id.drawer_layout);
        NavigationView navView=(NavigationView) findViewById(R.id.nav_view);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null) {
            // setDisplayHomeAsUpEnabled 将导航按钮显示出来
            actionBar.setDisplayHomeAsUpEnabled(true);
            // setHomeAsUpIndicator 导航按钮图标
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
        }
       //setCheckedItem 将Call菜单设定为默认选中
        navView.setCheckedItem(R.id.nav_call);
        // 设置监听器 用户点击后 会回调到 onNavigationItemSelected
        navView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                mDrawerLayout.closeDrawers();
                return true;
            }
        });
        FloatingActionButton fab=(FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //调用Snackbar.make 方法创建一个Snackebar 对象
                Snackbar.make(v,"Data deleted",Snackbar.LENGTH_SHORT)
                        //调用setAction 设置一个动作
                        .setAction("Undo", new View.OnClickListener() {
                            @Override
                            public void onClick(View v) {
                                Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
                            }
                        }).show();
            }
        });

        initFruits();
        RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycler_view);
        GridLayoutManager layoutManager=new GridLayoutManager(this,2);
        recyclerView.setLayoutManager(layoutManager);
        adapter=new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);

        //下拉刷新功能
        swipeRefresh=(SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        // 进度条颜色
        swipeRefresh.setColorSchemeColors(R.color.purple_500);
        // 设置一个下拉刷新的监听器，当触发了下拉刷新功能的时候回调refreshFruits();
        swipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                refreshFruits();
            }
        });
    }

    private void refreshFruits(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        initFruits();
                        adapter.notifyDataSetChanged();
                        swipeRefresh.setRefreshing(false);
                    }
                });
            }
        }).start();
    }

    private void initFruits() {
        fruitList.clear();
        for(int i=0;i<50;i++){
            Random random=new Random();
            int index=random.nextInt(fruits.length);
            fruitList.add(fruits[index]);
        }
    }


    private void setSupportActionBar(Toolbar toolbar) {
    }
    public boolean onCreateOptionsMenu(Menu menu){
        getMenuInflater().inflate(R.menu.toolbal,menu);
        return  true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You Clicked Delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.setting:
                Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show();
                break;
                // HomeAsUp 按钮的id永远是 android.R.id.home
            case  android.R.id.home:
                // openDrawer 将滑动菜单显示出来
                mDrawerLayout.openDrawer(GravityCompat.START);
            default:

        }
        return true;
    }
}
```

## 12.7 可折叠式标题栏 

### 12.7.1 CollapsingToolbarLayout 

```
CollapsingToolbarLayout 做为AppBarLayout 的直接子布局进行使用
AppBarLayout  又是 CoordinatorLayout 的子布局 
```

#### 水果标题栏布局

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FruitActivity">
    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:id="@+id/appBar">
        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/collapsing_toolbar"
      // 指定主题 android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            // 指定CollapsingToolbarLayout 在趋于折叠以及折叠之后的背景色 
            app:contentScrim="?attr/colorOnPrimary"
      // 指定如何滚动隐藏 
      //scroll 表示 跟随水果内容详情的滚动一起滚动
      //exitUntilCollapsed 表示滚动完成折叠之后保留在界面上 
          app:layout_scrollFlags="scroll|exitUntilCollapsed">
            
        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

#### 标题栏具体内容

```
<com.google.android.material.appbar.CollapsingToolbarLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/collapsing_toolbar"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="?attr/colorOnPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
        <ImageView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/fruit_image_view"
            android:scaleType="centerCrop"
            //指定parallax 折叠过程中会产生一定的错位偏移 
            app:layout_collapseMode="parallax"/>
            <androidx.appcompat.widget.Toolbar
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:id="@+id/toobal"
        // 指定当前控件在CollapsingToolbarLayout 折叠过程中的
        // 折叠模式 指定pin 表示折叠过程中 位置始终保持不变
                app:layout_collapseMode="pin"/>
        </com.google.android.material.appbar.CollapsingToolbarLayout>
```

#### 水果内容详情

```
    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        
    </androidx.core.widget.NestedScrollView>
```

```
NestedScrollView 可以使用滚动的方式查看屏幕以外的数据，还增加了嵌套响应滚动事件的功能 
只能允许一个直接子布局，可在里面嵌套一个LinearLayout，然后在LinearLayout放入具体内容 
```

```
<LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
            <androidx.cardview.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="15dp"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="35dp"
                app:cardCornerRadius="4dp">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:id="@+id/fruit_content_text"
                    android:layout_margin="10dp"/>
            </androidx.cardview.widget.CardView>

        </LinearLayout>
```

####  添加一个悬浮按钮

```
<com.google.android.material.floatingactionbutton.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        //app:layout_anchor 指定了一个锚点 
        //设定为appBar 悬浮按钮会出现在 水果标题栏的区域内
        app:layout_anchor="@id/appBar"
      // app:layout_anchorGravity 将悬浮按钮定位到标题栏区域的右下角
        app:layout_anchorGravity="bottom|end"/>
```

#### 活动代码

```

import androidx.annotation.NonNull;
import androidx.appcompat.app.ActionBar;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;

import android.content.Intent;
import android.os.Bundle;
import android.view.MenuItem;
import android.widget.ImageView;
import android.widget.TextView;

import com.bumptech.glide.Glide;
import com.google.android.material.appbar.CollapsingToolbarLayout;

public class FruitActivity extends AppCompatActivity {

    public static final String FRUIT_NAME="fruit_name";
    public static final String FRUIT_IMAGE_ID="fruit_image_id";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fruit);
       // 获取传入的水果名和水果图片
        Intent intent=getIntent();
        String fruitName= intent.getStringExtra(FRUIT_NAME);
        int fruitImageId= intent.getIntExtra(FRUIT_IMAGE_ID,0);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        CollapsingToolbarLayout collapsingToolbar=(CollapsingToolbarLayout)
                findViewById(R.id.collapsing_toolbar);
        ImageView fruitImagedView=(ImageView) findViewById(R.id.fruit_image_view);
        TextView fruitContentText=(TextView) findViewById(R.id.fruit_content_text);
        ////Toolbar 标准用法  做为ActionBar显示 并启动HomeAsUp 按钮
        setSupportActionBar(toolbar);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null){
            actionBar.setDisplayHomeAsUpEnabled(true);
        }
        //调用collapsingToolbar.setTitle 将水果名设置成当前界面标题
        collapsingToolbar.setTitle(fruitName);
        //glide 加载传入的图片 并设置到标题栏上
        Glide.with(this).load(fruitImageId).into(fruitImagedView);
        String fruitContent=generateFruitContent(fruitName);
        fruitContentText.setText(fruitContent);
    }

    private String generateFruitContent(String fruitName) {
        StringBuilder fruitContent =new StringBuilder();
        for(int i=0;i<500;i++){
            fruitContent.append(fruitName);
        }
        return fruitContent.toString();
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case  android.R.id.home:
                finish();
                return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

#### 修改适配器

```
package com.example.materialtest;

import android.content.Context;
import android.content.Intent;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.cardview.widget.CardView;
import androidx.recyclerview.widget.RecyclerView;

import com.bumptech.glide.Glide;

import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {
    private  Context mContext;
    private List<Fruit>mFruitList;
    //ViewHolder 自定义的类 继承自RecyclerView.ViewHolder
    // 在ViewHolder中添加了fruitView变量包窜子项最外层布局的实例
    //随后在 onCreateViewHolder注册实例
    static class ViewHolder extends RecyclerView.ViewHolder{
        CardView cardView;
        ImageView fruitImage;
        TextView fruitName;
        //构造函数中传入一个View参数，这个参数通常为RecycleView子项的最外参数
        // 利用View.findViewById获取布局中的ImageView和TextView实例
        public ViewHolder(View view){
            super(view);
            cardView=(CardView) view;
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
        if(mContext==null){
            mContext=parent.getContext();
        }
        View view= LayoutInflater.from(mContext)
                .inflate(R.layout.fruit_item,parent,false);//加载布局

        //传入水果活动
        final ViewHolder holder=new ViewHolder(view);
        holder.cardView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int position =holder.getAdapterPosition();
                System.out.println(position);
                Fruit fruit= mFruitList.get(position+1);
                Intent intent=new Intent(mContext,FruitActivity.class);
                intent.putExtra(FruitActivity.FRUIT_NAME,fruit.getName());
                intent.putExtra(FruitActivity.FRUIT_IMAGE_ID,fruit.getImageId());
                mContext.startActivity(intent);
            }
        });

        return holder;
    }
    // onBindViewHolder 方法对RecyclerView的子项进行赋值，通过mFruitList.get(position)获取当前滚动的Fruit实例
    // 随后将数据设置给ImageView和TextView
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Fruit fruit=mFruitList.get(position);
        holder.fruitName.setText(fruit.getName());
        Glide.with(mContext).load(fruit.getImageId()).into(holder.fruitImage);
    }
    //getItemCount 获取RecyclerView一共有多少子项，直接返回数据长度即可
    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}

```

#### BUG

```
 java.lang.RuntimeException: Unable to start activity ComponentInfo{com.example.materialtest/com.example.materialtest.FruitActivity}: java.lang.IllegalStateException: This Activity already has an action bar supplied by the window decor. Do not request Window.FEATURE_SUPPORT_ACTION_BAR and set windowActionBar to false in your theme to use a Toolbar instead.
      
```

#### 问题原因

```
由于我在代码中使用了ToolBar，并在activity中调用了setSupportActionBar(toolbar);
setSupportActionBar(toolbar);
```

#### 解决方案

```
给报错的activity增加对应的theme或者修改theme主题
```

```
解决方法：将parent属性继承NoActionBar的样式就OK了
```

#### bug

```
  java.lang.ArrayIndexOutOfBoundsException: length=73; index=-1
```

```
返回出错  return new viewHoler(view);
```

#### 解决方法

```
return  holder;
```

### 12.7.2 充分利用系统状态栏空间

​	android:fitsSystemWindows 可将背景图与系统状态栏融合 

```
android:fitsSystemWindows="true" 需要设置在所有的f
```

​	创建新的主题 

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="FruitActivityTheme" parent="AppTheme">
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
</resources>
```

#### 全部代码

```
public class MainActivity extends AppCompatActivity {

    private DrawerLayout mDrawerLayout;
    private Fruit[] fruits={new Fruit("Apple",R.drawable.apple),new Fruit("Banana",R.drawable.banana),
            new Fruit("Orange",R.drawable.orange),new Fruit("Watermelon",R.drawable.watermelon),
            new Fruit("Pear",R.drawable.pear),
            new Fruit("Grape",R.drawable.grape),new Fruit("Pineapple",R.drawable.pineapple),
            new Fruit("Strawberry",R.drawable.strawberry),
            new Fruit("Cherry",R.drawable.cherry), new Fruit("Mango",R.drawable.mango)};
    private List<Fruit> fruitList=new ArrayList<>();

    private FruitAdapter adapter;

    private SwipeRefreshLayout swipeRefresh;
    @SuppressLint("ResourceAsColor")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        androidx.appcompat.widget.Toolbar toolbar=(androidx.appcompat.widget.Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        mDrawerLayout=(DrawerLayout) findViewById(R.id.drawer_layout);
        NavigationView navView=(NavigationView) findViewById(R.id.nav_view);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null) {
            // setDisplayHomeAsUpEnabled 将导航按钮显示出来
            actionBar.setDisplayHomeAsUpEnabled(true);
            // setHomeAsUpIndicator 导航按钮图标
            actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
        }
       //setCheckedItem 将Call菜单设定为默认选中
        navView.setCheckedItem(R.id.nav_call);
        // 设置监听器 用户点击后 会回调到 onNavigationItemSelected
        navView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                mDrawerLayout.closeDrawers();
                return true;
            }
        });
        FloatingActionButton fab=(FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //调用Snackbar.make 方法创建一个Snackebar 对象
                Snackbar.make(v,"Data deleted",Snackbar.LENGTH_SHORT)
                        //调用setAction 设置一个动作
                        .setAction("Undo", new View.OnClickListener() {
                            @Override
                            public void onClick(View v) {
                                Toast.makeText(MainActivity.this, "Data restored", Toast.LENGTH_SHORT).show();
                            }
                        }).show();
            }
        });

        initFruits();
        RecyclerView recyclerView=(RecyclerView) findViewById(R.id.recycler_view);
        GridLayoutManager layoutManager=new GridLayoutManager(this,2);
        recyclerView.setLayoutManager(layoutManager);
        adapter=new FruitAdapter(fruitList);
        recyclerView.setAdapter(adapter);

        //下拉刷新功能
        swipeRefresh=(SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        // 进度条颜色
        swipeRefresh.setColorSchemeColors(R.color.purple_500);
        // 设置一个下拉刷新的监听器，当触发了下拉刷新功能的时候回调refreshFruits();
        swipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                refreshFruits();
            }
        });
    }

    private void refreshFruits(){
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(2000);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        initFruits();
                        adapter.notifyDataSetChanged();
                        swipeRefresh.setRefreshing(false);
                    }
                });
            }
        }).start();
    }

    private void initFruits() {
        fruitList.clear();
        for(int i=0;i<50;i++){
            Random random=new Random();
            int index=random.nextInt(fruits.length);
            fruitList.add(fruits[index]);
        }
    }


    public void setSupportActionBar(Toolbar toolbar) {
    }
    public boolean onCreateOptionsMenu(Menu menu){
        getMenuInflater().inflate(R.menu.toolbal,menu);
        return  true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You Clicked Delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.setting:
                Toast.makeText(this, "You clicked Settings", Toast.LENGTH_SHORT).show();
                break;
                // HomeAsUp 按钮的id永远是 android.R.id.home
            case  android.R.id.home:
                // openDrawer 将滑动菜单显示出来
                mDrawerLayout.openDrawer(GravityCompat.START);
            default:

        }
        return true;
    }
}
```

```
package com.example.materialtest;

public class Fruit {
    private  String name;
    private  int imageId;
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

```
package com.example.materialtest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.ActionBar;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;

import android.content.Intent;
import android.os.Bundle;
import android.view.MenuItem;
import android.widget.ImageView;
import android.widget.TextView;

import com.bumptech.glide.Glide;
import com.google.android.material.appbar.CollapsingToolbarLayout;

public class FruitActivity extends AppCompatActivity {

    public static final String FRUIT_NAME="fruit_name";
    public static final String FRUIT_IMAGE_ID="fruit_image_id";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_fruit);
       // 获取传入的水果名和水果图片
        Intent intent=getIntent();
        String fruitName= intent.getStringExtra(FRUIT_NAME);
        int fruitImageId= intent.getIntExtra(FRUIT_IMAGE_ID,0);
        Toolbar toolbar=(Toolbar) findViewById(R.id.toobal);
        CollapsingToolbarLayout collapsingToolbar=(CollapsingToolbarLayout)
                findViewById(R.id.collapsing_toolbar);
        ImageView fruitImagedView=(ImageView) findViewById(R.id.fruit_image_view);
        TextView fruitContentText=(TextView) findViewById(R.id.fruit_content_text);
        ////Toolbar 标准用法  做为ActionBar显示 并启动HomeAsUp 按钮
        setSupportActionBar(toolbar);
        ActionBar actionBar=getSupportActionBar();
        if(actionBar!=null){
            actionBar.setDisplayHomeAsUpEnabled(true);
        }
        //调用collapsingToolbar.setTitle 将水果名设置成当前界面标题
        collapsingToolbar.setTitle(fruitName);
        //glide 加载传入的图片 并设置到标题栏上
        Glide.with(this).load(fruitImageId).into(fruitImagedView);
        String fruitContent=generateFruitContent(fruitName);
        fruitContentText.setText(fruitContent);
    }

    private String generateFruitContent(String fruitName) {
        StringBuilder fruitContent =new StringBuilder();
        for(int i=0;i<500;i++){
            fruitContent.append(fruitName);
        }
        return fruitContent.toString();
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()){
            case  android.R.id.home:
                finish();
                return true;
        }
        return super.onOptionsItemSelected(item);
    }
}
```

```
package com.example.materialtest;

import android.content.Context;
import android.content.Intent;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.cardview.widget.CardView;
import androidx.recyclerview.widget.RecyclerView;

import com.bumptech.glide.Glide;

import java.util.List;

public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder> {
    private  Context mContext;
    private List<Fruit>mFruitList;
    //ViewHolder 自定义的类 继承自RecyclerView.ViewHolder
    // 在ViewHolder中添加了fruitView变量包窜子项最外层布局的实例
    //随后在 onCreateViewHolder注册实例
    static class ViewHolder extends RecyclerView.ViewHolder{
        CardView cardView;
        ImageView fruitImage;
        TextView fruitName;
        //构造函数中传入一个View参数，这个参数通常为RecycleView子项的最外参数
        // 利用View.findViewById获取布局中的ImageView和TextView实例
        public ViewHolder(View view){
            super(view);
            cardView=(CardView) view;
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
        if(mContext==null){
            mContext=parent.getContext();
        }
        View view= LayoutInflater.from(mContext)
                .inflate(R.layout.fruit_item,parent,false);//加载布局

        //传入水果活动
        final ViewHolder holder=new ViewHolder(view);
        holder.cardView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int position =holder.getAdapterPosition();
                System.out.println(position);
                Fruit fruit= mFruitList.get(position);
                Intent intent=new Intent(mContext,FruitActivity.class);
                intent.putExtra(FruitActivity.FRUIT_NAME,fruit.getName());
                intent.putExtra(FruitActivity.FRUIT_IMAGE_ID,fruit.getImageId());
                mContext.startActivity(intent);
            }
        });

        return  holder;
    }
    // onBindViewHolder 方法对RecyclerView的子项进行赋值，通过mFruitList.get(position)获取当前滚动的Fruit实例
    // 随后将数据设置给ImageView和TextView
    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Fruit fruit=mFruitList.get(position);
        holder.fruitName.setText(fruit.getName());
        Glide.with(mContext).load(fruit.getImageId()).into(holder.fruitImage);
    }
    //getItemCount 获取RecyclerView一共有多少子项，直接返回数据长度即可
    @Override
    public int getItemCount() {
        return mFruitList.size();
    }
}

```

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.materialtest">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme"
        tools:targetApi="31">
        <activity
            android:name=".FruitActivity"
            android:exported="false"
            android:theme="@style/FruitActivityTheme"/>
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="Fruits">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FruitActivity"
    android:fitsSystemWindows="true">
    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="250dp"
        android:id="@+id/appBar"
        android:fitsSystemWindows="true">
        <com.google.android.material.appbar.CollapsingToolbarLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/collapsing_toolbar"
            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
            app:contentScrim="?attr/colorOnPrimary"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            android:fitsSystemWindows="true">
        <ImageView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/fruit_image_view"
            android:scaleType="centerCrop"
            app:layout_collapseMode="parallax"
            android:fitsSystemWindows="true"/>
            <androidx.appcompat.widget.Toolbar
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:id="@+id/toobal"
                app:layout_collapseMode="pin"/>
        </com.google.android.material.appbar.CollapsingToolbarLayout>
    </com.google.android.material.appbar.AppBarLayout>
    <androidx.core.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
            <androidx.cardview.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="15dp"
                android:layout_marginLeft="15dp"
                android:layout_marginRight="15dp"
                android:layout_marginTop="35dp"
                app:cardCornerRadius="4dp">
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:id="@+id/fruit_content_text"
                    android:layout_margin="10dp"/>
            </androidx.cardview.widget.CardView>

        </LinearLayout>
    </androidx.core.widget.NestedScrollView>

    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp"
        android:src="@drawable/ic_comment"
        app:layout_anchor="@id/appBar"
        app:layout_anchorGravity="bottom|end"/>
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.drawerlayout.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/drawer_layout"
    tools:context=".MainActivity">

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <com.google.android.material.appbar.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
            <androidx.appcompat.widget.Toolbar
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:id="@+id/toolbar"
                android:background="?attr/colorPrimary"
                android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                app:layout_scrollFlags="scroll|enterAlways|snap"/>
        </com.google.android.material.appbar.AppBarLayout>

        <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/swipe_refresh"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

                <androidx.recyclerview.widget.RecyclerView
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:id="@+id/recycler_view"
                    app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
        </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>


        <com.google.android.material.floatingactionbutton.FloatingActionButton
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/fab"
            android:layout_gravity="bottom|end"
            android:layout_margin="16dp"
            android:src="@drawable/ic_done"
            app:elevation="8dp"/>
    </androidx.coordinatorlayout.widget.CoordinatorLayout>
    <com.google.android.material.navigation.NavigationView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/nav_view"
        android:layout_gravity="start"
        app:menu="@menu/nav_menu"
        app:headerLayout="@layout/nav_header"/>

</androidx.drawerlayout.widget.DrawerLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    app:cardCornerRadius="4dp">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="100dp"
        android:scaleType="centerCrop"/>
         <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:layout_margin="5dp"
             android:textSize="16sp"/>
    </LinearLayout>
</androidx.cardview.widget.CardView>
```

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="180dp"
    android:padding="10dp"
    android:background="?attr/colorPrimary">
    <de.hdodenhof.circleimageview.CircleImageView
        android:layout_width="70dp"
        android:layout_height="70dp"
        android:id="@+id/icon_image"
        android:src="@drawable/nav_icon"
        android:layout_centerInParent="true"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/mail"
        android:layout_alignParentBottom="true"
        android:text ="tonygreendev@gmail.com"
        android:textColor="#FFF"
        android:textSize="14sp"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/username"
        android:layout_above="@+id/mail"
        android:text ="Tony Green"
        android:textColor="#FFF"
        android:textSize="14sp"/>

</RelativeLayout>
```

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
<group android:checkableBehavior="single">

    <item android:title="Call"
        android:id="@+id/nav_call"
        android:icon="@drawable/nav_call"
        />
    <item android:title="Friends"
        android:id="@+id/nav_friends"
        android:icon="@drawable/nav_friends"
        />
    <item android:title="Location"
        android:id="@+id/nav_location"
        android:icon="@drawable/nav_location"
        />
    <item android:title="Mail"
        android:id="@+id/nav_mail"
        android:icon="@drawable/nav_mail"
        />
    <item android:title="Task"
        android:id="@+id/nav_task"
        android:icon="@drawable/nav_task"
        />
</group>
</menu>
```

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/backup"
        android:icon="@drawable/ic_backup"
        android:title="Backup"
        app:showAsAction="always"/>
    <item
        android:id="@+id/delete"
        android:icon="@drawable/ic_delete"
        android:title="Delete"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/setting"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never"/>

</menu>
```

```
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Primary brand color. -->
        <item name="colorPrimary">@color/purple_500</item>
        <item name="colorPrimaryVariant">@color/purple_700</item>
        <item name="colorOnPrimary">@color/white</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item><!-- 无标题 -->
        <!-- Secondary brand color. -->
        <item name="colorSecondary">@color/teal_200</item>
        <item name="colorSecondaryVariant">@color/teal_700</item>
        <item name="colorOnSecondary">@color/black</item>
        <!-- Status bar color. -->
        <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
        <!-- Customize your theme here. -->
    </style>
</resources>
```

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="FruitActivityTheme" parent="AppTheme">
        <item name="android:statusBarColor">@android:color/transparent</item>
    </style>
</resources>
```

### bug

```
Duplicate class android.support.v4.app.INotificationSideChannel found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.app.INotificationSideChannel$Stub found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.app.INotificationSideChannel$Stub$Proxy found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.IResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.IResultReceiver$Stub found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.IResultReceiver$Stub$Proxy found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.ResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.ResultReceiver$1 found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.ResultReceiver$MyResultReceiver found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2) Duplicate class android.support.v4.os.ResultReceiver$MyRunnable found in modules core-1.5.0-runtime (androidx.core:core:1.5.0) and support-compat-27.0.2-runtime (com.android.support:support-compat:27.0.2)
```

#### 解决方法

```
在编译Android项目的时候遇到重复类的问题，但是就是查不清楚到底是哪个库引入出错了，最后想到是新引入了 exoplayer 库，会不会是 exoplayer 使用了老的 supportV4 包，而我们现在用的是AndroidX呢的原因呢？那么如果第三方的库采用兼容的模式，是否可以解决问题呢？那么我们可以在gradle.properties 文件中进行如下修改：
//gradle.properties
android.useAndroidX=true
android.enableJetifier=true
```


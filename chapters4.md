# chapters4

## 4.1 碎片 Fragment

​	碎片是一种可以嵌入到活动当中的UI片段，能让程序更加合理的大屏幕的空间。类似于活动，包含布局以及生命周期。

## 4.2 使用方式

### 4.2.1 简单用法

 在一个活动中添加两个碎片 

​	左侧碎片布局left_fragment.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/button"
        android:layout_gravity="center_horizontal"
        android:text="Button"
        android:textAllCaps="false"/>

</LinearLayout>
```

 右侧碎片布局 right_fragment.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#00ff00">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:textSize="20sp"
        android:text="This is right fragment"/>
</LinearLayout>
```

​	创建类 LeftFragment 和RightFragment 继承自Fragment

```
package com.example.fragmenttest;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class LeftFragment extends Fragment {
    //重写了 onCreateView
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
       View view=inflater.inflate(R.layout.left_fragment,container,
               false);
       return  view;
    }
}

```

```
package com.example.fragmenttest;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class RightFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        View view=inflater.inflate(R.layout.right_fragment,container,
                false);
        return view;
    }
}

```

​	修改activity.main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    //使用<Fragment> 在布局中添加标签
    <fragment
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:id="@+id/left_fragment"
        // 显示的指明要添加的碎片类名
        android:name="com.example.fragmenttest.LeftFragment"/>
    <fragment
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:id="@+id/right_fragment"
        android:name="com.example.fragmenttest.RightFragment"/>
</LinearLayout>
```

### 4.2.2 动态添加碎片

​	1创建待添加的碎片实例

```
package com.example.fragmenttest;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class AnotherRightFragment extends Fragment {
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view=inflater.inflate(
                R.layout.another_right_fragment,container,
                false
        );
        return view;
    }
}
```

​	创建待添加的碎片布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#fff000">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:textSize="20sp"
        android:text="This is another right fragment"/>

</LinearLayout>
```

 2 修改activity_main代码 利用FrameLayout 帧布局替换右侧碎片 所有控件默认摆放在左上角

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    <fragment
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:id="@+id/left_fragment"
        android:name="com.example.fragmenttest.LeftFragment"/>
    <FrameLayout
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:id="@+id/right_layout">
    </FrameLayout>


</LinearLayout>
```

3 修改MainActivity代码

```
package com.example.fragmenttest;

import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentManager;
import androidx.fragment.app.FragmentTransaction;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity  {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button=(Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                switch (v.getId()){
                    case R.id.button:
                        replaceFragment(new AnotherRightFragment());
                        break;
                    default:
                        break;
                }
            }
        });
        replaceFragment(new RightFragment());

    }
    private void replaceFragment(Fragment fragment){
       //获取FragmentManager，在活动中可以直接调用getSupportFragmentManager（）方法
        FragmentManager fragmentManager=getSupportFragmentManager();
        //开启一个事务，通过beginTransaction（）方法开启
        FragmentTransaction transaction=fragmentManager.beginTransaction();
        //向容器内添加或交替碎片，一般使用replace()方法实现，需要传入容器的id和待添加的碎片实例
        transaction.replace(R.id.right_layout,fragment);
        // 提交事务，调用transaction.commit()完成
        transaction.commit();
    }
}
```

### 4.2.3 在碎片中模拟返回栈

	        //调用addToBackStack（）方法，可以接受一个名字用于描述返回栈的状态，一般传入null即可
	    transaction.addToBackStack(null);

```
    private void replaceFragment(Fragment fragment){
       //获取FragmentManager，在活动中可以直接调用getSupportFragmentManager（）方法
        FragmentManager fragmentManager=getSupportFragmentManager();
        //开启一个事务，通过beginTransaction（）方法开启
        FragmentTransaction transaction=fragmentManager.beginTransaction();
        //向容器内添加或交替碎片，一般使用replace()方法实现，需要传入容器的id和待添加的碎片实例
        transaction.replace(R.id.right_layout,fragment);
        //调用addToBackStack（）方法，可以接受一个名字用于描述返回栈的状态，一般传入null即可
        transaction.addToBackStack(null);
        // 提交事务，调用transaction.commit()完成
        transaction.commit();
    }
```

### 4.2.4 碎片和活动之间进行通信

​	活动中调用碎片，利用FragmentManager的findFragmentById（），可从布局文件中获取碎片的实例

```
  RightFragment rightFragment=(RightFragment) getSupportFragmentManager()
                .findFragmentById(R.id.right_layout);
```

 调用findFragmentById（）方法，就可在活动中获得相应碎片的实例

碎片中调用活动的方法 ，每个碎片都可使用getActivity()方法来得到和当前碎片相关联的活动实例

```
MainActivity activity=(MainActivity) getActivity();
```

有了活动实例，就可在碎片中调用活动中的方法

## 4.3 碎片的生命周期 

### 4.3.1 碎片的状态和回调

#### 1 运行状态 

​	碎片是可见的，所关联的活动正处于运行状态，该碎片也处于运行状态

#### 2 暂停状态

​	一个活动进入暂停状态（由于另一个为占满屏幕的活动被添加到了栈顶），与它相关联的可见碎片就会进入到暂停状态

#### 3 停止状态

​	一个活动进入停止状态，与它相关联的碎片就会进去到停止状态，或通过调用FragmentManager的remove（） replace()方法将碎片移除。但如果在事务提交之前调用addToBackStack()方法，这时的碎片也会进入到停止状态。总体说，进入到停止状态的碎片对用户来说是完全不可见的

#### 4 销毁状态 

​		碎片依附活动存在，活动销毁，与之相关联碎片随之销毁；或通过调用FragmentManager 的remove（） replace()方法将碎片移除，但在事务提交之前没有调用addToBackStack()，那么也会进入销毁状态。

```
onAttach() 当碎片和活动建立关联的时候调用
onCreateView() 为活动创建视图（加载布局）调用
onActivityCreated() 确保与碎片相关联的活动一定已经创建完毕的时候调用
onDestoryView() 当与碎片关联的视图被移除的时候调用
onDetach()       当碎片和活动接触关联的时候调用
```

## 4.4 动态加载布局的技巧

### 	4.4.1 使用限定符 Qualifiers

 	在res目录下新建一个layout-large文件夹，里面新建布局，也叫activity_main.xml 

```
大小：	small  提供给小设备
	  normal 提供给中等设备
	  large  提供给大屏幕设备
	  xlarge 提供给超大屏幕设备
分辨率： ldpi 提供给低分辨率的设备
		mdpi 提供给中等分辨率
		hdpi 提供给高分辨率
		xhdpi 提供给超高分变率
		xxhdpi 超超高分辨率
方向： land 提供给横屏设备
		port 提供给竖屏设备
```

### 4.4.2 使用最小宽度限定符

 	允许我们对屏幕的宽度指定一个最小值，以这个最小值为临界点，屏幕大于这个值的设备就加载一个布局
 	sw600dp 表示在宽度大于等于600dp的设备上时，会加载layout-sw600dp 里面的主活动布局
 				在宽度小于600dp的设备上时，会加载layout里面的主活动布局

## 4.5 简易版的新闻应用

​	1 构建一个新闻的实体类

```
package com.example.fragmentbestpractice;

public class News {
    private String title;//新闻标题
    private String content;//新闻内容

    public String getTitle() {
        return title;
    }

    public String getContent() {
        return content;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public void setContent(String context) {
        this.content = content;
    }
}
```

​	2 构建新闻内容的布局 news_content_frag.xml

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/visibility_layout"
        android:orientation="vertical"
        // invisible 不可见，但保留控件所占控件
        //gone 控件不可见，不保留控件控件
        android:visibility="invisible">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:id="@+id/news_title"
            //控制控件里的元素在该控件的显示位置
            android:gravity="center"
           //让控件里的内容与控件边上有一定距离
           android:padding="10dp"
            android:textSize="20sp"/>
        <View
            android:layout_width="match_parent"
            android:layout_height="1dp"
            android:background="#000"/>
        <TextView
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:id="@+id/news_content"
            android:layout_weight="1"
            android:padding="15dp"
            android:textSize="18sp"/>

    </LinearLayout>
    <View
        android:layout_width="1dp"
        android:layout_height="match_parent"
        //布局相对于父项的位置
        //控件左边缘与父控件的左边缘对齐
        android:layout_alignParentLeft="true"
        android:background="#000"/>

</RelativeLayout>
```

 	3 新建NewsContentFragment类，继承自Fragment

```
package com.example.fragmentbestpractice;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class NewsContentFragment extends Fragment {
    private View view;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        //加载新闻布局
        view=inflater.inflate(R.layout.news_content_frag,container,
                false);
        return view;
    }
    //将新闻的标题和内容显示在界面上
    public void refresh(String newsTitle,String newsContent){
        View visibilityLayout=view.findViewById(R.id.visibility_layout);
        visibilityLayout.setVisibility(View.VISIBLE);
        TextView newsTitleText=(TextView) view.findViewById(R.id.news_title);
        TextView newsContentText=(TextView) view.findViewById(R.id.news_content);
        newsTitleText.setText(newsTitle);//刷新标题
        newsContentText.setText(newsContent);//刷新内容
    }
}

```

​	4 如果想在单页模式下使用，需要新建一个活动，命名为NewsContentActivity，并将布局名指定为news_content.xml，修改里面代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <fragment
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/news_content_fragment"
        android:name="com.example.fragmentbestpractice.NewsContentFragment"/>

</LinearLayout>
```

​	修改NewsContentActivity代码

```
package com.example.fragmentbestpractice;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Context;
import android.content.Intent;
import android.os.Bundle;

public class NewsContentActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_content);
        String newsTitle=getIntent().getStringExtra("news_title");//新闻标题
        String newsContent=getIntent().getStringExtra("news_content");//新闻内容
        //调用FragmentManager的 findFragmentById（）方法，在活动中获取碎片实例
        NewsContentFragment newsContentFragment=(NewsContentFragment)
                getSupportFragmentManager().
                        findFragmentById(R.id.news_content_fragment);
        //传入新闻内容和标题
        newsContentFragment.refresh(newsTitle,newsContent);
    }
    //启动活动 可以看到启动该活动需要传入的参数
    public static void actionStart(Context context,String newsTitle,
                                   String newsContent){
        Intent intent= new Intent(context,NewsContentActivity.class);
        intent.putExtra("news_title",newsTitle);
        intent.putExtra("news_content",newsContent);
        context.startActivity(intent);
    }
}
```

​	5 创建一个显示新闻列表的布局，新建news_title_frag.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <androidx.recyclerview.widget.RecyclerView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/news_title_recycle_view"/>

</LinearLayout>
```

​	6 有了recyclerview，就需要子项的布局，新建news_item.xml作为RecyclerView子项的布局

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:id="@+id/news_title"
    //maxLines="1"表示让这个TextView 单行显示
    android:maxLines="1"
    //用于设定当文本内容超出控件宽度时，文本的缩略方式，在尾部进行缩略
    android:ellipsize="end"
    android:textSize="18sp"
    //android:padding 在文本四周加上空白
    android:paddingLeft="10dp"
    android:paddingRight="10dp"
    android:paddingTop="15dp"
    android:paddingBottom="15dp">

</TextView>
```

 7 展示新闻列表的地方，新建NewsTitleFragment作为展示新闻的地方 

```
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {

        View view=inflater.inflate(R.layout.news_title_frag,container,
                false);
        RecyclerView newsTitleRecyclerView=(RecyclerView) view.
                findViewById(R.id.news_title_recycle_view);
        LinearLayoutManager layoutManager=new LinearLayoutManager(getActivity());
        newsTitleRecyclerView.setLayoutManager(layoutManager);
        NewsAdapter adapter=new NewsAdapter(getNews());
        newsTitleRecyclerView.setAdapter(adapter);
        return view;

    }
    //重写onActivityCreated（）方法 实现单页布局和双页布局交换
```

 	8 单页布局  单页模式下，只会加载一个碎片

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/news_title_layout">

    <fragment
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/news_title_fragment"
        android:name="com.example.fragmentbestpractice.NewsTitleFragment"/>

</FrameLayout>
```

​	9  双页布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <fragment
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1"
        android:id="@+id/news_title_fragment"
        android:name="com.example.fragmentbestpractice.NewsTitleFragment"/>
    <FrameLayout
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2"
        android:id="@+id/news_content_layout">
        <fragment
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/news_content_fragment"
            android:name="com.example.fragmentbestpractice.NewsContentFragment"/>
    </FrameLayout>


</LinearLayout>
```

​	10 在NewsTitleFragment中通过RecyclerView将新闻列表展现出来 ，新建一个NewsTitleFragment作为RecyclerView的适配器

```
        public NewsAdapter(List<News>newsList){
            mNewsList=newsList;
        }
        @Override
        public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
            View view= LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.news_item,parent,false);//加载布局
           final ViewHolder holder=new ViewHolder(view);
            view.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    News news =mNewsList.get (holder.getAdapterPosition());
                    if(isTwoPane){
                        //如果是双页模式 刷新newsContentFragment中的内容
                        //getFragmentManager 被弃用了
            NewsContentFragment newsContentFragment = (NewsContentFragment)
                       getParentFragmentManager().
                       findFragmentById(R.id.news_content_fragment);
                   newsContentFragment.refresh(news.getTitle(),
                               news.getContent());
                    }else {
                        //如果是单页模式，直接启动NewsContentActicity
                        NewsContentActivity.actionStart(getActivity(),
                                news.getTitle(),news.getContent());
                    }
                }
            });
            return holder;
        }

        @Override
        public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
            News news=mNewsList.get(position);
            holder.newsTitleText.setText(news.getTitle());
        }

        @Override
        public int getItemCount() {
            return mNewsList.size();
        }
    }
```

​	11 往RecyclerView中填充数据，修改NewsTitleFragement

```
    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {

        View view=inflater.inflate(R.layout.news_title_frag,container,
                false);
        RecyclerView newsTitleRecyclerView=(RecyclerView) view.
                findViewById(R.id.news_title_recycle_view);
        LinearLayoutManager layoutManager=new LinearLayoutManager(getActivity());
        newsTitleRecyclerView.setLayoutManager(layoutManager);
        NewsAdapter adapter=new NewsAdapter(getNews());
        newsTitleRecyclerView.setAdapter(adapter);
        return view;
    }
    private List<News> getNews(){
        List<News> newsList=new ArrayList<>();
        for(int i=1;i<=50;i++){
            News news=new News();
            news.setTitle("This is news title"+i);
            news.setContent("This is news context"+i+".");
            newsList.add(news);
        }
        return newsList;
    }
    private String getRandomLengthContent(String content){
        Random random= new Random();
        int length=random.nextInt(20)+1;
        StringBuilder builder =new StringBuilder();
        for(int i=0;i<length;i++){
            builder.append(content);
        }
        return builder.toString();
    }

```

###  onActivityCreated（）的替代

```
 @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);

        requireActivity().getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
                if (event.getTargetState() == Lifecycle.State.CREATED){
                    //在这里任你飞翔
                    if(getActivity().findViewById(R.id.news_content_layout)!=null){
                        isTwoPane=true;
                    }else {
                        isTwoPane=false;
                    }

                    getLifecycle().removeObserver(this);  //这里是删除观察者
                }
            }
        });
    }
```

### getFragmentManager的替代

```
NewsContentFragment newsContentFragment = (NewsContentFragment)
                        getParentFragmentManager().findFragmentById(R.id.news_content_fragment);
```


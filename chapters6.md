# chapters6

## 6.1 持久化技术

​	数据持久化就是指将那些内存中的瞬时数据保存在存储设备中，保证即使在手机或电脑关机的情况下，数据仍然不会丢失。保存在内存中的数据是瞬时状态的，保存在存储设备中的数据是持久状态的，持久化技术提供了一种机制可以让数据在瞬时状态和持久状态之间进行转换。

## 6.2 文件存储

​	最基本的一种数据存储方式，不对存储的内容进行任何格式化处理，所有数据都是原封不动的保存在文件中，比较适合存储一些简单的文本技术或二进制数据

### 6.2.1 数据存储在文件中

```
Context 的openFileOutput() 可以将数据存储到指定的文件中
							第一个参数 文件名，不可以包含路径。所有
								文件默认存储到/data/data/<package name>/file/目录下
							第二个参数 文件的操作模式 
								MODE_PRIVATE 默认操作模式 指定文件名是，所写入内容会覆盖原有
								MODE-APPEND	该文件已存在，往文件内添加，不新建文件
openFileOutput() 返回一个FileOutputStream 对象 以下代码展示了，如何将文本内容存在文件中
	public void save(String inputText){
        FileOutputStream out=null;
        BufferedWriter writer=null;
        try {
            out =openFileOutput("data",Context.MODE_PRIVATE);
            writer=new BufferedWriter(new OutputStreamWriter(out));
            writer.write(inputText);
        }
        catch (IOException e){
            e.printStackTrace();
        }
        finally {
            try {
                if(writer!=null){
                    writer.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
```

```
public class MainActivity extends AppCompatActivity {
    private EditText edit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        edit=(EditText) findViewById(R.id.edit);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        String inputText=edit.getText().toString();
        save(inputText);
    }
    public void save(String inputText){
        FileOutputStream out=null;
        BufferedWriter writer=null;
        try {
            out =openFileOutput("data",Context.MODE_PRIVATE);
            writer=new BufferedWriter(new OutputStreamWriter(out));
            writer.write(inputText);
        }
        catch (IOException e){
            e.printStackTrace();
        }
        finally {
            try {
                if(writer!=null){
                    writer.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
}
```

### 6.2.2 从文件中读取数据

``` 
Context 的openFileInput() 从文件中读取数据 参数即为文件名
	如何从文本中读取数据 
	    public String load(){
        FileInputStream in=null;
        BufferedReader reader=null;
        StringBuilder content=new StringBuilder();
        try {
            in=openFileInput("data");
            reader=new BufferedReader(new InputStreamReader(in));
            String line="";
            while((line=reader.readLine())!=null){
                content.append(line);
            }
        }
        catch (IOException e){
            e.printStackTrace();
        }
        finally {
            try {
                if(reader!=null){
                    reader.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        return content.toString();
    }
```

```
 private EditText edit;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        edit=(EditText) findViewById(R.id.edit);
        String inputText=load();
        //TextUtils.isEmpty(inputText)
        	传入的字符串为null或空值都会返回true
        if(!TextUtils.isEmpty(inputText)){
            edit.setText(inputText);
            //setSelection 将输入光标移动到文本的末尾位置以便继续输入
            edit.setSelection(inputText.length());
            Toast.makeText(this, "Restoring succeeded", Toast.LENGTH_SHORT).show();
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        String inputText=edit.getText().toString();
    }
    public String load(){
        FileInputStream in=null;
        BufferedReader reader=null;
        StringBuilder content=new StringBuilder();
        try {
            in=openFileInput("data");
            reader=new BufferedReader(new InputStreamReader(in));
            String line="";
            while((line=reader.readLine())!=null){
                content.append(line);
            }
        }
        catch (IOException e){
            e.printStackTrace();
        }
        finally {
            try {
                if(reader!=null){
                    reader.close();
                }
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        return content.toString();
    }
```

## 6.3 SharedPreferences 存储

​	使用键值对的方式存储数据。支持多种不同的数据存储类型

### 6.3.1 将数据存储到SharedPreferences

​	获取SharedPreferences对象

#### 	1 Context类中的getSharedPreferences()方法

​		第一个参数 用于指定SharePreferences文件的名称，如果没有会自动创建一个。文件放	在/data/data/<package name>/shared_prefs/目录下. 

​	第二个参数用于指定操作模式  只有MODE_PRIVATE这一种模式 

#### 	2 Activity类中的getPreferences()方法

​	只接受一个操作模式参数，会自动把当前活动的类名作为文件名

### 3 PreferenceManager 类中的getDefaultSharedPreferences()

​	这是一个静态方法，接受一个Context参数，自动使用当前应用程序的包名作为前缀名命名文件、

有了SharedPreferences对象后，在SharedPreferences文件存储数据分三步

​	1 使用SharedPreferences对象的edit() 获取SharedPreferences.Editor对象

​	2 向SharedPreferences.Editor对象中添加数据，添加布尔型就用 putBoolean 字符串型就用 putString

​	3 调用apply() 将添加的数据提交 完成存储

```
package com.example.sharedpreferencestest;

import androidx.appcompat.app.AppCompatActivity;

import android.app.Activity;
import android.content.Context;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;



public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button saveData =(Button) findViewById(R.id.button);
        saveData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Context ctx = MainActivity.this;
                SharedPreferences.Editor editor=getSharedPreferences("data",MODE_PRIVATE).edit();
                editor.putString("name","Tom");
                editor.putInt("age",20);
                editor.putBoolean("married",false);
                editor.apply();

            }
        });
    }
}
```

### 6.3.2 从SharedPreferences中读取数据

```
get 第一个是键 第二个是默认值 
        Button restoreData=(Button) findViewById(R.id.restore_data);
        restoreData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SharedPreferences pref=getSharedPreferences("data",MODE_PRIVATE);
                String name= pref.getString("name","");
                int age=pref.getInt("age",0);
                Boolean married=pref.getBoolean("married",false);
                Log.d("Mactivity","name is "+name);
                Log.d("Mactivity","age is "+age);
                Log.d("Mactivity","married is "+married);
            }
        });
```

### 6.3.3 实现记住密码的功能

​	利用之前的BroadcastBestPraceice 项目进行修改

​	1 修改activity_login代码

```
 <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        // <CheckBox> 复选框控件 用户可以通过点击的方式 进行选中或取消
        // 用这个控件表示用户是否需要记住密码
        <CheckBox
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/remeber_pass"/>
        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Remeber password"
            android:textSize="18sp"/>
    </LinearLayout>
        <Button
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:id="@+id/login"
            android:text="Login"/>
```

​	2 修改LoginActivity密码

```
    private EditText accountEdit;
    private EditText passwordEdit;
    private Button login;
    private SharedPreferences pref;
    private SharedPreferences.Editor editor;
    private CheckBox remeberPass;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        accountEdit=(EditText) findViewById(R.id.account);
        passwordEdit=(EditText) findViewById(R.id.password);
        pref= PreferenceManager.getDefaultSharedPreferences(this);
        remeberPass =(CheckBox) findViewById(R.id.remeber_pass);
        boolean isRemeber =pref.getBoolean("remeber_password",false);
        if(isRemeber){
            //把账号密码都设置到文本框
            String account =pref.getString("account","");
            String password=pref.getString("password","");
            accountEdit.setText(account);
            passwordEdit.setText(password);
            remeberPass.setChecked(true);
        }
         login=(Button) findViewById(R.id.login);
        login.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String account= accountEdit.getText().toString();
                String password= passwordEdit.getText().toString();
                //如果账号是admin 密码是123456 登录成功
                if(account.equals("admin")&&password.equals("123456")){
                    editor = pref.edit();
                    if(remeberPass.isChecked()){
                        //利用自带的方法 检查复选框是否选中
                        // 选中记住密码 没选中将文件中的数据清楚
                        editor.putBoolean("remeber_password",true);
                        editor.putString("account",account);
                        editor.putString("password",password);
                    }else {
                        editor.clear();
                    }
```

## 6.4 SQLite数据库存储

​	SQLite一款轻量级的关系型数据库，运算速度快，占用资源小。

### 6.4.1 创建数据库

​	SQLiteOpenHelper 抽象类 

​	oncreare（） 和onUpgrade（）方法

​	getReadableDatabase() 和getWriterableDatabase()  创建或打开一个现有的数据库，并返回一个可对数据库进行读写操作的对象。

​	构造函数 第一个参数 Context  

​					第二个参数 数据库名 

​					第三个参数 允许我们在查询数据的时候返回一个自定义的Cursor，一般传入null

​					第四个参数表示 当前数据库的版本号，可对其进行升级操作

​	文件放在/data/data/<package name>/databases/目录下.

```
SQlite 数据库类型 
	integer 整形
	real 浮点型
	text 文本类型
	blob 二进制
	使用primary key 将id设为主键
```

​	1 新建类MyDatabaseHelper extends SQLiteOpenHelper

```
public class MyDatabaseHelper extends SQLiteOpenHelper {
    //Book 的建表语句
    public static final String CREATE_BOOK ="create table book("
            +"id interger primary key autoincrement,"
            +"author text,"
            +"price real, "
            +"pages integer, "
            +"name text)";
    private Context mContext;
    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory,
                            int version){
        super(context, name, factory, version);
        mContext=context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        //利用execSQL 方法执行建表语句
        db.execSQL(CREATE_BOOK);
        Toast.makeText(mContext, "Created succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}
```

​	2 修改activity_main.xml代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/create_database"
        android:text="Create database"/>


</LinearLayout>
```

​	3 修改主活动代码

```
public class MainActivity extends AppCompatActivity {
    private MyDatabaseHelper dpHelper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        dpHelper =new MyDatabaseHelper(this,"BookStore.db",null,1);
        Button createDatabase= (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dpHelper.getWritableDatabase();
            }
        });
    }
}
```

####   adb 观察

​	没有权限的，

https://blog.csdn.net/m0_57898713/article/details/121896151?ops_request_misc=&request_id=&biz_id=102&utm_term=su:%20inaccessible%20or%20not%20found&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-121896151.nonecase&spm=1018.2226.3001.4187

### 6.4.2 升级数据库

​	新建一个表 用于分类 

​	需要用到onUpgrade 将原有的表进行删除，然后重建，在对版本号进行更改 

```
dpHelper =new MyDatabaseHelper(this,"BookStore.db",null,2);
```

```
public class MyDatabaseHelper extends SQLiteOpenHelper {
    //Book 的建表语句
    public static final String CREATE_BOOK ="create table Book("
            +"id integer primary key autoincrement ,"
            +"author text,"
            +"price real, "
            +"pages integer, "
            +"name text)";
    // Caregory 分类
    public static final String CREATE_CATEGORY="create table Category("
            +"id integer primary key autoincrement ,"
            +"category_name text, "
            +"category_code integer)";
    private Context mContext;
    public MyDatabaseHelper(Context context, String name,
                            SQLiteDatabase.CursorFactory factory,
                            int version){
        super(context, name, factory, version);
        mContext=context;
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        //利用execSQL 方法执行建表语句
        db.execSQL(CREATE_BOOK);
        db.execSQL(CREATE_CATEGORY);
        Toast.makeText(mContext, "Created succeeded", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("drop table if exists Book");
        db.execSQL("drop table if exists Category");
        onCreate(db);
    }
}
```

```
package com.example.databasetest;

import androidx.appcompat.app.AppCompatActivity;

import android.os.Bundle;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    private MyDatabaseHelper dpHelper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        dpHelper =new MyDatabaseHelper(this,"BookStore.db",null,2);
        Button createDatabase= (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dpHelper.getWritableDatabase();
            }
        });
    }
}
```

### 6.4.3 添加数据

```
insert 添加数据
		第一个参数 表名
		第二个参数 未指定添加数据下给某些可为空的列自动赋值
		第三个参数 ContentValues对象 有系列的put（）方法重载
```

```
                SQLiteDatabase db =dpHelper.getWritableDatabase();
                ContentValues values=new ContentValues();
                //开始组装第一条数据
                values.put("name","The Da Vinci Code");
                values.put("author","Dan");
                values.put("pages",454);
                values.put("prices",1654);
                db.insert("Book",null,values);
                values.clear();
                //第二组
                values.put("name","The  Vinci Code");
                values.put("author","Dn");
                values.put("pages",45);
                values.put("prices",154);
                db.insert("Book",null,values);
            }
```

### 6.4.4 更新数据

​	 

```
updata()  第一个参数 表名
		 第二个参数 ContentValues对象
		 第三个，四个参数 用于约束更新某一行或某几行数据，如果不指定默认更新所有
	Button updataData=(Button) findViewById(R.id.updata_data);
        updataData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db= dpHelper.getWritableDatabase();
                ContentValues values=new ContentValues();
                values.put("price",10.99);
                db.update("Book",values,"name=?"
                        ,new String[]{"The Da Vinci Code"});
            }
        });
```

### 6.4.5 删除数据

```
 delete() 第一个参数 表名
 		  第二个 三个 用于约束删除哪一行或者某几行，不指定默认删除所有
 		          Button deleteData=(Button) findViewById(R.id.delete_data);
        deleteData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db= dpHelper.getWritableDatabase();
                db.delete("Book","pages>?",new String[]{"30"});
            }
        });
```

### 6.4.6 查询数据

```
query() 第一个参数 表名
		第二个参数 用于指定查询第几列，不指定默认查询所有列
		第三个 四个参数用于约束查询第几行数据
		第五个参数 用于指定需要去group by的列，不指定则表示不需要对查询结果进行group by操作
		第六个参数 用于对group by之后数据进行进一步的过滤，不指定则表示不进行过滤
		第七个参数 用于指定查询结果的排序方式
	返回一个Cursor对象
```

```
        Button queryData=(Button) findViewById(R.id.query_data);
        queryData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                SQLiteDatabase db=dpHelper.getWritableDatabase();
                //查询Book表中的所有数据
                Cursor cursor= db.query("Book",null,
                        null,null,null,
                        null,null);
                //cursor.moveToFirst() 指针移动到第一行的位置
                if(cursor.moveToFirst()){
                    do{
                        //遍历Cursor 对象，取出数据并打印
                        //cursor.getColumnIndex 获取某一列在表中对应的位置索引
                        @SuppressLint("Range") String name= cursor.getString
                                (cursor.getColumnIndex("name"));
                        @SuppressLint("Range") String author=cursor.getString
                                (cursor.getColumnIndex("author"));
                        @SuppressLint("Range") int pages=cursor.getInt
                                (cursor.getColumnIndex("pages"));
                        @SuppressLint("Range") double price=cursor.getDouble
                                (cursor.getColumnIndex("price"));
                        Log.d("MainActivity", "book name is "+name);
                        Log.d("MainActivity", "book author is "+author);
                        Log.d("MainActivity", "book pages is "+pages);
                        Log.d("MainActivity", "book price is "+price);
                    }while(cursor.moveToNext());
                }
                //关闭cursor
                cursor.close();
            }
        });
```

### 6.4.7 SQL数据操作

## 6.5 LitePal操作数据库

### 6.5.1 简介

​	开源库-LitePal  一款开源的Android数据库框架，采用了对象映射关系（ORM）模式，将我们平时开发常用的一些数据库功能进行了封装

### 6.5.2 配置LitePal

 	1 导入依赖库

```
implementation 'org.litepal.guolindev:core:3.2.3'
```

​	2 在main 目录下新建一个 assets目录，然后在这个目录下，新建一个litepal.xml文件

```
<?xml version="1.0" encoding="utf-8" ?>
<litepal>
    <dbname value="Data" />  <!--数据库名-->
    <version value="1" />  <!--数据库版本号-->
    <!--list用于指定映射模型-->
    <list>

    </list>
</litepal>

```

​	3 修改AndroidManifest代码

```
android:name="org.litepal.LitePalApplication"
```

​	4 新版本中jencter已经被取消了 考虑使用镜像

```
 在settings.gradle 的repositories 中加入
        jcenter()
        maven{ url 'https://maven.aliyun.com/repository/jcenter'}
```

 

### 6.5.3 创建和升级数据库

​	1 新建类 Book

```
package com.example.litepaltest;

public class Book {
    private int id;
    private String author;
    private double price;
    private int pags;
    private String names;
    public int getId(){
        return id;
    }
    public void setId(int id){
        this.id=id;
    }
    public String getAuthor(){
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public double getPrice() {
        return price;
    }

    public void setPrice(double price) {
        this.price = price;
    }

    public int getPags() {
        return pags;
    }

    public void setPags(int pags) {
        this.pags = pags;
    }

    public String getNames() {
        return names;
    }

    public void setNames(String names) {
        this.names = names;
    }
}

```

​	2 将Book添加到映射模型中 修改litepal.xml

```
<?xml version="1.0" encoding="utf-8" ?>
<litepal>
    <dbname value="Data" />  <!--数据库名-->
    <version value="1" />  <!--数据库版本号-->
    <!--list用于指定映射模型-->
    <list>
    //<mapping> 声明要配置的映射模型类 
        <mapping class="com.example.litepaltest.Book"></mapping>
    </list>
</litepal>

```

​	3 任意进行一次数据库操作，BookStore.db数据就会自动创建

```
	protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button createDatabase= (Button) findViewById(R.id.create_database);
        createDatabase.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //点击一下，数据库自动创建
                LitePal.getDatabase();
            }
        });
    }
```

### 6.5.4 使用LitePal 添加数据

​	进行CRUD操作，需要让类表继承LitePalSupport

```
        Button addData=(Button) findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Book book=new Book();
                book.setAuthor("DAN");
                book.setPags(456);
                book.setNames("The love");
                book.setPrice(46);
                book.setPress("Unknow");
                //简单利用save就可以了
                book.save();
            }
        });
```

### 6.5.5 更新数据

#### 	1 对已存储的对象进行更新

```
        Button updataData=(Button) findViewById(R.id.updata_data);
        updataData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //对已存储对象进行更新
                Book book=new Book();
                book.setAuthor("DANd");
                book.setPags(454);
                book.setNames("The love for you");
                book.setPrice(496);
                book.setPress("Unknow");
                book.save();
                book.setPrice(592);
                book.save();
            }
        });
```

#### 	 2 updateAll 可以指定一个约束条件，如果不指定的话，默认更新所有

```
        Button updataData=(Button) findViewById(R.id.updata_data);
        updataData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Book book=new Book();
                book.setPrice(56);
                book.setPress("ANCHOR");
                //updateAll 可以指定一个约束条件，如果不指定的话，默认更新所有
                book.updateAll("name = ? and author = ?","The love for you",
                        "DANd");
            }
        });
    }
```

​	

```
 所有数据更新成默认值
                Book book=new Book();
                //setToDefault （） 参数 列名
                book.setToDefault("pags");
                //updateAll 可以指定一个约束条件，如果不指定的话，默认更新所有
                book.updateAll();
```

### 6.5.6 删除数据



```
        Button deleteData=(Button) findViewById(R.id.delete_data);
        deleteData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
            // LitePal.deleteAll 指定约束条件 
            					 不指定的话，删除所有数据
                LitePal.deleteAll(Book.class,"price < ?","53");
            }
        });
```

### 6.5.7 查询数据

```
List<Book> books=LitePal.findAll(Book.class);
        Button queryData =(Button) findViewById(R.id.query_data);
        queryData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                List<Book> books=LitePal.findAll(Book.class);
                for(Book book:books){
                    Log.d("MainActivity", "book name is "+book.getNames());
                    Log.d("MainActivity", "book author is "+book.getAuthor());
                    Log.d("MainActivity", "book press is "+book.getPress());
                    Log.d("MainActivity", "book pags is "+book.getPags());
                    Log.d("MainActivity", "book price is "+book.getPrice());
                }
            }
        });
 //查询book表的第一条数据
 Book books=LitePal.findFirst(Book.class);
 //最后一条数据
 Book books=LitePal.findLast(Book.class);
 //通过select 关键字只查询 name 和 author 这两列
 List<Book> books=LitePal.select("names","author").find(Book.class);
 //where（） 指定查询的约束条件
  List<Book> books=LitePal.where("price > ?","400").find(Book.class);
 // order() 用于指定结果的排序方式 desc降序  不写或者asc  表示升序
 List<Book> books=LitePal.order("price desc").find(Book.class);
 //Limit （） 方法用于查询结果的数量
 //offset（）用于指定查询结果的偏移量
```


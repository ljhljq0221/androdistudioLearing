# chapters7 

## 7.1 内容提供器简介

​	主要用于在不同的应用程序之间实现数据共享的功能，允许一个程序访问另一个程序中的数据，同时还能保证被访问数据的安全性，目前，使用内容提供器是Android 实现跨程序共享数据的标准方式。

## 7.2 运行时权限

### 7.2.1 Android 权限机制详解

### 7.2.2 程序运行时申请权限

​	CALL_PHONE 申请权限 

​	在安卓6.0以上的手机运行危险权限时都必须进行运行时权限处理

```
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button makeCall=(Button) findViewById(R.id.make_call);
        makeCall.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 判断用户是不是已经授权给我们了
                // ContextCompat.checkSelfPermission 第一个参数 context
                //                                    第二个参数 具体权限名
                //返回值与PackageManager.PERMISSION_GRANTED对比，相同已经授权，否则没有授权
                if(ContextCompat.checkSelfPermission(MainActivity.this,
                        Manifest.permission.CALL_PHONE)!=
                        PackageManager.PERMISSION_GRANTED){
                    //没有授权的话。需要调用ActivityCompat.requestPermissions 向用户申请授权
                    //第一个参数 Activity的实例 第二个参数是一个String 数组 第三个参数 请求码 为一组就可以
                    ActivityCompat.requestPermissions(MainActivity.this,
                            new String[]{Manifest.permission.CALL_PHONE},1);
                }else{
                    call();
                }
            }
        });
    }
    private void call(){
        try {
            //隐式Intent intent的action指定为 Intent.ACTION_CALL
            //             系统内置的打电话动作
            Intent intent= new Intent(Intent.ACTION_CALL);
            intent.setData(Uri.parse("tel:10086"));
            startActivity(intent);
        }
        catch (SecurityException e){
            e.printStackTrace();
        }
    }
    /*
        需要调用ActivityCompat.requestPermissions后，系统会弹出一个对话框，无论用户同意还是拒绝，都会
         回调onRequestPermissionsResult（）方法，授权的结果在grantResults中，只需要判断下授权结果，同意
         就会调用call（）拨打，不同意就说，没有权限。
    */
    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] ==
                        PackageManager.PERMISSION_GRANTED) {
                    call();
                } else {
                    Toast.makeText(this, "You denied the permission",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }
```

```
 <uses-permission android:name="android.permission.CALL_PHONE"/> 
 加入AndroidManifest中
```

## 7.3  访问其他程序中的数据

### 7.3.1 ContentResolver的基本用法

```
可通过Context中的getContentResolver()方法获取实例。ContentResolver 中提供了对数据的CRUD操作 insert updata delete query 
	使用一个Uri参数进行表名参数的替代 。
	内容URI 由两部分组成 authority 和path
		authority 对不同应用程序做区分，为了避免冲突，一般采用程序包名的方式进行命名
		path 对同一个应用程序的不同表进行区分，添加到authority 后面。
		此外，还需要在字符串的头部加上协议声明
		content：//com.example.app.provider/table
	有了URI字符串，还需要解析成Uri对象才可以作为参数传入。
	Uri uri= Uri.parse("content：//com.example.app.provider/table")
	随后就可以进行查询table1中的数据了
	Cursor cursor =getContentResolver.query(uri,//指定某个应用程序的某个表
							projection,//查询列名
							selection,//指定where的约束条件
							selectionArgs,//为where的占位符提供具体值
							sortOrder):// 指定查询结果的排序方式
```

```
查询完毕返回的是一个cursor对象，此时可将数据从Cursor中逐个取出 。读取的思路依然是通过移动游标的位置遍历Cursor的所有行，然后取出每行相应列的数据
 if(cursor!=null){
 	while(cursor.moveNext()){
 		String column1=cursor.getString(cursor.getColumnIndex("column1"));
 		int column2=cursor.getInt(cursor.getColumnIndex("column2"));
 	}
 	cursor.close();
 }
```

```
向table1中添加一条数据 
	ContentValues values =new ContenValues();
	values.put("column1","text")
	values.put("column2",1)
	getContentResolver.insert(uri,values)
```

```
更新数据
	ContentValues values =new ContenValues();
	values.put("column1"," ")
	getContentResolver.updata(uri,values,"column1=? and column2=?",
						new String[]{"text","1"} )
```

```
删除数据
		getContentResolver.delete(uri,values,"column1=? ",
						new String[]{"1"} )
```

### 7.3.2 读取系统联系人



```
public class MainActivity extends AppCompatActivity {
    ArrayAdapter<String> adapter;
    List<String> contactList= new ArrayList<>();
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ListView contactsView =(ListView) findViewById(R.id.contacts_view);
        adapter =new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1,
                contactList);
        contactsView.setAdapter(adapter);
        if(ContextCompat.checkSelfPermission(this,
                Manifest.permission.READ_CONTACTS)!=
                PackageManager.PERMISSION_GRANTED){
            ActivityCompat.requestPermissions(MainActivity.this,
                    new String[]{Manifest.permission.READ_CONTACTS},1);
            }else{
            readContacts();
          }

        }

        private void readContacts(){
            Cursor cursor=null;
            try {
   //ContactsContract.CommonDataKinds.Phone.CONTENT_URI
   //
                cursor= getContentResolver().query(ContactsContract.
                        CommonDataKinds.Phone.CONTENT_URI,null,null,null
                ,null);
                if(cursor!=null){
                    while(cursor.moveToNext()){
                        //获取联系人姓名
                        @SuppressLint("Range") String displayName =cursor.getString(cursor.getColumnIndex(
                                ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
                        Log.d("MainActivity", displayName);
                        //获取联系人手机号
                        @SuppressLint("Range") String number =cursor.getString(cursor.getColumnIndex(
                                ContactsContract.CommonDataKinds.Phone.NUMBER));
                        Log.d("MainActivity", number);
                        contactList.add(displayName+"\n"+number);
                    }
                    adapter.notifyDataSetChanged();
                }

            }catch (Exception e){
                e.printStackTrace();
            }finally {
                if(cursor!=null){
                    cursor.close();
                }
            }
        }

    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0 && grantResults[0] ==
                        PackageManager.PERMISSION_GRANTED) {
                    readContacts();
                } else {
                    Toast.makeText(this, "You denied the permission",
                            Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }
```

## 7.4 创建自己的内容提供器

### 7.4.1 创建内容提供器 步骤 

​	新建一个类进行继承ContentProvider 

```

public class MyProvider extends ContentProvider {
    // 初始化内容提供器的时候调用。在这里完成对数据库的创建和升级等操作，返回true表示内容提供器初始化成功，false失败
    @Override
    public boolean onCreate() {
        return false;
    }
	// 使用uri参数确定查询哪张表 projection 哪一列selection  selectionArgs 哪一行
		 sortOrder 对结果进行排序
    @Nullable
    @Override
    public Cursor query(@NonNull Uri uri, @Nullable String[] projection, @Nullable String selection, @Nullable String[]  selectionArgs, @Nullable String sortOrder) {
        return null;
    }
	// 添加数据 返回一个用于表示这条新纪录的URI
    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        return null;
    }
	// 更新数据  selection selectionArgs 约束哪些行
		受影响的行数将作为返回值进行返回
    @Override
    public int update(@NonNull Uri uri, @Nullable ContentValues values, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
	// 删除数据 被删除的行数将作为返回值
    @Override
    public int delete(@NonNull Uri uri, @Nullable String selection, @Nullable String[] selectionArgs) {
        return 0;
    }
	// 根据传入的内容URI 返回相应的MIME类型
    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }
}

```

```
一个标准的URI写法
	content://com.example.app.provider/table1
	还可在后面加一个ID 
	content://com.example.app.provider/table1/1
	表示访问com.example.app 这个应用的table1 表中id为1的数据 
	可以通配符分别匹配上述两种URI
	* 代表匹配任意长度的任意字符
	# 匹配任意长度的数字
		一个能够匹配任意表的内容URI格式可写为
		content://com.example.app.provider/*
		一个能匹配table1表中任意一行数据的内容URI格式为
		content://com.example.app.provider/table1/#
	随后借助UriMatcher类的就可轻松匹配内容Uri
	UriMatcher（）第一个参数 authority
				 第二个参数 path
				 第三个参数 自定义代码 
	返回值是某个能匹配这个Uri对象所对应的自定义代码，利用这个代码就可判断出调用方期望访问的是哪张表中的数据了
	
```

```
修改MyProvider 代码
	public class MyProvider extends ContentProvider {
    public static final int TABLE1_DIR=0;
    public static final int TABLE1_ITEM=1;
    public static final int TABLE2_DIR=2;
    public static final int TABLE2_ITEM=3;
    private static UriMatcher uriMatcher;
    static {
        uriMatcher=new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI("com.example.app.provider",
        "table1",TABLE1_DIR);
       uriMatcher.addURI("com.example.app.provider",
       "table1/#",TABLE1_ITEM);
        uriMatcher.addURI("com.example.app.provider",
        "table2",TABLE2_DIR);
        uriMatcher.addURI("com.example.app.provider",
        "table2/#",TABLE2_ITEM);
    }
    ...
    /*
    	当query（）方法调用时，通过UriMatch的match方法对传入的Uri对象进行匹配，通过返回的自定义代码就可判断出调用方的期望访问是什么数据
    	其余的insert updata delete 一样的
    */
        @Nullable
    @Override
    public Cursor query(@NonNull Uri uri,
    @Nullable String[] projection, 
    @Nullable String selection, 
    @Nullable String[] selectionArgs, 
    @Nullable String sortOrder) {
        switch (uriMatcher.match(uri)){
            case TABLE1_DIR:
                //查询table1中的所有数据
                break;
            case TABLE1_ITEM:
                //查询table1中的单行数据
                break;
            case TABLE2_DIR:
                //查询table2中的所有数据
                break;
            case TABLE2_ITEM:
                //查询table2中的单行数据
                break;
            default:
                break;
        }
        return null;
    }
    ...
   }
```

```
// 根据传入的内容URI 返回相应的MIME类型
    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        return null;
    }
    getType 所有内容提供器都必须提供的一个方法，用于获取Uri对象所对应的MIME类型，一个Uri对象所对应的MiME字符由3部分构成
    	1 必须以vnd开头
    	2 如果内容URI以路径结尾，则后接 android.cursor.dir/
    	  如果内容URI以id结尾，则后接 android.cursor.item/
    	3 最后接上vnd.<authority>.<path>
对于 content://com.example.app.provider/table1/1 这个内容URI，对应的MIME为
    vnd.android.cursor.item/vnd.com.app.provider.table1
```

```
继续完善代码  
	    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        switch (uriMatcher.match(uri)){
            case TABLE1_DIR:
                return "vnd.android.cursor.dir/vnd.com.app.provider.table1";
            case TABLE1_ITEM:
                return "vnd.android.cursor.item/vnd.com.app.provider.table1";
            case TABLE2_DIR:
                return "vnd.android.cursor.dir/vnd.com.app.provider.table2";
            case TABLE2_ITEM:
                return "vnd.android.cursor.item/vnd.com.app.provider.table2";
        }
        return null;
    }
```

### 7.4.2 实现跨程序数据共享

#### 	1 在DatabaseTest项目中修改代码

##### 		新建内容提供器 DatabasePrivider 

```
	修改DatabasePrivider 代码
	public class DatabaseProvider extends ContentProvider {
    public static final int BOOK_DIR=0;
    public static final int BOOK_ITEM=1;
    public static final int CATEGORY_DIR=2;
    public static final int CATEGORY_ITEM=3;
    public static final String AUTHORITY ="com.example.databasetest.provider";
    private static UriMatcher uriMatcher;
    private MyDatabaseHelper dpHelper;
    static {
        uriMatcher=new UriMatcher(UriMatcher.NO_MATCH);
        uriMatcher.addURI(AUTHORITY,"book",BOOK_DIR);
        uriMatcher.addURI(AUTHORITY,"book/#",BOOK_ITEM);
        uriMatcher.addURI(AUTHORITY,"category",CATEGORY_DIR);
        uriMatcher.addURI(AUTHORITY,"category/#",CATEGORY_ITEM);
    }
    public DatabaseProvider() {
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        // Implement this to handle requests to delete one or more rows.
        //删除数据
        SQLiteDatabase db=dpHelper.getWritableDatabase();
        int deleteRows=0;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                deleteRows =db.delete("Book",selection,selectionArgs);
                break;
            case BOOK_ITEM:
                String bookId=uri.getPathSegments().get(1);
                deleteRows =db.delete("Book","id = ?" ,
                        new String[]{bookId});
                break;
            case CATEGORY_DIR:
                deleteRows=db.delete("Category",selection,selectionArgs);
                break;
            case CATEGORY_ITEM:
                String categoryId =uri.getPathSegments().get(1);
                deleteRows=db.delete("Category","id = ?",
                        new String[]{categoryId});
                break;
            default:
                break;
        }
        return deleteRows;
    }

    @Override
    public String getType(Uri uri) {
        // TODO: Implement this to handle requests for the MIME type of the data
        // at the given URI.
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.book";

            case BOOK_ITEM:
                return "vnd.android.cursor.item/vnd.com.example.databasetest.provider.book";

            case CATEGORY_DIR:
                return "vnd.android.cursor.dir/vnd.com.example.databasetest.provider.category";

            case CATEGORY_ITEM:
                return "vnd.android.cursor.item/vnd.com.example.databasetest.provider.category";

        }
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {
        // TODO: Implement this to handle requests to insert a new row.
        //添加数据
        SQLiteDatabase db=dpHelper.getWritableDatabase();
        Uri uriReturn=null;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
            case BOOK_ITEM:
                long newBookId =db.insert("Book",null,values);
                uriReturn=Uri.parse("content://"+AUTHORITY+"/book/"+newBookId);
                break;
            case CATEGORY_DIR:
            case CATEGORY_ITEM:
                long newCategoryId =db.insert("Category",null,values);
                //Uri.parse（）可将一个内容解析成Uri对象
                uriReturn=Uri.parse("content://"+AUTHORITY+"/category/"+newCategoryId);
                break;
            default:
                break;
        }
        return uriReturn;
    }

    @Override
    public boolean onCreate() {
        // TODO: Implement this to initialize your content provider on startup.
        dpHelper =new MyDatabaseHelper(getContext(),"BookStore.db",null,2);
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                        String[] selectionArgs, String sortOrder) {
        // TODO: Implement this to handle query requests from clients.
       //查询数据、
        //获取SQLiteDatabase实例
        SQLiteDatabase db=dpHelper.getWritableDatabase();
        Cursor cursor=null;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                cursor=db.query("Book",projection,selection,selectionArgs,null
                ,null,sortOrder);
                break;
            case BOOK_ITEM:
                //uri.getPathSegments() 会将内容URI的权限之后的部门以“/”符合进行分割，并把分割过后的放在一个字符串中
                // 这个列表的第0个位置存放的就是路径，第一个位置就是id
                String bookId=uri.getPathSegments().get(1);
                cursor=db.query("Book",projection,"id=?",
                        new String[]{bookId},null,null,sortOrder);
                break;
            case CATEGORY_DIR:
                cursor=db.query("Category",projection,selection,selectionArgs,null
                        ,null,sortOrder);
                break;
            case CATEGORY_ITEM:
                String categoryId=uri.getPathSegments().get(1);
                cursor=db.query("Category",projection,"id=?",
                        new String[]{categoryId},null,null,sortOrder);
                break;
            default:
                break;
        }
        return  cursor;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
                      String[] selectionArgs) {
        // TODO: Implement this to handle requests to update one or more rows.
        //更新数据
        SQLiteDatabase db=dpHelper.getWritableDatabase();
        int updataedRows=0;
        switch (uriMatcher.match(uri)){
            case BOOK_DIR:
                updataedRows= db.update("Book",values,selection,selectionArgs);
                break;
            case BOOK_ITEM:
                String bookId=uri.getPathSegments().get(1);
                updataedRows=db.update("Book",values,"id = ?",
                        new String[]{bookId});
                break;
            case CATEGORY_DIR:
                updataedRows=db.update("Category",values,selection,selectionArgs);
                break;
            case CATEGORY_ITEM:
                String categoryId=uri.getPathSegments().get(1);
                updataedRows=db.update("Category",values,"id = ?",
                        new String[]{categoryId});
                break;
            default:
                break;
        }
        return updataedRows;
    }
}
```

##### 		 修改AndroidManifest代码

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.providertest">
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <!-- Android11新增 -->
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
    <queries>
        <provider android:authorities="com.example.databasetest.provider"/>
    </queries>
    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.ProviderTest"
        tools:targetApi="31">
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

#### 2  新建ProviderTest项目 

##### 	修改activity_main.xml代码

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/add_data"
        android:text="Add To Book"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/query_data"
        android:text="Query From Book"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/update_data"
        android:text="Update Book"/>
    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/delete_data"
        android:text="Delete From Book"/>
</LinearLayout>
```

```
修改MainActivity 代码
	package com.example.providertest;

import androidx.appcompat.app.AppCompatActivity;

import android.annotation.SuppressLint;
import android.content.ContentProvider;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;

public class MainActivity extends AppCompatActivity {
    private  String newId;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button addData =(Button) findViewById(R.id.add_data);
        addData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Uri uri=Uri.parse("content://com.example.databasetest.provider/book");
                ContentValues values=new ContentValues();
                values.put("name","A Clash of Kings");
                values.put("author","Martin");
                values.put("pages",1040);
                values.put("price",22.85);
                Uri newUri =getContentResolver().insert(uri,values);
                newId=newUri.getPathSegments().get(1);
            }
        });
        Button queryData=(Button) findViewById(R.id.query_data);
        queryData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Uri uri=Uri.parse("content://com.example.databasetest.provider/book");
                Cursor cursor =getContentResolver().query(uri,null,null,null,null);
                if(cursor!=null){
                    while(cursor.moveToNext()){
                        @SuppressLint("Range") String name=cursor.getString(cursor.getColumnIndex("name"));
                        @SuppressLint("Range") String author=cursor.getString(cursor.getColumnIndex("author"));
                        @SuppressLint("Range") int pages=cursor.getInt(cursor.getColumnIndex("pages"));
                        @SuppressLint("Range") double price=cursor.getDouble(cursor.getColumnIndex("price"));
                        Log.d("MainActivity", "book name is "+ name);
                        Log.d("MainActivity", "book author is "+ author);
                        Log.d("MainActivity", "pages name is "+ pages);
                        Log.d("MainActivity", "book price is "+ price);
                    }
                    cursor.close();
                }
            }
        });
        Button updateData=(Button) findViewById(R.id.update_data);
        updateData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Uri uri =Uri.parse("content://com.example.databasetest.provider/book/"+newId);
                ContentValues values=new ContentValues();
                values.put("name","A storm of Swords");
                values.put("pages",215);
                values.put("price",12.16);
                getContentResolver().update(uri,values,null,null);
            }
        });
        Button deleteData=(Button) findViewById(R.id.delete_data);
        deleteData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Uri uri =Uri.parse("content://com.example.databasetest.provider/book/"+newId);
                getContentResolver().delete(uri,null,null);
            }
        });
    }
}
```

#### 3 出现bug

##### 	项目中出现了错误

```
java.lang.IllegalArgumentException: Unknown URL content://com.example.databasetest.provider/book
        at android.content.ContentResolver.insert(ContentResolver.java:2145)
        at android.content.ContentResolver.insert(ContentResolver.java:2111)
        at com.example.providertest.MainActivity$1.onClick(MainActivity.java:32)
        at android.view.View.performClick(View.java:7448)
        at com.google.android.material.button.MaterialButton.performClick(MaterialButton.java:1119)
        at android.view.View.performClickInternal(View.java:7425)
        at android.view.View.access$3600(View.java:810)
        at android.view.View$PerformClick.run(View.java:28305)
        at android.os.Handler.handleCallback(Handler.java:938)
        at android.os.Handler.dispatchMessage(Handler.java:99)
        at android.os.Looper.loop(Looper.java:223)
        at android.app.ActivityThread.main(ActivityThread.java:7656)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:592)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:947)

```

##### 原因

```
一番研究后发现是因为测试用的模拟器的SDK是API 30的，该版本（Android 11）的更新中，改变了当前应用于本机其他应用进行交互的方式，由此若按照一些教程的例程去学习，会出现以上的一些访问权限问题。
```

##### 修改

```
在需要访问其他应用的Provider的应用的Manifest中加入元素，在该元素内声明想要访问的其他应用资源
	例如我的providertest应用需要访问databasetest内定义的Provider，那么我就在内写：
	<?xml version="1.0" encoding="utf-8"?>
<manifest 
          xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.providertest">

<!--    <uses-permission android:name="DatabaseProvider._READ_PERMISSION" />-->
<!--    <uses-permission android:name="DatabaseProvider._WRITE_PERMISSION" />-->

    <queries>
        <package android:name="com.example.databasetest" />
        
        <!-- 也可以单独指定provider -->
<!--        <provider android:authorities="com.example.databasetest.provider" />-->
    </queries>


    <application>
        ...
    </application>

</manifest>
```

- 在定义Provider的应用Manifest中声明可访问权限，然后在需要访问Provider的应用的Manifest中获取权限。

- 这是定义Provider的应用的Manifest

- ```
  <?xml version="1.0" encoding="utf-8"?>
  <manifest 
            xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.example.databasetest">
  
     <permission
         android:name="DatabaseProvider._READ_PERMISSION"
         android:protectionLevel="normal" />
  
     <permission
         android:name="DatabaseProvider._WRITE_PERMISSION"
         android:protectionLevel="normal" />
  
      <application ...>
          ...
          <provider
                  android:name=".DatabaseProvider"
                  android:authorities="com.example.databasetest.provider"
                  android:enabled="true"
                  android:exported="true">
          </provider>
          ...
      </application>
  
  </manifest>
  ```

  ## 7.5 GIt 时间-版本控制工具进阶

  ### 7.5.1 忽略文件 

  ```
  .gitignore 文件中的文件或者文件夹不会被添加到版本控制中，这个文件可以随意修改 
  
  ```

  ### 7.5.2 查看修改内容

  ```
  查看文件修改情况 
   git	status 
  查看文件更改的内容 diff
   git diff  可查看所有修改文件 
   git diff  app/src/main/java/com/example/providertest/MainActivity.java 查看某个修改文件
   	减号表示删除的部分
   	加号表示增加的部分
  ```

  ### 7.5.3 撤销未提交的修改 

  ```
  git checkout app/src/main/java/com/example/providertest/MainActivity.java 撤销尚未修改的代码
  ```

  ```
  取消添加的命令 
  	git reset HEAD app/src/main/java/com/example/providertest/MainActivity.java
  	随后在输入 git	status 
  			 git checkout app/src/main/java/com/example/providertest/MainActivity.java
  			  就可对内容进行修改了
  ```

  

### 	7.5.4 查看提交记录 

```
git log 
	如果提交记录过多，只想查看其中一条记录，可在命令行中指定该记录的id,并加上-1参数，表示我们只想看到一行记录
	git log e3156da0ef665ec6f84ab7bd8720b95ba5641359 -1
	如果想看这条提交记录具体修改了什么内容，可在命令行中加上 -p
	git log e3156da0ef665ec6f84ab7bd8720b95ba5641359 -1 -p
```


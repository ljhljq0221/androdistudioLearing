# chapters9

## 9.1 WebView的用法 

```

public class MainActivity extends AppCompatActivity {

    @SuppressLint("SetJavaScriptEnabled")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        WebView webView =(WebView) findViewById(R.id.web_view);
        //.getSettings().设置一些属性
        //setJavaScriptEnabled 让WebView支持JavaScript脚本
        webView.getSettings().setJavaScriptEnabled(true);
        //setWebViewClient 但一个网页跳转到另一个网页时，我们希望目标网页仍在当前WebView中显示，而不打开系统浏览器
        webView.setWebViewClient(new WebViewClient());
        //loadUrl 调入一个Uri  可展示相应的网页内容
        webView.loadUrl("http://www.baidu.com");
    }
}
```

```
声明权限
 <uses-permission android:name="android.permission.INTERNET"/>
```

## 9.2 HTTP协议访问网络

```
工作原理 客户端向服务器发出一条HTTP请求，服务端收到请求后会返回一些数据给客户端，然后客户端对这些数据进行解析和处理就可以。
```

```
向百度的服务器发起了一条HTTP请求，服务器分析出我们想访问的是百度的首页，于是把该网页的HTML代码进行返回，然后WebView在调用手机浏览器的内核对HTML进行解析，最后将页面进行展示
```

```
WebView 已经帮我们在后台处理好了发送Http请求，接受服务器响应，解析返回数据，以及最终的页面展示进行工作，
```

### 9.2.1 使用HttpURLConnection

```
        // 获取HttpURLConnection 实例
        // 一般来说 new出一个URL对象，并传入目标的网络地址，然后调用一下openConnection 方法就可以
        URL url= new URL("http://www.baidu.com");
        HttpURLConnection connection=(HttpURLConnection) url.openConnection();
        //得到HttpURLConnection 的实例之后，我们可以设置一下Http请求所使用的方法 ，有两个方法 GET POST
        //GET 表示 从服务器那里获取数据 POST则表示 希望提交数据给服务器
        connection.setRequestMethod("GET");
        //随后就可进行自由定制 比如连接超时 读取超时的毫秒数
        connection.setConnectTimeout(8000);
        connection.setReadTimeout(8000);
        //之后在调用getInputStream 方法就可以获取到服务器返回的输入流了，之后的认为就是对输入流进行读取
        InputStream in=connection.getInputStream();
        //最后调用disconnection（）方法将这个HTTP协议关闭掉 
        connection.disconnect();
* 提交数据给服务器 将HTTP请求码改为POST 并在获取输入流之前把要提交的数据写出即可 
* connection.setRequestMethod("POST");
  * 每条数据以键值对的形式存在，数据与数据之间用&隔开
  DataOutputStream out=new DataOutputStream(connection.getOutputStream());
 * out.writeBytes("username=admin&password=123456");
```

```
ScrollView 控件 可借助滚动的形式查看屏幕外的内容
```

```
public class MainActivity extends AppCompatActivity {

    TextView responseText;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button sendRequest=(Button) findViewById(R.id.send_request);
        sendRequest.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(v.getId()==R.id.send_request){
                    sendRequestWithHttpURLConnection();
                }
            }
        });
    }
    private void sendRequestWithHttpURLConnection(){
        //开启线程来发起网络请求
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection=null;
                BufferedReader reader=null;
                try {
                    URL url= new URL("http://www.baidu.com");
                    connection =(HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    InputStream in =connection.getInputStream();
                    //利用BufferedReader对获取的输入流进行读取，并将结果传入到showResponse
                    reader = new BufferedReader(new InputStreamReader(in));
                    StringBuilder response=new StringBuilder();
                    String line;
                    while((line= reader.readLine())!=null){
                        response.append(line);
                    }
                    showResponse(response.toString());
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    if(reader!=null){
                        try {
                            reader.close();
                        }catch (IOException e){
                            e.printStackTrace();
                        }
                    }
                    if (connection!=null){
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }
    private void  showResponse(final String response){
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                //这里面进行UI操作，将结果显示到界面上
                //android 不允许在子线程中进行UI操作，需要通过这个方法将线程切换到主线程，然后更新UI元素
                responseText =(TextView)findViewById(R.id.response_text);
                responseText.setText(response);
                System.out.println(responseText);
            }
        });
    }
}
```



#### 错误出现

```
Cleartext HTTP traffic to www.baidu.com not permitted
这是由于安卓高版本导致的联网问题
解决方法：
（1）APP改用https请求

（2）targetSdkVersion 降到27以下

（3）更改网络安全配置
（4）在AndroidManifest.xml配置文件的标签中直接插入
```

```
(3)更改网络安全配置
	1.在res文件夹下创建一个xml文件夹，然后创建一个network_security_config.xml文件，文件内容如下：
	<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
	2 在AndroidManifest.xml文件下的application标签增加以下属性：
	 android:networkSecurityConfig="@xml/network_security_config"
```

```
(4) 在AndroidManifest.xml配置文件的标签中直接插入 android:usesCleartextTraffic=“true”

```

### 9.2.2 使用OkHttp

```
 //依赖库
 implementation 'com.squareup.okhttp3:okhttp:3.10.0'
 //创建一个OkHttpClient实例
                    OkHttpClient client=new OkHttpClient();
//发起一条HTTP请求，需要创建一个Request对象，利用url方法设置目标的网络地址
   Request request=new Request.Builder().
                    url("http://www.baidu.com").build();
//之后调用OkHttpClient的newCall 方法创建一个Call对象，并调用他的execute 方法来发送请求
//并获取服务器返回的数据
     Response response=client.newCall(request).execute();
 //Response对象就是服务器返回的数据了 如下写法获取返回的具体内容
    String responseData =response.body().string();
 //如果是发起一条POST请求，需要构建一个RequestBody 对象来存放提交的参数 
 RequestBody requestBody=new FormBody.Builder().add("username","admin").add("password","123456").build();
//随后在Request.Builder中调用Post方法，并将RequestBody对象传入 
Request request1=new Request.Builder().url("http://www.baidu.com")
                            .post(requestBody).build();
```

## 9.3 解析XML格式数据 

网络上数据最常用的两种格式 ：XML 和JSON 

### 9.3.1 Pull 解析方式

```
    private void parseXMLWithPull(String xmlData){
        try {
            XmlPullParserFactory factory=XmlPullParserFactory.newInstance();
            XmlPullParser xmlPullParser=factory.newPullParser();
     //xmlPullParser.setInput 将服务器返回的XML数据设置进去进行解析
            xmlPullParser.setInput(new StringReader(xmlData));
            //getEventType 获取当前解析事件 
            int eventType =xmlPullParser.getEventType();
            String id="";
            String name="";
            String version="";
            while (eventType!=xmlPullParser.END_DOCUMENT){
                String nodeName =xmlPullParser.getName();
                switch (eventType){
                    //开始解析某个节点
                    case XmlPullParser.START_TAG:{
                        if("id".equals(nodeName)){
                            id=xmlPullParser.nextText();
                        }else if("name".equals(nodeName)){
                            name=xmlPullParser.nextText();
                        }else if("version".equals(nodeName)){
                            version =xmlPullParser.nextText();
                        }
                        break;
                    }
                    //完成解析某个节点
                         case XmlPullParser.END_TAG:{
                             if("app".equals(nodeName)){
                                 Log.d("MainActivity",id);
                             }
                             break;
                         }
                    default:
                        break;
                }
                eventType=xmlPullParser.next();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

### 9.3.2 SAX 解析方式 

​	新建类继承自DefaultHandler

```
package com.example.networktest;

import android.util.Log;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;

public class MyHandler extends DefaultHandler {
    private String nodeName;
    private StringBuilder id;
    private StringBuilder name;
    private StringBuilder version;
    
    @Override
    public void startDocument() throws SAXException {
        id=new StringBuilder();
        name=new StringBuilder();
        version=new StringBuilder();
    }

    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        //记录当前节点名
        nodeName=localName;
    }

    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        //根据当前节点名，判断将内添加到那以恶搞String对象中 
        if("id".equals(nodeName)){
            id.append(ch,start,length);
        }else if("name".equals(nodeName)){
            name.append(ch,start,length);
        }else if("version".equals(nodeName)){
            version.append(ch,start,length);
        }
    }

    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        //打印出来
        if("app".equals(nodeName)){
        //trim() 避免回车或者换行符 
            Log.d("MainActivity",id.toString().trim());
        }
        id.setLength(0);
    }

    @Override
    public void endDocument() throws SAXException {
        super.endDocument();
    }
}

```

```
    private void parseXMLWithSAX(String xmlData){
        try {
            SAXParserFactory saxParserFactory=SAXParserFactory.newInstance();
            XMLReader xmlReader=saxParserFactory.newSAXParser().getXMLReader();
            MyHandler handler=new MyHandler();
            //将ContenteHandler的实例设置到 XML对象中 
            xmlReader.setContentHandler(handler);
            xmlReader.parse(new InputSource(new StringReader(xmlData)));
        }catch(Exception e){
            e.printStackTrace();
        }
 
    }
```

## 9.4 解析JSON格式数据 

​	对比 XML，JSON体积更小，网络传输的时候更省流量 

```
[{"id":5,"version":"5.5","name":"Clash if Clans"},
{"id":3,"version":"5.2","name":"Clash  Clans"},
{"id":6,"version":"5.2","name":"Clash if "},]
```

### 9.4.1 使用JSONObject

```
    private void parseJSONWithJSONObject(String jsonData){
        try {
            //将服务器返回的数据传入待一个JSONArray 对象中
            //遍历循环这个对象，取出的每一个元素都是一个JSONObject 对象
            JSONArray jsonArray=new JSONArray(jsonData);
            for (int i=0;i<jsonArray.length();i++){
                JSONObject jsonObject= jsonArray.getJSONObject(i);
                String id= jsonObject.getString("id");
                String name=jsonObject.getString("name");
                String version=jsonObject.getString("version");
                Log.d("MainActivity","id is + " + id);
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
```

### 9.4.2 使用GSON

#### 	添加依赖库

```
implementation 'com.google.code.gson:gson:2.8.9'
```

​	可以将一段JSON格式的字符串自动映射成一个对象

例如一段JSON格式的数据如下

```
{“name":"tom","age":"20"}
```

​	就可以定义一个Person类，加入name和age两个字段，然后简单的调用如下代码就可以将其JSON数据解析成一个Person对象了

```
Gson gson =new Gson();
Person person=gson.fromJson(jsonData,Person.class);
```

如果需要解析的是一段JSON数据，需要借助 TypeToken 将期望解析成的数据类型传入到fromJson()方法中

```
List<Person> people =gson.fromJson(jsonData,new 
				TypeToken<List<Person>>(){}.getType());
```

```
    private void parseJSONWithGSON(String jsonData){
        Gson gson= new Gson();
        List<App> apps =gson.fromJson(jsonData,new TypeToken<List<App>>(){}.getType());
        for(App app:apps){
            Log.d("MainActivity","id is + " + app.getId());
        }
    }

```

## 9.5 网络编程的最佳实践

 	通常将这些同意的网络操作提取到一个公共的类里，并提供一个静态方法，当想发起网络请求的时候，调用一下这个方法就可以 

### 	9.5.1 sendHttpRequest

```
package com.example.networktest;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class HttpUtil {
    public static  String sendHttpRequest(String address){
        HttpURLConnection connection=null;
        try {
            URL url=new URL(address);
            connection=(HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setConnectTimeout(8000);
            connection.setReadTimeout(8000);
            connection.setDoInput(true);
            connection.setDoOutput(true);
            InputStream in=connection.getInputStream();
            BufferedReader reader=new BufferedReader(new InputStreamReader(in));
            StringBuilder response =new StringBuilder();
            String line;
            while ((line =reader.readLine())!=null){
                response.append(line);
            }
            return response.toString();
        }catch (Exception e){
            e.printStackTrace();
            return  e.getMessage();
        }finally {
            if(connection!=null) {
                connection.disconnect();
            }
        }
    }
}

```

​	后续调用的时候之间这样使用

```
String address="http://www.baidu.com";
Steing response=HttpUtil.sendHttpRequest(address);
```

​	网络请求属于耗时操作，而 sendHttpRequest 方法内部没有开启线程，这样可能在调用的时候是的主线程被阻塞住。而直接在sendHttpRequest  方法内部开启线程发起HTTP请求，由于所有的耗时逻辑都是在子线程里进行的，sendHttpRequest 会在服务器还没来得及响应时候就执行结束了。

​	可采用JAVA的回调机制

​	首先定义一个接口， HttpCallbackListener

```
public interface HttpCallbackListener {
    // 表示当服务器成功响应我们请求的时候调用，参数为服务器返回的数据
    void onFinish(String response);
    // 表示网络操作出现错误的时候调用，参数记录着详细的错误信息
    void onError(Exception e);
}
```

​	然后在sendHttpRequest 方法里添加一个HttpCallbackListener 参数，并在方法内部开启子线程，在子线程里进行具体网络操作。子线程是无法通过return语句返回数据的，这里我们将服务器返回的数据传入了onFinish方法中，如果出现异常就将异常原因传入到onError 

```
public class HttpUtil {
    public static  void sendHttpRequest(final String address,final HttpCallbackListener listener){
        new Thread(new Runnable() {
            @Override
            public void run() {
                HttpURLConnection connection=null;
                try {
                    URL url=new URL(address);
                    connection=(HttpURLConnection) url.openConnection();
                    connection.setRequestMethod("GET");
                    connection.setConnectTimeout(8000);
                    connection.setReadTimeout(8000);
                    connection.setDoInput(true);
                    connection.setDoOutput(true);
                    InputStream in=connection.getInputStream();
                    BufferedReader reader=new BufferedReader(new InputStreamReader(in));
                    StringBuilder response =new StringBuilder();
                    String line;
                    while ((line =reader.readLine())!=null){
                        response.append(line);
                    }
                    if(listener!=null){
                        //调用onFinish方法
                        listener.onFinish(response.toString());
                    }
                }catch (Exception e){
                    //回调onError方法
                    if(listener!=null){
                        listener.onError(e);
                    }
                }finally {
                    if(connection!=null) {
                        connection.disconnect();
                    }
                }
            }
        }).start();
    }
}
```

​	sendHttpRequest 需要两个参数，我们在调用的时候还需要将HttpCallbackListener的 实例传入，如下

```
HttpUtil.sendHttpRequest(address,new HttpCallbackListener(){
	@override
	public void onFinish(String response){
		//根据返回内容执行具体逻辑
	}
		public void onError(Expection e){
		//对异常情况进行处理 
	}
});
```

### 9.5.2 sendOkHttpRequest

```
    //okhttp3.Callback 是okhttp库中自带的一个回调接口
    public static  void sendOkHttpRequest(String address,okhttp3.Callback callback){
        OkHttpClient client=new OkHttpClient();
        Request request=new Request.Builder().url(address).build();
        //okhttp在enqueue（）方法的内部已经帮我们开好子线程了 ，然后会在子线程里进行HTTP请求  
        client.newCall(request).enqueue(callback);
    }
```

​	在调用 sendOkHttpRequest 方法可以这样写

```
HttpUtil.sendHttpRequest("http://www.baidu.com",new okhttps.Callback(){
	@override
	public void onResponse (Call call,Response response)throws 
	IOException{
		//得到服务器返回的内容
		String responseData= reponse.body().string();
	}
	@override
	public void onFailure(Call call,IOException e){
		// 对异常情况进行处理 
	}
})
```


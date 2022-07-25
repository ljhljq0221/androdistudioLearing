# chapters14

## 14.3 创建数据库和表

```
db包用于存放数据库模型相关代码
gson包用于存放GSON模型相关代码
service包用于存放服务相关代码
util包用于存放工具相关代码
```

### 导入依赖

#### LitePal

```
implementation 'org.litepal.guolindev:core:3.2.3'

在settings.gradle 的repositories 中加入
        jcenter()
        maven{ url 'https://maven.aliyun.com/repository/jcenter'}
```

#### OkHttp

```
implementation 'com.squareup.okhttp3:okhttp:3.10.0'
```

#### GSON

```
implementation 'com.google.code.gson:gson:2.8.9'
```

#### Glide

```
implementation 'com.github.bumptech.glide:glide:4.4.0'
```

### 建表

#### 省表 Province

```
package com.coolweather.android.db;

import org.litepal.crud.LitePalSupport;

public class Province extends LitePalSupport {

    private  int id;

    private String provinceName;

    private int provinceCode;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getProvinceName() {
        return provinceName;
    }

    public void setProvinceName(String provinceName) {
        this.provinceName = provinceName;
    }

    public int getProvinceCode() {
        return provinceCode;
    }

    public void setProvinceCode(int provinceCode) {
        this.provinceCode = provinceCode;
    }
}

```

#### bug DataSupport 被遗弃

```
用LitePalSupport 替代DataSupport
```

#### 市表City

```
package com.coolweather.android.db;

import org.litepal.crud.LitePalSupport;

public class City extends LitePalSupport {

    private int id;

    private String cityName;

    private int cityCode;

    private int provinceId;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getCityName() {
        return cityName;
    }

    public void setCityName(String cityName) {
        this.cityName = cityName;
    }

    public int getCityCode() {
        return cityCode;
    }

    public void setCityCode(int cityCode) {
        this.cityCode = cityCode;
    }

    public int getProvinceId() {
        return provinceId;
    }

    public void setProvinceId(int provinceId) {
        this.provinceId = provinceId;
    }

}
```

#### 县表

```
package com.coolweather.android.db;

import org.litepal.crud.LitePalSupport;

public class County extends LitePalSupport {
    
    private int id;
    
    private String countyName;
    
    private String weatherId;
    
    private int cityId;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getCountyName() {
        return countyName;
    }

    public void setCountyName(String countyName) {
        this.countyName = countyName;
    }

    public String getWeatherId() {
        return weatherId;
    }

    public void setWeatherId(String weatherId) {
        this.weatherId = weatherId;
    }

    public int getCityId() {
        return cityId;
    }

    public void setCityId(int cityId) {
        this.cityId = cityId;
    }
}
```

### 配置litepal.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<litepal>
//数据库指定为cool_weather
    <dbname value="cool_weather"/>

    <version value="1"/>
    // 将三个实体类添加到映射列表中

    <list>
        <mapping class="com.coolweather.android.db.Province"/>
        <mapping class="com.coolweather.android.db.City"/>
        <mapping class="com.coolweather.android.db.County"/>
    </list>
</litepal>
```

### 配置LitePalApplication

```
    <application
        android:name="org.litepal.LitePalApplication"
```

### 第一阶段提交

```
git add .

git commit -m "加入创建数据库和表的各项配置"

git push origin main
```

## 14.4 遍历全国省市县数据 

### 与服务器的交互 

在util 包下增加一个HttpUtil 类

```
package com.coolweather.android.util;

import okhttp3.OkHttpClient;
import okhttp3.Request;

public class HttpUtil {

    public static void sendOkHttpRequest(String address,okhttp3.Callback callback){
        OkHttpClient client=new OkHttpClient();
        Request request=new Request.Builder().url(address).build();
        ////okhttp在enqueue（）方法的内部已经帮我们开好子线程了 ，然后会在子线程里进行HTTP请求  
        client.newCall(request).enqueue(callback);
    }

}
```

### 新建工具类 Utility 处理JSON格式数据

```
package com.coolweather.android.util;

import android.text.TextUtils;

import com.coolweather.android.db.City;
import com.coolweather.android.db.County;
import com.coolweather.android.db.Province;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

public class Utility {
    /*
        解析和处理服务器返回的省级数据
     */
    public static boolean handleProvinceResponse(String response){
        if(!TextUtils.isEmpty(response)){
            try {
                JSONArray allProvinces=new JSONArray(response);
                for(int i=0;i<allProvinces.length();i++){
                    JSONObject provinceObject= allProvinces.getJSONObject(i);
                    Province province=new Province();
                    province.setProvinceName(provinceObject.getString("name"));
                    province.setProvinceCode(provinceObject.getInt("id"));
                    province.save();
                }
                return  true;
            }catch (JSONException e){
                e.printStackTrace();
            }
        }
        return false;
    }
    
    /*
        解析和处理服务器返回的市级数据
     */
    public static boolean handleCityResponse(String response,int provinceId){
        if(!TextUtils.isEmpty(response)){
            try {
                JSONArray allCities=new JSONArray(response);
                for(int i=0;i<allCities.length();i++){
                    JSONObject cityObject= allCities.getJSONObject(i);
                    City city=new City();
                    city.setCityName(cityObject.getString("name"));
                    city.setCityCode(cityObject.getInt("id"));
                    city.setProvinceId(provinceId);
                    city.save();
                }
                return  true;
            }catch (JSONException e){
                e.printStackTrace();
            }
        }
        return false;
    }
    
    /*
        解析和处理服务器返回的县级数据
     */
    public static boolean handleCountyResponse(String response,int cityId){
        if(!TextUtils.isEmpty(response)){
            try {
                JSONArray allCounties=new JSONArray(response);
                for(int i=0;i<allCounties.length();i++){
                    JSONObject countyObject= allCounties.getJSONObject(i);
                    County county=new  County();
                    county.setCountyName(countyObject.getString("name"));
                    county.setWeatherId(countyObject.getString("weather_id"));
                    county.setCityId(cityId);
                    county.save();
                }
                return  true;
            }catch (JSONException e){
                e.printStackTrace();
            }
        }
        return false;
    }
}

```

### 书写界面 choose_area.xml

```
	定义了一个头布局进行作为标题栏。布局高度为actionBar高度，背景为colorPrimary，头布局中 TextView 用于显示标题内容。放置了一个Button 用于执行返回操作 
```

```
头布局的下面定义了一个 ListView 省市县的数据将在这里进行显示 
使用 ListView 为自动为每个子项添加一条分割线 
```



```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#fff">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/title_text"
            android:layout_centerInParent="true"
            android:textColor="#fff"
            android:textSize="20sp"/>

        <Button
            android:layout_width="25dp"
            android:layout_height="25dp"
            android:id="@+id/back_button"
            android:layout_marginLeft="10dp"
            android:layout_alignParentLeft="true"
            android:layout_centerVertical="true"
            android:background="@drawable/ic_backup"/>
    </RelativeLayout>

    <ListView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/list_view"/>

</LinearLayout>
```

### 用于遍历省市县数据的碎片

#### onActivityCreated 被弃用

#### 解决方法

```
 @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);

        requireActivity().getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
                if (event.getTargetState() == Lifecycle.State.CREATED){
                    //在这里任你飞翔
                  

                    getLifecycle().removeObserver(this);  //这里是删除观察者
                }
            }
        });
    }
```



```
public class ChooseAreaFragment extends Fragment {

    public static final int LEVEL_PROVINCE=0;

    public static final int LEVEL_CITY=1;

    public static final int LEVEL_COUNTY=2;

    private ProgressDialog progressDialog;

    private TextView titleText;

    private Button backButton;

    private ListView listView;

    private ArrayAdapter<String> adapter;

    private List<String> dataList=new ArrayList<>();

    /**
     * 省列表
     */
    private List<Province> provinceList;

    /**
     * 市列表
     */
    private List<City> cityList;

    /**
     * 县列表
     */
    private List<County> countyList;

    /**
     * 选中的省份
     */
    private  Province selectedProvince;

    /**
     * 选中的城市
     */
    private City selectedCity;

    /**
     * 当前选中的级别
     */
    private int currentLevel;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {

        View view=inflater.inflate(R.layout.choose_area,container,false);
        titleText=(TextView) view.findViewById(R.id.title_text);
        backButton=(Button) view.findViewById(R.id.back_button);
        listView=(ListView) view.findViewById(R.id.list_view);
        adapter=new ArrayAdapter<>(getContext(), android.R.layout.simple_list_item_1,dataList);
        listView.setAdapter(adapter);
        return view;

    }

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);

        requireActivity().getLifecycle().addObserver(new LifecycleEventObserver() {
            @Override
            public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Lifecycle.Event event) {
                if (event.getTargetState() == Lifecycle.State.CREATED){
                    //在这里任你飞翔
                    listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                        @Override
                        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                            if(currentLevel==LEVEL_PROVINCE) {
                                selectedProvince=provinceList.get(position);
                                queryCities();
                            }else if(currentLevel==LEVEL_CITY){
                                selectedCity=cityList.get(position);
                                queryCounties();
                            }
                        }
                    });
                    backButton.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            if(currentLevel==LEVEL_COUNTY){
                                queryCities();
                            }else if(currentLevel==LEVEL_CITY){
                                queryProvinces();
                            }
                        }
                    });
                    queryProvinces();
                    getLifecycle().removeObserver(this);  //这里是删除观察者
                }
            }
        });
    }

    /**
     * 查询全国所有的省，优先从数据库查询，如果没有查询到再去服务器上查询
     * 
     */
    private void queryProvinces(){
        titleText.setText("中国");
        backButton.setVisibility(View.GONE);
        provinceList = LitePal.findAll(Province.class);
        if(provinceList.size()>0){
            dataList.clear();
            for(Province province:provinceList){
                dataList.add(province.getProvinceName());
            }
            adapter.notifyDataSetChanged();
            listView.setSelection(0);
            currentLevel=LEVEL_PROVINCE;
        }else {
            String address="http://guolin.tech/api/china";
            queryFromServer(address,"province");
        }
    }

    /**
     * 查询全国所有的市，优先从数据库查询，如果没有查询到再去服务器上查询
     *
     */
    private void queryCities(){
        titleText.setText(selectedProvince.getProvinceName());
        backButton.setVisibility(View.VISIBLE);
        cityList = LitePal.where("provinced=?",String.valueOf(selectedProvince.getId())).find(City.class);
        if(cityList.size()>0){
            dataList.clear();
            for(City city:cityList){
                dataList.add(city.getCityName());
            }
            adapter.notifyDataSetChanged();
            listView.setSelection(0);
            currentLevel=LEVEL_CITY;
        }else {
            int provinceCode=selectedProvince.getProvinceCode();
            String address="http://guolin.tech/api/china/"+provinceCode;
            queryFromServer(address,"city");
        }
    }

    /**
     * 查询全国所有的县，优先从数据库查询，如果没有查询到再去服务器上查询
     *
     */
    private void queryCounties(){
        titleText.setText(selectedCity.getCityName());
        backButton.setVisibility(View.VISIBLE);
        countyList = LitePal.where("cityid=?",String.valueOf(selectedCity.getId())).find(County.class);
        if(countyList.size()>0){
            dataList.clear();
            for(County county:countyList){
                dataList.add(county.getCountyName());
            }
            adapter.notifyDataSetChanged();
            listView.setSelection(0);
            currentLevel=LEVEL_COUNTY;
        }else {
            int provinceCode=selectedProvince.getProvinceCode();
            int cityCode=selectedCity.getCityCode();
            String address="http://guolin.tech/api/china/"+provinceCode+"/"+cityCode;
            queryFromServer(address,"county");
        }
    }
    
    
    /**
     * 根据传入的地址和类型从服务器上查询省市县数据
     */
    private void queryFromServer(String address, final String type) {
        
        showProgressDialog();
        HttpUtil.sendOkHttpRequest(address, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                //通过runOnUiThread 方法回到主线程处理逻辑
                getActivity().runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        closeProgressDialog();
                        Toast.makeText(getContext(), "加载失败", Toast.LENGTH_SHORT).show();
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                String responseText =response.body().string();
                boolean result=false;
                if("province".equals(type)){
                    result= Utility.handleProvinceResponse(responseText);
                }else if("city".equals(type)){
                    result=Utility.handleCityResponse(responseText,selectedProvince.getId());
                }else if("county".equals(type)){
                    result=Utility.handleCountyResponse(responseText,selectedCity.getId());
                }
                if(result){
                    getActivity().runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            closeProgressDialog();
                            if("province".equals(type)) {
                                queryProvinces();
                            }else if("city".equals(type)) {
                                queryCities();
                            }else if("county".equals(type)){
                                queryCounties();
                            }
                        }
                    });
                }
            }
        });
    }
    
    
    /**
     * 显示进度对话框
     */
    private void showProgressDialog() {
        if(progressDialog==null){
            progressDialog=new ProgressDialog(getActivity());
            progressDialog.setMessage("正在加载...");
            progressDialog.setCanceledOnTouchOutside(false);
        }
        progressDialog.show();
    }

    /**
     * 关闭对话进度框
     */
    
    private void closeProgressDialog(){
        if(progressDialog!=null){
            progressDialog.dismiss();
        }
    }
}

```

### 修改主布局

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/choose_area_fragment"
        android:name="com.coolweather.android.ChooseAreaFragment"/>

</FrameLayout>
```

### 修改主题

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
       <style name="Theme.CoolWeather" parent="Theme.AppCompat.Light.NoActionBar">

    </style>
</resources>
```

### 声明许可

```
<uses-permission android:name="android.permission.INTERNET"/>
```

#### bug  org.json.JSONException: Value <!DOCTYPE of type java.lang.String cannot be converted to JSONArray

```
JSON 格式不对 
```

#### 解决

```
2022-07-14 07:45:51.542 26611-26664/com.coolweather.android D/responseone: Response{protocol=http/1.1, code=200, message=OK, url=http://guolin.tech/api/china5}
地址格式错误
```

#### 模拟器显示加载失败

```
是因为从Android 6.0开始引入了对Https的推荐支持，而到了Android 9.0的系统上面默认所有Http的请求都无法响应，解决办法如下 
```

#### 解决办法

```
 在AndroidManifest.xml配置文件的标签中直接插入 android:usesCleartextTraffic=“true”
```

### 第二阶段提交

```
git add .
git commit -m "完成省市县三级列表功能"
git push origin main 
```

### 14.5 显示天气信息

### 14.5.1 定义GSON实体类 

​	借助GSON进行 对天气进行解析 

​	返回数据类型格式

```
{"HeWeather":[{"basic":{"cid":"CN101190401","location":"苏州","parent_city":"苏州","admin_area":"江苏","cnty":"中国","lat":"38.91458893","lon":"121.61862183","tz":"+8.00","city":"苏州","id":"CN101190401","update":{"loc":"2022-07-13 09:26","utc":"2022-07-13 09:26"}},"update":{"loc":"2022-07-13 09:26","utc":"2022-07-13 09:26"},"status":"ok","now":{"cloud":"10","cond_code":"100","cond_txt":"晴","fl":"3","hum":"28","pcpn":"0.0","pres":"1014","tmp":"6","vis":"16","wind_deg":"183","wind_dir":"南风","wind_sc":"2","wind_spd":"8","cond":{"code":"100","txt":"晴"}},"daily_forecast":[{"date":"2022-07-14","cond":{"txt_d":"小雨"},"tmp":{"max":"9","min":"2"}},{"date":"2022-07-15","cond":{"txt_d":"多云"},"tmp":{"max":"8","min":"1"}},{"date":"2022-07-16","cond":{"txt_d":"晴"},"tmp":{"max":"8","min":"3"}},{"date":"2022-07-17","cond":{"txt_d":"中雨"},"tmp":{"max":"9","min":"2"}},{"date":"2022-07-18","cond":{"txt_d":"阵雨"},"tmp":{"max":"12","min":"4"}},{"date":"2022-07-19","cond":{"txt_d":"晴"},"tmp":{"max":"10","min":"3"}}],"aqi":{"city":{"aqi":"36","pm25":"18","qlty":"优"}},"suggestion":{"comf":{"type":"comf","brf":"较舒适","txt":"白天有雨，风力较强，这种天气条件下，人们会感到有些凉意，但大部分人完全可以接受。"},"sport":{"type":"sport","brf":"较不宜","txt":"有降水，且风力较强，推荐您在室内进行各种健身休闲运动；若坚持户外运动，请注意防风保暖。"},"cw":{"type":"cw","brf":"不宜","txt":"不宜洗车，未来24小时内有雨，如果在此期间洗车，雨水和路上的泥水可能会再次弄脏您的爱车。"}},"msg":"所有天气数据均为模拟数据，仅用作学习目的使用，请勿当作真实的天气预报软件来使用。"}]}
```

#### basic 类

```
"basic":
{"cid":"CN101190401",
"location":"苏州",
"parent_city":"苏州",
"admin_area":"江苏","cnty":"中国","lat":"38.91458893",
"lon":"121.61862183",
"tz":"+8.00","city":"苏州",
"id":"CN101190401",
"update":{"loc":"2022-07-13 09:26","utc":"2022-07-13 09:26"}},"update":{"loc":"2022-07-13 09:26","utc":"2022-07-13 09:26"}
```

```
package com.coolweather.android.gson;

import com.google.gson.annotations.SerializedName;

public class Basic {
    @SerializedName("city")
    public String cityName;

    @SerializedName("id")
    public String weatherId;

    public Update update;

    public class Update{
        @SerializedName("loc")
        public String updateTime;
    }
}
```

#### AQI类

```
"aqi":{
	"city":{
		"aqi":"36",
		"pm25":"18",
		"qlty":"优"}}
```

```
package com.coolweather.android.gson;

public class AQI {
    public AQICity city;
    
    public class AQICity{
        public String aqi;
        public String pm25;
    }
}
```

#### Now类

```
now":{"cloud":"10","cond_code":"100","cond_txt":"晴","fl":"3","hum":"28","pcpn":"0.0","pres":"1014","tmp":"6","vis":"16","wind_deg":"183","wind_dir":"南风","wind_sc":"2","wind_spd":"8","cond":{"code":"100","txt":"晴"}},
```

```
package com.coolweather.android.gson;

import com.google.gson.annotations.SerializedName;

public class Now {
    
    @SerializedName("tmp")
    public String temperature;
    
    @SerializedName("cond")
    public More more;
    
    public class More{
        
        @SerializedName("txt")
        public String info;
    }
}
```

#### Suggestion类

```
suggestion":{
	"comf":{"type":"comf","brf":"较舒适","txt":"白天有雨，风力较强，这种天气条件下，人们会感到有些凉意，但大部分人完全可以接受。"}
,
	"sport":{"type":"sport","brf":"较不宜","txt":"有降水，且风力较强，推荐您在室内进行各种健身休闲运动；若坚持户外运动，请注意防风保暖。"},

	"cw":{"type":"cw","brf":"不宜","txt":"不宜洗车，未来24小时内有雨，如果在此期间洗车，雨水和路上的泥水可能会再次弄脏您的爱车。"}}
```

```
package com.coolweather.android.gson;

import com.google.gson.annotations.SerializedName;

public class Suggestion {
    
    @SerializedName("comf")
    public Comfort comfort;
    
    @SerializedName("cw")
    public CarWash carWash;
    
    public Sport sport;
    
    public class Comfort{
        
        @SerializedName("txt")
        public String info;
    }
    
    public class CarWash{
        @SerializedName("txt")
        public String info;
    }
    
    public class Sport{
        @SerializedName("txt")
        public String info;
    }
}
```

#### Forecast类

```
"daily_forecast":
	[{"date":"2022-07-14",
	"cond":{
		"txt_d":"小雨"},
	"tmp":
		{"max":"9",
		"min":"2"}}
	,{"date":"2022-07-15","cond":{"txt_d":"多云"},"tmp":{"max":"8","min":"1"}},
	{"date":"2022-07-16","cond":{"txt_d":"晴"},"tmp":{"max":"8","min":"3"}},
	{"date":"2022-07-17","cond":{"txt_d":"中雨"},"tmp":{"max":"9","min":"2"}},
	{"date":"2022-07-18","cond":{"txt_d":"阵雨"},"tmp":{"max":"12","min":"4"}},
	{"date":"2022-07-19","cond":{"txt_d":"晴"},"tmp":{"max":"10","min":"3"}}]
```

```
这是一个数组，数组的每一项包含未来一条的天气信息 
我们只需要定义出单日天气的实体类即可，然后在声明实体类引用的时候使用集合类型来进行声明
```

```
package com.coolweather.android.gson;

import com.google.gson.annotations.SerializedName;

public class Forecast {
    public String data;

    @SerializedName("tmp")
    public Temperature temperature;

    @SerializedName("cond")
    public More more;

    public class Temperature{

        public String max;

        public String min;
    }

    public class More{

        @SerializedName("txt_d")
        public String info;
    }
}
```

#### 总的实例类 Weather

```
package com.coolweather.android.gson;

import com.google.gson.annotations.SerializedName;

import java.util.List;

public class Weather {
    
    public String status;
    
    public Basic basic;
    
    public AQI aqi;
    
    public Now now;
    
    public Suggestion suggestion;
    
    @SerializedName("daily_forecast")
    public List<Forecast> forecastList;
}
```

### 14.5.2 编写天气界面 

#### title.xml 头布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/title_city"
        android:layout_centerInParent="true"
        android:textColor="#fff"
        android:textSize="20sp"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/title_update_time"
        android:layout_marginRight="10dp"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"
        android:textColor="#fff"
        android:textSize="16sp"/>


</RelativeLayout>
```

##### bug Android Studio关于出现Cannot resolve symbol ‘@+id/***‘的解决办法

```
可通过运行运行Build->Clean运行来重建工程。
```

#### now.xml 当前天气信息布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="15dp">
  
  // 当前气温
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/degree_text"
        android:layout_gravity="end"
        android:textColor="#fff"
        android:textSize="60sp"/>
    // 天气概况
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/weather_info_text"
        android:layout_gravity="end"
        android:textColor="#fff"
        android:textSize="20sp"/>

</LinearLayout>
```

#### forecast.xml 未来几天天气信息布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="15dp"
    android:background="#8000">
    
    //标题
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="15dp"
        android:layout_marginTop="15dp"
        android:text="预报"
        android:textColor="#fff"
        android:textSize="20sp"/>
    //未来几天信息的布局
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:id="@+id/forecast_layout">
        
    </LinearLayout>

</LinearLayout>
```

#### forecast_item 未来天气信息的子布局

```
?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="15dp">
    
    // 天气预报日期
    <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="2"
        android:layout_gravity="center_vertical"
        android:id="@+id/date_text"
        android:textColor="#fff" />
    // 天气概况
    <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_gravity="center_vertical"
        android:gravity="center"
        android:layout_weight="1"
        android:id="@+id/info_text"
        android:textColor="#fff" />
   // 最高温度
   <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:id="@+id/max_text"
        android:layout_gravity="center"
        android:layout_weight="1"
        android:gravity="right"
        android:textColor="#fff"/>
  //最低温度
  <TextView
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:id="@+id/min_text"
        android:layout_gravity="center"
        android:layout_weight="1"
        android:gravity="right"
        android:textColor="#fff"/>
</LinearLayout>
```

#### aqi.xml 空气质量信息布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="15dp"
    android:background="#8000">
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="15dp"
        android:layout_marginTop="15dp"
        android:text="空气质量"
        android:textColor="#fff"
        android:textSize="20sp"/>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="15dp">
        
        // 显示AQI指数
        <RelativeLayout
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1">
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:layout_centerInParent="true">
                
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:id="@+id/aqi_text"
                    android:layout_gravity="center"
                    android:textColor="#fff"
                    android:textSize="40sp"/>
                
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:text="AQI指数"
                    android:textColor="#fff"/>
            </LinearLayout>
        </RelativeLayout>
        
        //显示PM2.5指数
        <RelativeLayout
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1">
            
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:layout_centerInParent="true">
                
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:id="@+id/pm25_text"
                    android:layout_gravity="center"
                    android:textColor="#fff"
                    android:textSize="40sp"/>
                
                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:text="PM2.5指数"
                    android:textColor="#fff"/>
            </LinearLayout>
        </RelativeLayout>
    </LinearLayout>

</LinearLayout>
```

#### suggestion.xml 生活建议信息布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="15dp"
    android:background="#8000">
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="15dp"
        android:layout_marginTop="15dp"
        android:text="生活建议"
        android:textColor="#fff"
        android:textSize="20sp"/>
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/comfort_text"
        android:layout_margin="15dp"
        android:textColor="#fff"/>
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/car_wash_text"
        android:layout_margin="15dp"
        android:textColor="#fff"/>
    
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/sport_text"
        android:layout_margin="15dp"
        android:textColor="#fff"/>

</LinearLayout>
```

#### activity_weather.xml 天气活动布局 

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/purple_500">

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/weather_layout"
        android:scrollbars="none"
        android:overScrollMode="never">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <include layout="@layout/title"/>

            <include layout="@layout/now"/>

            <include layout="@layout/forecast"/>

            <include layout="@layout/aqi"/>

            <include layout="@layout/suggestion"/>
        </LinearLayout>
    </ScrollView>

</FrameLayout>
```

###  14.5.3 将天气显示到界面上 

#### 在Utility类中加入 解析天气JSON数据方法

```

    /**
     * 将返回的JSON数据解析成Weather实体类
     */
    public static Weather handleWeatherResponse(String response){
        try {
            /**
             * 通过 JSONObject 和JSONArray 将天气数据中的主体内容解析出来
             * 在通过定义的GSON实体类 利用fromJson方法将JSON数据转换成Weather对象
             */
            JSONObject jsonObject=new JSONObject(response);
            JSONArray jsonArray= jsonObject.getJSONArray("HeWeather");
            String weatherContent=jsonArray.getJSONObject(0).toString();
            return new Gson().fromJson(weatherContent,Weather.class);
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
```

#### 修改WeatherActivity 代码

```
package com.coolweather.android;

import androidx.appcompat.app.AppCompatActivity;

import android.content.SharedPreferences;
import android.os.Bundle;
import android.preference.PreferenceManager;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.LinearLayout;
import android.widget.ScrollView;
import android.widget.TextView;
import android.widget.Toast;

import com.coolweather.android.gson.Forecast;
import com.coolweather.android.gson.Weather;
import com.coolweather.android.util.HttpUtil;
import com.coolweather.android.util.Utility;

import java.io.IOException;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.Response;

public class WeatherActivity extends AppCompatActivity {

    private ScrollView weatherLayout ;

    private TextView titleCity;

    private TextView titleUpdateTime;

    private TextView degreeText;

    private TextView weatherInfoText;

    private LinearLayout forecastLayout;

    private TextView aqiText;

    private TextView pm25Text;

    private TextView comfortText;

    private TextView carWashText;

    private TextView sportText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_weather);
        //初始化各控件
        weatherLayout=(ScrollView) findViewById(R.id.weather_layout);
        titleCity=(TextView) findViewById(R.id.title_city);
        titleUpdateTime=(TextView) findViewById(R.id.title_update_time);
        degreeText=(TextView) findViewById(R.id.degree_text);
        weatherInfoText=(TextView) findViewById(R.id.weather_info_text);
        forecastLayout=(LinearLayout) findViewById(R.id.forecast_layout);
        aqiText=(TextView) findViewById(R.id.aqi_text);
        pm25Text=(TextView) findViewById(R.id.pm25_text);
        comfortText=(TextView) findViewById(R.id.comfort_text);
        carWashText=(TextView) findViewById(R.id.car_wash_text);
        sportText=(TextView) findViewById(R.id.sport_text);
        SharedPreferences prefs= PreferenceManager.getDefaultSharedPreferences(this);
        String weatherString =prefs.getString("weather",null);
        if(weatherString!=null){
            //有缓存时直接解析天气数据
            Weather weather= Utility.handleWeatherResponse(weatherString);
            showWeatherInfo(weather);
        }else {
            //无缓存的时候去服务器查询天气
            String weatherId=getIntent().getStringExtra("weather_id");
            weatherLayout.setVisibility(View.INVISIBLE);
            requestWeather(weatherId);
        }
    }

    /**
     * 处理并展示 Weather 实体类中的数据
     * @param weather
     */
    private void showWeatherInfo(Weather weather) {

        String cityName= weather.basic.cityName;
        String updateTime=weather.basic.update.updateTime.split(" ")[1];
        String degree=weather.now.temperature+"℃";
        String weatherInfo=weather.now.more.info;
        titleCity.setText(cityName);
        titleUpdateTime.setText(updateTime);
        degreeText.setText(degree);
        weatherInfoText.setText(weatherInfo);
        forecastLayout.removeAllViews();
        for(Forecast forecast:weather.forecastList){
            View view= LayoutInflater.from(this).inflate(R.layout.forecast_item,forecastLayout,false);
            TextView dateText=(TextView) view.findViewById(R.id.date_text);
            TextView infoText=(TextView) view.findViewById(R.id.info_text);
            TextView maxText=(TextView) view.findViewById(R.id.max_text);
            TextView minText=(TextView) view.findViewById(R.id.min_text);
            dateText.setText(forecast.data);
            infoText.setText(forecast.more.info);
            maxText.setText(forecast.temperature.max);
            minText.setText(forecast.temperature.min);
            forecastLayout.addView(view);
        }
        if(weather.aqi!=null){
            aqiText.setText(weather.aqi.city.aqi);
            pm25Text.setText(weather.aqi.city.pm25);
        }
        String comfort="舒适度： "+weather.suggestion.comfort.info;
        String carWash="洗车指数： "+weather.suggestion.carWash.info;
        String sport="运动建议： "+weather.suggestion.sport.info;
        comfortText.setText(comfort);
        carWashText.setText(carWash);
        sportText.setText(sport);
        weatherLayout.setVisibility(View.VISIBLE);
    }

    /**
     * 根据天气id 请求城市天气信息
     */

    private void requestWeather( final String weatherId) {

        String weatherUrl="http://guolin.tech/api/weather?cityid="+weatherId+
                "&key=497d6108248a4c0598aa6757544fb148";
        //调用HttpUtil.sendOkHttpRequest 发出地址请求
        HttpUtil.sendOkHttpRequest(weatherUrl, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(WeatherActivity.this, "获取天气信息失败", Toast.LENGTH_SHORT).show();
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {

                final  String responseText=response.body().string();
                //将返回的JSON格式转换为Weather对象
                final Weather weather=Utility.handleWeatherResponse(responseText);
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        //服务器返回的status 状态是ok 请求天气成功
                        if(weather!=null&&"ok".equals(weather.status)){
                            // 将返回的数据缓存到SharedPreferences
                            SharedPreferences.Editor editor= PreferenceManager.
                                    getDefaultSharedPreferences(WeatherActivity.this).edit();
                            editor.putString("weather",responseText);
                            editor.apply();
                            //调用showWeatherInfo(weather)进行内容显示 
                            showWeatherInfo(weather);
                        }else {
                            Toast.makeText(WeatherActivity.this, "获取天气信息失败", Toast.LENGTH_SHORT).show();
                        }
                    }
                });

            }
        });
    }

}
```

#### 修改ChooseAreaFragment 代码

```
   listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                        @Override
                        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                            if(currentLevel==LEVEL_PROVINCE) {
                                selectedProvince=provinceList.get(position);
                                queryCities();
                            }else if(currentLevel==LEVEL_CITY){
                                selectedCity=cityList.get(position);
                                queryCounties();
                            }else if(currentLevel==LEVEL_COUNTY){
                                String weatherId=countyList.get(position).getWeatherId();
                                Intent intent=new Intent(getActivity(),WeatherActivity.class);
                                intent.putExtra("weather_id",weatherId);
                                startActivity(intent);
                                getActivity().finish();
                            }
                        }
                    });
```

#### 修改MainActivity 代码

```
package com.coolweather.android;

import androidx.appcompat.app.AppCompatActivity;

import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.preference.PreferenceManager;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // 缓存数据的判断
        SharedPreferences prefs= PreferenceManager.getDefaultSharedPreferences(this);
        if(prefs.getString("weather",null)!=null){
            Intent intent=new Intent(this,WeatherActivity.class);
            startActivity(intent);
            finish();
        }
    }
}
```

### 14.5.4 获取每日一图

 获取每日一图的接口

```
http;//guolin.tech/api/bing_pic
```

####  修改activity_weather.xml代码

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/purple_500">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/bing_pic_img"
        android:scaleType="centerCrop"/>
       ...
        </FrameLayout>
```

#### 修改WeatherActivity 代码

```
    private ImageView bingPicImg;
   
   loadBingPic();
   private void loadBingPic()
    {
        String requestBingPic = "http://guolin.tech/api/bing_pic";
        Log.d("WeatherActivity", requestBingPic);
        HttpUtil.sendOkHttpRequest(requestBingPic, new Callback() {
            @Override
            public void onFailure(@NotNull Call call, @NotNull IOException e) {
                e.printStackTrace();
            }

            @Override
            public void onResponse(@NotNull Call call, @NotNull Response response) throws IOException {
                final String bingPic = response.body().string();
                SharedPreferences.Editor editor = PreferenceManager.getDefaultSharedPreferences(WeatherActivity.this).edit();
                Log.d("WeatherActivity", bingPic);
                editor.putString("bing_pic",bingPic);
                editor.apply();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Glide.with(WeatherActivity.this).load("http://cn.bing.com/th?id=OHR.BasaltGiants_ROW7516697595_1920x1080.jpg&rf=LaDigue_1920x1081920x1080.jpg").into(bingPicImg);
                    }
                });
            }
        });
    }
```

#### 图片加载失败

```
W/Glide: Load failed for Request bing pic with error: undefined method `get_text' for nil:NilClass with size [480x764]
    class com.bumptech.glide.load.engine.GlideException: Failed to load resource
```

#### 接口网址问题

```
有时候网址不对 
Request bing pic with error: undefined method `get_text' for nil:NilClass
```

```
Glide.with(WeatherActivity.this).load("http://cn.bing.com/th?id=OHR.BasaltGiants_ROW7516697595_1920x1080.jpg&rf=LaDigue_1920x1081920x1080.jpg").into(bingPicImg);
```

#### 背景融合 修改WeatherActivity 

```
        super.onCreate(savedInstanceState);
        if(Build.VERSION.SDK_INT>=21){
            View decorView =getWindow().getDecorView();
            decorView.setSystemUiVisibility(
                    View.SYSTEM_UI_FLAG_FULLSCREEN|
                            View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
            getWindow().setStatusBarColor(Color.TRANSPARENT);
        }
```

#### 第三阶段提交

```
git add .
git commit -m "加入显示天气信息的功能"
git push origin mian
```

## 14.6  手动更新天气和切换城市

### 14.6.1 手动更新天气 

​	 依靠下拉刷新实现

#### 依赖库swiperefreshlayout

```
 implementation "androidx.swiperefreshlayout:swiperefreshlayout:1.0.0"
```

#### 修改activity_weather.xml 代码

```
    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/swipe_refresh">
        
        <ScrollView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/weather_layout"
            android:scrollbars="none"
            android:overScrollMode="never">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:fitsSystemWindows="true">

                <include layout="@layout/title"/>

                <include layout="@layout/now"/>

                <include layout="@layout/forecast"/>

                <include layout="@layout/aqi"/>

                <include layout="@layout/suggestion"/>
            </LinearLayout>
        </ScrollView>

    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>

```

修改WeatherActivity 代码

```
    public SwipeRefreshLayout swipeRefresh;
    private String mWeadtherId;
    
            swipeRefresh=(SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        swipeRefresh.setColorSchemeResources(R.color.purple_500);
        
         if(weatherString!=null){
            //有缓存时直接解析天气数据
            Weather weather= Utility.handleWeatherResponse(weatherString);
            mWeadtherId = weather.basic.weatherId;
            showWeatherInfo(weather);
        }else {
            //无缓存的时候去服务器查询天气
            mWeadtherId =getIntent().getStringExtra("weather_id");
            weatherLayout.setVisibility(View.INVISIBLE);
            requestWeather(mWeadtherId);
        }
        swipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                requestWeather(mWeadtherId);
            }
        });
```

```
private void requestWeather( final String weatherId) {

        String weatherUrl="http://guolin.tech/api/weather?cityid="+weatherId+
                "&key=497d6108248a4c0598aa6757544fb148";
        //调用HttpUtil.sendOkHttpRequest 发出地址请求
        HttpUtil.sendOkHttpRequest(weatherUrl, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(WeatherActivity.this, "获取天气信息失败", Toast.LENGTH_SHORT).show();
                        swipeRefresh.setRefreshing(false);
                    }
                });
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {

                final  String responseText=response.body().string();
                //将返回的JSON格式转换为Weather对象
                final Weather weather=Utility.handleWeatherResponse(responseText);
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        //服务器返回的status 状态是ok 请求天气成功
                        if(weather!=null&&"ok".equals(weather.status)){
                            // 将返回的数据缓存到SharedPreferences
                            SharedPreferences.Editor editor= PreferenceManager.
                                    getDefaultSharedPreferences(WeatherActivity.this).edit();
                            editor.putString("weather",responseText);
                            editor.apply();
                            mWeadtherId=weather.basic.weatherId;
                            //调用showWeatherInfo(weather)进行内容显示
                            showWeatherInfo(weather);
                        }else {
                            Toast.makeText(WeatherActivity.this, "获取天气信息失败", Toast.LENGTH_SHORT).show();
                        }
                        swipeRefresh.setRefreshing(false);
                    }
                });

            }
        });
    }
```

### 14.6.2 切换城市 

#### 利用滑动菜单实现更换城市

#### 更改title.xml 布局

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize">

    <Button
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:id="@+id/nav_button"
        android:layout_marginLeft="10dp"
        android:layout_centerVertical="true"
        android:background="@drawable/ic_home"/>
    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/title_city"
        android:layout_centerInParent="true"
        android:textColor="#fff"
        android:textSize="20sp"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/title_update_time"
        android:layout_marginRight="10dp"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"
        android:textColor="#fff"
        android:textSize="16sp"/>


</RelativeLayout>
```

#### 修改activity_weather.xml 代码

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/bing_pic_img"
        android:scaleType="centerCrop" />
    
    <androidx.drawerlayout.widget.DrawerLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/drawer_layout">

        <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/swipe_refresh">

            <ScrollView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:id="@+id/weather_layout"
                android:scrollbars="none"
                android:overScrollMode="never">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:fitsSystemWindows="true">

                    <include layout="@layout/title"/>

                    <include layout="@layout/now"/>

                    <include layout="@layout/forecast"/>

                    <include layout="@layout/aqi"/>

                    <include layout="@layout/suggestion"/>
                </LinearLayout>
            </ScrollView>

        </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
        
        <fragment
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/choose_area_fragment"
            android:name="com.coolweather.android.ChooseAreaFragment"
            android:layout_gravity="start"/>
    </androidx.drawerlayout.widget.DrawerLayout>
```

#### 修改WeatherActivity 代码

```
      public DrawerLayout drawerLayout;

    private Button navButton;

  
  protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if(Build.VERSION.SDK_INT>=21){
            View decorView =getWindow().getDecorView();
            decorView.setSystemUiVisibility(
                    View.SYSTEM_UI_FLAG_FULLSCREEN|
                            View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
            getWindow().setStatusBarColor(Color.CYAN);
        }
        setContentView(R.layout.activity_weather);
        //初始化各控件
        bingPicImg=(ImageView) findViewById(R.id.bing_pic_img);

        swipeRefresh=(SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        swipeRefresh.setColorSchemeResources(R.color.purple_500);

        drawerLayout=(DrawerLayout) findViewById(R.id.drawer_layout);

        navButton=(Button) findViewById(R.id.nav_button);
        navButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                drawerLayout.openDrawer(GravityCompat.START);
            }
        });
        ...
 }
```

#### 修改ChooseAreaFragment 代码

```
else if(currentLevel==LEVEL_COUNTY){
                                String weatherId=countyList.get(position).getWeatherId();
                                if(getActivity()instanceof  MainActivity){
                                    Intent intent=new Intent(getActivity(),WeatherActivity.class);
                                    intent.putExtra("weather_id",weatherId);
                                    startActivity(intent);
                                    getActivity().finish(); 
                                }else if(getActivity() instanceof WeatherActivity){
                                    WeatherActivity activity=(WeatherActivity) getActivity();
                                    activity.drawerLayout.closeDrawers();
                                    activity.swipeRefresh.setRefreshing(true);
                                    activity.requestWeather(weatherId);
                                }
                                
```

#### 第四阶段提交

```
git add .
 git commit -m "新增切换城市和手动更新天气的功能"
 git push origin main
```

## 14.7 后台自动更新天气

### 新建AutoUpdateService 

```
package com.coolweather.android.service;

import android.app.AlarmManager;
import android.app.PendingIntent;
import android.app.Service;
import android.content.Intent;
import android.content.SharedPreferences;
import android.os.IBinder;
import android.os.SystemClock;
import android.preference.PreferenceManager;

import com.coolweather.android.gson.Weather;
import com.coolweather.android.util.HttpUtil;
import com.coolweather.android.util.Utility;

import java.io.IOException;

import okhttp3.Call;
import okhttp3.Callback;
import okhttp3.Response;

public class AutoUpdateService extends Service {
    public AutoUpdateService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        return null;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        updateWeather();
        updateBingPic();
        AlarmManager manager=(AlarmManager) getSystemService(ALARM_SERVICE);
        int anHour=8*60*60*1000;
        long triggerAtTime = SystemClock.elapsedRealtime()+anHour;
        Intent i=new Intent(this,AutoUpdateService.class);
        PendingIntent pi=PendingIntent.getService(this,0,i,0);
        manager.cancel(pi);
        manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,triggerAtTime,pi);
       return super.onStartCommand(intent, flags, startId);
    }

    /**
     * 更新天气信息
     */
    private void updateWeather(){
        SharedPreferences prefs= PreferenceManager.getDefaultSharedPreferences(this);
        String weatherString= prefs.getString("weather",null);
        if(weatherString!=null){
            //有缓存的时候直接解析数据
            Weather weather= Utility.handleWeatherResponse(weatherString);
            String weatherId =weather.basic.weatherId;

            String weatheUri ="http://guolin.tech/api/weather?cityid="+weatherId
                    +"&key=497d6108248a4c0598aa6757544fb148";
            HttpUtil.sendOkHttpRequest(weatheUri, new Callback() {
                @Override
                public void onFailure(Call call, IOException e) {
                    e.printStackTrace();
                }

                @Override
                public void onResponse(Call call, Response response) throws IOException {

                    String responseText =response.body().string();
                    Weather weather=Utility.handleWeatherResponse(responseText);
                    if(weather!=null&&"ok".equals(weather.status)){
                        SharedPreferences.Editor editor=PreferenceManager
                                .getDefaultSharedPreferences(AutoUpdateService.this).edit();
                        editor.putString("weather",responseText);
                        editor.apply();
                    }
                }
            });
        }
    }

    /**
     * 更新每日一图
     */
    private void updateBingPic(){

        String requestBingPic="http://guolin.tech/api/bing_pic";
        HttpUtil.sendOkHttpRequest(requestBingPic, new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
                e.printStackTrace();
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {

                String bingPic=response.body().string();
                SharedPreferences.Editor editor=PreferenceManager.
                        getDefaultSharedPreferences(AutoUpdateService.this).edit();
                editor.putString("bing_pic",bingPic);
                editor.apply();
            }
        });
    }
}
```

### 修改WeatherActivity 代码

```
 private void showWeatherInfo(Weather weather) {

  ...
        weatherLayout.setVisibility(View.VISIBLE);
        Intent intent=new Intent(this, AutoUpdateService.class);
        startService(intent);
    }
```

### 第五阶段提交

```
git add .
 git commit -m "新增后台自动更新天气功能"
 git push origin main
```

## 14.8 修改图标和名称

```
android:icon="@mipmap/logo" 
```

#### 修改程序名称  values/string.xml

```
<resources>
    <string name="app_name">酷欧天气</string>
</resources>
```

### 最后提交

```
git add .
 git commit -m "x"
 git push origin main
```


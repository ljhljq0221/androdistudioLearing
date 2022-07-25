# chapters11

## 11.1 基于位置的服务 

​	确定用户位置 

​	1 通过gps

​	2 通过网络定位 

#### bug

 java.lang.RuntimeException: Unable to start activity ComponentInfo{com.example.lbstest/com.example.lbstest.MainActivity}: java.lang.NullPointerException: Attempt to invoke virtual method 'void com.baidu.location.LocationClient.registerLocat

```
mLocationClient =new LocationClient(getApplicationContext());
```

解决方法

```
   LocationClient.setAgreePrivacy(true);
        //在对LocationClient实例化之前调用LocationClient的setAgreePrivacy(true)。
        // 之后LocationClient还会有红线存在，需要对其进行try-catch就行了。
```

#### 针对registerLocationListener 过时问题

```
将原代码中public class MyLocationListener implements BDLocationListener
改为public class MyLocationListener extends BDAbstractLocationListener
```

### 11.3 百度定位

### 11.3.1 确定自己位置经纬度

```
public class MainActivity extends AppCompatActivity {

    public LocationClient mLocationClient;

    private TextView positionText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        LocationClient.setAgreePrivacy(true);
        //在对LocationClient实例化之前调用LocationClient的setAgreePrivacy(true)。
        // 之后LocationClient还会有红线存在，需要对其进行try-catch就行了。
        try {
            mLocationClient =new LocationClient(getApplicationContext());
        } catch (Exception e) {
            e.printStackTrace();
        }
        mLocationClient.registerLocationListener(new MyLocationListener());
        positionText =(TextView) findViewById(R.id.position_text_view);
        List<String> permissionList=new ArrayList<>();
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.ACCESS_FINE_LOCATION)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.ACCESS_FINE_LOCATION);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.READ_PHONE_STATE)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.READ_PHONE_STATE);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.WRITE_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
        }
        if(!permissionList.isEmpty()){
            String[]permissions=permissionList.toArray(new String[permissionList.size()]);
            ActivityCompat.requestPermissions(MainActivity.this,permissions,1);
        }else {
            requestLocation();
        }
    }
    private void  requestLocation(){
        mLocationClient.start();
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0) {
                    for (int result : grantResults) {
                        if (result != PackageManager.PERMISSION_GRANTED) {
                            Toast.makeText(this, "必须同意所有权限才能使用本程序", Toast.LENGTH_SHORT).show();
                            finish();
                            return;
                        }
                    }
                    requestLocation();
                } else {
                    Toast.makeText(this, "发生未知错误", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }

    public class MyLocationListener extends BDAbstractLocationListener {
        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    StringBuilder currentPosition =new StringBuilder();
                    currentPosition.append("纬度").append(bdLocation.getLatitude()).append("\n");
                    currentPosition.append("经度").append(bdLocation.getLongitude()).append("\n");
                    currentPosition.append("定位方式");
                    if(bdLocation.getLocType()==BDLocation.TypeGpsLocation){
                        currentPosition.append("GPS");
                    }else if(bdLocation.getLocType()==BDLocation.TypeNetWorkLocation){
                        currentPosition.append ("网络");
                    }
                    positionText.setText(currentPosition);
                }
            });
        }

    }

}
```

更新当前位置

```
 private void  requestLocation(){

        initLocation();
        mLocationClient.start();
    }

    private void initLocation() {
        LocationClientOption option=new LocationClientOption();
        option.setScanSpan(5000);
        mLocationClient.setLocOption(option);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mLocationClient.stop();
    }
```

### 11.3.3 选择定位方式 

```
    private void initLocation() {
        LocationClientOption option=new LocationClientOption();
        option.setScanSpan(5000);
        option.setLocationMode(LocationClientOption.LocationMode.Device_Sensors);
        mLocationClient.setLocOption(option);
    }
```

### 11.3.4 看的懂的位置信息 

```
  private void initLocation() {
        LocationClientOption option=new LocationClientOption();
        option.setScanSpan(5000);
        // GPS定位
        //option.setLocationMode(LocationClientOption.LocationMode.Device_Sensors);
        option.setIsNeedAddress(true);
        mLocationClient.setLocOption(option);
    }

```

```
    public class MyLocationListener extends BDAbstractLocationListener {
        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    StringBuilder currentPosition =new StringBuilder();
                    currentPosition.append("纬度： ").append(bdLocation.getLatitude()).append("\n");
                    currentPosition.append("经度： ").append(bdLocation.getLongitude()).append("\n");
                    currentPosition.append("国家： ").append(bdLocation.getCountry()).append("\n");
                    currentPosition.append("省： ").append(bdLocation.getProvince()).append("\n");
                    currentPosition.append("市： ").append(bdLocation.getCity()).append("\n");
                    currentPosition.append("区： ").append(bdLocation.getDistrict()).append("\n");
                    currentPosition.append("街道： ").append(bdLocation.getStreet()).append("\n");
                    currentPosition.append("定位方式： ");
                    if(bdLocation.getLocType()==BDLocation.TypeGpsLocation){
                        currentPosition.append("GPS");
                    }else if(bdLocation.getLocType()==BDLocation.TypeNetWorkLocation){
                        currentPosition.append ("网络");
                    }
                    positionText.setText(currentPosition);
                }
            });
        }

    }
```

## 11.4 使用百度地图

### 11.4.1 地图显示出来 

```
    <com.baidu.mapapi.map.MapView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/bmapView"
        android:clickable="true"
        android:visibility="gone"/>
        
```

```
//初始化操作要在 setContentView 方法前调用
        SDKInitializer.initialize(getApplicationContext());

        //百度地图


        LocationClient.setAgreePrivacy(true);
        //在对LocationClient实例化之前调用LocationClient的setAgreePrivacy(true)。
        // 之后LocationClient还会有红线存在，需要对其进行try-catch就行了。
        try {
            mLocationClient =new LocationClient(getApplicationContext());
        } catch (Exception e) {
            e.printStackTrace();
        }
        setContentView(R.layout.activity_main);
        mapView=(MapView) findViewById(R.id.bmapView);
```

```
protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
       mapView.onPause();
    }

    private void  requestLocation(){

        initLocation();
        mLocationClient.start();
    }

    private void initLocation() {
        LocationClientOption option=new LocationClientOption();
        option.setScanSpan(5000);
        // GPS定位
        //option.setLocationMode(LocationClientOption.LocationMode.Device_Sensors);
        option.setIsNeedAddress(true);
        mLocationClient.setLocOption(option);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mLocationClient.stop();
       mapView.onDestroy();
    }
```



#### bug



```
    java.lang.RuntimeException: Unable to start activity ComponentInfo{com.example.lbstest/com.example.lbstest.MainActivity}: com.baidu.mapapi.common.BaiduMapSDKException: not agree privacyMode, please invoke SDKInitializer.setAgreePrivacy(Context, boolean) function
    Caused by: com.baidu.mapapi.common.BaiduMapSDKException: not agree privacyMode, please invoke SDKInitializer.setAgreePrivacy(Context, boolean) function
    
```

#### 错误原因

```
在SDK各功能组件使用之前都需要调用“SDKInitializer.initialize(getApplicationContext())”，因此建议在应用创建时初始化SDK引用的Context为全局变量。
```

#### 解决方法

```
新建一个自定义的Application，在其onCreate方法中完成SDK的初始化。示例代码如下：
public class DemoApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        //在使用SDK各组件之前初始化context信息，传入ApplicationContext   
        SDKInitializer.initialize(this);
        //自4.3.0起，百度地图SDK所有接口均支持百度坐标和国测局坐标，用此方法设置您使用的坐标类型.
        //包括BD09LL和GCJ02两种坐标，默认是BD09LL坐标。
        SDKInitializer.setCoordType(CoordType.BD09LL);
    }
}
```

```
在AndroidManifest.xml文件中声明该Application 
android:name='com.example.lbstest.DemoApplicarion'
```

#### bug

```
仍然报错 
Caused by: com.baidu.mapapi.common.BaiduMapSDKException: not agree privacyMode, please invoke SDKInitializer.setAgreePrivacy(Context, boolean) function
```

#### 解决方法

```
对该类进行修改 
public class DemoApplicarion extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        //
        SDKInitializer.setAgreePrivacy(getApplicationContext(),true);

        //在使用SDK各组件之前初始化context信息，传入ApplicationContext
        SDKInitializer.initialize(this);
        //自4.3.0起，百度地图SDK所有接口均支持百度坐标和国测局坐标，用此方法设置您使用的坐标类型.
        //包括BD09LL和GCJ02两种坐标，默认是BD09LL坐标。
        SDKInitializer.setCoordType(CoordType.BD09LL);
        //增加新的同意声明
        LocationClient.setAgreePrivacy(true);
    }
}

```

### 11.4.2 移动到我的位置 

```
BaiduMap 类 地图的总控制器 
// 对地图进行缩放
// MapStatusUpdateFactory.zoomTo 接受一个float型参数，用与设置缩放级别 
MapStatusUpdate update=MapStatusUpdateFactory.zoomTo(12.5f);
// 把MapStatusUpdate 传入到baiduMap.animateMapStatus方法中即可完成
//缩放功能
baiduMap.animateMapStatus(update);

// 移动到某一个经纬度 
//LatLng 类存放经纬度，第一个参数纬度，第二个参数经度
LatLng ll=new LatLng(39.12,116.04);
//MapStatusUpdateFactory.newLatlng 返回一个 MapStatusUpdate 对象
MapStatusUpdate update=MapStatusUpdateFactory.newLatlng(ll);
// 把MapStatusUpdate 传入到baiduMap.animateMapStatus方法中 完成移动
baiduMap.animateMapStatus(update);
```

```
   private BaiduMap baiduMap;

   private boolean isFirstLocate=true;
```

```
        mapView=(MapView) findViewById(R.id.bmapView);
        baiduMap=mapView.getMap();
```

```
    private void navigateTo(BDLocation location){
        if(isFirstLocate){
            LatLng ll=new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update= MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);
            update=MapStatusUpdateFactory.zoomBy(16f);
            baiduMap.animateMapStatus(update);
            isFirstLocate=false;
        }
    }
```

```
       public void onReceiveLocation(BDLocation bdLocation) {
            if(bdLocation.getLocType()==BDLocation.TypeGpsLocation||
            bdLocation.getLocType()==BDLocation.TypeNetWorkLocation){
                navigateTo(bdLocation);
            }
```

### 11.4.3 让我显示在地图上

```
//MyLocationData.Builder 封装设备当前位置 
MyLocationData.Builder locationBuilder=new MyLocationData.Builder();
locationBuilder.latitude(39.1);
locationBuilder.longitude(116.20);
//封装信息设置完成后 需要调用build 方法 生成一个MyLocationData实例 
传入到 BaiduMap的setMyLocationData()方法中
MyLocationData LocationData=LocationData .builder();
baiduMap.setMyLocationData(LocationData)
```

```
  baiduMap=mapView.getMap();
        baiduMap.setMyLocationEnabled(true);
```

```
 private void navigateTo(BDLocation location){
        if(isFirstLocate){
            LatLng ll=new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update= MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);
            update=MapStatusUpdateFactory.zoomBy(16f);
            baiduMap.animateMapStatus(update);
            isFirstLocate=false;
        }
        MyLocationData.Builder builder=new MyLocationData.Builder();
        //纬度
        builder.latitude(location.getLatitude());
        //经度
        builder.longitude(location.getLongitude());
        MyLocationData locationData=builder.build();
        baiduMap.setMyLocationData(locationData);
    }
```

```
   @Override
    protected void onDestroy() {
        super.onDestroy();
        mLocationClient.stop();
       mapView.onDestroy();
       baiduMap.setMyLocationEnabled(false);
    }
```

## 全部代码

```
package com.example.lbstest;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.Manifest;
import android.content.pm.PackageManager;
import android.os.Bundle;
import android.widget.TextView;
import android.widget.Toast;

import com.baidu.location.BDAbstractLocationListener;
import com.baidu.location.BDLocation;
import com.baidu.location.BDLocationListener;
import com.baidu.location.LocationClient;
import com.baidu.location.LocationClientOption;
import com.baidu.mapapi.SDKInitializer;
import com.baidu.mapapi.map.BaiduMap;
import com.baidu.mapapi.map.MapStatusUpdate;
import com.baidu.mapapi.map.MapStatusUpdateFactory;
import com.baidu.mapapi.map.MapView;
import com.baidu.mapapi.map.MyLocationData;
import com.baidu.mapapi.model.LatLng;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    public LocationClient mLocationClient;

    private TextView positionText;

   private MapView mapView;

   private BaiduMap baiduMap;

   private boolean isFirstLocate=true;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //初始化操作要在 setContentView 方法前调用
        SDKInitializer.initialize(getApplicationContext());

        //百度地图


        LocationClient.setAgreePrivacy(true);
        //在对LocationClient实例化之前调用LocationClient的setAgreePrivacy(true)。
        // 之后LocationClient还会有红线存在，需要对其进行try-catch就行了。
        try {
            mLocationClient =new LocationClient(getApplicationContext());
        } catch (Exception e) {
            e.printStackTrace();
        }
        setContentView(R.layout.activity_main);
        mapView=(MapView) findViewById(R.id.bmapView);
        baiduMap=mapView.getMap();
        baiduMap.setMyLocationEnabled(true);
        mLocationClient.registerLocationListener(new MyLocationListener());
        positionText =(TextView) findViewById(R.id.position_text_view);
        List<String> permissionList=new ArrayList<>();
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.ACCESS_FINE_LOCATION)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.ACCESS_FINE_LOCATION);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.READ_PHONE_STATE)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.READ_PHONE_STATE);
        }
        if(ContextCompat.checkSelfPermission(MainActivity.this, Manifest
                .permission.WRITE_EXTERNAL_STORAGE)!= PackageManager.PERMISSION_GRANTED){
            permissionList.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);
        }
        if(!permissionList.isEmpty()){
            String[]permissions=permissionList.toArray(new String[permissionList.size()]);
            ActivityCompat.requestPermissions(MainActivity.this,permissions,1);
        }else {
            requestLocation();
        }

    }
    /*
        进行尺度缩放以及经纬度移动
     */
    private void navigateTo(BDLocation location){
        if(isFirstLocate){
            LatLng ll=new LatLng(location.getLatitude(),location.getLongitude());
            MapStatusUpdate update= MapStatusUpdateFactory.newLatLng(ll);
            baiduMap.animateMapStatus(update);
            update=MapStatusUpdateFactory.zoomBy(16f);
            baiduMap.animateMapStatus(update);
            isFirstLocate=false;
        }
        MyLocationData.Builder builder=new MyLocationData.Builder();
        //纬度
        builder.latitude(location.getLatitude());
        //经度
        builder.longitude(location.getLongitude());
        MyLocationData locationData=builder.build();
        baiduMap.setMyLocationData(locationData);
    }



    @Override
    protected void onResume() {
        super.onResume();
        mapView.onResume();
    }

    @Override
    protected void onPause() {
        super.onPause();
       mapView.onPause();
    }

    private void  requestLocation(){

        initLocation();
        mLocationClient.start();
    }

    private void initLocation() {
        LocationClientOption option=new LocationClientOption();
        option.setScanSpan(5000);
        // GPS定位
        //option.setLocationMode(LocationClientOption.LocationMode.Device_Sensors);
        option.setIsNeedAddress(true);
        mLocationClient.setLocOption(option);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mLocationClient.stop();
       mapView.onDestroy();
       baiduMap.setMyLocationEnabled(false);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        switch (requestCode) {
            case 1:
                if (grantResults.length > 0) {
                    for (int result : grantResults) {
                        if (result != PackageManager.PERMISSION_GRANTED) {
                            Toast.makeText(this, "必须同意所有权限才能使用本程序", Toast.LENGTH_SHORT).show();
                            finish();
                            return;
                        }
                    }
                    requestLocation();
                } else {
                    Toast.makeText(this, "发生未知错误", Toast.LENGTH_SHORT).show();
                }
                break;
            default:
        }
    }

    public class MyLocationListener extends BDAbstractLocationListener {
        @Override
        public void onReceiveLocation(BDLocation bdLocation) {
            if(bdLocation.getLocType()==BDLocation.TypeGpsLocation||
            bdLocation.getLocType()==BDLocation.TypeNetWorkLocation){
                navigateTo(bdLocation);
            }
            runOnUiThread(new Runnable() {
                @Override
                public void run() {
                    StringBuilder currentPosition =new StringBuilder();
                    currentPosition.append("纬度： ").append(bdLocation.getLatitude()).append("\n");
                    currentPosition.append("经度： ").append(bdLocation.getLongitude()).append("\n");
                    currentPosition.append("国家： ").append(bdLocation.getCountry()).append("\n");
                    currentPosition.append("省： ").append(bdLocation.getProvince()).append("\n");
                    currentPosition.append("市： ").append(bdLocation.getCity()).append("\n");
                    currentPosition.append("区： ").append(bdLocation.getDistrict()).append("\n");
                    currentPosition.append("街道： ").append(bdLocation.getStreet()).append("\n");
                    currentPosition.append("定位方式： ");
                    if(bdLocation.getLocType()==BDLocation.TypeGpsLocation){
                        currentPosition.append("GPS");
                    }else if(bdLocation.getLocType()==BDLocation.TypeNetWorkLocation){
                        currentPosition.append ("网络");
                    }
                    positionText.setText(currentPosition);
                }
            });
        }

    }

}
```

## 11.5 Git时间-版本控制工具的高级用法

### 11.5.1 分支的用法 

​	分支的作用在现有的代码的基础上开辟一个分叉口，使得代码可以在主干线和分支线上同时进行开发，且互不干扰。

#### 查看分支

```
git branch
```

#### 创建分支

```
git branch version1.0
```

#### 切换分支

```
git checkout version1.0
```

#### 合并操作

```
git checkout master
git merge version1.0
```

#### 删除分支

```
git branch -D version1.0
```

### 11.5.2 与远程版本库协作

####  将远程版本库的代码下载到本地

```
git clone https://github.com/example/test.git
```

#### 将修改的代码同步到远程版本库中

```
git push origin master
```

#### 如何将远程版本库上的修改同步到本地 

```
git fetch origin master
```

#### 观察具体修改了哪些内容

```
git diff origin/master
```

#### 合并修改内容

```
git merge origin/master
```

#### 从远程版本库上获取最新的代码并合并到本地 

```
git pull origin master
```


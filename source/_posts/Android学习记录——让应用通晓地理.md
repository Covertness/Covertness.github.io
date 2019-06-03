title: Android学习记录——让应用通晓地理
urlname: Android学习记录——让应用通晓地理
date: 2015/05/17 13:08:45
categories:
- 学习记录
tags:
- Android
- 定位
- 地图
- Restful

---
![](https://image.covertness.cn/android_rangyingyongtongxiaodili_screencast-Genymotion-2015-05-17_15.32.14.804.gif)

一个可以把现在的心情记录到地图上的 APP ，基于高德 LBS 开放平台。
<!-- more -->

## 需要用到的工具
- [Android Studio](http://developer.android.com/sdk/index.html) Android 集成开发环境
- [Genymotion](https://www.genymotion.com/) Android 模拟器
- [高德 LBS 开放平台](http://lbs.amap.com/) 提供定位、地图及其数据存储
- [Android Asynchronous Http Client](http://loopj.com/android-async-http/) Android 异步 HTTP 请求库

Android Studio 和 Genymotion 的基本使用方法可参考[《Android学习记录——开发环境搭建》](http://covertness.me/2015/03/28/Android学习记录——开发环境搭建/)。

## 创建应用——MoodMap
### 1. 使用 Android Studio 创建一个名为 MoodMap 的 Android 项目
配置保持默认参数即可：
> Minimum SDK: API 16: Android 4.1 (Jelly Bean)
> Activity Name: MainActivity
> Layout Name: activity_main
> Title: MoodMap
> Menu Resource Name: menu_main

## 导入高德 LBS 开放平台相关 SDK
### 1. 注册高德 LBS 开放平台账号并获取相关 SDK 的 KEY
注册账号后在[我的 KEY](http://lbs.amap.com/console/)页面获取 Android 平台 SDK 和 Rest 服务接口的 KEY。

### 2. 下载 SDK
- [高德地图Android SDK（2D矢量） V2.4.1](http://lbs.amap.com/api/android-sdk/down/)
- [高德地图 Android云图 SDK v1.1.0](http://lbs.amap.com/api/android-cloud-sdk/down/)
- [高德地图定位 SDK V1.3.2](http://lbs.amap.com/api/android-location-sdk/down/)

### 3. 配置 SDK 库文件
将 SDK 包中对应的 .jar 文件复制到当前 app 项目下的 libs 目录中；然后再 Android Studio 中依次右击 .jar文件，选择 Add as library ，将其导入当前 app 项目。

## 创建一张云图
通过以下命令请求 Rest 服务接口创建一张用于存储心情数据的云图：
```bash
curl -d "key=RestKey&name=mymap" "http://yuntuapi.amap.com/datamanage/table/create"
```

RestKey为刚才申请的 Rest 服务接口的 KEY，注意保存返回结果中的tableid，后面需要保存到 AndroidManifest.xml 文件中。

## 导入 Android Asynchronous Http Client
在项目 build.gradle 文件的 dependencies 节中添加一行`compile 'com.loopj.android:android-async-http:1.4.5'`。

## 实现 APP 的功能
### 1. 界面
res/layout/activity_main.xml: 
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <com.amap.api.maps2d.MapView
        xmlns:android="http://schemas.android.com/apk/res/android" android:id="@+id/map"
        android:layout_width="fill_parent" android:layout_height="fill_parent" />

</RelativeLayout>
```

res/layout/choose_mood.xml:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent" >

    <ImageButton
        android:id="@+id/happyButton"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:background="@null"
        android:src="@drawable/face_happy"
        android:contentDescription="@string/happy" />

    <ImageButton
        android:id="@+id/smileButton"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:background="@null"
        android:src="@drawable/face_smile"
        android:contentDescription="@string/smile" />

    <ImageButton
        android:id="@+id/sadButton"
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:background="@null"
        android:src="@drawable/face_sad"
        android:contentDescription="@string/sad" />

</LinearLayout>
```

res/values/strings.xml:
```xml
<resources>
    <string name="app_name">MoodMap</string>

    <string name="action_settings">Settings</string>
    <string name="your_mood">现在的心情</string>
    <string name="happy">开心</string>
    <string name="sad">伤心</string>
    <string name="smile">微笑</string>
</resources>
```

复制以下3张图片到 res/drawable/ 目录下：
- [face_happy.png](http://covertness.qiniudn.com/android_rangyingyongtongxiaodili_face_happy.png)
- [face_sad.png](http://covertness.qiniudn.com/android_rangyingyongtongxiaodili_face_sad.png)
- [face_smile.png](http://covertness.qiniudn.com/android_rangyingyongtongxiaodili_face_smile.png)

### 2. 逻辑
MainActivity.java:
```java
package me.covertness.moodmap;

import java.util.ArrayList;
import java.util.List;

import android.app.Dialog;
import android.content.Context;
import android.content.pm.ApplicationInfo;
import android.content.pm.PackageManager;
import android.location.Location;
import android.os.Handler;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.ImageButton;

import com.amap.api.cloud.model.AMapCloudException;
import com.amap.api.cloud.model.CloudItem;
import com.amap.api.cloud.model.CloudItemDetail;
import com.amap.api.cloud.model.LatLonPoint;
import com.amap.api.cloud.search.CloudResult;
import com.amap.api.cloud.search.CloudSearch;
import com.amap.api.location.AMapLocation;
import com.amap.api.location.AMapLocationListener;
import com.amap.api.location.LocationManagerProxy;
import com.amap.api.location.LocationProviderProxy;
import com.amap.api.maps2d.AMap;
import com.amap.api.maps2d.CameraUpdateFactory;
import com.amap.api.maps2d.LocationSource;
import com.amap.api.maps2d.MapView;
import com.amap.api.maps2d.model.BitmapDescriptor;
import com.amap.api.maps2d.model.BitmapDescriptorFactory;
import com.amap.api.maps2d.model.LatLng;
import com.amap.api.maps2d.model.LatLngBounds;
import com.amap.api.maps2d.model.Marker;
import com.amap.api.maps2d.model.MarkerOptions;
import com.loopj.android.http.*;
import org.apache.http.Header;
import org.json.JSONException;
import org.json.JSONObject;


public class MainActivity extends ActionBarActivity implements LocationSource, AMapLocationListener,
        CloudSearch.OnCloudSearchListener {
    private String amapRestfulKey;
    private String cloudTableId;
    private MapView mapView;
    private AMap aMap;
    private LocationManagerProxy locationManagerProxy;
    private CloudSearch cloudSearch;
    private CloudSearch.Query currentQuery;
    private OnLocationChangedListener locationChangedListener;
    private AMapLocation currentLocation = null;
    private static AsyncHttpClient httpClient = new AsyncHttpClient();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        try {
            ApplicationInfo app = this.getPackageManager().getApplicationInfo(this.getPackageName(),PackageManager.GET_META_DATA);
            amapRestfulKey = app.metaData.getString("me.covertness.moodmap.restful.apikey");
            cloudTableId = app.metaData.getString("me.covertness.moodmap.cloud.tableid");
        } catch (PackageManager.NameNotFoundException e) {
            Log.e("metadata", "Failed to load meta-data, NameNotFound: " + e.getMessage());
        } catch (NullPointerException e) {
            Log.e("metadata", "Failed to load meta-data, NullPointer: " + e.getMessage());
        }

        mapView = (MapView) findViewById(R.id.map);
        mapView.onCreate(savedInstanceState);

        if (aMap == null) {
            aMap = mapView.getMap();
            aMap.setLocationSource(this);   // 设置定位监听
            // 设置为true表示显示定位层并可触发定位，false表示隐藏定位层并不可触发定位，默认是false
            aMap.setMyLocationEnabled(true);
        }

        cloudSearch = new CloudSearch(this);
        cloudSearch.setOnCloudSearchListener(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        mapView.onDestroy();
    }

    @Override
    protected void onPause() {
        super.onPause();

        mapView.onPause();
    }

    @Override
    protected void onResume() {
        super.onResume();

        mapView.onResume();
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);

        mapView.onSaveInstanceState(outState);
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }



    @Override
    public void activate(OnLocationChangedListener onLocationChangedListener) {
        locationChangedListener = onLocationChangedListener;

        if (locationManagerProxy == null) {
            locationManagerProxy = LocationManagerProxy.getInstance(this);
        }

        //此方法为每隔固定时间会发起一次定位请求，为了减少电量消耗或网络流量消耗，
        //注意设置合适的定位时间的间隔，并且在合适时间调用removeUpdates()方法来取消定位请求
        //在定位结束后，在合适的生命周期调用destroy()方法
        //其中如果间隔时间为-1，则定位只定一次
        locationManagerProxy.requestLocationData(
                LocationProviderProxy.AMapNetwork, -1, 10, this);
    }

    @Override
    public void deactivate() {
        locationChangedListener = null;

        if (locationManagerProxy != null) {
            locationManagerProxy.removeUpdates(this);
            locationManagerProxy.destroy();
        }
        locationManagerProxy = null;
    }


    @Override
    public void onLocationChanged(AMapLocation aMapLocation) {
        if (locationChangedListener != null && aMapLocation != null) {
            int retCode = aMapLocation.getAMapException().getErrorCode();
            if (retCode == 0) {
                currentLocation = aMapLocation;
                locationChangedListener.onLocationChanged(currentLocation);// 显示系统小蓝点
                updateMapData();
                showMoodDialog();
            } else {
                Log.e("locate", aMapLocation.getAMapException().getErrorMessage());
            }
        }
    }

    /**
     * 此方法已经废弃
     */
    @Override
    public void onLocationChanged(Location location) {

    }

    @Override
    public void onStatusChanged(String provider, int status, Bundle extras) {

    }

    @Override
    public void onProviderEnabled(String provider) {

    }

    @Override
    public void onProviderDisabled(String provider) {
    }


    @Override
    public void onCloudSearched(CloudResult result, int rCode) {
        if (rCode == 0) {
            if (result != null && result.getQuery() != null) {
                if (result.getQuery().equals(currentQuery)) {
                    //获取云数据
                    List<CloudItem> mCloudItems = result.getClouds();
                    if (mCloudItems != null && mCloudItems.size() > 0) {
                        aMap.clear();
                        PoiOverlay mPoiCloudOverlay = new PoiOverlay(this, aMap, mCloudItems);
                        mPoiCloudOverlay.removeFromMap();
                        mPoiCloudOverlay.addToMap();
                        mPoiCloudOverlay.zoomToSpan();
                    } else {
                        Log.d("search", "result is empty");
                    }
                }
            } else {
                Log.d("search", "result is empty");
            }
        } else {
            Log.e("search", "error code: " + rCode);
        }
    }

    @Override
    public void onCloudItemDetailSearched(CloudItemDetail cloudItemDetail, int i) {

    }

    private void showMoodDialog() {
        final Dialog moodDialog = new Dialog(this);
        moodDialog.setContentView(R.layout.choose_mood);
        moodDialog.setTitle(R.string.your_mood);

        final JsonHttpResponseHandler publishMoodHandler = new JsonHttpResponseHandler() {
            @Override
            public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
                try {
                    if (response.getInt("status") == 1) {
                        final Handler handler = new Handler();
                        handler.postDelayed(new Runnable() {
                            @Override
                            public void run() {
                                updateMapData();
                            }
                        }, 5000);
                    } else {
                        Log.e("publishMood", "failed: " + response.getString("info"));
                    }
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        };

        final ImageButton happyButton = (ImageButton) moodDialog.findViewById(R.id.happyButton);
        // if button is clicked, close the custom dialog
        happyButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                moodDialog.dismiss();
                publishMood(happyButton.getContentDescription().toString(), publishMoodHandler);
            }
        });

        final ImageButton smileButton = (ImageButton) moodDialog.findViewById(R.id.smileButton);
        // if button is clicked, close the custom dialog
        smileButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                moodDialog.dismiss();
                publishMood(smileButton.getContentDescription().toString(), publishMoodHandler);
            }
        });

        final ImageButton sadButton = (ImageButton) moodDialog.findViewById(R.id.sadButton);
        // if button is clicked, close the custom dialog
        sadButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                moodDialog.dismiss();
                publishMood(sadButton.getContentDescription().toString(), publishMoodHandler);
            }
        });

        moodDialog.show();
    }

    private void publishMood(String mood, AsyncHttpResponseHandler responseHandler) {
        RequestParams params = new RequestParams();
        params.put("key", amapRestfulKey);
        params.put("tableid", cloudTableId);
        params.put("loctype", "1");

        JSONObject jsonObject = new JSONObject();
        try {
            jsonObject.put("_name", mood);
            jsonObject.put("_location",
                    currentLocation.getLongitude() + "," + currentLocation.getLatitude());
        } catch (JSONException e) {
            e.printStackTrace();
        }
        params.put("data", jsonObject.toString());
        httpClient.post("http://yuntuapi.amap.com/datamanage/data/create", params, responseHandler);
    }

    private void updateMapData() {
        //圆形查询范围
        CloudSearch.SearchBound bound = new CloudSearch.SearchBound(new LatLonPoint(
                currentLocation.getLatitude(), currentLocation.getLongitude()), 50000);
        try {
            //构造查询对象
            currentQuery = new CloudSearch.Query(cloudTableId, "", bound);

            //异步搜索
            cloudSearch.searchCloudAsyn(currentQuery);
        } catch (AMapCloudException e) {
            e.printStackTrace();
        }
    }
}


class PoiOverlay {
    private final List<CloudItem> mPois;
    private final AMap mAMap;
    private final Context mainContext;
    private ArrayList<Marker> mPoiMarks = new ArrayList<>();

    public PoiOverlay(Context context, AMap amap, List<CloudItem> pois) {
        mainContext = context;
        mAMap = amap;
        mPois = pois;
    }

    public void addToMap() {
        for (int i = 0; i < mPois.size(); i++) {
            Marker marker = mAMap.addMarker(getMarkerOptions(i));
            marker.setObject(i);
            mPoiMarks.add(marker);
        }
    }

    public void removeFromMap() {
        for (Marker mark : mPoiMarks) {
            mark.remove();
        }
    }

    public void zoomToSpan() {
        if (mPois != null && mPois.size() > 0) {
            if (mAMap == null)
                return;
            LatLngBounds bounds = getLatLngBounds();
            mAMap.moveCamera(CameraUpdateFactory.newLatLngBounds(bounds, 20));
        }
    }

    private LatLngBounds getLatLngBounds() {
        LatLngBounds.Builder b = LatLngBounds.builder();
        for (int i = 0; i < mPois.size(); i++) {
            b.include(new LatLng(mPois.get(i).getLatLonPoint().getLatitude(),
                    mPois.get(i).getLatLonPoint().getLongitude()));
        }
        return b.build();
    }

    private MarkerOptions getMarkerOptions(int index) {
        return new MarkerOptions()
                .position(
                        new LatLng(mPois.get(index).getLatLonPoint()
                                .getLatitude(), mPois.get(index)
                                .getLatLonPoint().getLongitude()))
                .title(getTitle(index)).snippet(getSnippet(index))
                .icon(getBitmapDescriptor(index));
    }

    protected BitmapDescriptor getBitmapDescriptor(int index) {
        String title = mPois.get(index).getTitle();
        if (title.equals(mainContext.getString(R.string.happy))) {
            return BitmapDescriptorFactory.fromResource(R.drawable.face_happy);
        } else if (title.equals(mainContext.getString(R.string.smile))) {
            return BitmapDescriptorFactory.fromResource(R.drawable.face_smile);
        } else if (title.equals(mainContext.getString(R.string.sad))) {
            return BitmapDescriptorFactory.fromResource(R.drawable.face_sad);
        } else {
            return null;
        }
    }

    protected String getTitle(int index) {
        return mPois.get(index).getTitle();
    }

    protected String getSnippet(int index) {
        return mPois.get(index).getSnippet();
    }
}
```

### 3. 修改AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.covertness.moodmap" >

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.CHANGE_CONFIGURATION" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <meta-data
            android:name="com.amap.api.v2.apikey" android:value="AndroidKey"/>
        <meta-data
            android:name="me.covertness.moodmap.restful.apikey" android:value="RestKey"/>
        <meta-data
            android:name="me.covertness.moodmap.cloud.tableid" android:value="CloudTableId"/>

        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

**注意将代码中的`AndroidKey`、`RestKey`和`CloudTableId`替换成自己应用的。**


## 测试应用
使用 Android 4.1 或以上版本的手机打开 MoodMap （Genymotion模拟器需要先打开GPS并设置当前的位置），选择一个表情，等待片刻便能在地图上看到自己所在的位置多了一个表情，如下图所示。
![](https://image.covertness.cn/android_rangyingyongtongxiaodili_screencast-Genymotion-2015-05-17_15.32.14.804.gif)

## 要点总结
### 1. 界面
应用初始化完成后会创建一个[自定义的对话框](http://www.mkyong.com/android/android-custom-dialog-example/)，其布局文件位于 choose_mood.xml 文件中，当用户做出选择后该对话框即关闭。

### 2. 逻辑
此应用启动后首先初始化 2D 地图 SDK，然后发起定位请求，获得位置信息后显示对话框让用户选择当前的心情，之后保存用户的心情到云图，最后更新云图数据显示到地图上。期间使用到 2D 地图 SDK 、 定位 SDK 和 云图 SDK ，具体使用方法可参看实现代码及文档。

### 3. AndroidManifest.xml
应用将用到第三方服务的 KEY 保存到了 meta-data 字段，可在 Java 代码中通过如下方式获取：
```java
        try {
            ApplicationInfo app = this.getPackageManager().getApplicationInfo(this.getPackageName(),PackageManager.GET_META_DATA);
            amapRestfulKey = app.metaData.getString("me.covertness.moodmap.restful.apikey");
            cloudTableId = app.metaData.getString("me.covertness.moodmap.cloud.tableid");
        } catch (PackageManager.NameNotFoundException e) {
            Log.e("metadata", "Failed to load meta-data, NameNotFound: " + e.getMessage());
        } catch (NullPointerException e) {
            Log.e("metadata", "Failed to load meta-data, NullPointer: " + e.getMessage());
        }
```

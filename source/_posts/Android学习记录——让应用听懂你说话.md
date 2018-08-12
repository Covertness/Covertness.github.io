title: Android学习记录——让应用听懂你说话
date: 2015/05/02 17:03:25
categories:
- 学习记录
tags:
- Android
- Gradle
- 语音识别
- 百度开放服务

---
![](https://image.covertness.me/android_rangyingyongtindongnishuohua_screencast-Genymotion-2015-05-02_17.17.19.305.gif)

一个简单的语音识别Demo，基于百度语音识别云服务。
<!-- more -->

## 需要用到的工具
- [Android Studio](http://developer.android.com/sdk/index.html) Android 集成开发环境
- [Genymotion](https://www.genymotion.com/) Android 模拟器
- [百度语音识别服务](http://developer.baidu.com/wiki/index.php?title=docs/cplat/media/voice) 提供云端语音识别

Android Studio 和 Genymotion 的基本使用方法可参考[《Android学习记录——开发环境搭建》](http://covertness.me/2015/03/28/Android学习记录——开发环境搭建/)。

## 创建应用—— Listen
### 1. 使用 Android Studio 创建一个名为 Listen 的 Android 项目
配置保持默认参数即可：
> Minimum SDK: API 16: Android 4.1 (Jelly Bean)
> Activity Name: MainActivity
> Layout Name: activity_main
> Title: Listen
> Menu Resource Name: menu_main

## 导入百度语音识别 SDK
### 1. 注册百度开放云平台账号并开启语音识别服务
步骤比较简单，具体可参考[《开发手册》](http://bcs.duapp.com/cplat-01/mediacloud%2Fvoice%2FBaidu-Voice-Android-SDK-Manual.pdf)的集成指南一节

### 2. 下载 Android SDK
目前最新版本为[1.6.2](http://bcs.duapp.com/cplat-01/mediacloud%2Fvoice%2FBaidu-Voice-SDK-Android-1.6.2.zip)

### 3. 配置 SDK 库文件
将 SDK 包中 libs 目录下的所有文件和文件夹复制到当前 app 项目下的 libs 目录中；然后再 Android Studio 中右击 galaxy.jar，选择 Add as library ，将其导入当前 app 项目；最后由于 SDK 中还包含 .so 库，还需在项目 build.gradle 中指定 jni 库的目录为 libs 目录。最终 build.gradle 文件如下所示。
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "me.covertness.listen"
        minSdkVersion 16
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.0.0'
    compile files('libs/galaxy.jar')
}
```

## 实现语音识别的功能
### 1. 修改界面
activity_main.xml: 
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent" android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    android:paddingBottom="@dimen/activity_vertical_margin" tools:context=".MainActivity">

    <Button
        android:layout_width="188dip"
        android:layout_height="100dip"
        android:text="@string/start_listening"
        android:layout_centerInParent="true"
        android:onClick="startListening"
        android:textSize="48sp" />

</RelativeLayout>
```

strings.xml:
```xml
<resources>
    <string name="app_name">Listen</string>

    <string name="error_title">错误</string>
    <string name="error_message">出错了，错误码：</string>
    <string name="start_listening">听我说</string>
    <string name="listen_dialog_title">语音识别</string>
    <string name="listen_dialog_wait">请说话</string>
    <string name="listen_dialog_listening">倾听中</string>
    <string name="listen_dialog_speech">你刚才说了：</string>
    <string name="listen_dialog_noword">抱歉，没有听到你的声音，请重试</string>
    <string name="listen_dialog_networkerror">网络连接失败，请稍后重试</string>
    <string name="action_settings">Settings</string>
</resources>
```


### 2. 修改逻辑
MainActivity.java:
```java
package me.covertness.listen;

import android.app.AlertDialog;
import android.app.ProgressDialog;
import android.content.Context;
import android.content.DialogInterface;
import android.os.Handler;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;

import com.baidu.voicerecognition.android.VoiceRecognitionClient;
import com.baidu.voicerecognition.android.VoiceRecognitionConfig;

import java.util.ArrayList;


public class MainActivity extends ActionBarActivity {
    private VoiceRecognitionClient baiduRecognitionClient;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        baiduRecognitionClient = VoiceRecognitionClient.getInstance(this.getApplicationContext());
        baiduRecognitionClient.setTokenApis(your_API_KEY, your_SECRET_KEY);
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

    public void startListening(View view) {
        MyVoiceRecognitionListener listener = new MyVoiceRecognitionListener(this);
        int ret = baiduRecognitionClient.startVoiceRecognition(
                listener, new VoiceRecognitionConfig()
        );
        if (ret == VoiceRecognitionClient.START_WORK_RESULT_WORKING) {
            ProgressDialog ringProgressDialog = ProgressDialog.show(this,
                    getString(R.string.listen_dialog_title), getString(R.string.listen_dialog_wait),
                    false);
            listener.setRecognitionDialog(ringProgressDialog);
        } else {
            AlertDialog alertDialog = new AlertDialog.Builder(this).create();
            alertDialog.setTitle(R.string.error_title);
            alertDialog.setMessage(getString(R.string.error_message) + ret);
            alertDialog.setButton(AlertDialog.BUTTON_NEUTRAL, "OK", new DialogInterface.OnClickListener() {
                public void onClick(DialogInterface dialog, int which) {
                }
            });
            alertDialog.show();
        }
    }
}


class MyVoiceRecognitionListener implements VoiceRecognitionClient.VoiceClientStatusChangeListener {
    private final Context mainContext;
    private ProgressDialog recognitionDialog;
    private String recogntionResult = "";

    MyVoiceRecognitionListener(Context context) {
        this.mainContext = context;
    }

    public void setRecognitionDialog(ProgressDialog recognitionDialog) {
        this.recognitionDialog = recognitionDialog;
    }

    @Override
    @SuppressWarnings (value="unchecked")
    public void onClientStatusChange(int status, Object obj) {
        switch(status){
            case VoiceRecognitionClient.CLIENT_STATUS_START_RECORDING:
                Log.i("ClientStatusChange", "start recording");
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_SPEECH_START:
                recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_listening));
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_SPEECH_END:
                if (recogntionResult.length() > 0) {
                    recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_speech) + recogntionResult);
                } else {
                    recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_noword));
                }
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_FINISH:
                closeDialog();
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_UPDATE_RESULTS:
                ArrayList<String> words = (ArrayList<String>)obj;
                StringBuilder sb = new StringBuilder();
                for (String w : words) {
                    sb.append(w);
                }
                recogntionResult = sb.toString();
                break;
            default:
                break;
        }
    }

    @Override
    public void onNetworkStatusChange(int i, Object o) {

    }

    @Override
    public void onError(int type, int code) {
        String errorMessage;
        if (code == VoiceRecognitionClient.ERROR_CLIENT_NO_SPEECH) {
            errorMessage = mainContext.getString(R.string.listen_dialog_noword);
        } else if (code == VoiceRecognitionClient.ERROR_NETWORK_CONNECT_ERROR) {
            errorMessage = mainContext.getString(R.string.listen_dialog_networkerror);
        } else {
            errorMessage = mainContext.getString(R.string.error_message) + code;
        }

        recognitionDialog.setMessage(errorMessage);
        closeDialog();
    }

    private void closeDialog() {
        Handler handler = new Handler();
        handler.postDelayed(new Runnable() {
            public void run() {
                recognitionDialog.dismiss();
            }
        }, 3000);  // 3000 milliseconds
    }
}
```

**注意将代码中的`your_API_KEY`和`your_SECRET_KEY`替换成自己应用的。**

### 3. 修改AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.covertness.listen" >

    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.WRITE_SETTINGS"/>

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
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

## 测试应用
使用 Android 4.1 或以上版本的手机打开 Listen ，点击`听我说`按钮，看到`请说话`提示后对麦克风说一句话，稍后便可看到屏幕上显示自己刚说过的话了，如下图所示。
![](https://image.covertness.me/android_rangyingyongtindongnishuohua_screencast-Genymotion-2015-05-02_17.17.19.305.gif)

## 要点总结
### 1. 界面
Listen主界面只有一个居中布局（`android:layout_centerInParent="true"`）的按钮，当启动语音识别后会出现一个[ProgressDialog](http://developer.android.com/reference/android/app/ProgressDialog.html)类型的对话框，ProgressDialog多用于后台正在工作时告知用户等待。

### 2. 逻辑
Listen首先对用户的讲话进行录音，然后通过网络将其发送至百度语音识别的云端进行处理，完成后的识别结果通过网络返回，最终显示在界面上。由于 SDK 已经对整个过程进行了封装，因而我们只需对其进行一些个性化的配置，并处理其返回的结果即可。

对其进行配置可通过VoiceRecognitionConfig来完成，目前 SDK 支持对语种、识别语言的领域等参数进行设置，具体可参考 SDK 中的 API 文档。

对返回结果的处理是通过实现接口VoiceRecognitionClient.VoiceClientStatusChangeListener来完成的。
```java
class MyVoiceRecognitionListener implements VoiceRecognitionClient.VoiceClientStatusChangeListener {
    private final Context mainContext;
    private ProgressDialog recognitionDialog;
    private String recogntionResult = "";

    MyVoiceRecognitionListener(Context context) {
        this.mainContext = context;
    }

    public void setRecognitionDialog(ProgressDialog recognitionDialog) {
        this.recognitionDialog = recognitionDialog;
    }

    @Override
    @SuppressWarnings (value="unchecked")
    public void onClientStatusChange(int status, Object obj) {
        switch(status){
            case VoiceRecognitionClient.CLIENT_STATUS_START_RECORDING:
                Log.i("ClientStatusChange", "start recording");
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_SPEECH_START:
                recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_listening));
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_SPEECH_END:
                if (recogntionResult.length() > 0) {
                    recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_speech) + recogntionResult);
                } else {
                    recognitionDialog.setMessage(mainContext.getString(R.string.listen_dialog_noword));
                }
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_FINISH:
                closeDialog();
                break;
            case VoiceRecognitionClient.CLIENT_STATUS_UPDATE_RESULTS:
                ArrayList<String> words = (ArrayList<String>)obj;
                StringBuilder sb = new StringBuilder();
                for (String w : words) {
                    sb.append(w);
                }
                recogntionResult = sb.toString();
                break;
            default:
                break;
        }
    }

    @Override
    public void onNetworkStatusChange(int i, Object o) {

    }

    @Override
    public void onError(int type, int code) {
        String errorMessage;
        if (code == VoiceRecognitionClient.ERROR_CLIENT_NO_SPEECH) {
            errorMessage = mainContext.getString(R.string.listen_dialog_noword);
        } else if (code == VoiceRecognitionClient.ERROR_NETWORK_CONNECT_ERROR) {
            errorMessage = mainContext.getString(R.string.listen_dialog_networkerror);
        } else {
            errorMessage = mainContext.getString(R.string.error_message) + code;
        }

        recognitionDialog.setMessage(errorMessage);
        closeDialog();
    }

    private void closeDialog() {
        Handler handler = new Handler();
        handler.postDelayed(new Runnable() {
            public void run() {
                recognitionDialog.dismiss();
            }
        }, 3000);  // 3000 milliseconds
    }
}
```

可以看出其主要是通过状态变化（`onClientStatusChange`）来告知结果的。识别结果的捕获主要通过`CLIENT_STATUS_UPDATE_RESULTS`状态告知，需要注意的是在一次识别过程中此状态可能会被多次触发，识别结果每次都被全量返回，因而这个Demo仅保存了最终的识别结果。另外两个接口方法用来处理出错的情况，具体可参考 SDK 中的[开发手册](http://bcs.duapp.com/cplat-01/mediacloud%2Fvoice%2FBaidu-Voice-Android-SDK-Manual.pdf) 4.2 节。

### 3. AndroidManifest.xml
SDK 需要如下权限才能正常运行，否则在初始化（`VoiceRecognitionClient.getInstance`）时应用便会崩溃。
```xml
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.WRITE_SETTINGS"/>
```
title: Android学习记录——开发环境搭建
date: 2015/03/28 14:02:23
categories:
- 学习记录
tags:
- Android
- Android Studio
- Genymotion

---
![](http://covertness.qiniudn.com/android_kaifahuanjindajian_mybrowse-finish.png)

使用Android Studio开发Android应用的基本流程，记录了从环境搭建到一个浏览器应用（见上图）开发完成的整个过程。
<!-- more -->

## 需要用到的工具
- [Android Studio](http://developer.android.com/sdk/index.html) 一款官方推荐的Android开发集成环境，基于IntelliJ IDEA
- [Genymotion](https://www.genymotion.com/#!/download) Android模拟器，用于调试应用，也可以直接使用Android手机进行调试

## 安装Android Studio
### 1. 下载Android Studio
可以直接从[Android官网](http://developer.android.com/sdk/index.html)上下载，完成之后进行安装即可。

### 2. 启动Android Studio
启动时如提示未安装JVM，按照提示给出的链接下载安装后再次打开Android Studio。

### 3. 配置Android Studio
第一次使用Android Studio会自动弹出导入配置提示框，保持默认选择ok继续；然后弹出欢迎界面，点击Next出现选择配置提示，保持默认即可，而后出现安卓SDK的License说明，Accept即可；最后选择Finish后会自动下载安装Android SDK，过程可能比较漫长请耐心等待，完成后再次点击Finish完成整个配置过程。

## 创建第一个应用——MyBrowse
完成初始化配置后，Android Studio便会显示开始界面，现在便可以开始创建自己的应用了。
### 1. 选择Start a new Android Studio project
### 2. 配置此应用的一些基本信息，如图所示
### 3. 选择应用运行的Android平台，一般高版本系统可以运行低版本的应用，反之不可以，这里保持默认
### 4. 选择应用的activity，activity类似于窗口的概念，一个应用可以有多个activity，这里选择的是应用加载时的第一个activity，这里保持默认
### 5. 为这个activity设置一些基本的参数，如图所示
完成对应用的配置后，Android Studio会使用Gradle（一种自动构建工具）根据刚才的配置创建应用，结束后注意查看窗口下方有无Error Message，如果没有则说明应用创建成功。

## 安装Genymotion
创建完成后应用已经可以运行，下面安装Genymotion来测试下这个应用（如果使用真实Android手机测试参考[《Android Studio真机调试》](http://blog.csdn.net/zkxhhf/article/details/9270243)）。
### 1. 下载Genymotion
在[Genymotion官网](https://www.genymotion.com/#!/download)创建账号并邮件激活，之后便可下载安装Genymotion。

### 2. 创建虚拟Android手机
首先启动genymotion，登录之前注册的账号；然后点击Add，选择一款与应用运行的Android平台版本相符的手机；最后下载完之后即可启动此手机。

### 3. 安装Android Studio的Genymotion插件
此插件可以在Android Studio中方便地打开Genymotion里的虚拟Android手机。首先打开Android Studio，定位到菜单项File > Settings；然后选择Plugins，点击Browse Repositories；最后右击Genymotion选择Download and install即可。

## 运行应用
点击Android Studio上方类似播放按钮的绿色按键，然后在弹出的Choose Device对话框中选择一款的（虚拟）Android手机即可，等待片刻后便可在对应的（虚拟）Android手机上看到应用已经打开（如果手机处于锁屏状态请先解锁）。

## 实现MyBrowse的功能
刚才运行的应用除了在屏幕上显示一行文字外没有任何其他用处，下面将为这个应用添加浏览器的功能，这样就可以使用它上网了。
### 1. 双击res/layout文件夹中的browse_main.xml，便可看到现在应用的预览，和刚才在手机中显示的一样，现在我们开始修改它

### 2. 在下方选中Text标签页便可看到browse_main.xml的内容，然后将文件的内容用下面的代码覆盖
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <EditText
            android:id="@+id/edit_url"
            android:inputType="textUri"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:hint="@string/edit_url" />

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="visitWeb"
            android:text="@string/button_go" />

    </LinearLayout>

    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <WebView
            android:id="@+id/myWebView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    </LinearLayout>
</LinearLayout>
```


### 3. 打开res/values文件夹中的strings.xml，其内容替换成下面的文本
```xml
<resources>
    <string name="app_name">MyBrowse</string>

    <string name="edit_url">要访问的网页链接</string>
    <string name="button_go">-></string>
    <string name="action_settings">Settings</string>
</resources>
```


### 4. 打开java/me.covertness.mybrowse中的BrowseActivity.java文件，将文件的内容用下面的代码覆盖
```java
package me.covertness.mybrowse;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.EditText;


public class BrowseActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.browse_main);
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_browse, menu);
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

    public void visitWeb(View view) {
        EditText editText = (EditText) findViewById(R.id.edit_url);
        String url = editText.getText().toString();

        WebView myWebView = (WebView)findViewById(R.id.myWebView);
        myWebView.setWebViewClient(new WebViewClient());
        myWebView.getSettings().setJavaScriptEnabled(true);
        myWebView.loadUrl(url);
    }
}
```


### 5. 替换manifests/AndroidManifest.xml为以下内容
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.covertness.mybrowse" >

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".BrowseActivity"
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
按之前运行应用中的方法打开修改后的应用，在顶部输入框中输入[http://baidu.com](http://baidu.com)，然后点击右侧的箭头，片刻之后便可在下方看到百度的首页了。
![](http://covertness.qiniudn.com/android_kaifahuanjindajian_mybrowse-finish.png)

## 要点总结
下面依照修改文件的顺序从界面、逻辑、权限三个方面对应用涉及的知识进行总结。
### 1. 界面
browse_main.xml是界面元素的描述文件，MyBrowse界面上存在三个部件：输入框、箭头按钮和网页浏览器，其依次对应文件中的[EditText](http://developer.android.com/guide/topics/ui/controls/text.html)、[Button](http://developer.android.com/guide/topics/ui/controls/button.html)和[WebView](http://developer.android.com/guide/webapps/webview.html)。除此之外文件中还存在名为[LinearLayout](http://developer.android.com/guide/topics/ui/layout/linear.html)的节，用来规定各个部件在界面中如何布局。

### 2. 逻辑
BrowseActivity.java是应用逻辑的描述文件，MyBrowse只实现了一个功能，当箭头按钮被按下时让网页浏览器显示输入框里填写的URL对应的页面。这个逻辑在此文件的visitWeb函数中实现。
```java
public void visitWeb(View view) {
    EditText editText = (EditText) findViewById(R.id.edit_url);
    String url = editText.getText().toString();

    WebView myWebView = (WebView)findViewById(R.id.myWebView);
    myWebView.setWebViewClient(new WebViewClient());
    myWebView.getSettings().setJavaScriptEnabled(true);
    myWebView.loadUrl(url);
}
```

此函数在箭头按钮被点击时执行：首先从输入框中获取url并将其存入变量中；然后获取网页浏览器对应的句柄，覆盖默认的链接打开行为（避免在网页浏览器中打开其他链接时调用系统的浏览器），允许运行网页中的JavaScript代码；最后载入页面内容。其中部分逻辑在应用初始化时执行效率更高，此处仅是为了方便代码阅读。

### 3. 权限
AndroidManifest.xml告诉Android系统如何启动应用，是一个重要的应用配置文件。MyBrowse需要访问因特网的权限，因此需要在此文件中添加如下配置：`<uses-permission android:name="android.permission.INTERNET" />`。
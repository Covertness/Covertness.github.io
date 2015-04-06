title: Android学习记录——在线影音播放器
date: 2015/04/06 14:49:51
categories:
- 学习记录
tags:
- Android
- 音视频处理
- ExoPlayer
- Gradle

---
![](http://covertness.qiniudn.com/android_zaixianyingyinbofangqi_app_screencast.gif)

一个简单的在线视频播放应用，基于Google去年新推出的ExoPlayer开源框架。
<!-- more -->

## 需要用到的工具
- [Android Studio](http://developer.android.com/sdk/index.html) Android集成开发环境
- [Genymotion](https://www.genymotion.com/) Android模拟器
- [Git](http://git-scm.com) 用于下载位于GitHub上的ExoPlayer代码

Android Studio和Genymotion的基本使用方法可参考[《Android学习记录——开发环境搭建》](http://covertness.me/2015/03/28/Android学习记录——开发环境搭建/)，Git的使用说明可参考[《git - 简明指南》](http://rogerdudler.github.io/git-guide/index.zh.html)。


## 创建应用——ExoPlayerTest
### 1. 使用Android Studio创建一个名为ExoPlayerTest的Android项目
配置除了以下参数之外与MyBrowse项目基本一致：
Minimum SDK: API 16: Android 4.1 (Jelly Bean)
Activity Name: MainActivity
Layout Name: activity_main
Title: ExoPlayerTest
Menu Resource Name: menu_main

### 2. 获取ExoPlayer
在Terminal中定位到ExoPlayerTest项目的上层目录，然后执行如下命令：
```bash
$ git clone https://github.com/google/ExoPlayer.git
```


### 3. 把ExoPlayer添加到ExoPlayerTest项目
修改ExoPlayerTest项目的以下Gradle脚本，添加相应的依赖项：
```
// settings.gradle
include ':app', ':..:ExoPlayer:library'

// build.gradle
dependencies {
    classpath 'com.novoda:bintray-release:0.2.7'
}

// app/build.gradle
dependencies {
    compile project(':..:ExoPlayer:library')
}
```


## 实现播放在线视频的功能
### 1. 修改界面（activity_main.xml）
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">

    <com.google.android.exoplayer.VideoSurfaceView android:id="@+id/surface_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="center"/>

</FrameLayout>
```


### 2. 修改逻辑（MainActivity.java）
```java
package me.covertness.exoplayertest;

import android.media.MediaCodec;
import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.os.Handler;
import android.net.Uri;
import android.view.Menu;
import android.view.MenuItem;
import android.view.Surface;
import android.view.SurfaceHolder;
import android.widget.Toast;

import com.google.android.exoplayer.ExoPlaybackException;
import com.google.android.exoplayer.ExoPlayer;
import com.google.android.exoplayer.MediaCodecAudioTrackRenderer;
import com.google.android.exoplayer.MediaCodecTrackRenderer;
import com.google.android.exoplayer.MediaCodecVideoTrackRenderer;
import com.google.android.exoplayer.TrackRenderer;
import com.google.android.exoplayer.VideoSurfaceView;
import com.google.android.exoplayer.audio.AudioTrack;
import com.google.android.exoplayer.source.DefaultSampleSource;
import com.google.android.exoplayer.source.FrameworkSampleExtractor;


public class MainActivity extends ActionBarActivity implements ExoPlayer.Listener,
        SurfaceHolder.Callback, MediaCodecVideoTrackRenderer.EventListener,
        MediaCodecAudioTrackRenderer.EventListener {
    private ExoPlayer player;                           // ExoPlayer
    private final Handler mainHandler;                  // 负责音视频事件分发
    private TrackRenderer videoRenderer;                // 视频渲染器
    private TrackRenderer audioRenderer;                // 音频渲染器

    private VideoSurfaceView surfaceView;               // 播放界面

    public MainActivity() {
        mainHandler = new Handler();
    }

    private void preparePlayer() {
        if (player == null) {
            player = ExoPlayer.Factory.newInstance(2, 1000, 5000);
            player.addListener(this);

            Uri uri = Uri.parse("http://covertness.qiniudn.com/android_zaixianyingyinbofangqi_test_baseline.mp4");

            DefaultSampleSource sampleSource =
                    new DefaultSampleSource(new FrameworkSampleExtractor(this, uri, null), 2);
            videoRenderer = new MediaCodecVideoTrackRenderer(sampleSource,
                    null, true, MediaCodec.VIDEO_SCALING_MODE_SCALE_TO_FIT, 5000, null, mainHandler,
                    this, 50);
            audioRenderer = new MediaCodecAudioTrackRenderer(sampleSource, null, true, mainHandler, this);

            player.prepare(videoRenderer, audioRenderer);
        }
    }

    private void startPlayer() {
        player.sendMessage(videoRenderer, MediaCodecVideoTrackRenderer.MSG_SET_SURFACE, surfaceView.getHolder().getSurface());
        player.setPlayWhenReady(true);
    }

    private void releasePlayer() {
        if (player != null) {
            player.release();
            player = null;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        surfaceView = (VideoSurfaceView) findViewById(R.id.surface_view);
        surfaceView.getHolder().addCallback(this);

        preparePlayer();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        releasePlayer();
    }

    @Override
    protected void onPause() {
        super.onPause();

        releasePlayer();
    }

    @Override
    protected void onResume() {
        super.onResume();

        preparePlayer();
        startPlayer();
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

    // ExoPlayer.Listener 实现

    @Override
    public void onPlayerStateChanged(boolean playWhenReady, int playbackState) {

    }

    @Override
    public void onPlayWhenReadyCommitted() {

    }

    @Override
    public void onPlayerError(ExoPlaybackException error) {
        Toast.makeText(getApplicationContext(), R.string.play_fail, Toast.LENGTH_LONG).show();
    }

    // SurfaceHolder.Callback 实现

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        startPlayer();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {

    }

    // MediaCodecVideoTrackRenderer.EventListener 实现

    @Override
    public void onDroppedFrames(int count, long elapsed) {

    }

    @Override
    public void onVideoSizeChanged(int width, int height, float pixelWidthHeightRatio) {
        surfaceView.setVideoWidthHeightRatio(
                height == 0 ? 1 : (width * pixelWidthHeightRatio) / height);
    }

    // MediaCodecAudioTrackRenderer.EventListener 实现

    @Override
    public void onAudioTrackInitializationError(AudioTrack.InitializationException e) {

    }

    @Override
    public void onAudioTrackWriteError(AudioTrack.WriteException e) {

    }

    @Override
    public void onDrawnToSurface(Surface surface) {

    }

    @Override
    public void onDecoderInitializationError(MediaCodecTrackRenderer.DecoderInitializationException e) {

    }

    @Override
    public void onCryptoError(MediaCodec.CryptoException e) {

    }
}
```


### 3. 添加一个字符串资源(strings.xml)
```xml
<resources>
    <string name="app_name">ExoPlayerTest</string>
    <string name="action_settings">Settings</string>

    <string name="play_fail">播放失败！</string>
</resources>
```


### 4. 修改AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="me.covertness.exoplayertest" >

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.AppCompat.NoActionBar" >                      <!--开启全屏模式-->
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name"
            android:configChanges="keyboardHidden|orientation|screenSize" >       <!--监听一些可能导致activity重新创建的事件，防止其重启-->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```


## 测试应用
使用Android 4.1或以上版本的手机打开ExoPlayerTest，等待片刻后便可欣赏到邓紫棋的这首2015新曲——《多远都要在一起》。


## 要点总结
### 1. 界面
ExoPlayerTest使用[FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html)承载视频界面，此布局上的控件呈现堆叠效果。第一个添加的控件被放在最底层，最后一个添加到框架布局中的视图显示在最顶层，上一层的控件会覆盖下一层的控件。

### 2. 逻辑
ExoPlayerTest启动后便直接开始加载播放视频，应用暂停或退出都直接停止视频的播放并释放相关的资源，尚未实现用户可操控的逻辑。因而可归纳为加载视频、播放视频和销毁资源三个步骤，对应MainActivity.java中的三个函数preparePlayer、startPlayer和releasePlayer。

```java
    private void preparePlayer() {
        if (player == null) {
            player = ExoPlayer.Factory.newInstance(2, 1000, 5000);
            player.addListener(this);

            Uri uri = Uri.parse("http://covertness.qiniudn.com/android_zaixianyingyinbofangqi_test_baseline.mp4");

            DefaultSampleSource sampleSource =
                    new DefaultSampleSource(new FrameworkSampleExtractor(this, uri, null), 2);
            videoRenderer = new MediaCodecVideoTrackRenderer(sampleSource,
                    null, true, MediaCodec.VIDEO_SCALING_MODE_SCALE_TO_FIT, 5000, null, mainHandler,
                    this, 50);
            audioRenderer = new MediaCodecAudioTrackRenderer(sampleSource, null, true, mainHandler, this);

            player.prepare(videoRenderer, audioRenderer);
        }
    }
```

preparePlayer首先初始化一个ExoPlayer实例，然后使用ExoPlayer默认的采样源DefaultSampleSource从[URL](http://covertness.qiniudn.com/android_zaixianyingyinbofangqi_test_baseline.mp4)中获取视频数据，最后指定MediaCodecVideoTrackRenderer和MediaCodecAudioTrackRenderer为相应的视频和音频渲染器。
需要注意的是因为MediaCodecVideoTrackRenderer和MediaCodecAudioTrackRenderer仅支持[Android原生的几种视频格式](http://developer.android.com/guide/appendix/media-formats.html)，因而这个应用遇到一些其他格式的视频时并不能正常播放，此时onPlayerError会被触发，ExoPlayerTest在这里通过Toast给用户一个提示，告知播放失败。

```java
    private void startPlayer() {
        player.sendMessage(videoRenderer, MediaCodecVideoTrackRenderer.MSG_SET_SURFACE, surfaceView.getHolder().getSurface());
        player.setPlayWhenReady(true);
    }
```

startPlayer发送一个消息给ExoPlayer，告诉它开始播放视频。

```java
    private void releasePlayer() {
        if (player != null) {
            player.release();
            player = null;
        }
    }
```

releasePlayer释放ExoPlayer所占用的资源。

### 3. AndroidManifest.xml
此应用需要全屏显示，通过添加如下配置实现：
```xml
android:theme="@style/Theme.AppCompat.NoActionBar"
```

默认当手机屏幕旋转时系统会重新创建Activity，按照现在的逻辑这样会导致视频重新从头播放，可以通过以下配置规避：
```xml
android:configChanges="keyboardHidden|orientation|screenSize"
```
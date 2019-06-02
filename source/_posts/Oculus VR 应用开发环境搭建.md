title: Oculus VR 应用开发环境搭建
urlname: Oculus VR 应用开发环境搭建
date: 2016/01/10 15:43:24
categories:
- 学习记录
tags:
- 虚拟现实

---
![](https://image.covertness.me/vr_yingyongkaifahuanjindajian_samsung-gear-vr.jpg)

搭建基于 Unity 引擎制作虚拟现实应用的开发环境。
<!-- more -->

## 需要用到的工具
- [Unity 5](http://unity3d.com/cn/get-unity/download?ref=personal) 内置 VR 支持的 3D 应用开发引擎
- [Android SDK](http://developer.android.com/sdk/installing/index.html?pkg=tools) 可选，开发 Gear VR 的应用需要，Oculus DK2 可略过
- Oculus VR 设备 可以是 Oculus DK2 (Desktop) 或者 Gear VR，对于应用开发来说两者仅有少量配置差异，下面均以 Gear VR 为例
- [支持蓝牙的游戏手柄](https://support.oculus.com/hc/en-us/articles/205698678-Bluetooth-Gamepads-controllers-for-Gear-VR) 可选，开发 Gear VR 的应用需要，Oculus DK2 可略过

## 创建第一个应用——Roll-a-ball
### 下载 Unity 官方示例 Roll-a-ball 的源码
```
git clone https://github.com/nicksuch/Roll-a-ball.git
```

将这个项目重命名为 Roll-a-ball-VR ，后续将对这个示例修改以支持 VR 设备。

### 使用 Unity 打开 Roll-a-ball-VR
在项目标签页下打开 `Assets > _Scenes` 然后选择 `MiniGame`，然后就可以在 `Scene` 上看到如下所示的预览界面。
![](https://image.covertness.me/vr_yingyongkaifahuanjindajian_1.PNG)

### 修改配置以支持 VR
打开 Player 设置标签页（位于`Edit > Project Settings > Player`），然后选择 Android 图标的子页，定位到 Other Settings ， Graphics APIs 选择 OpenGLES3 ， 勾选 Virtual Reality Supported ，去掉 GPU Skinning 、 Android TV Compatibility ，将 Bundle Identifier 修改为 `com.covertness.rollball`， Minimum API Level 修改为 19。最后定位到菜单 `Assets > Import Package > Custom Package` 导入 [Oculus Utilities Package](https://developer.oculus.com/downloads/game-engines/0.1.3.0-beta/Oculus_Utilities_for_Unity_5/)。完成后如下图所示。
![](https://image.covertness.me/vr_yingyongkaifahuanjindajian_2.PNG)

### 编译部署应用
1. 在进行编译部署前请确保 Android SDK 已经安装并正确配置（可参考之前的文章[《Android学习记录——开发环境搭建》](http://covertness.me/2015/03/28/Android学习记录——开发环境搭建/)）。
2. [获取 osig 签名文件](https://developer.oculus.com/osig/)并把它放到项目目录 `Assets/Plugins/Android/assets` 下，目录如不存在可自行创建。
3. 打开 Build 设置（位于`File > Build Settings`），选择 Android 平台并将 Texture Compression 设置为 ASTC，然后点击 Build 即可，如果已经连入手机并处于调试模式也可直接点击 Build And Run，完成后将手机放入 Gear VR 里即可启动应用。
4. 需要注意的是 Roll-a-ball-VR 这个应用需要使用游戏手柄控制，所以想要实际体验游戏还需提前配置好支持蓝牙的游戏手柄。

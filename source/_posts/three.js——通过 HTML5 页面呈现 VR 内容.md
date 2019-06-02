title: three.js——通过 HTML5 页面呈现 VR 内容
urlname: three.js——通过 HTML5 页面呈现 VR 内容
date: 2016/12/25 15:01:34
categories:
- 学习记录
tags:
- 虚拟现实

---
![](https://image.covertness.me/vr_player_vr.jpg)

three.js 是一个在网页上绘制 3D 图形的 Javascript 库，它同时支持 WebVR 接口，因而可以通过它很容易地在普通的 HTML5 页面上呈现 VR 内容。
<!-- more -->

VR 随着相关硬件的快速升级而日益普及，作为网络主要入口的浏览器也已构建了相关的支持接口——[WebVR](https://webvr.info/)。接下来我们便通过 three.js 在网页上播放一段 VR 视频。需要说明的是由于目前大部分浏览器还未支持 WebVR 接口，因此我们需要一个 [webvr-polyfill](https://github.com/googlevr/webvr-polyfill) 库，通过它才能在大多数浏览器上直接访问 VR 视频内容。完整的示例可以[点击这里](http://covertness.coding.me/vr-player/)访问（为了保证体验请使用 [Chrome 浏览器](http://www.google.cn/chrome/browser/)访问）。

# 播放 360 度视频
现阶段大部分 VR 相关的视频还是 360 度视频，即能够提供 360 度访问视角，不过就视频格式来说除了拥有[少数几个 VR 相关的参数](https://github.com/google/spatial-media/blob/master/docs/spherical-video-v2-rfc.md)外与传统视频并无太大区别。可以先通过[点击这里](https://image.covertness.me/vr_player_Dance.mp4)访问这个示例的原始视频，我们会看到除了画面有些畸变外与普通视频并无差异。画面的畸变是由于要提供 360 度视角的缘故，而普通的视频只能提供一个平面上的视野，需要在一帧的画面上提供全景的视野就需要将多个画面拼接到一个画面上，最终就变成了刚才看到的那样。而要正常播放这样的视频我们需要将画面还原，其中一种方法便是重新渲染到一个全景的物体上。360 度视频的视角是一个环绕的 360 度，因此只需要将它渲染到一个球体上，然后我们从球体中央观看便是正常的画面了。下面就通过 three.js 来实现这样的效果：
```js
  const WIDTH = window.innerWidth;
  const HEIGHT = window.innerHeight;

  const scene = new THREE.Scene();          // 构建一个场景

  const geometry = new THREE.SphereGeometry(1, 48, 48);    // 创建一个球体， three.js 可以创建很多 3D 物体，可以参考 http://www.52jb.net/biancheng/6921.html

  geometry.applyMatrix(new THREE.Matrix4().makeScale(-1, 1, 1));  // 对球体进行矩阵转换，因为我们最终需要从球体中央进行观看，具体转换逻辑可以参看 http://www.opengl-tutorial.org/beginners-tutorials/tutorial-3-matrices/

  const videoTexture = new THREE.VideoTexture(videoElement);
  videoTexture.minFilter = THREE.LinearFilter;
  videoTexture.magFilter = THREE.LinearFilter;
  videoTexture.format = THREE.RGBFormat;
  videoTexture.generateMipmaps = false;
  videoTexture.needsUpdate = true;

  const material = new THREE.MeshBasicMaterial({ map: videoTexture });  // 创建一个素材，将要渲染的视频应用到它上面

  const mesh = new THREE.Mesh(geometry, material);  // 把素材应用到球体上
  scene.add(mesh);  // 将球体放入场景中

  const camera = new THREE.PerspectiveCamera(75, WIDTH / HEIGHT, 0.1, 100);  // 创建视角，相关参数可参考 http://blog.csdn.net/lingedeng/article/details/7302204

  // 选用 WebGL 进行渲染
  const renderer = new THREE.WebGLRenderer({ antialias: false });
  renderer.setSize(WIDTH, HEIGHT);
  document.body.appendChild(renderer.domElement);

  const animate = (time) => {
    requestAnimationFrame(animate);
    mesh.rotation.y = time * 0.0005;  // 由于还未加入 WebVR 控制接口，为了可以看到整个视角我们先让球体旋转起来
    renderer.render(scene, camera);
  };

  requestAnimationFrame(animate);
```

通过上述逻辑我们就可以以正确的视角播放这段视频了，想要观看这段代码的效果[点击这里](http://covertness.coding.me/vr-player/display.html)。

# 通过 VR 控制视角
上面的示例通过正确的视角播放了这段视频，但并没有加入 VR 相关的控制功能。好在 three.js 已经支持 WebVR 相关的接口，我们只需要添加几行代码就可以加入 VR 控制功能：
```js
  const controls = new THREE.VRControls(camera);  // 应用 VR 设备的位置数据到视角上，这样在旋转 VR 设备时就能同步更新视角了

  const effect = new THREE.VREffect(renderer);  // 通过 WebVR 调用 VR 设备接口进行渲染
  effect.setSize(WIDTH, HEIGHT);

  let vrDisplay = null;
  const animate = (time) => {
    controls.update();
    effect.render(scene, camera);
    vrDisplay.requestAnimationFrame(animate);
  };

  // 获取 VR 设备
  navigator.getVRDisplays().then(function (displays) {
    if (displays.length > 0) {
      vrDisplay = displays[0];

      vrDisplay.requestAnimationFrame(animate);
    }
  });


  // 开启双目的 VR 模式，这个模式可以很方便地在普通的移动设备浏览器上体验 VR 内容
  const onResize = () => {
    effect.setSize(window.innerWidth, window.innerHeight);
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
  };
  const onVRDisplayPresentChange = () => {
    onResize();
  }

  window.addEventListener('resize', onResize);
  window.addEventListener('vrdisplaypresentchange', onVRDisplayPresentChange);
  document.querySelector('button#vr').addEventListener('click', () => {
    vrDisplay.requestPresent([{ source: renderer.domElement }]);
  });
```

在运行上述代码之前还需要引入相关依赖：
- [VRControls.js](https://github.com/mrdoob/three.js/blob/dev/examples/js/controls/VRControls.js)和[VREffect.js](https://github.com/mrdoob/three.js/blob/dev/examples/js/effects/VREffect.js) three.js 对 WebVR 的适配
- [webvr-polyfill.js](https://github.com/googlevr/webvr-polyfill/blob/master/build/webvr-polyfill.js) polyfill WebVR 接口，让大部分未支持 WebVR 的浏览器也能够使用

通过 webvr-polyfill.js 的处理便可以在 PC 端或移动端的 Chrome 浏览器中直接访问 VR 内容，当在 PC 端访问时可以通过鼠标控制 VR 的视角，当在移动端访问时通过陀螺仪来进行控制。此外在移动端访问时我们还可以点击页面右上角的按钮切换到双目的 VR 模式，然后带上 VR 头盔体验更好的效果。
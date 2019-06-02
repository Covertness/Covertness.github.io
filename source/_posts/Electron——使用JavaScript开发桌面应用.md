title: Electron——使用JavaScript开发桌面应用
urlname: Electron——使用JavaScript开发桌面应用
date: 2016/03/13 14:15:34
categories:
- 学习记录
tags:
- JavaScript
- 桌面应用

---
![](https://image.covertness.me/electron_electron-icon-avatar.png)

Electron 让开发传统的桌面应用像制作网页一样简单。
<!-- more -->

虽然 PC 端的桌面应用越来越多地被 Web 应用所取代，但是传统的桌面应用拥有很多独特的优势，如更高的工作效率、多样的通知机制和更易于与操作系统结合等等，许多场景下仍然有其用武之地。
过去开发桌面应用程序是一个极其繁琐和低效的过程，需要掌握操作系统相关接口、学习界面开发框架、应用版本管理、程序 Bug 跟踪以及安装包制作等许多知识，学习成本高且代码复用率低。相比之下 Web 应用的开发则方便许多，基本一致的设计接口，大量可复用的开源库，活跃的社区支持。 [Electron](http://electron.atom.io/) 便是将 Web 开发的种种便利用于桌面应用的开发，降低了桌面应用开发的难度，提升了开发效率。
下面使用 Electron 制作一款经典的贪吃蛇游戏，基于开源社区的支持我们甚至都不需要写任何代码即可完成这样一款游戏。

## 需要用到的工具
- [Git](http://git-scm.com) 用于获取开源社区的代码
- [Node.js](https://nodejs.org) JavaScript 运行时环境，用于 Electron 搭建 Web 运行环境
- [npm](https://www.npmjs.com) JavaScript 包管理工具，用于获取 Electron 及其他 JavaScript 库

## 获取 Electron 应用开发模版
这里使用 [electron-quick-start](https://github.com/atom/electron-quick-start) 作为开发 Electron 应用的模版，在此基础上进行修改。
1. 获取 electron-quick-start

    ```bash
    $ git clone https://github.com/atom/electron-quick-start
    ```

2. 修改 main.js ，将程序的入口页面定位于 app 目录下

    ```javascript
    'use strict';

    const electron = require('electron');
    // Module to control application life.
    const app = electron.app;
    // Module to create native browser window.
    const BrowserWindow = electron.BrowserWindow;

    // Keep a global reference of the window object, if you don't, the window will
    // be closed automatically when the JavaScript object is garbage collected.
    let mainWindow;

    function createWindow () {
      // Create the browser window.
      mainWindow = new BrowserWindow({width: 800, height: 600});

      // and load the index.html of the app.
      mainWindow.loadURL('file://' + __dirname + '/app/index.html'); // 修改 app 的入口页面，稍后会将贪吃蛇程序的代码放到 app 目录下

      // Open the DevTools.
      // mainWindow.webContents.openDevTools(); // 不显示开发者工具

      // Emitted when the window is closed.
      mainWindow.on('closed', function() {
        // Dereference the window object, usually you would store windows
        // in an array if your app supports multi windows, this is the time
        // when you should delete the corresponding element.
        mainWindow = null;
      });
    }

    // This method will be called when Electron has finished
    // initialization and is ready to create browser windows.
    app.on('ready', createWindow);

    // Quit when all windows are closed.
    app.on('window-all-closed', function () {
      // On OS X it is common for applications and their menu bar
      // to stay active until the user quits explicitly with Cmd + Q
      if (process.platform !== 'darwin') {
        app.quit();
      }
    });

    app.on('activate', function () {
      // On OS X it's common to re-create a window in the app when the
      // dock icon is clicked and there are no other windows open.
      if (mainWindow === null) {
        createWindow();
      }
    });
    ```

## 获取贪吃蛇游戏的代码
[html5-snake](https://github.com/JDStraughan/html5-snake) 是一款基于 HTML5 制作的贪吃蛇游戏，我们将它放到 app 目录下即可。
```bash
$ git clone https://github.com/JDStraughan/html5-snake.git app
```

## 测试程序
只需执行 `npm start` 就可以启动这个游戏了，如下所示
![](https://image.covertness.me/electron_snake-preview.gif)

## 打包程序
使用 `electron-packager` 就可以将程序打包为指定平台的可执行程序。
```bash
$ npm install electron-packager
$ ./node_modules/.bin/electron-packager ./ snake --out ../snake --all --version=0.36.0
```
完成后即可在上层目录的 snake 下看到 Linux 、 Windows 和 Mac OS 的可执行程序。
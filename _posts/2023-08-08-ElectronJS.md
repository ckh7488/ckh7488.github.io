---
title: "[ElectronJS] Electron IPC"
author: "ckh"
date: "2023-08-08 23:31:41 +0800"
categories: ["ElectronJS", "JS"]
tags: ["ElectronJS"]  
---

## Electron
웹을 개발하는 방법대로, 플랫폼에 무관하게(chromium만 실행되면) 실행파일을 만들어주는 툴이다.
``vscode``, ``arduinoIDE`` 등, 여러 기업에서 ``Electron``을 이용하여 실행파일을 개발하였다.
간단히 ``구조``와 가장 중요한 ``IPC에 관련된 부분``에 대해 알아보자.

## 구조
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/552640ec-7a20-4245-a91d-1f4d2993f214)
### ``Main process`` 
 NodeJS가 직접 돌아가는 프로세스로, Node Native Module을 사용 할 수 있다.  
### ``Renderer process``
 유저가 직접 보는 화면이 동작하는 프로세스.
  
 ``Main``은 창을 실행파일의 창을 띄우는 역할과 ipc 및 창 자체를 관리하는 부분을 담당.
 ``renderer``는 창안에서 화면을 표시해주는 부분을 담당하며, web을 개발하듯이 화면을 구성할 수 있다.

  
## 통신
renderer 부분이 ``OS api``의 기능이 필요할 때 두가지 방법이 존재한다.
>1. IPC기능 이용
>2. ``contextIsolation: false`` 를 하는 방법(보안문제)  

1번 방법은 electron에서 제공해주는 IPC(sharede memory방식으로 예상)를 이용한다. 
2번 방법은 그냥 node 모듈을 renderer에서 사용하는 것을 허가하는 방법. 

2번 방법은 딱히 설명이 필요 없으므로 넘어간다.(보안문제만 괜찮으면 가장 편하고, 효율적이다)
1번 방법은 아래에 설명하며, 각각의 파일의 역할도 같이 설명한다.

### IPC 구조
> 1. Main.js
> 2. preloader.js
> 3. renderer.js
위의 세 파일을 통해 main과 renderer가 통신한다.

#### Main
![image](https://github.com/ckh7488/ckh7488.github.io/assets/75701998/577d6f1b-ffc6-4f04-8893-3b72d358ad35)
```javascript
//mianwindow 정의
  mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728,
    icon: getAssetPath('icon.png'),
    webPreferences: {
      // contextIsolation: false,
      preload: app.isPackaged
        ? path.join(__dirname, 'preload.js')
        : path.join(__dirname, '../../.erb/dll/preload.js'),
    },
  });

//윈도우 창 이벤트에 따른 기능 구현 예시
 mainWindow.on('ready-to-show', () => {
    if (!mainWindow) {
      throw new Error('"mainWindow" is not defined');
    }
    if (process.env.START_MINIMIZED) {
      mainWindow.minimize();
    } else {
      mainWindow.show();
    }
  });
 mainWindow.on('closed', () => {
    mainWindow = null;
  });

//mainprocess에서 reneder에 데이터 전송 예시
mainWindow?.webContents.send("udpData", mydatas);

// ipcMain 설정. reneder가 ipc-example 채널로 함수 호출 시 main에서 동작하는 함수 정의
// event.reply를 통해 renderer에 데이터를 리턴 할 수 있다. (event.reply 안 쓸 경우, void foo() 형태, reply 쓰면 returnType foo() 형태.
// event.reply를 renderer에서 받을 경우, 해당 채널 (현재 예시는 ipc-example)에 대한 이벤트 콜백함수를 만들어 줘야 한다.
ipcMain.on('ipc-example', async (event, arg) => {
  const msgTemplate = (pingPong: string) => `IPC test: ${pingPong}`;
  console.log(msgTemplate(arg));
  event.reply('ipc-example', msgTemplate('pong'));
});
```
#### Main의 역할 
1. 창의 관리(창 이벤트 처리)
2. node native 함수 구현 및 IPC에 API 노출
3. webcontents함수를 이용하여 renderer에 함수 호출

모든 통신은 첫번째 인자로 channel이 들어가는데 이는 임의의 텍스트로, 양쪽간에 잘 맞춰주기만 하면 된다.
위의 예제에서 ipcMain.on('ipc-example', ...)로 되어있는 함수를 renderer에서 호출하고 싶으면 같은 채널 'ipc-example'로 첫번쨰 인자를 맞춰주면 이 함수를 실행시킨다.
```javascript
//renderer
window.electron.ipcRenderer.sendMessage('ipc-example', ['ping']);
//renderer에서 쓸 수 있는 ipc 객체는 웹 객체인 window에 붙어서 나온다.
```

### preloader
```javascript
//main.js (preloader 정의 부분)
  mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728,
    icon: getAssetPath('icon.png'),
    webPreferences: {
      // contextIsolation: false,
      preload: app.isPackaged
        ? path.join(__dirname, 'preload.js')
        : path.join(__dirname, '../../.erb/dll/preload.js'),
    },
  });

//preloader.js
import { contextBridge, ipcRenderer, IpcRendererEvent } from 'electron';
const electronHandler = {
  ipcRenderer: {
    sendMessage(channel: Channels, ...args: unknown[]) {
      ipcRenderer.send(channel, ...args);
    },
   //define other renderer functions
};
//window 객체에 `electron`객체로 위의 electronHandler 노출.
contextBridge.exposeInMainWorld('electron', electronHandler);

//renderer
window.electron.ipcRenderer.sendMessage('ipc-example', ['ping']);

```
#### preloader역할
1. ``contextBridge``, ``ipcRenderer``를 이용하여 main에 함수 노출.
 * {func1 : ()=>{ipcRenderer 사용 함수 정의}} 이런 식의 객체 생성 후, ``contextBridge.exposeInMainWorld``를 통하여 renderer에 노출.

### renderer
```javascript
window.electron.ipcRenderer.once('ipc-example', (arg) => {
  // eslint-disable-next-line no-console
  console.log(arg);
});
window.electron.ipcRenderer.sendMessage('ipc-example', ['ping']);
```
#### ipc 사용법  
window 객체 ->  preloader에서 정의한 electron 객체 -> ipcRenderer -> preloader에서 정의한 함수 사용  
  
#### renderer역할  
실행창의 내부 보이는 화면을 나타냄.




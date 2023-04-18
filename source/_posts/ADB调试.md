title: ADB调试
author: Jie
date: 2022-11-11 21:25:41
tags:
  - ADB
  - Android
---

## 连接方式
### 1. USB 连接
1. 通过 USB 连接来正常使用 adb 需要以下步骤：  
2. 确认硬件状态正常(包括 Android 设备处于正常开机状态，USB 连接线和各种接口完好)。
3. Android 设备的开发者选项和 USB 调试模式已开启(可以在「设置」-「开发者选项」-「USB调试」打开USB调试)。
4. 确认设备驱动状态正常(安装ADB驱动程序)。
5. 通过 USB 线连接好电脑和设备后确认状态。
6. 通过 `adb devices` 命令查看设备连接情况。
<!-- more -->
### 2. WLAN 连接（需要 USB 线）
借助 USB 通过 WiFi 连接来正常使用 adb 需要以下步骤：  
操作步骤：  
1. 将 Android 设备与要运行 adb 的电脑连接到同一个 WiFi。
2. 将设备与电脑通过 USB 线连接(可通过 adb devices 命令查看设备连接情况)。
3. 通过 adb tcpip 5555 命令让设备在 5555 端口监听 TCP/IP 连接。
4. 断开 USB 连接。
5. 找到设备的 IP 地址(可以在「设置」-「关于手机」-「状态信息」-「IP地址」查看 IP 地址)。
6. 通过 `adb connect <device-ip-address>` 命令使用 IP 地址将 Android 设备与电脑连接。
7. 通过 `adb devices` 命令查看设备连接情况。
8. 使用完毕后可通过 `adb disconnect <device-ip-address>` 命令断开无线连接。

### 3. WLAN 连接（无需借助 USB 线）
**注：需要 root 权限。不借助 USB 通过 WiFi 连接来正常使用 adb 需要以下步骤：**  
1. 在 Android 设备上安装一个终端模拟器(可通过Terminal Emulator for Android Downloads下载)。
2. 将 Android 设备与要运行 adb 的电脑连接到同一个 WiFi。
3. 打开 Android 设备上的终端模拟器，在里面依次运行命令：
```
su
setprop service.adb.tcp.port 5555
```
4. 找到设备的 IP 地址(可以在「设置」-「关于手机」-「状态信息」-「IP地址」查看 IP 地址)。
5. 通过 `adb connect <device-ip-address>` 命令使用 IP 地址将 Android 设备与电脑连接。
6. 通过 `adb devices` 命令查看设备连接情况。

### 4.  WiFi 连接转为 USB 连接
通过adb usb命令以USB模式重新启动ADB：
```
adb usb
```

## 连接手机
手机通过usb连接电脑
在电脑输入
```
adb devices
```
![](/images/pasted-2.png)
开启adb调试之后在电脑上输入
```
  adb shell
```

## 对电脑进行授权
配置好adb的电脑，在C:\Users\用户名\.android文件夹下有一个adbkey.pub文件，右键打开终端，将这个文件夹复制到手机的/data/misc/adb/文件夹下并且重命名为adb_keys  
具体操作如下：
```
adb push adb_keys /data/misc/adb/adb_keys
```
![](/images/pasted-3.png)
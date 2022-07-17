---

layout: default
title: "AOSP"
date: 2022-07-17 00:00:00 -0000
categories: Environment

---
# AOSP build
## 我的编译环境

* Ubuntu18.04 虚拟机
* 内存 16G
* SSD 300G

**AOSP资源信息**  

|   |   |   |   |   |   | 
| -------- | --- | ------- | ----------------- | -------- | -------------------- |  
| Build ID| Tag | Version | Supported devices | Download | Security patch level |  
| -------- | --- | ------- | ----------------- | -------- | -------------------- |  
| SP1A.210812.016.C1 | android-12.0.0_r31 |	Android12 | Pixel 3, Pixel 3 XL | [link](https://cs.android.com/android/platform/superproject/+/android-12.0.0_r31:) | 2021-10-05 |

\
**Pixel 3 binaries for Android 12.0.0 (SP1A.210812.016.B2)**

|   |   |   |   |
| ------------------ | ------- | -------- | ---------------- |
| Hardware Component | Company | Download | SHA-256 Checksum |
| ------------------ | ------- | -------- | ---------------  |
| Vendor image | Google| [Link](https://dl.google.com/dl/android/aosp/google_devices-blueline-sp1a.210812.016.c1-0b9f3bc0.tgz) | ede315cdeef5b97e9a1903b87b367d5fab30ea6e4ba0e9a8432d9b3cd21fb622 |
| GPS, Audio, Camera, Gestures, Graphics, DRM, Video, Sensors | Qualcomm | [Link](https://dl.google.com/dl/android/aosp/qcom-blueline-sp1a.210812.016.c1-3a8f8e14.tgz) | bb817fc9a7fa906efb37880cb58e5c9f951d06e6d3ae302fb286fdd6f73fe804 |

## 安装前初始化配置

### 初始化配置

**升级系统指令（有可能用的到吧）**
```
sudo apt-get update && sudo apt-get upgrade
```
**安装必要的库**
```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5-dev lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig openjdk-11-jdk htop tree p7zip-full python python3 
```
```
snap install --classic kotlin
```

**配置Git**
```
mkdir aosp12 && cd aosp12
git config --global user.name "git用户名"
git config --global user.email "git邮箱，可以随便填"
mkdir ~/bin
PATH=~/bin:$PATH
```
**安装repo包管理工具**
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
**设置python3为默认版本才能同步源码**   
最新的repo 需要Python3,所以我们下面要设置下python3为默认版本,设置优先级, 数值越大, 优先级越高
```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 100
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150
```
**开始同步源码 ( Android 12.0版本 )**   
--depth=1 指令代表之同步当前分支的代码
```
repo init -u https://android.googlesource.com/platform/manifest -b android-12.0.0_r31 --depth=1
```
**同步代码**   
最好指定核心数：m -j8 （使用8个核心，一定要小于等于给虚拟机分配的核心数）
```
repo sync -j8
```
**下载驱动**   
切记Android版本与驱动是一一对应的
```
wget -cP ~/aosp12 https://dl.google.com/dl/android/aosp/google_devices-blueline-sp1a.210812.016.c1-0b9f3bc0.tgz && wget -cP ~/aosp12 https://dl.google.com/dl/android/aosp/qcom-blueline-sp1a.210812.016.c1-3a8f8e14.tgz

tar -C ~/aosp12/ -xzf ~/aosp12/google_devices-blueline-sp1a.210812.016.c1-0b9f3bc0.tgz
tar -C ~/aosp12/ -xzf ~/aosp12/qcom-blueline-sp1a.210812.016.c1-3a8f8e14.tgz
```

**安装驱动**
```
./extract-google_devices-walleye.sh
./extract-qcom-walleye.sh
```
## 开始进行编译
**如果编译Android 8 或更老的版本需要使用Python2**  
*本文不需要改,直接用Python3*
```
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 150
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 100
```  
**开始编译**
```
source build/envsetup.sh
```
或
```
. build/envsetup.sh
```
**lunch 指定目标设备**   
*通过提示选择*
```
lunch
```

  
*直接指定*

|  |  |  |
| ---- | ------- | ----------------- |
|Device|Code name|Build configuration|
| ---- | ------- | ----------------- |
| Pixel 3 |	blueline | aosp_blueline-userdebug |

```
lunch aosp_blueline-userdebug
```
**开始编译**   
编译时最好指定核心数：m -j8 （使用8个核心，一定要小于等于给虚拟机分配的核心数），不指定很有可能编译失败
```
m -j8
```
## 编译完成

**压缩**   
7zr a -t7z -mmt100 压缩出来的文件的路径 要压缩的路径
我们需要的img和android-info.txt
```
7zr a -t7z -mmt100 ~/pixel2_aosp12.0.7z ~/aosp12/out/target/product/walleye/*.img ~/aosp12/out/target/product/walleye/android-info.txt
```
**查看压缩包里面的文件**   
7z l 包名.7z

## 编译结束准备上传
**上传到谷歌云盘**  
**GitHub上传工具源码**  

https://github.com/astrada/google-drive-ocamlfuse/wiki/Headless-Usage-&-Authorization
```
sudo add-apt-repository ppa:alessandro-strada/ppa
sudo apt-get update
sudo apt-get install google-drive-ocamlfuse
```

1. 谷歌云验证这条命令就不演示了，上面给了GitHub的网址，里面有详细操作的命令
2. 填写密钥
3. 建个文件夹
4. 把你的刷机包挪到你所绑定的这个文件夹也就是MyGoogleDrive,他自己就会自动上传   
   
```
mkdir ~/MyGoogleDrive
google-drive-ocamlfuse -label MyGoogleDrive ~/MyGoogleDrive
```


## 问题修复 

#### 开机后wifi有感叹号, 时间无法同步解决办法**
```
adb root
adb shell settings put global captive_portal_https_url https://www.google.cn/generate_204
adb shell settings put global ntp_server time.ustc.edu.cn
```
#### adb 权限问题连不上
```
sudo vim /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", MODE="0666"
```
#### linux 文件限制
````
sudo vim /etc/sysctl.conf
fs.inotify.max_user_watches=524288
sudo sysctl -p
````
#### 注意事项
1. 内存最好>=16G
2. 编译时最好指定核心数：m -j8 （使用8个核心，一定要小于等于给虚拟机分配的核心数），不指定很有可能编译失败
3. 编译android 12 时，需要kotlin运行时

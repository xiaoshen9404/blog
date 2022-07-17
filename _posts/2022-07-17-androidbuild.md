---

layout: default
title: "AOSP"
date: 2022-07-17 00:00:00 -0000
categories: Environment

---

# 配套版本源码阅读
https://cs.android.com/android/platform/superproject/+/android-8.0.0_r34:

# 下载pixel 2 所配套安卓版本的驱动
https://developers.google.com/android/drivers#walleyeopd3.170816.023

# 我们所用的编译环境是
谷歌云台湾节点，在海外不受限制
Ubuntu18.04版本
SSD 300G

# -------------------------------安装前初始化配置----------------------------------
# 使用的版本是Ubuntu 18.04
# 升级下系统
sudo apt-get update && sudo apt-get upgrade

# 安装必要的库
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5-dev lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig openjdk-8-jdk htop tree p7zip-full python python3

# 初始化配置
mkdir aosp8 && cd aosp8
git config --global user.name "这里填你自己的英文名 随便填"
git config --global user.email "这里填你自己的邮箱 随便填"
mkdir ~/bin
PATH=~/bin:$PATH

# 在这里注意一下，由于谷歌最新的repo(源码同步工具是在python3上跑的)
# 所以我们下面要设置下python3为默认版本
# 当然也有老版本的repo源码同步工具
# 老版本的网址在这里 https://source.android.com/setup/develop#old-repo-python2
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo

# 设置python3为默认版本才能同步源码
# 设置优先级, 数值越大, 优先级越高
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 100
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 150

# 开始同步源码 ( Android 8.0版本 )
# 末尾加了--depth=1会快很多 因为这样他不会把历史的代码记录给从头到尾克隆过来（只克隆最新的历史记录）
repo init -u https://android.googlesource.com/platform/manifest -b android-8.0.0_r34 --depth=1

# 给个100个线程同步一下（如果你使用国内源同步的话, 比如中科大源，清华源. 只能选择4, 5个线程这样子，会有限制）
# 同步代码线程多点无所谓，线程100，200随便加，同步失败了再输入一遍这个代码就行
# 同步失败了不会伤代码，只需重新同步就行
repo sync -j5

# 下载pixel2驱动解压并调制
# 如果要编译其他手机, 请看这里https://developers.google.com/android/drivers
# 找到你与安卓版本对应的手机型号
# ！！！切记要看清楚，不同小版本的安卓比如安卓8.0的r13 和r14分支对应的驱动会不一样
# ！！！切记切记下载驱动手机型号千万别看错了，刷错了就成砖无解！！！
wget -cP ~/aosp8 https://dl.google.com/dl/android/aosp/google_devices-walleye-opd3.170816.023-cc502994.tgz && wget -cP ~/aosp8 https://dl.google.com/dl/android/aosp/qcom-walleye-opd3.170816.023-75f61a87.tgz

tar -C ~/aosp8/ -xzf ~/aosp8/google_devices-walleye-opd3.170816.023-cc502994.tgz
tar -C ~/aosp8/ -xzf ~/aosp8/qcom-walleye-opd3.170816.023-75f61a87.tgz


# 这里需要安装一下 pixel2 的驱动文件, 要一个一个来
./extract-google_devices-walleye.sh
./extract-qcom-walleye.sh

# ---------------------------------开始进行编译------------------------------------
# 编译的时候是python2上运行的
# 所以编译的时候要设置回来
# 先把python2设置默认版本要不然编译会出错
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 150
sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 100

# 开始编译
. build/envsetup.sh
# lunch 一下选择你手机对应的设备名字
# 具体设备名字看这里https://source.android.com/setup/build/running#selecting-device-build
lunch
# 开始编译
m
# -----------------------------------编译完成------------------------------------

# 7z压缩一下
# 给个100线程压缩一下
# 使用方法
# 7zr a -t7z -mmt100 压缩出来的文件的路径 要压缩的路径
# 我们这里只需要他的img文件还有一个android-info.txt就行了
7zr a -t7z -mmt100 ~/pixel2_aosp8.0.7z ~/aosp8/out/target/product/walleye/*.img ~/aosp8/out/target/product/walleye/android-info.txt

# 查看压缩包里面的文件
7z l 包名.7z

# --------------------------------编译结束准备上传----------------------------------
# 上传到谷歌云盘
# GitHub上传工具源码
# https://github.com/astrada/google-drive-ocamlfuse/wiki/Headless-Usage-&-Authorization

sudo add-apt-repository ppa:alessandro-strada/ppa
sudo apt-get update
sudo apt-get install google-drive-ocamlfuse

# 1.谷歌云验证这条命令就不演示了，上面给了GitHub的网址，里面有详细操作的命令
# 2.填写密钥

# 3.然后建个文件夹
mkdir ~/MyGoogleDrive
google-drive-ocamlfuse -label MyGoogleDrive ~/MyGoogleDrive

# 然后把你的刷机包挪到你所绑定的这个文件夹也就是MyGoogleDrive
# 他自己就会自动上传

# -----------------------------刷机过程----------------------------

# 开机后wifi有感叹号, 时间无法同步解决办法
adb root
adb shell settings put global captive_portal_https_url https://www.google.cn/generate_204
adb shell settings put global ntp_server time.ustc.edu.cn

# adb 权限问题连不上
sudo vim /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", MODE="0666"

# linux 文件限制
sudo vim /etc/sysctl.conf
fs.inotify.max_user_watches=524288
sudo sysctl -p


# 注意事项
1. 内存最好>=16G
2. 编译时最好指定核心数：m -j8 （使用8个核心，一定要小于等于给虚拟机分配的核心数），不指定很有可能编译失败
3. 编译android 12 时，需要kotlin运行时




```
参考资料
https://www.jianshu.com/p/baf7a71446e0
```
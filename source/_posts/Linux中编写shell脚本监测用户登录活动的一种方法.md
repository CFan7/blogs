---
title: Linux中编写shell脚本监测用户登录活动的一种方法
date: 2023-10-01 14:00:00
categories:
- Linux
cover: https://626c-blogs-7gj9u1232e248417-1313350496.tcb.qcloud.la/blogs/post%3A%20Linux%E4%B8%AD%E7%BC%96%E5%86%99shell%E8%84%9A%E6%9C%AC%E7%9B%91%E6%B5%8B%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E6%B4%BB%E5%8A%A8%E7%9A%84%E4%B8%80%E7%A7%8D%E6%96%B9%E6%B3%95/ICARUS-CLI.png?sign=2a28f5ec80290e8b0c86a37d3459a89c&t=1696190508

---



整理一些曾经的blogs到这里。本篇是操作系统课程实验的一部分，改编自本人的CSDN账号。

<!--more-->

## 平台

腾讯云轻量服务器CentOS 7.6 64bit	
mac终端（基于bash)

## 实验目的
在Linux中编写shell脚本实现监测用户登录活动
## 实验内容

在终端运行usr_monitor  username（username是键盘输入的用户名），先输出当前系统已登陆的用户名单，随后检查输入的用户名是否在其中：在则输出user [username] has logon后结束；不在则输出waiting user [username] …并持续等待，每5秒进行一次检查，直到用户username登陆。
## 实验流程
#### 1.编写shell脚本
```shell
if [ $# -ne 1 ] # 利用$#判断命令行有几个参数
then
        echo "Usage: usr_monitor username" # 参数不对就报错
        exit
fi
user_input=$1 # 将第一个参数赋给user_input
echo -e "You will monitor [$user_input]\n"
user=$(who)
username=`who |awk '{print $1}'` # 只截取登陆了的用户名
echo -e "Current user list is:\n$username\n"

compare=$(echo $username | grep "${user_input}") #获取username中与user_input相同的部分

while [ "$compare" == "" ]
do
        echo "waiting user [$user_input] ..."
        sleep 5
        username=`who |awk '{print $1}'`
        result=$(echo $username | grep "${user_input}")
done
echo "[$user_input] is log on"
```

#### **2.创建新用户**
首先切换到root模式下

```shell
$ sudo su
```
创建用户，取个名字（此后用username代替）
```shell
$ useradd username
```
**设置密码**

```shell
$ passwd username
```


 #### 3.给脚本赋予执行权限


```shell
$ chmod u+x usr_monitor
```
当前用户获得了该脚本的执行权限后，可以直接调用该脚本名来**执行该脚本**

```shell
$ usr_monitor username
```
极可能会报错
```shell
-bash: usr_monitor: command not_found
```

**原因**：赋予权限是正确的，但shell是根据环境变量PATH的值，依次在其列出的几个目录中搜索键盘命令的，而脚本并没有存放在这些目录中。
**解决方案**（仍不完美：每次开启新进程都需要输入）**：**
```shell
$ PATH=$PATH:./
$ export PATH
```
#### **此时再执行脚本便可成功**

#### 4.登陆（登陆应在脚本运行中进行）
Linux中登陆分为本地登陆和远程登录。本次实验采用SSH远程登录，不用本地登陆的原因放在后面（还是我太菜）。
首先在腾讯云界面中找到公网IP地址
然后在mac终端输入

```shell
$ ssh username@公网IP
```
确认后输入之前给新用户设置的密码即可连接
**如果你使用的是虚拟机，那么只需要知道你的IP，再开一个终端输入上述命令同样可以解决问题。**
## 效果
<img src="https://626c-blogs-7gj9u1232e248417-1313350496.tcb.qcloud.la/blogs/post%3A%20Linux%E4%B8%AD%E7%BC%96%E5%86%99shell%E8%84%9A%E6%9C%AC%E7%9B%91%E6%B5%8B%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E6%B4%BB%E5%8A%A8%E7%9A%84%E4%B8%80%E7%A7%8D%E6%96%B9%E6%B3%95/%E5%B7%B2%E7%99%BB%E5%BD%95.png?sign=55f10b2dc486700e5afe3ba75db77012&t=1696190540" alt="Logon" style="zoom:67%;" />

---
title: 笔记-Linux
categories: 好的学习笔记
tags: 运维
---
# Linux 实操命令

### 快捷键

* #：跳转相同



### 修改ip

> vi /etc/sysconfig/network-scripts

* 将BOOTPORTO改为static，自动获取改为静态
* ONBOOT改为yes，修改配置后自动启动网卡
* 添加IPPADDR和NETMASK

![image-20230310113345215](http://rqvj3u2c0.hn-bkt.clouddn.com/note/image-20230310113345215.png)





### 关闭Selinux

> vi /etc/selinux/config
>
> 将SELINUX=disabled
>
> reboot

![image-20230426102258868](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230426102258868.png)





### CPU100%

最近在服务器上安装了postgresql，结果第二天发现服务器 cpu 100%了。

> #查看进程情况
>
> top



![image-20230518102046844](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230518102046844.png)



可以看到，【kdevtmpfsi】这个不知道什么东西，给CPU干爆了，于是Google后得知这是一个挖矿病毒，于是乎我立即把它停掉，并且把文件也删掉了。

> #终止进程
>
> kill -9 31749
>
> #查找并删除文件
>
> find / -name "\*kdevtmpfsi\*" 
>
> rm -rf 



CPU 立即就正常了。但是不过几分钟，有飙上去了............................

于是分析了一下，应该是启动某个服务的时候带进来了病毒，结果果然是，把postgres服务停了，再把进程删了，CPU就正常了。但是这样治标不治本，于是又Google了。参考这里给出的处理方式，[记一次服务器linux（centos7）被postgres病毒攻击的事故](https://www.jianshu.com/p/bd6b89acf789) 。



### 执行sh脚本文件

#### 错误1：

>  ---> Running in db95be48be52
>
>  not foundst



#### 错误2：

> : invalid option: set: -
> set: usage: set [-abefhkmnptuvxBCHP] [-o option-name] [--] [arg ...]



#### 解决：

原因主要是因为你可能在Windows环境下打开过.sh文件，那么无形中就会改变文件的一些属性，比如换行符的问题。





### 启动docker服务错误

#### 错误1：

> Error response from daemon: driver failed programming external connectivity on endpoint server (61827684079d5a2bc5da93af38fadf8f8e67f2726031c4661e8c0485c37813c0):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 9091 -j DNAT --to-destination 192.168.20.2:9091 ! -i br-5381b5984774: iptables: No chain/target/match by that name



#### 解决：

docker 服务启动的时候，docker服务会向iptables注册一个链，以便让docker服务管理的containner所暴露的端口之间进行通信。

在开发环境中，如果你删除了iptables中的docker链，或者iptables的规则被丢失了（例如重启firewalld），docker就会报iptables。

> #重启docker后再启动服务
>
> systemctl restart docker









### cat 文件追加

> 文件内容打印到标准输出设备。
>
> 打印到控制台可以用来展示，打印到文本可以用来查看

![image-20230329102952055](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329102952055.png)

![image-20230329103004499](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329103004499.png)





### sed 的 /g

> 没有/g只匹配第一个就结束，有则是全部

![image-20230329103019415](https://note-1305755407.cos.ap-nanjing.myqcloud.com/note/image-20230329103019415.png)



### Nginx 403-forbidden

#### 错误

**日志：**

> directory index of "/data/files/" is forbidden, client: 192.168.31.1, server: 192.168.31.128, request: "GET /files/ HTTP/1.1", host: "192.168.31.128"



**查看SELinux：**

> 命令：getenforce
>
> Enforcing *# 出现这个表示已强制执行安全策略了*



**SELinux共有3个状态：**

* enforcing （执行中）
* permissive （不执行但产生警告）
* disabled（关闭）



#### 解决

1、临时关闭（重启机器后失效）

> setenforce 0



2、永久关闭（需要重启机器）

> sed -i s/SELINUX=enforcing/SELINUX=disabled /etc/selinux/config







# GIT 命令

##### 1、删除文件

> git rm -r 文件名



##### 2、记住密码

> git config --global credential.helper store



# Mac 快捷键

### Command键
主要用于“文本”和“文件”的Windows的ctrl键，一些文本的复制、粘贴、撤销、全选、查找都是一样的。


### Option键
更多选择操作，配合Cmd
> 有事儿没事儿，多加一个Opt，或许有意外惊喜
* 强制退出：Cmd + Opt + Esc
* 打开路径：Cmd + Opt + P
* 隐藏其他：Cmd + Opt + H
* 并排：Cmd + Opt + 上/下


### Ctrl键
主要用于系统的快捷键，单独使用。相当于Windows的Ctrl+Alt，比如窗口切换。

### 窗口
* 隐藏当前：Cmd + H
* 隐藏其他：Cmd + Opt + H


* 关闭：Cmd + W
* 退出：Cmd + Q

* 全屏展开：Cmd + Ctrl + F

### 文件夹
* 新建文件夹：Cmd + Shift + N
* 删除文件夹：Cmd + Delete

### 跳转
* Ctrl实现上下左右光标调整
* Option实现上下移动

### 删除
* 删除行：Cmd + Shift + K
  
* 删除后面所有：Ctrl + K
* 删除后面单个：Ctrl + D

* 删除前面所有：Cmd + Delete

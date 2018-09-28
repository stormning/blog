---
title: centos7安装virtualbox
date: 2018-03-14 14:00
tags: [centos,virtualbox]
categories: [运维]
---
### 安装Virtualbox
#### 利用yum安装
```bash
cd /etc/yum.repos.d
wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
yum --enablerepo=epel install dkms
yum install VirtualBox-5.2
```
#### 添加Virtualbox的用户(可选)
```bash
usermod -a -G vboxusers 用户名称
```

#### 安装一个种子镜像（需要远程图形界面）
安装XQuartz
1、启动XQuartz
2、登陆远程主机：
ssh -X username@host
输入密码登陆
3、登陆后输入setenv回车，查看DISPLAY变量，对DISPLAY变量设置：
setenv DISPLAY localhost:10.0
4、可输入xclock，如果弹出窗口，说明可以使用远程主机的图形界面了。
5、运行VirtualBox &
6、退出时，先关闭图形窗口，然后命令行上输入exit退出

问题：XQuartz响应太慢
ssh -c arcfour,blowfish-cbc -XC host.com
ssh -Y -C -o CompressionLevel=9  -c arcfour,blowfish-cbc user@hostname
如果加密选项没有，也可以省略

### web方式管理virtualbox
&emsp;&emsp;有时候，virtualbox所处的服务器是台没有显示器的远程主机，我们需要远程管理虚拟主机。可以[利用x11 server来远程渲染图形界面来操作主机](http://slyak.com/2018/03/13/ssh-x11-mac)，也可以安装virtualbox的图形管理界面。

&emsp;&emsp;但是考虑到可能其他用户也有管理虚拟主机的需求，所以采用web方式（免安装x11 server）是最正确的姿势。既然是当工具使用，当然要是最快捷的方式（什么繁琐的安装过程我不care），非docker莫属。

#### 运行virtualbox的webserver

##### 配置vbox.cfg
```bash
echo "\
VBOXWEB_USER=root
VBOXWEB_HOST=0.0.0.0
VBOXWEB_PORT=18083
VBOXWEB_TIMEOUT=300
VBOXWEB_CHECK_INTERVAL=5
VBOXWEB_THREADS=100
VBOXWEB_KEEPALIVE=100
VBOXWEB_LOGFILE=/var/log/vboxweb.log"\
>/etc/vbox/vbox.cfg
```
#### 运行vboxserv
```bash
systemctl enable vboxweb-service
systemctl start vboxweb-service
```
#### 运行和webserver通信的phpvirtualbox
```bash
docker run --name vbox_http --restart=always -p 80:80 \
    -e SRV1_PORT_18083_TCP=192.168.xx.xx:18083 -e SRV1_NAME=Server1 -e SRV1_USER=root -e SRV1_PW='123456' \
    -d jazzdd/phpvirtualbox
```
注意：上面的用户名密码是你宿主机的密码。
从上面命令可以看出phpvirtualbox是可以管理多个远程的virtualbox的

#### 运行增强包
```bash
wget https://download.virtualbox.org/virtualbox/5.2.16/Oracle_VM_VirtualBox_Extension_Pack-5.2.16-123759.vbox-extpack
VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-5.2.16-123759.vbox-extpack 
```

#### WEB控制台（可选）
```bash
docker stop vbox_http
systemctl stop vboxweb-service.service

VBoxManage modifyvm 虚拟机名称 --vrdeport 3389-4000
VBoxManage modifyvm 虚拟机名称 --vrde on

systemctl start vboxweb-service.service
docker start vbox_http
```

问题1：The machine 'xxx' is already locked for a session (or being unlocked)
解决方法：
```bash
VBoxManage startvm 虚拟机名称 --type emergencystop
```

问题2：The Adobe Flash plugin is not installed.
解决方法：换成firefox就ok了


#### 最终效果图
浏览器输入ip和端口号即可访问
{% asset_img virtualbox.png phpvirtualbox %}

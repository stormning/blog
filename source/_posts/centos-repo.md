---
title: 基于centos的软件自助安装服务器
date: 2018-04-13 14:57
tags: [centos,离线安装,镜像]
categories: [运维,DevOps]
---

## 软件安装自助服务
### 目标
1. 导出文件方式、便携、离线部署
2. 根据导出的文件自动化安装离线仓库
3. 基于离线仓库自动化安装软件，支持多机分发
4. 对未涵盖的软件，可以在离线仓库管理端，进行自助式生成

### 仓库实现思路
1. 利用--downloadonly下载docker离线安装文件
2. createrepo创建离线docker的repo
3. rpm -Uvh --force --nodeps *.rpm 安装docker
4. docker镜像导入[ nginx镜像服务器和其他源管理工具, ansible管理镜像（会将所有repo拷贝至各个宿主机） ]
5. 根据各种安装需求准备repo配置文件

{% asset_img architecture.png %}

### 使用仓库
1. 准备工作
	1. 利用仓库提供的源管理工具，自助添加其他需要的源
	2. 编写软件运行自动化脚本
2. 实际使用
	2. 利用Ansible Tower分发repo到各个宿主机(分发仓库中的repo)
	3. 利用Ansible Tower分发软件自动化运行脚本到各个宿主机

## 仓库详细实现步骤

#### 利用--downloadonly下载docker安装包
如：java <br/>
yum install --downloadonly --downloaddir=/opt/yum/docker docker

#### 利用createrepo+nginx提供服务
createrepo {repo仓库基本路径} /{不同软件根目录}
并通过定时任务+reposync同步资源

#### 其他自定义软件
1. 上传文件
2. 预处理脚本
3. yum install --downloadonly --downloaddir=/opt/yum/xx xx
4. repo模板(下载官方repo定义，修改其中地址为离线服务器地址)










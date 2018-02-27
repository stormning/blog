---
title: hexo牵手github爱上docker
date: "2018/02/26 17:25"
tags: [容器]
---

## 前言
一直想找个软件写个人博客，要求很简单，文章可移植性高，数据可以自动备份（有时候虚拟主机会抽风的）。可能我太挑剔，一直找不到我喜欢的，比如wordpress，html格式太重不够简洁，数据库要写脚本定时备份。

突然发现hexo是不错的选择，可以用markdown写作，而且代码可以托管到github。随便扫描了一眼安装过程，嗯，太麻烦，万一虚拟机挂了岂不是还要重新这个痛苦过程？我的虚拟机咋这么脆弱：），想起了docker的write once use anywhere 理念，遂打算将其docker化。捣鼓了两天终成正果：

[github地址](https://github.com/stormning/env-tool-suite/tree/master/software/hexo)

## 架构

{% asset_img hexo.png %}

1. 通过clone你的github站点进行初始化.
2. 如果clone结果为空 , 我们会使用`hexo init`来进行默认的初始化.
3. 推送初始化结果到你到github仓库.
4. 使用`hexo generate`生成静态文件.
5. 我们提交一些修改（比如换皮肤、写博客）到github.
6. Github利用webhook进行通知.
7. Webhook服务器接收到事件，从服务器拉取最新的修改.
8. 使用`hexo generate`生成静态文件.
9. 通过浏览器我们可以立刻看到修改结果.

综上，所有事情都会通过这个神奇的docker镜像，自动化完成。

### 逻辑架构

{% asset_img hexo-module.png %}

## 安装和运行步骤
### 准备工作
1. 如果你已经有基于hexo的博客，直接用它的博客地址就OK了，否则需要创建一个空的博客（记住，什么都不要有，包括README）.
2. 添加webhook，包括payload url(服务器ip和端口号4001)和secret, 选择content type "application/json" 
{% asset_img webhook.png %}

### 使用nginx作为你的web服务器
nginx的配置文件，请将命令行中的“your\_domain\_or\_ip”，替换成你的域名或ip
``` bash
curl -Ls https://raw.githubusercontent.com/stormning/env-tool-suite/master/software/hexo/hexo.conf \
    | sed "s|slyak.com|your_domain_or_ip|g" > /etc/nginx/conf.d/hexo.conf
service nginx restart
```
``` bash
docker run -idt -p 4001:4001 --name hexo \
    -e GIT_URL=your_blog_ssh_url \
    -e GIT_ACCOUNT=your_github_account \
    -e WEB_HOOK_SECRET=your_web_hook \
    -v /var/hexo:/var/hexo \
    slyak/hexo
```

### 或者用hexo自带的web服务器
``` bash
docker run -idt -p 4000:4000 -p 4001:4001 --name hexo \
    -e GIT_URL=your_blog_ssh_url \
    -e GIT_ACCOUNT=your_github_account \
    -e WEB_HOOK_SECRET=your_web_hook \
    -v /var/hexo:/var/hexo \
    slyak/hexo hexo server
```

### 设置SSH key
``` bash
docker logs -f hexo
```

你将看到如下:

{% asset_img rsa_key.png %}

把key拷贝到github到设置里，不要忘了勾选"Allow write access", 这个配置完后, hexo会继续下一步的初始化工作.

{% asset_img rsa.png %}

### 环境变量和端口描述

Docker run 环境变量:

| 环境变量 | 描述 |  是否必须  |
| --------    | :-----   | :----: |
| GIT_URL     | github仓库的SSH地址，注意不是https |   Y    |
| GIT_ACCOUNT | 你的github账号（email） |   Y    |
| WEB_HOOK_SECRET | Github webook 密码 |   Y    |
| HEXO_HOME | Hexo 安装路径, 默认 `/var/hexo` |   N    |

端口:

| 端口号 | 描述 |
| --------    | :----- |
| 4000     | Hexo web服务器的端口 (如果你使用hexo的服务器作为web服务器的话) |
| 4001 | Webhook服务器的端口 |


### 有用的链接
1. [Hexo皮肤](https://hexo.io/themes/)
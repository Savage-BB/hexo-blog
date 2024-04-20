---
title: Hexo 部署 — Ubuntu
date: 2023-05-29 
description: Hexo 部署在Ubuntu云服务器上
categories: other
tags: 
  - git
  - hexo
---
# Hexo 部署 — Ubuntu

### 1.安装git

```bash
apt update
apt install git
git --versioin  //安装完成后可以看到git版本
```

### 2.创建git用户

- 创建git用户

```bash
adduser git
```

- 获取权限

```bash
chmod 740 /etc/sudoers
```

- 增加改回权限

```bash
vim /etc/sudoers  

//在root后面增加git用户的权限，有时候不是root而是admin或者其他初始管理员名称
root    ALL=(ALL)       NOPASSWD:ALL
git     ALL=(ALL)       NOPASSWD:ALL
```

- 退回权限

```bash
chmod 400 /etc/sudoers
```

### 3.创建git仓库

- 创建网站根目录

```bash
mkdir /home/hexo
```

- 给git用户对这个文件夹的操作权限

```bash
chown git:git -R /home/hexo
```

### 4.设置git `hook`自动部署

- 建立`git`仓库

```bash
cd /home/git  //**这个很重要，使用nginx挂载需要这个地址
git init --bare blog.git  //初始化git仓库
```

- 赋予`git`用户对这个仓库的权限

```bash
chown git:git -R blog.git
```

- 在`/home/hexo/blog.git` 下，有一个自动生成的`hooks`文件夹添，添加`git` 钩子`post-receive`

```bash
vim blog.git/hooks/post-receive
```

- 编写文件

```bash
#!/bin/bash 
 git --work-tree=/home/hexo --git-dir=/home/git/blog.git checkout -f
 
 //work-tree 是提交的代码存放的地址  
 //--git-dir 是git仓库地址
```

- 修改文件权限，使其可执行

```bash
chmod +x /home/git/blog.git/hooks/post-receive
```

### 5.配置本地Hexo

- 修改`_config.yml` 中的配置

```bash
deploy:
  type: git
  repo: git@127.23.43.23:/home/git/blog.git   //127.23.43.23 是你自己的服务器的IP
  branch: master
```

- 部署

```bash
hexo clean
hexo g
hexo d  //输入完成，还需要输入git用户的密码
```

现在`cd /home/hexo`目录下已经可见看到提交的代码了

![Untitled](/pic/Hexo%20%E9%83%A8%E7%BD%B2%20%E2%80%94%20Ubuntu%2040a44028e3ca4b709be834b1dae02bed/Untitled.png)

### 挂载静态资源

- 下载`nginx`

```bash
apt install nginx
```

- 修改配置文件，将`/home/hexo`目录挂载到`nginx`

```bash
vim /etc/nginx/sites-available/default  //vim 默认配置文件 
```

- 我这里直接放在网站根目录80端口，用浏览器访问我的网站会直接跳到博客页面（HTTP请求默认是80端口）。

```bash
    # 网站的根目录
    root /home/hexo;
```

[Hexo博客部署到腾讯云服务器(使用宝塔面板) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/128649492)
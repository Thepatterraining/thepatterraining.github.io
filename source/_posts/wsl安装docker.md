---
title: wsl安装docker
date: 2021-06-07
tags: ['docker','wsl']
category: docker
article: wsl安装docker
---

# wsl安装docker

wsl 使用的是ubuntu 20的版本，所以可以直接使用`apt`进行安装，docker官网的安装也是推荐使用的apt进行安装。

需要先更新apt安装包

```
sudo apt-get update
```

安装一些依赖

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

添加Docker官方的GPG密钥：
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

添加docker的源

```
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新apt安装包

```
sudo apt-get update
```

安装docker

```
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

安装完成后启动docker

```
sudo service docker start
```

测试，运行hello world

```
sudo docker run hello-world
```


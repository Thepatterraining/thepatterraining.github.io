---
title: wsl安装go环境
date: 2020-08-21 10:12:47
tags: ['go']
category: go
article: wsl安装go环境
---

# wsl

wsl是可以在windows里面运行linux的一个软件。是微软官方发行的。

## 安装go

先去到安装目录`/usr/local/src`。

从go官网下载go tar包。

```
 sudo wget https://golang.org/dl/go1.16.3.linux-amd64.tar.gz
```

然后解压

```
sudo tar -zxvf go1.16.3.linux-amd64.tar.gz
```

接下来需要配置环境变量。可以设置在/etc/profile文件里面也可以设置在其他地方，我用的是zsh，所以我的环境变量配置在~/.zshrc文件里面。

执行`vim ~/.zshrc`。然后添加变量信息

```
export GOROOT="/usr/local/src/go"
export PATH="$PATH:$GOROOT/bin"
```

接下来重新加载一下配置文件

```
source ~/.zshrc
```

现在go就安装好了。可以执行`go version`查看go的版本。

## 创建第一个go 项目


我把项目放在`~/go`目录下。所以把这个目录添加到配置文件里面。因为这个目录也相当于`$HOME/go`。所以我们直接添加。

执行`vim ~/.zshrc`。然后添加变量信息

```
export GOPATH="$HOME/go"
```


接下来重新加载一下配置文件

```
source ~/.zshrc
```

在项目目录下面创建第一个文件。

```
vim ~/go/hello.go
```

添加下面的代码

```

package main

import "fmt"

func main () {
        fmt.Printf("hello world\n")

}
```

进行编译

```
go build ~/go/hello.go
```

编译完成出现`hello`文件，直接执行。

```
~/go/hello
```

就可以看到输出了。
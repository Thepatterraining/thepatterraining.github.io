---
title: k8s之etcd基本操作
date: 2018-11-27 15:11:07
tags: ['k8s','etcd']
category: etcd
article: k8s之etcd一
---


# 安装
可以参考etcd的github 
>https://github.com/etcd-io/etcd

如果是mac的话，简单的方式应该是运行如下命令
> brew install etcd

# 介绍

etcd是一个分布式键值存储，旨在可靠，快速地保存和提供对关键数据的访问。它通过分布式锁定，领导者选举和写入障碍实现可靠的分布式协调。etcd集群旨在实现高可用性和永久数据存储和检索。

# 启动

可以使用`goreman`来在本地启动多个节点

<!--more-->


[goreman](https://github.com/mattn/goremam) 是一个go写的多进程管理，是foreman的go版本，运行如下代码进行下载
>go get github.com/mattn/goreman

启动goreman
> goreman start

启动的时候需要提供一个Procfile文件可以使用-f参数，默认是使用当前目录下的Procfile文件

我们来创建一个Procfile文件，然后写入如下内容

     # Use goreman to run `go get github.com/mattn/goreman`
    etcd1: etcd --name infra1 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
    etcd2: etcd --name infra2 --listen-client-urls http://127.0.0.1:22379 --advertise-client-urls http://127.0.0.1:22379 --listen-peer-urls http://127.0.0.1:22380 --initial-advertise-peer-urls http://127.0.0.1:22380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
    etcd3: etcd --name infra3 --listen-client-urls http://127.0.0.1:32379 --advertise-client-urls http://127.0.0.1:32379 --listen-peer-urls http://127.0.0.1:32380 --initial-advertise-peer-urls http://127.0.0.1:32380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof
    #proxy: etcd grpc-proxy start --endpoints=127.0.0.1:2379,127.0.0.1:22379,127.0.0.1:32379 --listen-addr=127.0.0.1:23790 --advertise-client-url=127.0.0.1:23790 --enable-pprof

然后保存在启动goreman

上面的内容我们是定义了3个etcd的节点

    etcd1: etcd --name infra1 --listen-client-urls http://127.0.0.1:2379 --advertise-client-urls http://127.0.0.1:2379 --listen-peer-urls http://127.0.0.1:12380 --initial-advertise-peer-urls http://127.0.0.1:12380 --initial-cluster-token etcd-cluster-1 --initial-cluster 'infra1=http://127.0.0.1:12380,infra2=http://127.0.0.1:22380,infra3=http://127.0.0.1:32380' --initial-cluster-state new --enable-pprof

我们来看一下这些参数
- name  节点的名称
- listen-client-urls 指定etcd服务器绑定的本地地址以接受传入连接。要侦听所有接口的端口，请指定0.0.0.0侦听IP地址。
- advertise-client-urls 建议用于客户端通信的url列表，该值用于etcd代理或etcd成员与节点通信,不可以使用localhost这种，因为这种地址无法从远程计算机进行访问
- listen-peer-urls 指定etcd服务器绑定的本地地址以接受传入连接。要侦听所有接口的端口，请指定0.0.0.0侦听IP地址。
- initial-advertise-peer-urls 建议用于节点之间通信的url列表，不可以使用localhost这种，因为这种地址无法从远程计算机进行访问
- initial-cluster-token 集群的token值，用来区分不同的集群
- initial-cluster 集群中所有initial-advertise-peer-urls的合集
- initial-cluster-state 集群状态 new是新建

# 操作
## 使用etcdctl工具进行操作

etcdctl默认是v2,我们先调整到v3
> export ETCDCTL_API=3

### 查看版本
etcdctl version
### 写入
写入使用put指令，put后面跟键值对
> etcdctl put foo bar
### 读取
使用get指令读取，get后面跟要读取的键
> etcdctl get foo

如果想读取十六进制，使用--hex参数
> etcdctl get foo --hex

如果想只读取值，使用--print-value-only参数
> etcdctl get foo --print-value-only

如果想根据前缀获取，使用--prefix
> etcdctl get foo --prefix

还可以使用limit参数限制数量
> etcdctl get foo --prefix --liimit=2

#### 读取之前版本的记录

可以使用--rev参数来获取之前操作时候的值,比如

    etcdctl put foo bar
    etcdctl put foo bar1
    etcdctl put foo bar2

这个时候我们读取，给我们的值肯定是bar2
> etcdctl get foo

我们可以通过--rev=3参数来获取之前的bar1值，--rev=2来获取bar值
> etcdctl get foo --rev=3

#### 读取大于等于给定键的键值对
可以使用参数 --from-key来获取大于等于给定键的键值，这个参数是使用键的字节进行比较
> etcdctl get foo --from-key
### 删除
使用del指令删除键值对
> etcdctl del foo

可以使用上面提到的--prefix来删除所有这个前缀的键值，还有--from-key参数也可以使用。这里面有一个新参数--prev-kv，这个参数返回删除的键值
> etcdctl del foo --prev-kv

### 观察者
使用watch指令可以观测指定键的操作，注意这个应该在另外一个终端中进行，而且同样要改变成v3版本，不然默认v2是监测不到改变的
> etcdctl watch foo

可以是使用-i参数进入交互模式
> etcdctl watch -i
然后输入你要监测的键
> watch foo

#### 观察历史记录
这个操作也可以使用上面的--rev参数，它会把之前的历史操作都显示出来，也会继续观察之后的操作
> etcdctl watch foo --rev=1

# 关于通信
etcd本身使用grpc作为其消息传递协议，也就是我们在项目中使用的时候，使用grpc进行和etcd的通信。对于没有grpc支持的语言，或者你不想使用grpc的话，etcd也提供了json grpc网关。此网关提供restful代理，将http/json请求转换为grpc消息。
详细使用方法，直接去github上面看吧
> https://github.com/etcd-io/etcd/blob/master/Documentation/dev-guide/api_grpc_gateway.md

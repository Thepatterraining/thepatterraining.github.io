---
title: 网络原理自顶向下三可靠数据传输原理实现GBN滑动窗口协议
date: 2020-12-12 15:28:49
tags: ['计算机网络']
category: 计算机网络
article: 网络原理自顶向下三可靠数据传输原理实现GBN滑动窗口协议
---

# 网络原理自顶向下三可靠数据传输原理实现GBN滑动窗口协议

在GBN中，允许发送方发送多个分组而不需等待确认，但它也受限于在流水线中未确认的分组数不能超过某个最大允许数N。

我们需要
- base 基序号 最早未确认分组的序号
- nextSeqNum下一个序号 最小的未使用序号

则可以将序号范围分割成四段
- 0 至 base - 1 已经发送并被确认的分组
- base 至 nextSeqNum - 1 已经发送没有被确认的分组
- nextSeqNum 至 base + N - 1 可以被发送的分组序号
- base + N 至 无穷大  不能使用的序号

已经被发送的可以但未被确认的可以看成是一个在序号范围内长度为N的窗口。随着协议允许，窗口往前滑动。因此N常被成为`窗口长度`，GBN协议也被称为`滑动窗口协议`

#### 发送端

GBN的发送方必须响应三种类型的事件
- 上层的调用。检测有没有可以使用的序号，如果有就发送。
- 收到ACK。GBN使用`累计确认`。即收到的N之前的全部确认。
- 超时事件。如果出现超时，就重传所有已发送未确认的分组。


``` php
//序号
$base = 1;
$nextSeqNum = 1;
function rdt_send($data) {
    //生成校验和
    $checkSum = generateCheckSum($data);
    //组装报文
    $packet[$nextSeqNum] = make_pkt($data, $checkSum, $nextSeqNum);
    //调用网络层传输 
    udt_send($packet[$nextSeqNum]);
    //只启动一个定时器
    if ($base == $nextSeqNum) {
        start_timer();
    }
    //如果超时
    if (timeout()) {
        //重传所有已发送的分组
        for ($i = $base; $i <= $nextSeqNum - 1; $i++) {
            udt_send($packet[$i]);
        }
    } else if ($isAck = rdt_rev() && check($isAck)) {
        //等待接收方回传ack 并且没有出现错误
        //获取确认的序号
        $ackNum = getAckNum($isAck);
        //确认这个序号之前所有的分组
        $base = $ackNum;
        //如果确认了最新的分组，那么停止定时器
        if ($base == $nextSeqNum) {
            stop_timer();
        } else {
            start_timer();
        }
    }
}
```

#### 接收端

GBN的接收端较为简单，因为接收端只需要按序号交付数据就可以了。如果数据没有按序到达接收端，接收端只需要直接丢弃，因为发送端会重传所有分组。

``` php
//期望的序号
$expackNum = 1;
function rdt_rev($packet) {
    //差错检测通过了并且报文序号正确
    if (check($packet) && $packet['num'] == $expackNum) {
        //解析报文
        $data = extract($packet);
        //序号对的
        //没有错，把数据交付给应用层并回传ack
        //把数据给应用层
        deliver_data($data);
        //回传ACK
        $ack = make_pkt(1, $expackNum);
        $expackNum++;
        udt_send($ack);
    } else {
        //没通过差错检测或者序号错误，我们回传一个上一个ack，告诉发送端上一个分组我们收到了，当前分组没收到。
        //回传ACK
        $ack = make_pkt(1, $expackNum);
        udt_send($ack);
    }
}
```


### SR

在SR中，和GBN不同，SR是给每一个分组设置定时器，发送端只确认重传当前分组，而不是所有分组。接收端在接收到乱序的分组的时候会进行缓存，当前面的分组到达以后一起提交给应用层。

#### 发送端

- 等待上层调用。这里和GBN一样
- 超时。超时哪个重传哪个
- 收到ACK。如果收到的是最小序号的ACK，那么base可以往前移动，也就是窗口滑动。如果收到其他序号的ACK。那么把这些ACK缓存。

``` php
//序号
$base = 1;
$nextSeqNum = 1;
function rdt_send($data) {
    //生成校验和
    $checkSum = generateCheckSum($data);
    //组装报文
    $packet[$nextSeqNum] = make_pkt($data, $checkSum, $nextSeqNum);
    //调用网络层传输 
    udt_send($packet[$nextSeqNum]);
    //每个启动一个定时器
    start_timer($nextSeqNum);    
    //如果超时
    if ($timeNum = timeout()) {
        //重传超时的分组
        udt_send($packet[$timeNum]);
    } else if ($isAck = rdt_rev() && check($isAck)) {
        //等待接收方回传ack 并且没有出现错误
        //获取确认的序号
        $ackNum = getAckNum($isAck);
        stop_timer($ackNum);
        //判断这个ACK是不是base
        if ($ackNum == $base) {
            ++$base;
            //判断缓存有没有
            while(array_key_exists(++$ackNum, $cache)) {
                //如果下一个ack已经收到了，那么就把base接着往前移动
                ++$base;
            }
        } else {
            //缓存起来
            $cache[$ackNum] = $ackNum;
        }
    }
}
```

#### 接收端

- 序号在rcv_base 至 rcv_base + N - 1内的分组被正确接受。如果该分组不是期望的分组，那么缓存，如果是，那么给应用层并且看缓存里面有没有后续，有就直接一起给应用层
- 序号在rcv_base - N 至 rcv_base - 1内的分组被正确接受。返回一个确认ACK。表示我已经收到了。
- 其他情况。忽略分组

``` php
//期望的序号
$expackNum = 1;
function rdt_rev($packet) {
    //差错检测通过了并且报文序号正确
    if (check($packet)) {
        if ($packet['num'] > $expackNum && $packet['num'] < $expackNum + N - 1) {
            if ($packet['num'] == $expackNum) {
                //是我们期望的，直接给应用层
                //解析报文
                $data = extract($packet);
                //序号对的
                //没有错，把数据交付给应用层并回传ack
                //把数据给应用层
                deliver_data($data);
                //回传ACK
                $ack = make_pkt(1, $expackNum);
                $expackNum++;
                udt_send($ack);
                //查询缓存里面有没有
                $key = $expackNum + 1;
                while(array_key_exists($key, $cache)) {
                    //如果下一个分组已经收到了，那么给应用层，并且滑动窗口
                    deliver_data($cache[$expackNum]);
                    ++$expackNum;
                    $key++;
                }   
            } else {
                //缓存分组
                $cache[$packet['num']] = $packet;
                //回传ACK
                $ack = make_pkt(1, $packet['num']);
                udt_send($ack);
            }
        }
    } else {
        //没通过差错检测或者序号错误，我们回传一个上一个ack，告诉发送端上一个分组我们收到了，当前分组没收到。
        //回传ACK
        $ack = make_pkt(1, $expackNum);
        udt_send($ack);
    }
}
```

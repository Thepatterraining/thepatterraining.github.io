---
title: 网络原理自顶向下三可靠数据传输原理实现SR选择重传协议
date: 2020-12-12 15:28:49
tags: ['计算机网络']
category: 计算机网络
article: 网络原理自顶向下三可靠数据传输原理实现SR选择重传协议
---

# 网络原理自顶向下三可靠数据传输原理实现SR选择重传协议

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

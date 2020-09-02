---
title: thinkphp-queue队列使用
date: 2020-08-25 10:12:47
tags: ['php','thinkphp','queue']
category: php
article: thinkphp-queue队列使用
---

# thinkphp-queue队列使用

在我们写程序的时候，经常会用到队列来完成一些操作，关于队列的介绍和使用场景，注意事项可以看我的这个文章[你不知道的队列使用技巧](https://blog.csdn.net/Thepatterraining/article/details/105344675)

## 在tp里面使用队列

### 安装

`tp`框架提供了一个扩展包，叫做`think-queue`。我们先来安装这个扩展包。

> composer require topthink/think-queue

### 配置消息队列

等待安装完成之后，我们需要进行配置，消息队列的消息存放在哪里，可以配置成redis。

配置在你的`config/queue.php`里面。

```php

'default'     => 'redis',  //默认是sync，改成redis

'connections' => [
        'redis'    => [
            'type'       => 'redis',
            'queue'      => 'default',
            'host'       => env('queue.host', '127.0.0.1'),
            'port'       => env('queue.port', 6379),
            'password'   => env('queue.password', ''),
            'select'     => 0,      // 使用哪一个 db，默认为 db0
            'timeout'    => 0,      // redis连接的超时时间
            'persistent' => false,  // 是否是长连接
        ],
    ],

```

### 创建消息

配置完成以后我们就可以开始使用了。

在我们的`controller`里面把一个消息推送到队列里面。这里我们定义一个队列名称叫做`message`,定义一个处理队列消息的消费者类`app\common\queue\consumer`。然后调用`Queue`门面的`push`方法，把消费者，队列名称，数据传入进去就可以了。这个时候就会把数据放到`message`这个队列里面。然后消费者取出数据进行处理。

```php
<?php
namespace app\api\controller;

use app\BaseController;
use think\facade\Queue;

class Test extends BaseController
{

    protected $consumer = 'app\common\queue\consumer';  //消费者类

    protected $queue = 'message'; //队列名称

    public function test() {
        if ($this->request->isPost()) {
            //要推送到队列里面的数据
            $jobData = [];
            $jobData["a"] = 'a';
            $jobData['b'] = 'b';

            $res = Queue::push($this->consumer, $data, $this->queue);
            return json([]);
        }
    }
}

```

### 消费消息

我们接下来实现我们上面定义的消费者。来处理我们的逻辑。

消费者`app\common\queue\consumer`类。

```php
 <?php
  namespace app\common\queue;

  use think\queue\Job;

  class consumer {
      /**
       * fire方法是消息队列默认调用的方法
       * @param Job            $job      当前的任务对象
       * @param array|mixed    $data     发布任务时自定义的数据
       */
      public function fire(Job $job,$data)
      {
          // 有些消息在到达消费者时,可能已经不再需要执行了
          $isJobStillNeedToBeDone = $this->check($data);
          if(!$isJobStillNeedToBeDone){
              $job->delete();  //删除任务
              return;
          }
        
          //执行任务
          $isJobDone = $this->doJob($data);
        
          if ($isJobDone) {
              // 如果任务执行成功， 记得删除任务
              $job->delete();
          }else{
              if ($job->attempts() > 3) {
                  //通过这个方法可以检查这个任务已经重试了几次了
  				  $job->delete();
              }
          }
      }
      
      /**
       * 有些消息在到达消费者时,可能已经不再需要执行了
       * @param array|mixed    $data     发布任务时自定义的数据
       * @return boolean                 任务执行的结果
       */
      private function check($data){
          return true;
      }

      /**
       * 根据消息中的数据进行实际的业务处理...
       */
      private function doJob($data) 
      {
          dump($data);
          return true;
      }
  }

```

这样我们就完成了代码的逻辑，也就是发布消息，消费消息。

接下来我们启动这个队列。

### 启动队列

启动队列有两种方式。

- work
- listen

`work`方式启动。这种方式是单进程运行。如果你更新了代码需要手动重启队列。

> php think queue:work --queue message  //我们刚刚定义的队列名称

`listen`方式启动。这种方式是master-worker模型。一个master主进程来监听，当请求进来了启动一个work子进程来运行上面的work方式启动。

> php think queue:listen --queue message

我更推荐listen方式来运行。这种方式更新代码后也不需要手动重启。





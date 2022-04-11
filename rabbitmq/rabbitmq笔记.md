# # rabbitmq笔记

## mq基本概念

#### mq优势

1. 应用解耦

2. 异步提速：生产者把消息发送给mq之后，消费者操作用异步的方式解决。生产生不需要花费时间等待消费者的操作。

3. 消峰填谷：
   
   消峰：秒杀活动中，用户把请求发送到mq，把消息在mq中存储，系统中慢慢的在mq中获取消息。
   
   填谷：秒杀活动过后，系统慢慢获取mq中请求的过程称为填谷.

#### mq缺点

1. 系统可用性降低

2. 系统复杂度提高

3. 一致性问题

## RabbitMQ概念

#### 简介

mq使用AMQP协议，一种网络协议，应用层。为消息中间件设计。不受不同客户端、不同中间件产品、不同开发语言的限制。

![](C:\Users\zjw\AppData\Roaming\marktext\images\2022-04-11-14-23-26-image.png)

生产者、消费者通过connection与mq server建立tcp连接。connection中有多个channel，用于与不同的内容建立不同的连接，但是多个channel公用同一个tcp。一个server中包括多个virtual host(虚拟机)。一个虚拟机中有多个exchange用于分发消息，queue用于存储消息。生产者将消息发给exchange，消费者从对应的queue中获取消息。

#### 安装

1. 安装rabbitmq

```shell
sudo apt-get install rabbitmq-server
```

2. 下载管理工具

```shell
sudo rabbitmq-plugins enable rabbitmq_management
```

#### 添加用户

##### 命令行方式

1. 打开管理工具
   
   ```shell
   wget localhost:15672/cli/rabbitmqadmin
   ```

2. 添加用户(账号：test ,密码：test)
   
   ```shell
   sudo rabbitmqctl add_user test test
   ```

3. 给test用户添加访问所有exchange的权限
   
   ```shell
   sudo rabbitmqctl set_permissions -p / test ".*" ".*" ".*"
   ```

##### 用户界面方式

1. 打开/usr/lib/rabbitmq/lib/rabbitmq_server-x.x.x/ebin下的rabbit.app(x.x.x为自己的版本号)
   
   ```shell
   vi /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.2/ebin/rabbit.app
   ```

2. 修改
   
   ```shell
   {loopback_users, [<<"guest">>]},
   ```

        为

```
{loopback_users, [guest]},
```

      []内为可以登陆的用户，添加用户后建议删除guest用户，使用自己创建的用户登陆

    3. 在本地浏览器打开      http://ip地址:15672     ，进行修改

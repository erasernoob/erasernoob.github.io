---
title: 'MQ消息队列（未完结）'
date: 2024-12-06T11:36:00+08:00
draft: false
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: ""     
categories: ["笔记"]
tags: ["中间件","后端"]
---
# Message Queue

### 同步调用

- 时效性强，等待到全部结果之后再返回。
- 拓展性差、性能下降、**级联失败**

### 异步调用

异步调用通常基于消息通知的方式，包含三个角色

- 消息接收者
- 消息发送者
- 消息代理:管理、暂存、转发消息

#### 优劣

- 偶合度低，拓展性强
- 无序等待，性能好
- 故障隔离
- 缓存消息、流量削峰填谷
- 不能立即得到调用结果，时效性差
- 业务安全依赖于**broker**的可靠性

### 架构

- `exchange`  交换机
- `queue` 消息队列
- `publisher`
- `consumer`

### 数据隔离

使用虚拟主机（virtual host）以及用户的不同，实现数据隔离。

### Work Queues

多个消费者都被绑定到同一个队列中。默认的会将消息依次的轮询投递给每一个消费者。  

- 同一条消息只会被一个消费者处理
- 可以通过设置`prefetch`来控制消费者预取的消息数量，只有完成以后，才能继续取下一个消息。实现**能者多劳**。

### Fanout交换机（广播）

 将收到的消息，全部广播给与其关联的队列。

### Direct交换机

校验消息所带`routeKey`与每一个与交换机相连的队列的特定的`bindingKey`进行对照对比。

### Topic交换机

同样的基于RoutingKey做消息路由，但是routingKey时多个单词的集合，以`.`分割

指定`BindingKey`时使用通配符

- #：代指0或多个单词
- *：仅代指一个单词

### 声明队列和交换机

`Exchange` `Queue` `Binding`

基于`Bean`的声明方式：再配置类中添加新的生成`Bean`

- `Binding`类作为绑定队列和交换机的绑定关系。

基于注解的声明方式

```java
@RabbitListener(bindings = @QeueBinding(
	value = @Queue(name = "direct.queue1", durable="true"),
    exchange = @Exchange(name= "hmall.driect", type = ExchangeTypes.DIRECT),
    key = {"red", "blue"}
))
public void listenerDirectQueue(String msg) {
	System.out.println("接收到Direct消息：" + msg);
}
```

 





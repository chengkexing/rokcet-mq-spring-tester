1、首先构建 rocket-comment
maven install 将 rocket-comment 安装到本地, 之后刷新依赖即可启动其他项目！

2、测试结果

## 1、普通模式 normal-message:

普通消息为 Apache RocketMQ 中最基础的消息，区别于有特性的顺序消息、定时/延时消息和事务消息。本文为您介绍普通消息的应用场景、功能原理、使用方法和使用建议。

    典型场景一：微服务异步解耦

    以在线的电商交易场景为例，上游订单系统将用户下单支付这一业务事件封装成独立的普通消息并发送至Apache
    RocketMQ服务端，下游按需从服务端订阅消息并按照本地消费逻辑处理下游任务。每个消息之间都是相互独立的，且不需要产生关联。

    典型场景二：数据集成传输

    以离线的日志收集场景为例，通过埋点组件收集前端应用的相关操作日志，并转发到 Apache RocketMQ 。每条消息都是一段日志数据，Apache
    RocketMQ 不做任何处理，只需要将日志数据可靠投递到下游的存储系统和分析系统即可，后续功能由后端应用完成。

    使用限制

    普通消息仅支持使用MessageType为Normal主题，即普通消息只能发送至类型为普通消息的主题中，发送的消息的类型必须和主题的类型一致。

1. 当发送信息到topic 不指定tags 时, 所有的Group 都会消费达到消息分裂效果。
2. Group 必须是唯一的 Tags 可以不是唯一的，当一个Tags 对应多个不同的Group 时，这个Tag 会被每一个Group 消费
3. 当消息消费失败默认会进行重试操作（默认为消息间隔）再次失败后将会跳过该条，继续消费下一条，但在下次间隔期到来继续消费一次。

重试次数/间隔  
1 10秒  
2 30秒  
3 1分钟  
4 2分钟  
5 3分钟  
6 4分钟  
7 5分钟  
8 6分钟  
9 7分钟  
10 8分钟  
11 9分钟  
12 10分钟  
13 20分钟  
14 30分钟  
15 1小时  
16 2小时  
若重试次数超过16次，后面每次重试间隔都为2小时。

delayLevelWhenNextConsume 属性设置后 重试间隔修改为 (n)s  
maxReconsumeTimes 属性设置后 修改重试次数 (n)

```java
// 如下为 重复三次 每次之间间隔 5min 三次之后进入死信队列
@RocketMQMessageListener(
		topic = Constant.TOPIC,
		selectorExpression = Constant.Tags.ERROR_TEST_TAG,
		consumerGroup = Constant.Group.ERROR_TEST_GROUP_B,
		delayLevelWhenNextConsume = 5,
		maxReconsumeTimes = 3
)
```


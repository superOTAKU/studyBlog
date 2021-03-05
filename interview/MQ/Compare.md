# RabbitMQ And Kafka

了解两个中间件的区别，以便在不同的场景下选择不同的解决方案。

## RabbitMQ

典型的消息中间件，它的特性主要有：
+ 支持消息队列和发布/订阅模式
+ 良好的消息路由支持（根据路由规则，将Producer的消息路由到不同的队列）
+ 容错，消费失败后消息重回消息队列
+ 消息被消费后，就从队列中删除
+ 无序性，不能保证消费者顺序消费消息

## Kafka

Topic: 主题 

Partition: 分区

主题下可有多个分区，每个分区的消息顺序存储，分区可以分布在不同的节点。

特性：
+ 消息持久性，分区下的消息消费后不会删除
+ 顺序消费，合理的规划分区，每个分区一个消费者，即可保证消息消费的顺序性
+ 可靠性，Kafka的客户端需要自己维护partition index，包括错误重试，持久化partition指针等，都需要客户端自己去实现

## 总结

RabbitMQ适合用于投递消息命令，对消息顺序不敏感，不需要重复消费消息的场景。

Kafka适合于高并发，顺序处理消息，可重复消费消息的场景，如流式日志处理。

Tips：纯扯淡，没有太多应用经验。

## 参考

https://www.cloudamqp.com/blog/2019-12-12-when-to-use-rabbitmq-or-apache-kafka.html

https://zhuanlan.zhihu.com/p/161224418

https://www.cnblogs.com/haolujun/p/9632835.html

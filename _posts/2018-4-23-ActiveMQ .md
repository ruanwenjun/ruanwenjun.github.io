---
layout: post
title: ActiveMQ入门学习!
---

目录：
- [安装](#安装)
- [使用队列传输消息](#使用队列传输消息)
- [使用队列接收消息](#使用队列接收消息)
- [使用广播传输消息](#使用广播传输消息)
- [使用广播接收消息](#使用广播接收消息)



ActiveMQ是Apache出品的一个消息队列系统，完全采用Java实现的，很好的支持了JMS规范。

# 安装
1. 将安装包下载到Linux上
2. 解压
3. 进去解压后的文件夹，bin目录下有一个`activemq`.

使用
- ./activemq start启动
- ./activemq status查看运行状态
- ./activemq stop停止

如果遇到用户名不是 localhost的情况，需要修改/etc/hosts文件，在里面加上一行192.168.25.128 localhost，即把自己的IP映射为localhost

启动完毕后，在浏览器输入：192.168.25.128:8161/admin即可出现页面
（默认端口是8161）

# 使用队列传输消息
```java
// 测试队列发送消息
@Test
public void testProducer() throws JMSException {
	// 1.创建连接工厂
	ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.128:61616");
	// 2.创建连接
	Connection connection = connectionFactory.createConnection();
	// 3.开启连接
	connection.start();
	// 4.创建session对象
	Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	// 5.创建消息队列
	Queue queue = session.createQueue("测试消息队列");
	// 6.创建消息的生产者
	MessageProducer producer = session.createProducer(queue);
	// 7.创建字符串消息
	TextMessage message = session.createTextMessage("测试 ActiveMQ,使用消息队列");
	// 8.使用生产者发送消息
	producer.send(message);
	// 9.关闭资源
	producer.close();
	session.close();
	connection.close();
}
```
然后网站上消息队列中出现一条消息，表示成功了。

但是由于此版本的activemq依赖的是jdk1.7,使用jdk1.8会出现看不了消息的异常，但是无伤大雅。

解决方案：使用高版本的activemq或者使用jdk1.7

# 使用队列接收消息
```java
// 测试队列接收消息
@Test
public void testConsumer() throws JMSException, IOException {
	ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.128:61616");
	// 1.创建连接
	Connection connection = connectionFactory.createConnection();
	// 2.开启连接
	connection.start();
	// 3.创建session对象
	Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	// 4.创建消息接收者
	Queue queue = session.createQueue("测试消息队列");
	// 5.创建接收者
	MessageConsumer consumer = session.createConsumer(queue);
	// 6.设置监听
	consumer.setMessageListener(new MessageListener() {
		public void onMessage(Message message) {
			TextMessage tm = (TextMessage) message;
			// 打印消息
			try {
				System.out.println(tm.getText());
			} catch (JMSException e) {
				e.printStackTrace();
			}
		}
	});

	// 等待控制台输入
	System.in.read();
	// 7.关闭资源
	consumer.close();
	session.close();
	connection.close();

}
```
控制台打印：测试 ActiveMQ,使用消息队列

说明消息接收成功，并且网站后台发现消息没有了，说明使用队列传输消息在消息被接收时，消息将会从队列中消失。

# 使用广播传输消息
```java
// 测试广播发送消息
@Test
public void testTopicProducer() throws JMSException {
	// 1.创建连接工厂
	ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.128:61616");
	// 2.创建连接
	Connection connection = connectionFactory.createConnection();
	// 3.开启连接
	connection.start();
	// 4.创建session对象
	Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	// 5.创建消息队列
	Topic topic = session.createTopic("测试广播传输消息");
	// 6.创建消息的生产者
	MessageProducer producer = session.createProducer(topic);
	// 7.创建字符串消息
	TextMessage message = session.createTextMessage("测试 ActiveMQ,使用广播");
	// 8.使用生产者发送消息
	producer.send(message);
	// 9.关闭资源
	producer.close();
	session.close();
	connection.close();
}
```

# 使用广播接收消息
```java
// 测试广播接收消息
@Test
public void testTopicConsumer() throws JMSException, IOException {
	ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.25.128:61616");
	// 1.创建连接
	Connection connection = connectionFactory.createConnection();
	// 2.开启连接
	connection.start();
	// 3.创建session对象
	Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	// 4.创建消息接收者
	Topic topic = session.createTopic("测试广播传输消息");
	// 5.创建接收者
	MessageConsumer consumer = session.createConsumer(topic);
	// 6.设置监听
	consumer.setMessageListener(new MessageListener() {
		public void onMessage(Message message) {
			TextMessage tm = (TextMessage) message;
			// 打印消息
			try {
				System.out.println(tm.getText());
			} catch (JMSException e) {
				e.printStackTrace();
			}
		}
	});

	// 等待控制台输入
	System.in.read();
	// 7.关闭资源
	consumer.close();
	session.close();
	connection.close();

}
```
值得注意的是，广播发送消息默认的是发送就没有了，并不会等待接收者接收，如果发送的时候没有接收者接收，那么也会消失，但是这是可以通过设置使消息变得持久。

---

对比队列和广播两种方式：

这两种方式都可以实现消息传输，不同在于，队列是一对一的传输，广播是一对多的传输，即队列方式中，一个消息只能传输到一个接收者，而广播方式中，一个消息可以有多个接收者。这有点像观察者模式。
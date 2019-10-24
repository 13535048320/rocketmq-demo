## rocketMQ安装
### 1. 安装
下载地址: http://mirror.bit.edu.cn/apache/rocketmq/
```
解压
    unzip rocketmq-all-4.4.0-source-release.zip

编译
    cd rocketmq-all-4.4.0/
    mvn -Prelease-all -DskipTests clean install -U
    cd distribution/target/apache-rocketmq
```

### 2. Broker Master 配置
```
vi conf/broker.conf

#broker名字，最好修改，以便区分,主从必须一致,否则不能从slave消费
brokerName=broker-a

#所属集群名字,注意多主多从集群名必须一致
brokerClusterName=cluster
#0 表示 Master， >0 表示 Slave
brokerId=0
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#主从监听端口
haListenPort=10912
#删除文件时间点，默认凌晨 0点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
#宿主机IP
brokerIP1=192.168.9.215
#slave可读
slaveReadEnable=true

#开启权限控制
aclEnable=true
```

### 3. Broker Slave 配置
```
vi conf/broker-slave.conf

#broker名字，最好修改，以便区分,主从必须一致,否则不能从slave消费
brokerName=broker-a

#所属集群名字,注意多主多从集群名必须一致
brokerClusterName=cluster
#0 表示 Master， >0 表示 Slave
brokerId=1
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10913
#主从监听端口
#haListenPort=10914
#Master宿主机IP和Ha端口
haMasterAddress=192.168.9.215:10912
#删除文件时间点，默认凌晨 0点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole= SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
#宿主机IP
brokerIP1=192.168.9.215
#slave可读
slaveReadEnable=true

#开启权限控制
aclEnable=true
```

### 4. plain_acl.yml配置
```
# 全局白名单
globalWhiteRemoteAddresses:
#- 172.30.20.*

#帐号设置
accounts:
- accessKey: test
  secretKey: abc123456
  #帐号级白名单
  whiteRemoteAddress:
  #管理权限
  admin: false
  #默认topic权限，该值默认为DENY(拒绝)
  defaultTopicPerm: DENY
  #默认消费组权限，该值默认为DENY(拒绝)，建议值为SUB。
  defaultGroupPerm: SUB
  #DENY拒绝;PUB拥有发送权限;SUB拥有订阅权限
  topicPerms:
  - TopicTestConcurrent=PUB|SUB
  #DENY拒绝;PUB拥有发送权限;SUB拥有订阅权限
  groupPerms:
  # the group should convert to retry topic
  - concurrent_producer=PUB|SUB

- accessKey: test2
  secretKey: abc123456
  whiteRemoteAddress:
  - 172.30.20.*
  # if it is admin, it could access all resources
  admin: true
```

### 5. 启动
```
启动nameserver
    nohup sh bin/mqnamesrv &

启动broker master
    nohup sh bin/mqbroker -c conf/broker.conf &
    
启动broker slave
    nohup sh bin/mqbroker -n localhost:9876 -c conf/broker-slave.conf &
```

### 6. 关闭
```
sh bin/mqshutdown broker

sh bin/mqshutdown namesrv
```

### 7. 查看状态
```
bin/mqadmin clusterList -n "127.0.0.1:9876"

-n 指定nameserver
```

### 8. 安装控制台
```
git clone https://github.com/apache/rocketmq-externals.git

cd rocketmq-externals/rocketmq-console/

修改配置
    vi src/main/resources/application.properties
    
    server.contextPath=
    #访问端口
    server.port=8080
    #spring.application.index=true
    spring.application.name=rocketmq-console
    spring.http.encoding.charset=UTF-8
    spring.http.encoding.enabled=true
    spring.http.encoding.force=true
    #logback配置文件路径
    logging.config=classpath:logback.xml
    #if this value is empty,use env value rocketmq.config.namesrvAddr  NAMESRV_ADDR | now, you can set it in ops page.default localhost:9876
    #Name Server地址，修改成你自己的服务地址
    rocketmq.config.namesrvAddr=localhost:9876
    #if you use rocketmq version < 3.5.8, rocketmq.config.isVIPChannel should be false.default true
    rocketmq.config.isVIPChannel=false
    #rocketmq-console's data path:dashboard/monitor
    rocketmq.config.dataPath=/tmp/rocketmq-console/data
    #set it false if you don't want use dashboard.default true
    rocketmq.config.enableDashBoardCollect=true

编译
    mvn clean package -Dmaven.test.skip=true

启动
    nohup java -jar target/rocketmq-console-ng-1.0.1.jar &
    #如果配置文件没有填写Name Server
    nohup java -jar target/rocketmq-console-ng-1.0.1.jar --server.port=8080 --rocketmq.config.namesrvAddr='localhost:9876' &

访问
    http://localhost:8080
```

### 9. 生产者示例
<b>Maven依赖</b>
```
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>4.5.1</version>
        </dependency>
```

```
public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {

        //声明并初始化一个producer
        //需要一个producer group名字作为构造方法的参数，这里为concurrent_producer
        DefaultMQProducer producer = new DefaultMQProducer("concurrent_producer");

        //设置NameServer地址,此处应改为实际NameServer地址，多个地址之间用；分隔
        //NameServer的地址必须有，但是也可以通过环境变量的方式设置，不一定非得写死在代码里
        producer.setNamesrvAddr("10.1.54.121:9876;10.1.54.122:9876");

        //调用start()方法启动一个producer实例
        producer.start();

        //发送10条消息到Topic为TopicTest，tag为TagA，消息内容为“Hello RocketMQ”拼接上i的值
        for (int i = 0; i < 10; i++) {
            try {
                Message msg = new Message("TopicTestConcurrent",// topic
                        "TagA",// tag
                        ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET)// body
                );

                //调用producer的send()方法发送消息
                //这里调用的是同步的方式，所以会有返回结果，同时默认发送的也是普通消息
                SendResult sendResult = producer.send(msg);

                //打印返回结果，可以看到消息发送的状态以及一些相关信息
                System.out.println(sendResult);
            } catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }

        //发送完消息之后，调用shutdown()方法关闭producer
        producer.shutdown();
    }
}
```

### 10. 消费者示例
```
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {

        //声明并初始化一个consumer
        //需要一个consumer group名字作为构造方法的参数，这里为concurrent_consumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("concurrent_consumer");

        //同样也要设置NameServer地址
        consumer.setNamesrvAddr("10.1.54.121:9876;10.1.54.122:9876");

        //这里设置的是一个consumer的消费策略
        //CONSUME_FROM_LAST_OFFSET 默认策略，从该队列最尾开始消费，即跳过历史消息
        //CONSUME_FROM_FIRST_OFFSET 从队列最开始开始消费，即历史消息（还储存在broker的）全部消费一遍
        //CONSUME_FROM_TIMESTAMP 从某个时间点开始消费，和setConsumeTimestamp()配合使用，默认是半个小时以前
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET);

        //设置consumer所订阅的Topic和Tag，*代表全部的Tag
        consumer.subscribe("TopicTestConcurrent", "*");

        //设置一个Listener，主要进行消息的逻辑处理
        //注意这里使用的是MessageListenerConcurrently这个接口
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
                                                            ConsumeConcurrentlyContext context) {

                System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);

                //返回消费状态
                //CONSUME_SUCCESS 消费成功
                //RECONSUME_LATER 消费失败，需要稍后重新消费
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        //调用start()方法启动consumer
        consumer.start();

        System.out.println("Consumer Started.");
    }
}

```


### 11. 修改nameserver端口(扩展)
```
创建conf/namesrv.properties

添加内容 listenPort=9876

启动
    nohup sh bin/mqnamesrv -c conf/namesrv.properties &
```

### 12. 注册成服务(扩展)
<b>nameserver</b>
```
vi /usr/lib/systemd/system/rocketmq-nameserver.service

    [Unit]
    Description=rocketMQ nameserver
    Requires=network.service
    
    [Service]
    Type=simple
    ExecStart=/bin/bash -c "/opt/rocketmq/rocketmq-all-4.4.0/distribution/target/apache-rocketmq/bin/mqnamesrv"
    ExecStop=/bin/bash -c "/opt/rocketmq/rocketmq-all-4.4.0/distribution/target/apache-rocketmq/bin/mqshutdown namesrv"
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
    
```

<b>broker</b>
```
vi /usr/lib/systemd/system/rocketmq-broker.service

    [Unit]
    Description=rocketMQ broker
    Requires=network.service
    
    [Service]
    Type=simple
    ExecStart=/bin/bash -c "/opt/rocketmq/rocketmq-all-4.4.0/distribution/target/apache-rocketmq/bin/mqbroker -c /opt/rocketmq/rocketmq-all-4.4.0/distribution/target/apache-rocketmq/conf/broker.conf"
    ExecStop=/bin/bash -c "/opt/rocketmq/rocketmq-all-4.4.0/distribution/target/apache-rocketmq/bin/mqshutdown broker"
    Restart=on-failure
    
    [Install]
    WantedBy=multi-user.target
```

<b>启动</b>
```
启动
    systemctl start rocketmq-nameserver
    systemctl start rocketmq-broker
    
自启
    systemctl enable rocketmq-nameserver
    systemctl enable rocketmq-broker
    
关闭
    systemctl stop rocketmq-nameserver
    systemctl stop rocketmq-broker
```

### 13. mqadmin 命令(扩展)
<b>创建Topic</b>
```
sh bin/mqadmin updateTopic –n localhost:9876 –c DefaultCluster –t topicExample

-n nameserver
-c 集群名称
-t topic名称
```

<b>删除Topic</b>
```
sh mqadmin deleteTopic –n localhost:9876 –c DefaultCluster –t topicExample
```

<b>创建（修订）订阅组</b>
```
sh mqadmin updateSubGroup –n localhost:9876 –c DefaultCluster –g groupExample -w 1

-n nameserver
-c 集群名称
-t topic名称
-g 组名
-w 发现消息堆积后，将Consumer的消费请求重定向到另外一台Slave机器
```

### 14. 问题
#### 14.1 Algorithm HmacSHA1 not available
```
vi /opt/rocketmq-4.5.1/bin/tools.sh
在${JAVA_HOME}/jre/lib/ext后加上ext文件夹的绝对路径
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn256m -XX:PermSize=128m -XX:MaxPermSize=128m"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${BASE_DIR}/lib:${JAVA_HOME}/jre/lib/ext:/usr/java/jdk1.8.0_65/jre/lib/ext"
```

#### 14.2 Cannot allocate memory
```
修改bin/ 下的服务启动脚本 runserver.sh 、runbroker.sh 中对于内存的限制，​改成如下示例：

JAVA_OPT="${JAVA_OPT} -server -Xms128m -Xmx128m -Xmn128m -XX:PermSize=128m -XX:MaxPermSize=128m"
```

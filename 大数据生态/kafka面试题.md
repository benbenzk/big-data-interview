**消息队列的作用和使用场景**

通过异步处理提高响应时间，削峰填谷：

场景：数据比较集中且实时要求不是太高，如果同步处理，假如业务高峰需要4台服务支撑，那么在业务高峰过了之后，就会出现资源闲置，如果引入消息队列的话，将数据放到消息队列后直接返回成功，提升了响应时间，真正的业务在消息队列后面消费处理，可能2台服务就能够支撑的住，而且流量更加均匀。

降低系统间的耦合度：

场景：数据不止一方依赖，可能多个系统都需要这份数据，如果由发送方直接调用，那么如果新增业务也依赖数据，就得修改发送方代码；使用消息队列的话，将发送方和接收方解耦，发送方将数据扔到消息队列，依赖方可根据需求消费处理。

**使用消息队列会带来哪些问题？**

系统复杂度提高，可用性降低，不仅需要考虑消息队列的可用性，还要考虑数据的一致性

**如何做的消息队列选型，为什么选择kafka？**

kafka和rocketmq吞吐量可达百万级，比activemq、rabbitmq要高一个数量级 kafka和rocketmq都是分布式架构，高可用 kafka和rocketmq都是毫秒级低延时，rocketmq甚至到微秒级  rocketmq不支持队列层面的广播，kafka的consumer group支持组间广播，组内负载均衡  kafka可能会产生消息重复，业务做好幂等

**kafka相关概念与消费模型**

broker：kafka集群中的一个节点

topic：主题是kafka的逻辑上的队列

partition：一个topic可以包含一个或多个partition，每个partition的消息数据都是单独存储的，offset也是单独维护的，partition内部消息有序 partition复制因子：一个topic的所有分区会分布到各个broker上，允许设置复制因子使分区可以在其他节点上留存备份，在主分区所在broker宕机时，可以作为新的主分区继续提供服务

consumer group：一个topic可以有消费者组消费消息，kafka为每个消费者组单独管理每个分区的消费偏移量offset，消费者组间是广播模式，对于一个消费者组内是负载均衡，即一条消息可以被多个消费者组消费，只能被一个消费者组内的其中一个消费者消费；消费者组内的每个成员负责一定数量的分区，当消费者组内的消费者发生变动时，会触发分区的重平衡

pull消费模型：消费者向负责分区主动拉取消息，分区的消费偏移量通过一个默认的topic来记录，也可以显示指定

pull模型的优点：消费速度可以由消费者自由控制 逐条消费或者批量消费可以由消费者自由控制 broker设计更简单

**kafka的消息存储**

kafka的消息存储在磁盘上，一个kafka topic分为一个或多个partition，每个partition单独存储自己的消息数据

partition将数据记录到.log文件中，为了避免文件过大影响查询效率，将文件分段处理

记录消息到.log文件中的同时，会记录消息offset和物理偏移地址的映射作为索引，提升查找性能；

这个索引并不是按消息的顺序依次记录的，而是每隔一定字节的数据记录一条索引，降低了索引文件的大小

kafka查找消息时，只需要根据文件名和offset进行二分查找，找到对应的日志分段后，查找.index文件找到物理偏移地址，然后查.log读取消息内容

**消费组与分区重平衡**

当有新的消费者加入到消费者组时，原本的分区就需要重新分配；比如一个topic有30个分区，原本只有两个消费者，每人负责15个分区，当新加入一个消费者时，并没有分区可以给他消费，只能是将30个分区重新分配。

每个消费者组都会有一个broker负责协调（称为group coordinator），各个消费者通过发送心跳的方式向组协调者同步状态，当有消费者一定时间没有给组协调者发送心跳或者有新的消费者加入到消费者组时，就会触发消费组的重平衡操作。

新加入消费者触发重平衡： 

1.新加入消费者向组协调者发送joinGroup请求，携带订阅的topic信息

2.此后组协调者收到组内其他消费者的心跳请求时，在响应中告诉消费者要重平衡

3.组内原有消费者会重新发送joinGroup请求到组协调者

4.组协调者根据发送joinGroup请求的先后选出消费者leader，将topic和分区信息响应给各个消费者

5.被选为leader的消费者将分区分配好

6.各消费者发送SyncGroup请求给组协调者请求新分配好的分区信息，其中消费者leader会携带分配好的分区信息

7.组协调者将各个消费者负责的分区信息响应给消费者，重平衡完成

消费者主动离开导致重平衡

1.消费者发送leaveGroup请求给组协调者

2.此后组协调者收到组内其他消费者的心跳请求时，在响应中告诉消费者要重平衡

3.消费者会重新发送joinGroup请求到组协调者

4.组协调者根据发送joinGroup请求的先后选出消费者leader，将topic和分区信息响应给各个消费者

5.被选为leader的消费者将分区分配好

6.各消费者发送SyncGroup请求给组协调者请求新分配好的分区信息，其中消费者leader会携带分配好的分区信息

7.组协调者将各个消费者负责的分区信息响应给消费者，重平衡完成

消费者失去心跳导致重平衡

1.消费者一定时间内没有发送心跳信息给组协调者

2.此后组协调者收到组内其他消费者的心跳请求时，在响应中告诉消费者要重平衡

3.消费者会重新发送joinGroup请求到组协调者

4.组协调者根据发送joinGroup请求的先后选出消费者leader，将topic和分区信息响应给各个消费者

5.被选为leader的消费者将分区分配好

6.各消费者发送SyncGroup请求给组协调者请求新分配好的分区信息，其中消费者leader会携带分配好的分区信息

7.组协调者将各个消费者负责的分区信息响应给消费者，重平衡完成



**kafka如何保证不丢失消息？**

- 复制因子：创建topic的时候指定复制因子大于1时，一个分区被分配到一个broker上，同时会在其他broker上维护一个分区副本；
- isr列表：分区及其副本分别为leader和follower，leader对外提供读写服务，follower会向leader发送同步请求，拉取最新的数据，如果follower和leader的消息差距保持在一定范围之内，那么这个follower在isr列表内；当分区leader所在broker宕机，会从isr列表中选举一个follower作为新的leader提供服务
- 通过kafka的acks参数可以控制消息的发送行为，acks可选的值有0、1、all；当设置为0时，生产者消息发送成功即为成功，不关心是否写入到磁盘及后续操作；当设置为1时，消息发送到分区leader后写入磁盘即为成功；当设置为all时，消息发送到分区leader并写入磁盘后，同步给isr列表中的所有分区副本后即为成功

**kafka高可用**

broker启动会尝试向zookeeper创建临时节点：/controller，第一个broker选举成功成为集群的controller，其余节点都会在/controller注册watcher监控controller状态；当controller挂掉，所有broker感知到，重新尝试选举controller

controller节点通过zookeeper监控各broker状态，如果由broker挂掉，controller负责从其负责的leader分区的isr列表中选举一个作为新的leader

kafka副本和leader选举

**kafka高性能原因**

零拷贝、利用操作系统页缓存、磁盘顺序写 kafka零拷贝原理 分区、分段、建立索引 生产者、消费者批处理

**Kafka中的ISR、AR又代表什么？ISR的伸缩又指什么**

ISR:In-Sync Replicas 副本同步队列

AR:Assigned Replicas 所有副本

ISR是由leader维护，follower从leader同步数据有一些延迟（包括延迟时间replica.lag.time.max.ms和延迟条数replica.lag.max.messages两个维度,  当前最新的版本0.10.x中只支持replica.lag.time.max.ms这个维度），任意一个超过阈值都会把follower剔除出ISR, 存入OSR（Outof-Sync Replicas）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

**Kafka中的HW、LEO、LSO、LW等分别代表什么？**

HW:High Watermark 高水位，取一个partition对应的ISR中最小的LEO作为HW，consumer最多只能消费到HW所在的位置上一条信息

LEO:LogEndOffset 当前日志文件中下一条待写信息的offset

HW/LEO这两个都是指最后一条的下一条的位置而不是指最后一条的位置

LSO:Last Stable Offset 对未完成的事务而言，LSO 的值等于事务中第一条消息的位置(firstUnstableOffset)，对已完成的事务而言，它的值同 HW 相同

LW:Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值

**Kafka中是怎么体现消息顺序性的？**

kafka每个partition中的消息在写入时都是有序的，消费时，每个partition只能被每一个group中的一个消费者消费，保证了消费时也是有序的。整个topic不保证有序。如果为了保证topic整个有序，那么将partition调整为1.

**Kafka中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？**

拦截器->序列化器->分区器

**Kafka生产者客户端的整体结构是什么样子的？**

**Kafka生产者客户端中使用了几个线程来处理？分别是什么？**

2个，主线程和Sender线程。主线程负责创建消息，然后通过分区器、序列化器、拦截器作用之后缓存到累加器RecordAccumulator中。Sender线程负责将RecordAccumulator中消息发送到kafka中.

**Kafka的旧版Scala的消费者客户端的设计有什么缺陷？**

**消费组中的消费者个数如果超过topic的分区，那么就会有消费者消费不到数据”这句话是否正确？如果不正确，那么有没有什么hack的手段？**

不正确，通过自定义分区分配策略，可以将一个consumer指定消费所有partition。

**消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?**

offset+1

**有哪些情形会造成重复消费？**

消费者消费后没有commit offset(程序崩溃/强行kill/消费耗时/自动提交偏移情况下unscrible)

**那些情景下会造成消息漏消费？**

消费者没有处理完消息 提交offset(自动提交偏移 未处理情况下程序异常结束)

**KafkaConsumer是非线程安全的，那么怎么样实现多线程消费？**

1.在每个线程中新建一个KafkaConsumer 2.单线程创建KafkaConsumer，多个处理线程处理消息（难点在于是否要考虑消息顺序性，offset的提交方式）

**简述消费者与消费组之间的关系**

消费者从属与消费组，消费偏移以消费组为单位。每个消费组可以独立消费主题的所有数据，同一消费组内消费者共同消费主题数据，每个分区只能被同一消费组内一个消费者消费。

**当你使用kafka-topics.sh创建（删除）了一个topic之后，Kafka背后会执行什么逻辑？**

创建:在zk上/brokers/topics/下节点 kafkabroker会监听节点变化创建主题 删除:调用脚本删除topic会在zk上将topic设置待删除标志，kafka后台有定时的线程会扫描所有需要删除的topic进行删除

**topic的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？**

可以

**topic的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？**

不可以

**创建topic时如何选择合适的分区数？**

根据集群的机器数量和需要的吞吐量来决定适合的分区数

**Kafka目前有那些内部topic，它们都有什么特征？各自的作用又是什么？**

consumer_offsets 以下划线开头，保存消费组的偏移

**优先副本是什么？它有什么特殊的作用？**

优先副本 会是默认的leader副本 发生leader变化时重选举会优先选择优先副本作为leader

**Kafka有哪几处地方有分区分配的概念？简述大致的过程及原理**

创建主题时 如果不手动指定分配方式 有两种分配方式 消费组内分配

**简述Kafka的日志目录结构**

每个partition一个文件夹，包含四类文件.index .log .timeindex leader-epoch-checkpoint .index .log .timeindex 三个文件成对出现 前缀为上一个segment的最后一个消息的偏移 log文件中保存了所有的消息 index文件中保存了稀疏的相对偏移的索引  timeindex保存的则是时间索引 leader-epoch-checkpoint中保存了每一任leader开始写入消息时的offset  会定时更新 follower被选为leader时会根据这个确定哪些消息可用

**Kafka中有那些索引文件？**

如上

**如果我指定了一个offset，Kafka怎么查找到对应的消息？**

1.通过文件名前缀数字x找到该绝对offset 对应消息所在文件 2.offset-x为在文件中的相对偏移 3.通过index文件中记录的索引找到最近的消息的位置 4.从最近位置开始逐条寻找

**如果我指定了一个timestamp，Kafka怎么查找到对应的消息？**

原理同上 但是时间的因为消息体中不带有时间戳 所以不精确

**聊一聊你对Kafka的Log Retention的理解**

kafka留存策略包括 删除和压缩两种 删除: 根据时间和大小两个方式进行删除 大小是整个partition日志文件的大小 超过的会从老到新依次删除  时间指日志文件中的最大时间戳而非文件的最后修改时间 压缩: 相同key的value只保存一个 压缩过的是clean 未压缩的dirty  压缩之后的偏移量不连续 未压缩时连续

**聊一聊你对Kafka的Log Compaction的理解**

**聊一聊你对Kafka底层存储的理解（页缓存、内核层、块层、设备层）**

**聊一聊Kafka的延时操作的原理**

**聊一聊Kafka控制器的作用**

**消费再均衡的原理是什么？（提示：消费者协调器和消费组协调器）**

**Kafka中的幂等是怎么实现的**

pid+序号实现，单个producer内幂等

**Kafka中的事务是怎么实现的（只要简历上不写精通Kafka一般不会问到，这个问题的答案太太太太长了...）**

**Kafka中有那些地方需要选举？这些地方的选举策略又有哪些？**

**失效副本是指什么？有那些应对措施？**

**多副本下，各个副本中的HW和LEO的演变过程**

**为什么Kafka不支持读写分离？**

**Kafka在可靠性方面做了哪些改进？（HW, LeaderEpoch）**

**Kafka中怎么实现死信队列和重试队列？**

**Kafka中的延迟队列怎么实现**

**Kafka中怎么做消息审计**

**Kafka中怎么做消息轨迹**



**Kafka有哪些指标需要着重关注？**

生产者关注MessagesInPerSec、BytesOutPerSec、BytesInPerSec 消费者关注消费延迟Lag

**怎么计算Lag(注意read_uncommitted和read_committed状态下的不同)**

参考如何监控kafka消费Lag情况

**Kafka的那些设计让它有如此高的性能？**

零拷贝，页缓存，顺序写

**Kafka有什么优缺点？**

**还用过什么同质类的其它产品，与Kafka相比有什么优缺点？**

**为什么选择Kafka?**

吞吐量高，大数据消息系统唯一选择。

**在使用Kafka的过程中遇到过什么困难？怎么解决的？**

**怎么样才能确保Kafka极大程度上的可靠性？**
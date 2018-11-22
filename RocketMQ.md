#### 应用解耦

以电商系统为例，用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障或者因为升级等原因暂时不可用，都会造成操作异常，影响用户体验

当转变成基于消息队列的方式后，系统可用性就高多了，比如物流系统因为发生故障，需要几分钟的时间来修复，在这几分钟的时间里，物流系统要处理的内容被缓存在消息队列里，用户的下单操作可以正常完成。当物流系统恢复后，补充处理存储在消息队列里的订单消息即可，终端用户感知不到物流系统发生过几分钟的故障

![](D:\note\image\应用解耦.jpg)



#### 流量削峰

如果没有缓冲机制，不可能承受住短时大流量的冲击。通过利用消息队列，把大量的请求暂存起来，分散到相对长的一段时间内处理，能大大提高系统的稳定性和用户体验

使用消息队列进行流量消费，很多时候不是因为能力不够，而是由于经济性的考量。比如有的业务系统，流量最高峰也不会超过一万`QPS`，而平时只有一千左右的`QPS`。这种情况下我们就可以用个普通性能的服务器（只支持一千左右的`QPS`就可以），然后加个消息队列作为高峰期的缓冲，无须花大笔资金部署能处理上万`QPS`的服务器



#### 消息分发

数据的产生方只需要把各种的数据写入一个消息队列即可，数据使用方根据各自需求订阅感兴趣的数据



#### 名词

+ Producer	生产者
+ Consumer 消费者
+ Broker 负责暂存、传输                                  实际存储消息的数据节点
+ NameServer 整个消息队列中的状态服务器   服务发现节点   
+ Topic  区分不同类型的消息
+ Message Queue 类似于分区，消息可以并行地向各个Message Queue发送
+ Tag 消息子类型  服务器端基于Tag进行过滤
+ Key 消息在业务层面的唯一标识码，主要用于通过命令行查询消息

![](D:\note\image\RocketMQ各角色.jpg)



**RocketMQ的存储模型**

RocketMQ的消息的存储是由ConsumeQueue和CommitLog配合来完成的，ConsumeQueue中只存储很少的数据，消息主体都是通过CommitLog来进行读写

+ CommitLog：消息主体以及元数据的存储主体，对CommitLog建立一个ConsumeQueue，每个ConsumeQueue对应一个MessageQueue，所以只要有Commit Log 在，Consume Queue即使数据丢失，仍然可以恢复出来
+ Consume Queue：一个消息的逻辑队列，存储了这个Queue在CommitLog中的起始offset，log大小和MessageTag的hashCode。每个Topic下的每个Queue都有一个对应的ConsumerQueue文件，例如Topic中有三个队列，每个队列中的消息索引都会有一个编号，编号从0开始，往上递增。并由此一个位点offset的概念



#### 分布式消息系统中，如何避免重复消费

造成消息重复的根本原因是：网络不可靠。只要通过网络交换数据，就无法避免这个问题。所以解决这个的办法就是绕过这个问题。那么问题就变成了：`如果消费端收到两条一样的消息，应该怎样处理？`

+ 消费端处理消息的业务逻辑保存幂等性
+ 保证每条消息 都有唯一编号且保证消息处理成功与去重表的日志同时出现

通过幂等性，不管来多少条重复消息，可以实现处理的结果都一样。再利用一张日志表来记录已经处理成功的消息的ID，如果新到的消息ID已经在日志表中，那么就可以不再处理这条消息，避免消息的重复处理



----



消费者分为两种类型。一个是DefaultMQPushConsumer，由系统控制读取操作，收到消息后自动调用传入的处理方法来处理；另一个是DefaultMQPullConsumer，读取操作中的大部分功能由使用者自主控制



Push方式是Server端接收到消息后，主动把消息推送给Client端，实时性高。对于一个提供队列消息服务的Server来说，用Push方式主动推送有很多弊端：首先是加大Server端的工作量，进而影响Server的性能；其次，Client的处理能力各不相同，Client的状态不受Server控制，如果Client不能即使处理Server推送过来的消息，会造成各种潜在的问题



Pull方式是Client端循环地从Server端拉取消息，主动权在Client手里，自己拉去到一定量消息后，处理妥当了再接着取。Pull方式的问题是循环拉取消息的间隔不好设定，间隔太短就处在一个“忙等”的状态，浪费资源；每个Pull的时间间隔太长，Server端有消息到来时，有可能没有被及时处理



“长轮询”方式通过Client端和Server端的配合，达到既拥有Pull的优点，又能达到保证实时性的功能

Broker最长阻塞时间，默认设置是15秒，注意是Broker在没有消息的时候才阻塞，有消息会立刻方法

服务端接到新消息请求后，如果队列里没有新消息，并不急于返回，通过一个循环不断查看状态，每次waitForRuning一段时间（默认是5秒），然后再Check。默认情况下当Broker一直没有新消息，，第上次Check的时候，等待时间超过Request里面的Broker-SuspendMaxTimeMills，就返回空结果。在等待的过程中，Broker收到了新的消息后会直接调用notifyMessageArriving函数返回请求结果。

“长轮询”的核心是，Broker端HOLD住客户端过来的请求一小段时间，在这个时间内有新消息到底，就利用现有的连接立刻返回消息给Consumer。“长轮询”的主动权还是掌握在Consumer手中，Broker即使有大量消息积压，也不会主动推送给Consumer。

长轮询方式的局限性，是在HOLD住Consumer请求的时候需要占用资源，它时候用在消息队列这种客户端连接数可控的场景中





#### DefaultMQPushConsumer

Consumer的GroupName用于把多个Consumer组织到一起，提高并发处理能力，GroupName需要和消息模式（MessageModel）配合使用

+ 在Clustering模式下，同一个ConsumerGroup（GroupName相同）里的每个Consumer只消费所订阅消息的一部分内容，同一个ConsumerGroup里所有的Consumer消费的内容合起来才是所订阅Topic内容的整体，从而达到负载均衡的目的
+ 在Broadcasting模式下，同一个ConsumerGroup里的每个Consumer都能消费到所订阅Topic的全部消息，也就是一个消息会被多次分发，被多个Consumer消费





#### 存储队列位置信息

RocketMQ中，一种类型的消息会放到一个Topic里，为了能够并行，一般一个Topic会有多个Message Queue（也可以设置成一个），Offset是指某个Topic下的一条消息在某个Message Queue里的位置，通过Offset的值可以定位到这条消息，或者指示Consumer从这条消息开始向后继续处理



![](D:\note\image\OffsetStore.jpg)

Offset主要分为本地文件类型和Broker代存的类型。对于DefaultMQPushConsumer来说，默认是CLUSTERING模式，也就是同一个Consumer group里的多个消费者每人消费一部分，各自收到的消息内容不一样。这种情况下，由Broker端存储和控制Offset的值，使用RemoteBrokerOffsetStore

在DefaultMQPushConsumer里的BROADCASTING模式下，每个Consumer都收到这个Topic的全部消息，各个Consumer间相互没有干扰，RocketMQ使用LocalFileOffsetStore，把Offset存到本地

在使用DefaultMQPushConsumer的时候，我们不用关心OffsetStore的事，但是如果是PullConsumer，要自己处理OffsetStore，推荐持久化存储



#### NameServer

NameServer是整个消息队列中的状态服务器，集群的各个组件通过它来了解全局的信息。同时，各个角色的机器都要定期向NameServer上报自己的状态，超时不上报的话，NameServer会认为某个机器出故障不可用了，其他的组件会把这个机器从可用列表里移除



#### 为什么不用ZooKeeper

ZooKeeper为分布式应用程序提供协调服务，ZooKeeper功能很强大，而RocketMQ用不到那些复杂的功能，只需要一个轻量级的元数据服务器就足够了



分布式系统各个角色间的通信效率很关键，通信效率的高低直接影响系统性能



NameServer在RocketMQ集群中扮演调度中心的角色。各个Producer、Consumer上报自己的状态上去，同时从NameServer获取其他角色的状态信息。NameServer的功能虽然非常重要，但是被设计得很轻量级，代码量少并且几乎无磁盘存储，所有的功能都通过内存高效完成



#### Broker

Broker是RocketMQ的核心，大部分“重量级”工作都是由Broker完成的，包括接收Producer发过来的消息、处理Consumer的消费消息请求、消息的持久化存储、消息的HA机制以及服务端过滤功能



####  同步刷盘和异步刷盘

+ 异步刷盘：在返回写成功状态时，消息可能只是被写入了内存的PAGECACHE，写操作的返回快，吞吐量大；当内存里的消息量积累到一定程度时，统一触发写磁盘动作，快速写入
+ 同步刷盘：在返回写成功状态时，消息已经被写入磁盘。具体流程是，消息写入内存的PAGECACHE后，立刻通知刷盘线程刷盘，然后等待刷盘完成，刷盘线程执行完成后唤醒等待的线程，返回消息写成功的状态



#### 消息重复问题

对分布式消息队列来说，同时做到确保一定投递和不重复投递是很难的，也就是所谓的“有且只有一次”。RocketMQ选择了却确保一定投递，保证消息不丢失，但有可能造成重复消费

消息重复一般情况下不会发送，但是如果消息量大，网络有波动，消息重复就是个大概率事件。比如Producer有个函数setRetryTimesWhenSendFailed，设置在同步方式下自动重试的次数，默认值是2，这样当第一次发送消息时，Broker端接收到了消息但是没有正确返回发送成功的状态，就造成了消息重复

解决消息重复有两种方法：第一种方法是保证消费逻辑的幂等性（多次调用和一次调用效果相同）；另一种方法是维护一个已消费的记录，消费前查询这个消息是否被消费过。者两种方法都需要使用者自己实现


# RabbitMQ



## 安装



### Docker安装

[Rabbitmq - Official Image | Docker Hub](https://hub.docker.com/_/rabbitmq)

![image-20220526191429317](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261914437.png)

```sh
拉取镜像:
docker pull rabbitmq:3.10.1-management
运行容器:
docker run -d --hostname my-rabbit --name rabbitmq -p 5672:5672 -p 15672:15672  rabbitmq镜像id(只需要填前几位，确保与其他镜像id即可识别)

hostname:容器内的主机名
通讯端口:5672
web界面端口:15672
记得开启5672和15672端口
```

![image-20220526191423111](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261914213.png)

默认用户名和密码为guest

***

## Rabbitmq初识

![image-20220526191418592](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261914719.png)

1.生产者(Publisher): 发布消息到RabbitMQ中的交换机(Exchange)上

2.交换机(Exchange): 和生产者建立连接并接受生产者投递的消息

3.消费者(Consumer): 监听RabbitMQ中的Queue中的消息

4.队列(Queue): Exchange将消息分发到指定的Queue

5.路由(Routes):交换机转换消息到队列的规则

***

## AMQP 0.9.1 协议解析

**详细参考官方文档**:

[AMQP 0-9-1 Model Explained — RabbitMQ](https://www.rabbitmq.com/tutorials/amqp-concepts.html)

### AMQP协议简介

AMQP（高级消息队列协议）是一个网络协议。它支持符合要求的客户端应用（application）和消息中间件代理（messaging middleware broker）之间进行通信。

### 消息代理和他们所扮演的角色

消息代理（message brokers）从发布者（publishers）亦称生产者（producers）那儿接收消息，并根据既定的路由规则把接收到的消息发送给处理消息的消费者（consumers）。

由于AMQP是一个网络协议，所以这个过程中的发布者，消费者，消息代理 可以存在于不同的设备上。



### AMQP 0-9-1 模型简介

AMQP 0-9-1的工作过程如下图：消息（message）被发布者（publisher）发送给交换机（exchange），交换机常常被比喻成邮局或者邮箱。然后交换机将收到的消息根据路由规则分发给绑定的队列（queue）。最后AMQP代理会将消息投递给订阅了此队列的消费者，或者消费者按照需求自行获取。

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)

发布者（publisher）发布消息时可以给消息指定各种消息属性（message meta-data）。有些属性有可能会被消息代理（brokers）使用，然而其他的属性则是完全不透明的，它们只能被接收消息的应用所使用。

从安全角度考虑，网络是不可靠的，接收消息的应用也有可能在处理消息的时候失败。基于此原因，AMQP模块包含了一个消息确认（message acknowledgements）的概念：当一个消息从队列中投递给消费者后（consumer），消费者会通知一下消息代理（broker），这个可以是自动的也可以由处理消息的应用的开发者执行。当“消息确认”被启用的时候，消息代理不会完全将消息从队列中删除，直到它收到来自消费者的确认回执（acknowledgement）。



在某些情况下，例如当一个消息无法被成功路由时，消息或许会被返回给发布者并被丢弃。或者，如果消息代理执行了延期操作，消息会被放入一个所谓的死信队列中。此时，消息发布者可以选择某些参数来处理这些特殊情况。

队列，交换机和绑定统称为AMQP实体（AMQP entities）。

****

### AMQP是一个可编程的协议

AMQP 0-9-1是一个可编程协议，某种意义上说AMQP的实体和路由规则是由应用本身定义的，而不是由消息代理定义。包括像声明队列和交换机，定义他们之间的绑定，订阅队列等等关于协议本身的操作。

这虽然能让开发人员自由发挥，但也需要他们注意潜在的定义冲突。当然这在实践中很少会发生，如果发生，会以配置错误（misconfiguration）的形式表现出来。

应用程序（Applications）声明AMQP实体，定义需要的路由方案，或者删除不再需要的AMQP实体。

****

### 交换机和交换机类型

交换机是用来发送消息的AMQP实体。交换机拿到一个消息之后将它路由给一个或零个队列。它使用哪种路由算法是由交换机类型和被称作绑定（bindings）的规则所决定的。AMQP 0-9-1的代理提供了四种交换机

| Name（交换机类型）            | Default pre-declared names（预声明的默认名称） |
| ----------------------------- | ---------------------------------------------- |
| Direct exchange（直连交换机） | (Empty string) and amq.direct                  |
| Fanout exchange（扇型交换机） | amq.fanout                                     |
| Topic exchange（主题交换机）  | amq.topic                                      |
| Headers exchange（头交换机）  | amq.match (and amq.headers in RabbitMQ)        |

除交换机类型外，在声明交换机时还可以附带许多其他的属性，其中最重要的几个分别是：

- Name
- Durability （消息代理重启后，交换机是否还存在）
- Auto-delete （当所有与之绑定的消息队列都完成了对此交换机的使用后，删掉它）
- Arguments（依赖代理本身）

交换机可以有两个状态：持久（durable）、暂存（transient）。持久化的交换机会在消息代理（broker）重启后依旧存在，而暂存的交换机则不会（它们需要在代理再次上线后重新被声明）。然而并不是所有的应用场景都需要持久化的交换机。



****

####  默认交换机

默认交换机（default exchange）实际上是一个由消息代理预先声明好的没有名字（名字为空字符串）的直连交换机（direct exchange）。它有一个特殊的属性使得它对于简单应用特别有用处：那就是每个新建队列（queue）都会自动绑定到默认交换机上，绑定的路由键（routing key）名称与队列名称相同。

For example：当你声明了一个名为"search-indexing-online"的队列，AMQP代理会自动将其绑定到默认交换机上，绑定（binding）的路由键名称也是为"search-indexing-online"。因此，当携带着名为"search-indexing-online"的路由键的消息被发送到默认交换机的时候，此消息会被默认交换机路由至名为"search-indexing-online"的队列中。换句话说，默认交换机看起来貌似能够直接将消息投递给队列，尽管技术上并没有做相关的操作。

****

#### 直连交换机

直连型交换机（direct exchange）是根据消息携带的路由键（routing key）将消息投递给对应队列的。直连交换机用来处理消息的**单播路由**（unicast routing）（尽管它**也可以处理多播路由**）。下边介绍它是如何工作的：

- 将一个队列绑定到某个交换机上，同时赋予该绑定一个路由键（routing key）
- 当一个携带着路由键为`R`的消息被发送给直连交换机时，交换机会把它路由给绑定值同样为`R`的队列。

直连交换机经常用来循环分发任务给多个工作者（workers）。当这样做的时候，我们需要明白一点，在AMQP 0-9-1中，消息的负载均衡是发生在消费者（consumer）之间的，而不是队列（queue）之间。

直连型交换机图例：

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/exchange-direct.png)

****

#### 扇型交换机

扇型交换机（funout exchange）将消息路由给绑定到它身上的所有队列，**而不理会绑定的路由键**。如果N个队列绑定到某个扇型交换机上，当有消息发送给此扇型交换机时，交换机会将消息的拷贝分别发送给这所有的N个队列。**扇型用来交换机处理消息的广播路由**（broadcast routing）。

因为扇型交换机投递消息的拷贝到所有绑定到它的队列，所以他的应用案例都极其相似：

- 大规模多用户在线（MMO）游戏可以使用它来处理排行榜更新等全局事件
- 体育新闻网站可以用它来近乎实时地将比分更新分发给移动客户端
- 分发系统使用它来广播各种状态和配置更新
- 在群聊的时候，它被用来分发消息给参与群聊的用户。（AMQP没有内置presence的概念，因此XMPP可能会是个更好的选择）

扇型交换机图例：

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/exchange-fanout.png)

****

#### 主题交换机

主题交换机（topic exchanges）通过对消息的路由键和**队列到交换机的绑定模式**之间的匹配，将消息路由给一个或多个队列。主题交换机经常用来实现各种分发/订阅模式及其变种。主题交换机通常用来实现消息的多播路由（multicast routing）。

主题交换机拥有非常广泛的用户案例。无论何时，当一个问题涉及到那些想要有针对性的选择需要接收消息的 多消费者/多应用（multiple consumers/applications） 的时候，主题交换机都可以被列入考虑范围。

使用案例：

- 分发有关于特定地理位置的数据，例如销售点
- 由多个工作者（workers）完成的后台任务，每个工作者负责处理某些特定的任务
- 股票价格更新（以及其他类型的金融数据更新）
- 涉及到分类或者标签的新闻更新（例如，针对特定的运动项目或者队伍）
- 云端的不同种类服务的协调
- 分布式架构/基于系统的软件封装，其中每个构建者仅能处理一个特定的架构或者系统。

**什么是绑定模式，后面我们讲到RabbitMQ对AMQP协议具体实现的时候会看到**



****

####  头交换机

有时消息的路由操作会涉及到多个属性，此时使用消息头就比用路由键更容易表达，头交换机（headers exchange）就是为此而生的。头交换机使用多个消息属性来代替路由键建立路由规则。通过判断消息头的值能否与指定的绑定相匹配来确立路由规则。

我们可以绑定一个队列到头交换机上，并给他们之间的绑定使用多个用于匹配的头（header）。这个案例中，消息代理得从应用开发者那儿取到更多一段信息，换句话说，它需要考虑某条消息（message）是需要部分匹配还是全部匹配。上边说的“更多一段消息”就是"x-match"参数。当"x-match"设置为“any”时，消息头的任意一个值被匹配就可以满足条件，而当"x-match"设置为“all”的时候，就需要消息头的所有值都匹配成功。

头交换机可以视为直连交换机的另一种表现形式。头交换机能够像直连交换机一样工作，不同之处在于头交换机的路由规则是建立在头属性值之上，而不是路由键。路由键必须是一个字符串，而头属性值则没有这个约束，它们甚至可以是整数或者哈希值（字典）等。

****

### 队列

AMQP中的队列（queue）跟其他消息队列或任务队列中的队列是很相似的：它们存储着即将被应用消费掉的消息。队列跟交换机共享某些属性，但是队列也有一些另外的属性。

- Name
- Durable（消息代理重启后，队列依旧存在）
- Exclusive（只被一个连接（connection）使用，而且当连接关闭后队列即被删除）
- Auto-delete（当最后一个消费者退订后即被删除）
- Arguments（一些消息代理用他来完成类似与TTL的某些额外功能）

队列在声明（declare）后才能被使用。如果一个队列尚不存在，声明一个队列会创建它。如果声明的队列已经存在，并且属性完全相同，那么此次声明不会对原有队列产生任何影响。如果声明中的属性与已存在队列的属性有差异，那么一个错误代码为406的通道级异常就会被抛出。



****



#### 队列名称

队列的名字可以由应用（application）来取，也可以让消息代理（broker）直接生成一个。队列的名字可以是最多255字节的一个utf-8字符串。**若希望AMQP消息代理生成队列名，需要给队列的name参数赋值一个空字符串**：在同一个通道（channel）的后续的方法（method）中，我们可以使用空字符串来表示之前生成的队列名称。之所以之后的方法可以获取正确的队列名是因为通道可以默默地记住消息代理最后一次生成的队列名称。

**以"amq."开始的队列名称被预留做消息代理内部使用。如果试图在队列声明时打破这一规则的话，一个通道级的403 (ACCESS_REFUSED)错误会被抛出。**



****



#### 队列持久化

持久化队列（Durable queues）会被存储在磁盘上，当消息代理（broker）重启的时候，它依旧存在。没有被持久化的队列称作暂存队列（Transient queues）。并不是所有的场景和案例都需要将队列持久化。

**持久化的队列并不会使得路由到它的消息也具有持久性**。倘若消息代理挂掉了，重新启动，那么在重启的过程中持久化队列会被重新声明，无论怎样，只有经过持久化的消息才能被重新恢复。

****



####  绑定

绑定（Binding）是交换机（exchange）将消息（message）路由给队列（queue）所需遵循的规则。如果要指示交换机“E”将消息路由给队列“Q”，那么“Q”就需要与“E”进行绑定。绑定操作需要定义一个可选的路由键（routing key）属性给某些类型的交换机。路由键的意义在于从发送给交换机的众多消息中选择出某些消息，将其路由给绑定的队列。

拥有了交换机这个中间层，很多由发布者直接到队列难以实现的路由方案能够得以实现，并且避免了应用开发者的许多重复劳动。

如果AMQP的消息无法路由到队列（例如，发送到的交换机没有绑定队列），消息会被就地销毁或者返还给发布者。如何处理取决于发布者设置的消息属性。

****

### 消费者

消息如果只是存储在队列里是没有任何用处的。被应用消费掉，消息的价值才能够体现。在AMQP 0-9-1 模型中，有两种途径可以达到此目的：

- 将消息投递给应用 ("push API")
- 应用根据需要主动获取消息 ("pull API")

使用push API，应用（application）需要明确表示出它在某个特定队列里所感兴趣的，想要消费的消息。如是，我们可以说应用注册了一个消费者，或者说订阅了一个队列。一个队列可以注册多个消费者，也可以注册一个独享的消费者（当独享消费者存在时，其他消费者即被排除在外）。

**每个消费者（订阅者）都有一个叫做消费者标签的标识符。它可以被用来退订消息。消费者标签实际上是一个字符串。**



****

###  消息确认

消费者应用（Consumer applications） - 用来接受和处理消息的应用 - 在处理消息的时候偶尔会失败或者有时会直接崩溃掉。而且网络原因也有可能引起各种问题。这就给我们出了个难题，AMQP代理在什么时候删除消息才是正确的？AMQP 0-9-1 规范给我们两种建议：

- 当消息代理（broker）将消息发送给应用后立即删除。（使用AMQP方法：basic.deliver或basic.get-ok）
- 待应用（application）发送一个确认回执（acknowledgement）后再删除消息。（使用AMQP方法：basic.ack）

前者被称作自动确认模式（automatic acknowledgement model），后者被称作显式确认模式（explicit acknowledgement model）。在显式模式下，由消费者应用来选择什么时候发送确认回执（acknowledgement）。应用可以在收到消息后立即发送，或将未处理的消息存储后发送，或等到消息被处理完毕后再发送确认回执。

如果一个消费者在尚未发送确认回执的情况下挂掉了，那AMQP代理会将消息重新投递给另一个消费者。如果当时没有可用的消费者了，消息代理会死等下一个注册到此队列的消费者，然后再次尝试投递。

****

###  拒绝消息

当一个消费者接收到某条消息后，处理过程有可能成功，有可能失败。应用可以向消息代理表明，本条消息由于“拒绝消息（Rejecting Messages）”的原因处理失败了（或者未能在此时完成）。当拒绝某条消息时，应用可以告诉消息代理如何处理这条消息——销毁它或者重新放入队列。当此队列只有一个消费者时，请确认不要由于拒绝消息并且选择了重新放入队列的行为而引起消息在同一个消费者身上无限循环的情况发生。



****



###  Negative Acknowledgements

在多个消费者共享一个队列的案例中，明确指定在收到下一个确认回执前每个消费者一次可以接受多少条消息是非常有用的。这可以在试图批量发布消息的时候起到简单的负载均衡和提高消息吞吐量的作用。

注意，RabbitMQ只支持通道级的预取计数，而不是连接级的或者基于大小的预取。

****

###  消息属性和有效载荷（消息主体）

AMQP模型中的消息（Message）对象是带有属性（Attributes）的。有些属性及其常见，以至于AMQP 0-9-1 明确的定义了它们，并且应用开发者们无需费心思思考这些属性名字所代表的具体含义。例如：

- Content type（内容类型）
- Content encoding（内容编码）
- Routing key（路由键）
- Delivery mode (persistent or not)
  投递模式（持久化 或 非持久化）
- Message priority（消息优先权）
- Message publishing timestamp（消息发布的时间戳）
- Expiration period（消息有效期）
- Publisher application id（发布应用的ID）

有些属性是被AMQP代理所使用的，但是大多数是开放给接收它们的应用解释器用的。有些属性是可选的也被称作消息头（headers）。他们跟HTTP协议的X-Headers很相似。消息属性需要在消息被发布的时候定义。

AMQP的消息除属性外，也含有一个有效载荷 - Payload（消息实际携带的数据），它被AMQP代理当作不透明的字节数组来对待。消息代理不会检查或者修改有效载荷。消息可以只包含属性而不携带有效载荷。它通常会使用类似JSON这种序列化的格式数据，为了节省，协议缓冲器和MessagePack将结构化数据序列化，以便以消息的有效载荷的形式发布。AMQP及其同行者们通常使用"content-type" 和 "content-encoding" 这两个字段来与消息沟通进行有效载荷的辨识工作，但这仅仅是基于约定而已。

消息能够以持久化的方式发布，AMQP代理会将此消息存储在磁盘上。如果服务器重启，系统会确认收到的持久化消息未丢失。简单地将消息发送给一个持久化的交换机或者路由给一个持久化的队列，并不会使得此消息具有持久化性质：它完全取决与消息本身的持久模式（persistence mode）。将消息以持久化方式发布时，会对性能造成一定的影响（就像数据库操作一样，健壮性的存在必定造成一些性能牺牲）。

****

### 消息确认

由于网络的不确定性和应用失败的可能性，处理确认回执（acknowledgement）就变的十分重要。有时我们确认消费者收到消息就可以了，有时确认回执意味着消息已被验证并且处理完毕，例如对某些数据已经验证完毕并且进行了数据存储或者索引操作。

这种情形很常见，所以 AMQP 0-9-1 内置了一个功能叫做 消息确认（message acknowledgements），消费者用它来确认消息已经被接收或者处理。如果一个应用崩溃掉（此时连接会断掉，所以AMQP代理亦会得知），而且消息的确认回执功能已经被开启，但是消息代理尚未获得确认回执，那么消息会被从新放入队列（并且在还有还有其他消费者存在于此队列的前提下，立即投递给另外一个消费者）。

协议内置的消息确认功能将帮助开发者建立强大的软件。

****

###  AMQP 0-9-1 方法

AMQP 0-9-1由许多方法（methods）构成。方法即是操作，这跟面向对象编程中的方法没半毛钱关系。AMQP的方法被分组在类（class）中。这里的类仅仅是对AMQP方法的逻辑分组而已。在 [AMQP 0-9-1参考](https://www.rabbitmq.com/amqp-0-9-1-reference.html) 中有对AMQP方法的详细介绍。

让我们来看看交换机类，有一组方法被关联到了交换机的操作上。这些方法如下所示：

- exchange.declare
- exchange.declare-ok
- exchange.delete
- exchange.delete-ok

以上的操作来自逻辑上的配对：exchange.declare 和 exchange.declare-ok，exchange.delete 和 exchange.delete-ok. 这些操作分为“请求 - requests”（由客户端发送）和“响应 - responses”（由代理发送，用来回应之前提到的“请求”操作）。

如下的例子：客户端要求消息代理使用exchange.declare方法声明一个新的交换机：



![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/exchange-declare.png)

如上图所示，exchange.declare方法携带了好几个参数。这些参数可以允许客户端指定交换机名称、类型、是否持久化等等。



操作成功后，消息代理使用exchange.declare-ok方法进行回应：

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/exchange-declare-ok.png)

exchange.declare-ok方法除了通道号之外没有携带任何其他参数。

AMQP队列类的配对方法 - queue.declare方法 和 queue.declare-ok有着与其他配对方法非常相似的一系列事件：

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/queue-declare.png)

![enter image description here](https://www.rabbitmq.com/img/tutorials/intro/queue-declare-ok.png)

不是所有的AMQP方法都有与其配对的“另一半”。许多（basic.publish是最被广泛使用的）都没有相对应的“响应”方法，另外一些（如basic.get）有着一种以上与之对应的“响应”方法。

****

### 连接

AMQP连接通常是长连接。AMQP是一个使用TCP提供可靠投递的应用层协议。AMQP使用认证机制并且提供TLS（SSL）保护。当一个应用不再需要连接到AMQP代理的时候，需要优雅的释放掉AMQP连接，而不是直接将TCP连接关闭。



****

### 通道

有些应用需要与AMQP代理建立多个连接。无论怎样，同时开启多个TCP连接都是不合适的，因为这样做会消耗掉过多的系统资源并且使得防火墙的配置更加困难。AMQP 0-9-1提供了通道（channels）来处理多连接，可以**把通道理解成共享一个TCP连接的多个轻量化连接。**

在涉及多线程/进程的应用中，为每个线程/进程开启一个通道（channel）是很常见的，并且这些通道不能被线程/进程共享。

一个特定通道上的通讯与其他通道上的通讯是完全隔离的，因此每个AMQP方法都需要携带一个通道号，这样客户端就可以指定此方法是为哪个通道准备的。

****

###  虚拟主机

为了在一个单独的代理上实现多个隔离的环境（用户、用户组、交换机、队列 等），AMQP提供了一个虚拟主机（virtual hosts - vhosts）的概念。这跟Web servers虚拟主机概念非常相似，这为AMQP实体提供了完全隔离的环境。当连接被建立的时候，AMQP客户端来指定使用哪个虚拟主机。

****



## Java客户端开发指南

详细参考官方文档

[Java Client API Guide — RabbitMQ](https://www.rabbitmq.com/api-guide.html)



RabbitMQ Java 客户端使用`com.rabbitmq.client`作为它的顶级包。关键的类和接口有：

- Channel: 代表 AMQP 0-9-1通道，并提供了大多数操作（协议方法）。
- Connection: 代表 AMQP 0-9-1 连接
- ConnectionFactory: 构建`Connection`实例
- Consumer: 代表消息的消费者
- DefaultConsumer: 消费者通用的基类
- BasicProperties: 消息的属性（元信息）
- BasicProperties.Builder: `BasicProperties`的构建器



通过`Channel`（通道）的接口可以对协议进行操作。`Connection`（连接）用于开启通道，注册连接的生命周期内的处理事件，并且关闭不再需要的连接。`ConnectionFactory`用于实例化`Connection`对象，并且可以通过`ConnectionFactory`来进行诸如vhost、username等属性的设置。

****

### 初始化

***

#### 建立连接

**相关源码全部发布在下面的仓库中**

[源码仓库](https://gitee.com/DaHuYuXiXi/rabbitmq-client)

- RabbitmqClient封装建立连接用的相关属性

```
@Builder
public class RabbitmqClient {
     private String userName;
     private String password;
     private String virtualHost;
     private String host;
     private Integer port;

     public Connection getConnection() throws IOException, TimeoutException {
         ConnectionFactory connectionFactory = new ConnectionFactory();
         connectionFactory.setUsername(userName);
         connectionFactory.setPassword(password);
         connectionFactory.setVirtualHost(virtualHost);
         connectionFactory.setHost(host);
         connectionFactory.setPort(port);
         return connectionFactory.newConnection();
     }
}
```

- 工具类

```java
public class RabbitmqUtil {
    private final String keyPrefix="spring.rabbitmq.";
    private final YamlUtil yamlUtil;
    private final RabbitmqClient rabbitmqClient;

    public RabbitmqUtil(String ymlPath) {
        this.yamlUtil =new YamlUtil(ymlPath);
        this.rabbitmqClient=RabbitmqClient.builder()
                .userName(yamlUtil.get(keyPrefix+"username"))
                .password(yamlUtil.get(keyPrefix+"password"))
                .host(yamlUtil.get(keyPrefix+"host"))
                .port(Integer.valueOf(yamlUtil.get(keyPrefix+"port")))
                .virtualHost(yamlUtil.get(keyPrefix+"virtual-host"))
                .build();
    }

    public Connection getConnection() throws IOException, TimeoutException {
       return rabbitmqClient.getConnection();
    }
}
```

- 测试

```
        RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
        Connection connection = rabbitmqUtil.getConnection();
```

对于运行在本地的RabbitMQ节点而言，这些参数都有合适的默认值。

如果在创建连接前没有指定参数值，则会使用默认参数：

| Property     | Default Value                        |
| ------------ | ------------------------------------ |
| Username     | "guest"                              |
| Password     | "guest"                              |
| Virtual host | "/"                                  |
| Hostname     | "localhost"                          |
| port         | 5672正常通信端口,5671用于SSL加密通信 |

需要注意的是，默认情况下[guest（来宾）用户只能用本地进行连接](https://www.rabbitmq.com/access-control.html)。目的是为了限制已知凭证在生产系统中的使用。

```yml
#在配置文件中设置loopback_users为none,那么guest账号就可以进行远程连接了
loopback_users = none
```

****



#### 创建通道

```java
    public Channel createChannel() throws IOException {
        log.info("通道创建中...");
       return connection.createChannel();
    }
```

****

#### 关闭Rabbitmq连接

通过简单的对通道和连接进行关闭即可关闭掉RabbitMQ的连接：

```java
    public void close(){
        log.info("关闭rabbitmq连接中...");
        try {
            //channel.close(); 非必须
            connection.close();
        } catch (IOException e) {
            log.error("关闭连接过程中出现错误: ",e);
        }
    }
```



需要注意的是，将通道关闭掉不是必须的操作。因为无论何种情况，通道都会在底层的连接关闭时自动关闭掉。

****

#### 连接和通道的寿命

客户端[connections](https://www.rabbitmq.com/connections.html)是长连接。底层协议的设计和优化都考虑到了长连接的需求。这意味着对诸如消息发送之类的每个操作都建立一个连接的形式是极其不推荐的，那样做会产生大量的网络往返和开销。

[Channels](https://www.rabbitmq.com/channels.html) 虽然也是长期存活的，但是由于有大量的可恢复的协议错误会导致通道关闭，通道的存活期会比连接短一些。虽然每个操作都打开和关闭一个通道不是必须的操作，但是也不是不可行。有的选的情况下，还是优先考虑通道的复用为好。

类似于尝试从一个不存在的队列里消费消息这种 [通道级别的异常](https://www.rabbitmq.com/channels.html#error-handling) 会导致通道关闭。已经关闭的通道不可以再被使用，也不会再接收到如*消息投递*之类的服务器事件。RabbitMQ会记录下通道级别的异常，并且会为通道初始化一个关闭顺序

****

#### 提供本次连接的标记名称

RabbitMQ 节点可以持有有限的的客户端信息：

- 客户端的TCP节点（来源IP地址和端口）
- 使用的凭证

包括RabbitMQ Java客户端在内的AMQP 0-9-1客户端链接可以提供一个自定义标识符，一遍在[服务器日志](https://www.rabbitmq.com/logging.html) 和[管理界面](https://www.rabbitmq.com/management.html)中方便地对客户端进行区分。设置好后，日志内容管理界面中便会对标识符有所体现。标识符即为**客户端提供的连接名称**。名称可以用于标识应用或应用中特定的组件。虽然名称是可选的，但是强烈建议提供一个，这将大大简化某些操作任务。

![image-20220526191401910](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261914009.png)

newConnection方法提供了很多重载方法,其中一部分提供了此次连接名称的设置

```java
    public Connection getConnection(String connectionName) throws IOException, TimeoutException {
        connectionFactory.setUsername(userName);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost(virtualHost);
        connectionFactory.setHost(host);
        connectionFactory.setPort(port);
        return connectionFactory.newConnection(connectionName);
    }
```

![image-20220526191357387](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913513.png)

****

### 交换机和队列

交换机和队列在使用事前必须对他们进行声明。

简单来讲，对任何一种对象类型进行声明的目的是为了确保它们已经存在，并在需要的时候对其进行创建。

#### 客户端独占队列



以下代码声明了一个交换机以及一个[服务端命名的队列](https://www.rabbitmq.com/queues.html#server-named-queues)，然后将它们绑定到一起

```java
        channel.exchangeDeclare("dhy-exchange", "direct", true);
        //queueDeclare创建的队列名为""
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "dhy-exchange", "dhy");
```

这将会主动声明以下对象，这两个对象都可以使用附加参数进行自定义。但在这里，没有给他们俩定义特殊的参数。

- 持久化、非自动删除的“直连”形交换机
- 具有系统生成的名称的，非持久化、独占、自动删除的队列

![image-20220526191346576](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913692.png)

注意，当只有一个客户端打算独占队列时，这是一个典型的队列声明方式。队列不需要既定的名称，没有其他客户端使用此队列（独占），队列会被自动清理掉（自动删除）。如果有多个客户端消费打算消费一个既定名称的队列，一下代码更为合适：

```java
channel.exchangeDeclare(exchangeName, "direct", true);
channel.queueDeclare(queueName, true, false, false, null);
channel.queueBind(queueName, exchangeName, routingKey);
```

这将会主动进行以下声明：

- 持久化、非自动删除的“直连”交换机
- 拥有既定名称的，持久化、非独占、非自动删除的队列

许多`Channel`接口方法都是被重载的。这里用到的关于 `exchangeDeclare`, `queueDeclare` 和 `queueBind`的短结构的重载方法使用了合适的默认值，更易于使用。当然也有更多参数的长结构的重载方法，使用那些方法可以将一些必要的默认参数进行重写，进行更全面的控制。

***

#### 队列和交换机的被动声明

队列和交换机可以被动地进行声明。被动声明会简单地检查提供的名称所对应的实体是否存在。对成功检测到的队列来说，被动声明会返回跟非被动声明同样的信息，即队列中处于就绪状态的消费者和消息数量。

如果对应的实体不存在，操作会抛出一个通道级别的异常。然后通道就不可以继续使用了，需要打开一个新的通道。通常在进行被动声明的时候使用临时的一次性通道。

`Channel#queueDeclarePassive` 和 `Channel#exchangeDeclarePassive` 方法被用来进行被动声明。下边演示`Channel#queueDeclarePassive` 的使用：

```java
        AMQP.Queue.DeclareOk response = channel.queueDeclarePassive("queue-name");
        response.getMessageCount();
        response.getConsumerCount();
```

`Channel#exchangeDeclarePassive` 方法的返回值没包含什么有用的信息。只要方法正确返回，并且没有通道异常发生，就意味着交换机已经存在了。

***

#### 不等待服务器响应



一些常见的操作还带有“非等待”版本，这种版本的操作不会等待服务器的响应。例如，以下方法会声明一个队列并且通知服务器不要发送任何响应



```java
channel.queueDeclareNoWait(queueName, true, false, false, null);
```



“非等待”版本的操作会更具效率，但是安全保障较低，例如，它们更依赖心跳机制去检测失败的操作。如果不确定，就从标准版本的操作用起。“非等待”版本只是在高级拓扑结构（队列、绑定）的情况下需要。



***

#### 实体和消息的清除

可以显示地将队列和交换机删除：

```java
channel.queueDelete("queue-name")
```

也可以做到当队列为空时对其进行删除：

```java
channel.queueDelete("queue-name", false, true)
```

或者当它不再被使用的时候（没有任何消费者对其进行消费）：

```java
channel.queueDelete("queue-name", true, false)
```

队列可以被清除（删除里边的所有消息）：

```java
channel.queuePurge("queue-name")
```

***

#### 发布消息

```java
//发布消息到交换机
channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,null,"hello rabbitmq".getBytes(StandardCharsets.UTF_8));
```

想要实现更完善的控制，可以使用重载的变体来指定`mandatory`标识，或者发送预设好消息属性的消息。

```java
channel.basicPublish(exchangeName, routingKey, mandatory,
                     MessageProperties.PERSISTENT_TEXT_PLAIN,
                     messageBodyBytes);
```



> mandatory是强制的意思

当我们向某个交换机发送消息后，交换机发现消息无法被路由到任何一个绑定到该交换机的队列上，那么如果publiher发送消息时,将mandatory属性设置为了false(默认就是false),那么消息会被转交给alternate exchange兜底交换机，前提是该交换机存在，不存在会记录警告日志。

当我们向某个交换机发送消息后，交换机发现消息无法被路由到任何一个绑定到该交换机的队列上，那么如果publiher发送消息时,将mandatory属性设置为了true,该消息会被返回给publisher,对应的消息发送方需要提供一个处理回退消息的回调接口,可以通过该接口完成对路由失败消息的记录或者尝试将其转交给其他交换机发送。



mandatory属性具体说明大家也可以参考官方文档:

[Publishers — RabbitMQ](https://www.rabbitmq.com/publishers.html)



兜底交换机具体大家可以参考官方文档:

[Alternate Exchanges — RabbitMQ](https://www.rabbitmq.com/ae.html)

****

以下示例发送消息的时候会指定投递模式为2（持久化），优先级为1并且消息体类型（content-type）为"text/plain"。使用`Builder`类去创建一个需要指定多个属性的消息属性对象，例如：

```java
        //发布消息到交换机
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY,
                new AMQP.BasicProperties.Builder()
                        .contentType("text/plain")
                        .deliveryMode(2)
                        .priority(1)
                        .userId("dhy")
                        .build(),
                "hello2".getBytes(StandardCharsets.UTF_8));
```

以下是发布带有自定义headers消息的示例：

```java
Map<String, Object> headers = new HashMap<String, Object>();
headers.put("dhy", 18);

channel.basicPublish(exchangeName, routingKey,
             new AMQP.BasicProperties.Builder()
               .headers(headers)
               .build(),
               "hello3".getBytes(StandardCharsets.UTF_8));
```



以下例子会发布一条具有过期时间属性的消息：

```java
channel.basicPublish(exchangeName, routingKey,
             new AMQP.BasicProperties.Builder()
               .expiration("60000")
               .build(),
               "hello4".getBytes(StandardCharsets.UTF_8));
```

这里我们并没有展示所有的可能性。

注意`BasicProperties`是AMQP自动生成的持有类的内置类。

****

#### 通道和并发

应该尽量避免在线程间共享通道对象。应用应该尽可能为每个线程都使用单独的通道，而不是将通道共享给多个线程。

虽然可以安全地并发调用通道上的某些操作，但有些操作则不能并发调用，如果那样做会导致错误的帧交错在网络上，或造成重复确认等问题。

在共享的通道上并发执行发布会导致错误的帧交错在网络上，触发连接级别的协议异常并导致连接被代理直接关闭。因此，需要在应用程序代码中进行显式同步（必须在关键部分调用`Channel＃basicPublish`）。

线程之间共享通道也会干扰[发布者确认](https://www.rabbitmq.com/confirms.html)。最好能够完全避免在共享的通道上上进行并发发布，例如通过每个线程使用一个通道的方式实现并发。

也可以通过通道池的方式来避免在共享通道上并发发布消息：一旦一个线程使用完了某个通道，就将通道归还到池中，使得通道可以被其他线程再次使用。通道池可以视为一个特殊的同步解决方案。建议使用现成的池库来实现，而不是自己实现。例如开箱即用的 [Spring AMQP](https://projects.spring.io/spring-amqp/) 。

通道是吃资源的，而且大多数应用情景下同一个JVM进程很少会开放小几百的通道出来。设想我们应用的每个线程都持有一个通道（由于同一个通道不应被用于并发操作），单个JVM里上千个线程已经会是一个相当大的开销，这些开销本来是可以避免的。此外，一小部分快速的发布者可以很轻松地占满网络接口和代理节点。

一个需要避免的经典的反模式就是为每个发布的消息开放单独的通道。通道应该是长时间存活的。

一个线程用于消费，另一个线程在共享通道上推送是安全的。

服务推送投递（下边介绍）是以并发的方式分发的，并能确保每个通道顺序的固定。分发机制在每个连接中使用一个[java.util.concurrent.ExecutorService](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html)。使用`ConnectionFactory#setSharedExecutor` setter 的`ConnectionFactory`生成的所有连接都可以共享一个自定义的`executor`。

****



#### 通过订阅来接收消息(回调接口)



接收消息最高效的方式是使用`Consumer`消息推送接口设置订阅。消息在到达时被自动投递到其中，而不是显示的去请求。

当调用`Consumers`相关的接口方法时，单个订阅始终由其消费者标签引用。消费者标签可以由客户端或者服务器来生成，用于消费者的身份识别。想让RabbitMQ生成一个节点范围内的唯一标签，可以使用不含有消费者标签属性的`Channel#basicConsume` 重载，或者传递一个空字符串做为消费者标签，然后使用`Channel#basicConsume`返回的值。消费者标签同样用于清除消费者之用。

不同的消费者实例必须持有不同的消费者标签。非常不建议在同一个连接上出现重复的消费者标签，这回导致 [自动连接覆盖](https://www.rabbitmq.com/api-guide.html#connection-recovery) 问题，并在监控消费者时混淆监控数据。

实现`Consumer`最简单的方式是子类化`DefaultConsumer`。此子类的实例化对象可以当做`basicConsume`调用时的参数进行传递，用于设置订阅：

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,null,"你好,我是生产者".getBytes(StandardCharsets.UTF_8));
            log.info("发送消息...");
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}

```



```java
@Slf4j
public class Consumer implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            //不开启自动应答
            boolean autoAck = false;
            channel.basicConsume(QUEUE_NAME, autoAck, "myConsumerTag",
                    new DefaultConsumer(channel) {
                        @Override
                        public void handleDelivery(String consumerTag,
                                                   Envelope envelope,
                                                   AMQP.BasicProperties properties,
                                                   byte[] body)
                                throws IOException
                        {
                            String routingKey = envelope.getRoutingKey();
                            String contentType = properties.getContentType();
                            long deliveryTag = envelope.getDeliveryTag();
                            //手动确认消息收到
                            channel.basicAck(deliveryTag, false);
                            log.info("接收到消息: {} , 路由key为: {} ,类型为: {}",new String(body),routingKey,contentType);
                        }
                    });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}

```

**测试:**

```java
        Thread consumer = new Thread(new Consumer(),"消费者线程");
        Thread publisher = new Thread(new Publisher(),"生产者线程");
        consumer.start();
        publisher.start();
```

![image-20220526191335461](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913570.png)



这里由于我们设置了`autoAck = false`，需要手动对投递到`Consumer`的消息进行确认。最简便的方式就是如上边介绍的一样在`handleDelivery`中进行。

更复杂的消费者需要去覆写其他方法。特别说明的是，当通道和连接关闭时，`handleShutdownSignal`会被调用，`handleConsumeOk`会在调用其他`Consumer`回调之前被传递给消费者标签。

```java
    @Override
    public void handleConsumeOk(String consumerTag) {
        this._consumerTag = consumerTag;
    }
```

`Consumers`同样可以通过实现`handleCancelOk`和`handleCancel`方法来分别被告知是通过显式还是隐式方式进行取消。

你可以通过`Channel.basicCancel`显式地取消一个指定的`Consumer`。

```java
channel.basicCancel(consumerTag);
```

传递消费者标签。

就像发布者一样，这里同样也需要考虑到消费者的并发安全性。

消费者的回调的调度是在一个独立的线程池里完成的，这个线程池跟通道实例化的那个池是分开的。这表示`Consumers`可以安全的调用类似于`Channel#queueDeclare`和`Channel#basicCancel`这种链接和通道的阻塞方法。

每个通道都有自己的调度线程。对于大多数常见的每个`Channel`一个`Consumer`的场景下，这意味着消费者之间不会相互影响。需要注意，如果一个通道里有多个消费者，长时间运行的消费者会阻挡通道中其他消费者回调方法的调度。



****



#### 获取单条消息（拉取接口）

尝试去拉取消息，如果当前存在消息则返回，否则返回null,因此客户端大多需要不断轮询来获取消息，所以这种方式不推荐。

使用`Channel.basicGet`来进行消息的“拉取”。返回值是包含有头信息（属性）和消息体的`GetResponse`对象实例。



```java
@Slf4j
public class Consumer implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            boolean autoAck = false;
            while(true){
                GetResponse response = channel.basicGet(QUEUE_NAME, autoAck);
                if (response == null) {
                    log.info("当前无消息....");
                } else {
                    byte[] body = response.getBody();
                    log.info("msg: {}",new String(body));
                    channel.basicAck(response.getEnvelope().getDeliveryTag(),false);
                    break;
                }
            }
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}
```

![image-20220526191329133](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913247.png)

ChannIN类中源码其实也非常简单:

```java
@Override
    public GetResponse basicGet(String queue, boolean autoAck)
        throws IOException
    {
        validateQueueNameLength(queue);
        //构造命令
        AMQCommand replyCommand = exnWrappingRpc(new Basic.Get.Builder()
                                                  .queue(queue)
                                                  .noAck(autoAck)
                                                 .build());
        //命令执行
        Method method = replyCommand.getMethod();
        //判断执行结果
        if (method instanceof Basic.GetOk) {
            Basic.GetOk getOk = (Basic.GetOk)method;
            Envelope envelope = new Envelope(getOk.getDeliveryTag(),
                                             getOk.getRedelivered(),
                                             getOk.getExchange(),
                                             getOk.getRoutingKey());
            BasicProperties props = (BasicProperties)replyCommand.getContentHeader();
            byte[] body = replyCommand.getContentBody();
            int messageCount = getOk.getMessageCount();

            metricsCollector.consumedMessage(this, getOk.getDeliveryTag(), autoAck);
            //有结果，那么构造成GetResponse后返回
            return new GetResponse(envelope, props, body, messageCount);
        } else if (method instanceof Basic.GetEmpty) {
            //没有消息,那么返回的结果为空，会返回null
            return null;
        } else {
            throw new UnexpectedMethodError(method);
        }
    }
```



****



#### 处理无法路由的消息

如果发布的消息设置了`mandatory`标识，但是没有被成功路由，代理会将其返回给发送的客户端（通过`AMQP.Basic.Return`命令）。

客户端可以通过实现`ReturnListener`接口并调用`Channel.addReturnListener`来收到此类退还通知。如果客户端没有为特定的通道配置退还监听，那返回的相应消息会被默默地丢弃掉。

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            channel.addReturnListener(new RouteFailListener());
            channel.basicPublish(EXCHANGE_NAME,UNKNOWN_ROUTING_KEY,true,null,"你好,我是生产者".getBytes(StandardCharsets.UTF_8));
            log.info("发送消息...");
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}

```

例如，客户端发布了一条带有`mandatory`标识的消息，此消息设置了交换机类型为“直连”，但是交换机并没有绑定到队列上，此时退还监听就会被调用。

```java
        Thread publisher = new Thread(new Publisher(),"生产者线程");
        publisher.start();
```



日志如下:

```tex
19:52:56.103 [生产者线程] INFO com.dhy.util.RabbitmqUtil - 连接建立
19:52:56.232 [生产者线程] INFO com.dhy.util.RabbitmqUtil - 连接建立,客户端设置的连接名为dhy-connection
19:52:56.234 [生产者线程] INFO com.dhy.util.RabbitmqUtil - 通道创建中...
19:52:56.369 [生产者线程] INFO com.dhy.util.RabbitmqUtil - 准备channel中..
19:52:56.374 [生产者线程] INFO com.dhy.Publisher - 发送消息...
19:52:56.403 [AMQP Connection 110.40.155.17:5672] WARN com.dhy.RouteFailListener - 路由失败消息信息如下: replyCode=312 ,replyText=NO_ROUTE ,exchange=dhy-exchange ,routingKey=unknown ,body=你好,我是生产者

```

****

### 消费者操作线程池

默认情况下，消费者线程会通过一个新的`ExecutorService`线程池分配。

![image-20220526191321036](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913145.png)

**可以看出默认消费者线程大小是cpu核心数*2**



如果需要更大的控制权，可以使用`newConnection()`去应用`ExecutorService`以进行替代。这是一个应用一个比常规分配额更大的线程池的示例：

```java
ExecutorService es = Executors.newFixedThreadPool(20);
Connection conn = factory.newConnection(es);
```

当连接关闭时，默认提供的`ExecutorService`也会执行`shutdown()`，但是用户提供的`ExecutorService`（如上所示）则*不会*执行`shutdown()`。提供自定义`ExecutorService`的客户端必须确保其最终会被关闭（即调用`shutdown()` 方法），否则线程池会影响JVM的中止。

相同的executor服务可能会被多个连接共享，或者接连不断的重复使用、重复连接，但是无论如何当它关闭后是不可以再用的。



应该在有证据表明处理消费回调存在严重瓶颈时才去考虑使用这个功能。如果没有或者只有少量消费者回调需要执行，那默认分配的线程就足够了。即使偶尔会有消费者活动陡增的情况，最初的负载是很小的，并且线程资源的分配和不能无限扩大。



****



### 主机列表的使用

把一个`Address`数组传给`newConnection()`是没问题的。`Address`是一个`com.rabbitmq.client package`中包含 *主机* 和 *端口*组件的简单的便捷类。

例如：

```java
Address[] addrArr = new Address[]{ new Address(hostname1, portnumber1)
                                 , new Address(hostname2, portnumber2)};
Connection conn = factory.newConnection(addrArr);
```

这样会先去尝试连接`hostname1:portnumber1`，失败的话会再尝试`hostname2:portnumber2`。返回的连接对象是第一次成功的数组元素的(没抛出`IOException`的话)。这跟分别设置主机和端口然后依次调用`factory.newConnection()`直到成功的操作一毛一样。



如果同时也提供了`ExecutorService`（在`factory.newConnection(es, addrArr)`中使用），那线程池也是对应的第一次成功连接的那个。

****

### 使用AddressResolver接口实现服务发现

我们可以使用`AddressResolver`接口实现来改变连接时的端点解析算法：

```java
Connection conn = factory.newConnection(addressResolver);
```

`AddressResolver`接口类似于：

```java
public interface AddressResolver {

  List<Address> getAddresses() throws IOException;

}
```

就跟 [主机列表](https://www.rabbitmq.com/api-guide.html#address-array)一样，先尝试返回的第一个`Address`，如果失败了再试第二个，直到成功为止。

如果同时也提供了`ExecutorService`（在`factory.newConnection(es, addrArr)`中使用），那线程池也是对应的第一次成功连接的那个。

`AddressResolver`是实现自定义服务发现逻辑的最佳方式，客户端可以自动连接到首次启动时尚未出现故障的节点。

Java客户端附带了以下实现（详见javadoc）：

- `DnsRecordIpAddressResolver`：根据给定的主机名，返回其IP地址（针对DNS服务器平台的解析）。
- `DnsSrvRecordAddressResolver`：根据给定的服务的名字，返回其所在的主机名/端口对,可以实现类似于服务注册与发现功能，参考Eurkea,Consule

![image-20220526191304254](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261913343.png)

默认只有一个主机地址是走DnsRecordIpAddressResolver

![image-20220526191258861](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912952.png)

getAddresses方法会在newConnection方法中被回调

****

### autoDelete属性问题

- exchange自动删除的条件，有队列或者交换器绑定了本交换器，然后所有队列或交换器都与本交换器解除绑定，autoDelete=true时，此交换器就会被自动删除。
- 队列自动删除的条件，有消息者订阅本队列，然后所有消费者都解除订阅此队列，autoDelete=true时，此队列会自动删除，即使此队列中还有消息。



***



## Rabbitmq七种模式



### 简单队列模式

![(P) -> [|||] -> (C)](https://www.rabbitmq.com/img/tutorials/python-one.png)

**该模型很简单，就是生产者直接将消息放入队列中，消费者从队列取出消息进行消费**

> 实际是将消息放入默认直连交换机中,然后该交换机绑定指定队列

[RabbitMQ tutorial - "Hello World!" — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-one-java.html)

**生产者:**

```java
//如果使用默认交换机,传入空字符串即可
//对于默认交换机而言,路由key就是绑定到其上的队列名
channel.basicPublish("",QUEUE_NAME,true,null,"你好,我是生产者".getBytes(StandardCharsets.UTF_8));
```

![image-20220526191253105](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912187.png)

**消费者:**

```java
            channel.basicConsume(QUEUE_NAME,true,new DefaultConsumer(channel){
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
```

****

### 工作队列模式

![image-20220526191729073](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261917134.png)

**工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。**

> 上面这幅图还是先将消息放入默认交换机中，和简单队列相比，就是队列的消费者数量增加了，那么因为消息还是只能被消费一次，消息如何分发就成为了我们的关注点

[RabbitMQ tutorial - Work Queues — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-two-java.html)

消费者代码不变，生产者代码同样不变，我们只需要同时启动两个消费者即可，并且通过web界面，手动往队列中塞入消息进行测试:

![image-20220526191246040](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912136.png)



我们一共发送了6条消息，下面看看每个消费者都接收到了多少消息:

![image-20220526191240602](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912704.png)

**默认情况下，RabbitMQ 将按顺序将每条消息发送给下一个使用者。平均而言，每个消费者将获得相同数量的消息。这种分发消息的方式称为轮循机制。**

****

#### 消息确认机制

- 消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。RabbitMQ一旦向消费者传递了一条消息，便立即将该消息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费者的消息，因为它无法接收到。

- 为了保证消息在发送过程中不丢失，RabbitMQ引入消息应答机制，消息应答就是:消费者在接收到消息并且处理该消息之后，告诉RabbitMQ它已经处理了，RabbitMQ可以把该消息删除了。
  

> 如果消费者在指定超时时间内没有对某个消息做出应答，那么会强制关闭当前通道，并抛出PRECONDITION_FAILED通道级异常
>
> [PRECONDITION_FAILED](https://www.rabbitmq.com/consumers.html#acknowledgement-timeout)
>
> 默认超时时间为30分钟



****

##### 自动应答

消息发送后立即被认为已经传送成功，这种模式需要在高吞吐量和数据传输安全性方面做权衡,因为这种模式如果消息在接收到之前，消费者那边出现连接或者channel关闭，那么消息就丢失了,当然另一方面这种模式消费者那边可以传递过载的消息，没有对传递的消息数量进行限制,当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，所以这种模式仅适用在消费者可以高效并以某种速率能够处理这些消息的情况下使用。



> 建议不要采用自动应答

![image-20220526191234289](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912381.png)

****

##### 手动应答

- 确认消息

  ```java
  //第一个参数:确认哪一个消息
  //第二个参数:是否开启消息批量应答
  channel.basicAck(envelope.getDeliveryTag(),false);
  ```

  **批量应答怎么玩？**

  可以对手动确认进行批处理以减少网络流量。这是通过将确认方法的多个字段设置为 true 来完成的。

  > 当批处理字段设置为true时:
  >
  > 例如，假设通道 Ch 上有未确认的传递标记 5、6、7 和 8，当确认帧到达该通道时，delivery_tag设置为 8 且批处理标记设置为 true，则将确认从 5 到 8 的所有标记。
  >
  > 如果将批处理标记设置为 false，则交付 5、6 和 7 仍将不被确认。

  > 如果消费者拿到了消息但是直到断开连接前，都没有对消息进行应答，那么消息会重新入队

- 拒绝消息(两种方式)

  ```java
  //第一个参数:拒绝哪一个消息
  //第二个参数:是否将拒绝的消息重新入队
  channel.basicReject(envelope.getDeliveryTag(),true);
  ```

  ```java
  //第一个参数:拒绝哪一个消息
  //第二个参数:是否批量拒绝
  //第三个参数:是否将拒绝的消息重新入队
  //basic.nack 方法可以一次拒绝或重新排队多条消息。这就是它与 basic.reject 的区别。
  channel.basicNack(envelope.getDeliveryTag(),true,true);
  ```

  使用者无法立即处理交付，但其他实例可能能够处理。在这种情况下，可能需要将其重新排队，让另一个消费者接收并处理它。basic.reject 和 basic.nack 是用于此目的的两种协议方法。

​       此类消息可以被丢弃或死信或重新排队。此行为由**requeue**字段控制。当该字段设置为 true 时，代理将使用指定的传递标记将传递（或多个传递）重新排队。或者，当此字段设置为 false 时，如果已配置，则消息将被路由到[死信交换](https://www.rabbitmq.com/dlx.html)，否则将被丢弃。

​    当消息重新排队时，如果可能，它将被放置在其队列中的原始位置。如果不是（由于多个使用者共享队列时来自其他使用者的并发传递和确认），则消息将重新排队到更靠近队列头的位置。

更多消息发布确认细节可以参考官方文档:

[Consumer Acknowledgements and Publisher Confirms — RabbitMQ](https://www.rabbitmq.com/confirms.html)

****

#### 消息持久性

消息持久化需要将相关的队列先进行持久化，然后在发布消息时，将消息标记为持久化。

![image-20220526191200022](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261912103.png)

```java
 channel.basicPublish("",QUEUE_NAME,true, 
                      //消息添加持久化属性
                      MessageProperties.PERSISTENT_TEXT_PLAIN,("序号"+i).getBytes(StandardCharsets.UTF_8));
```

**Notice ! ! !**

> 将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉RabbitMQ将消息保存到磁盘，但是这里依然存在当消息刚准备存储在磁盘的时候但是还没有存储完，消息还在缓存的一个间隔点。此时并没有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。如果需要更强有力的持久化策略，则可以使用[发布者确认](https://www.rabbitmq.com/confirms.html)。

****

#### 公平调度和预取值

您可能已经注意到，调度仍然不完全符合我们的要求。例如，在有两个 worker 的情况下并且二者处理速度相差很大的情况下，一个 worker 会一直很忙，而另一个 worker 几乎不会做任何工作。好吧，RabbitMQ对此一无所知，仍然会均匀地调度消息。

发生这种情况是因为 RabbitMQ 只是在消息进入队列时调度消息。它不查看使用者的未确认消息的数量。它只是盲目地将第 n 条消息分派给第 n 个使用者。

![img](https://www.rabbitmq.com/img/tutorials/prefetch-count.png)

为了解决这个问题，我们可以使用具有预取计数 = 1 设置的基本 Qos 方法。这告诉 RabbitMQ 不要一次向一个工人发送多条消息。或者，换句话说，在处理并确认前一条消息之前，不要向工作人员发送新消息。相反，它会将其分派给下一个仍然不繁忙的工作人员。

```java
channel.basicQos(1)
```

> 举例: 消费端程序调用了 channel.basicQos(5) ，之后订阅了某个队列进行消费。 RabbitMq 会保存一个消费者的列表，每发送一条消息都会为对应的消费者计数，计数达到5后，那么RabbitMQ就不会向这个消费者再发消息。消费者确认了某条消息处理完后，RabbitMQ 将相应的计数减1之后消费者可以继续接收消息，直到再次到达计数上限。这种机制可以类比于 TCP IP中的"滑动窗口"

[消费者确认](https://www.rabbitmq.com/confirms.html)



****

### 发布订阅模式

[RabbitMQ tutorial - Publish/Subscribe — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)

把交换机（Exchange）里的消息发送给所有绑定该交换机的队列，忽略routingKey。

![img](https://www.rabbitmq.com/img/tutorials/python-three-overall.png)

**上面这种交换机被称为扇形交换机(fanout)**

> 给出的示例图中，两个队列都是临时队列,即队列名由服务端生成的队列,与唯一的客户端绑定，客户端断开连接后，队列自动被删除

发布订阅模式很简单，这里给出一个简单的例子:

- 准备好交换机和队列

```java
    public static final String EXCHANGE_NAME="dhy-exchange";
    public static final String QUEUE_NAME="dhy-queue";
    public static final String ROUTING_KEY="dhy";
    public static final String TEMP_QUEUE="";
    public static final String UNKNOWN_ROUTING_KEY ="unknown";

    public Channel prepareChannel() throws IOException, TimeoutException {
        RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml","dhy-connection");
        Channel channel = rabbitmqUtil.createChannel();
        //声明交换机和队列
        //非持久化、非自动删除的“扇形”形交换机
        channel.exchangeDeclare(EXCHANGE_NAME, FANOUT, false);
        //拥有既定名称的，非持久化、非独占、非自动删除的队列
        channel.queueDeclare(TEMP_QUEUE, false, false, false, null);
        //绑定交换机和队列
        channel.queueBind(TEMP_QUEUE, EXCHANGE_NAME, ROUTING_KEY);
        log.info("准备channel中..");
        return channel;
    }
```

- 消费者

  ![image-20220526191148793](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261911889.png)

  

- 生产者

  ![image-20220526191144497](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261911602.png)

  

  ![image-20220526191140578](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261911685.png)

****

### 路由模式

[RabbitMQ tutorial - Routing — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-four-python.html)

![img](https://www.rabbitmq.com/img/tutorials/direct-exchange.png)

![img](https://www.rabbitmq.com/img/tutorials/direct-exchange-multiple.png)

![img](https://www.rabbitmq.com/img/tutorials/python-four.png)

**上面是官网给出的三幅示例图,其实就是利用了直连交换机通过路由key去完成消息投递的特性，构建出了上面这种路由模式**

可以把上面交换机和队列的关系理解为map集合的关系:

```java
Map<List<RouteKey>,List<Queue>> exchange;
```

当交换机拿到一个RouteKey后，需要知道该把这个消息路由给哪些队列，怎么办呢？

```java
List<Queue> queues=exchange.get(Arrays.asList(key1,key2...))
```

**多个RouteKey可能共同指向同一个queue,也可能一个RouteKey指向多个队列，因此两者之间是多对多的关系**



下面给出使用演示：

- 常量准备

```java
    /**
     * 直接交换机
     */
    public static final String DIRECT_EXCHANGE="direct";
    /**
     * 队列一
     */
    public static final String QUEUE_ONE="queue_one";
    public static final String ONE_KEY="one";
    /**
     * 队列二
     */
    public static final String QUEUE_TWO="queue_two";
    public static final String TWO_KEY="two";
```



- 生产者代码准备

    ```java
    @Slf4j
    public class Publisher implements Runnable {
        @Override
        public void run() {
            try {
                RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
                Channel channel = rabbitmqUtil.createChannel();
                //声明直接交换机
                channel.exchangeDeclare(DIRECT_EXCHANGE, DIRECT,false);
                //分别发送两个消息,对应的路由key为one和two
                channel.basicPublish(DIRECT_EXCHANGE,ONE_KEY,false, null,"one".getBytes(StandardCharsets.UTF_8));
                channel.basicPublish(DIRECT_EXCHANGE,TWO_KEY,false, null,"two".getBytes(StandardCharsets.UTF_8));
            } catch (IOException | TimeoutException e  ) {
                log.error("出现异常: ",e);
            }
        }
    }
    ```

- 消费者代码准备

```java
@Slf4j
public class ConsumerOne implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            channel.queueDeclare(QUEUE_ONE,false,false,false,null);
            //绑定别忘了
            channel.queueBind(QUEUE_ONE,DIRECT_EXCHANGE,ONE_KEY);
            channel.basicConsume(QUEUE_ONE,true,new DefaultConsumer(channel){
                @SneakyThrows
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}
```

```java
@Slf4j
public class ConsumerTwo implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            channel.queueDeclare(QUEUE_TWO,false,false,false,null);
             channel.queueBind(QUEUE_TWO,DIRECT_EXCHANGE,TWO_KEY);
            channel.basicConsume(QUEUE_TWO,true,new DefaultConsumer(channel){
                @SneakyThrows
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}

```

- 先启动消费者，再启动生产者，因为需要先创建队列才行,否则交换机收到消息找不到队列，那么会直接丢弃消息

```java
        Thread consumer1 = new Thread(new ConsumerOne(),"消费者1");
        Thread consumer2 = new Thread(new ConsumerTwo(),"消费者2");
        Thread publisher = new Thread(new Publisher(),"生产者");
        consumer1.start();
        consumer2.start();
        Thread.sleep(1000);
        publisher.start();
```



![image-20220526191131968](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261911063.png)

****



### 主题模式

[RabbitMQ tutorial - Topics — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-five-python.html)

![img](https://www.rabbitmq.com/img/tutorials/python-five.png)

**上面的路由模式本质就是利用了直接交换机的routeKey绑定特性，但是直接交换机还是有一个坏处，就是无法模糊匹配，必须精确指定routekey才行，因此这就有了主体模式，提供模糊匹配的功能**

下面先讲讲具体是如何进行模糊匹配的:

- 发送到类型是topic交换机的消息的routing_key不能随意写，必须满足一定的要求，它必须是一个单词列表，以点号分隔开。这些单词可以是任意单词，比如说: “stock.usd.nyse” ， “nyse.vmw”，"quick.orange.rabbit"这种类型的。当然这个单词列表最多不能超过255个字节。
-  在这个规则列表中，其中有两个替换符：

```java
* 可以代替一个单词
# 可以代替零个或多个单词
```

**就拿上面那副图举个例子:**

- quick.orange.rabbit：被队列Q1Q2接收到
- quick.orange.fox：被队列Q1接收到
- lazy.brown.fox：被队列Q2接收到
- lazy.pink.rabbit：虽然满足队列Q2的两个绑定但是只会被接收一次
- quick.orange.male.rabbit：四个单词不匹配任何绑定会被丢弃

**类比:**

-  当一个队列绑定键是#,那么这个队列将接收所有数据，就有点像fanout了
-  如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是direct了

**实战演示:**

- 常量准备

```java
    /**
     * 主题交换机
     */
    public static final String TOPIC_EXCHANGE="topic";


    public static final String Q1_QUEUE="Q1";
    public static final String Q1_ROUTE_KEY="*.orange.*";

    public static final String Q2_QUEUE="Q2";
    public static final String Q2_ROUTE_KEY1="*.*.rabbit";
    public static final String Q2_ROUTE_KEY2="lazy.#";
```

- 生产者准备

  ```java
  @Slf4j
  public class Publisher implements Runnable {
      @Override
      public void run() {
          try {
              RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
              Channel channel = rabbitmqUtil.createChannel();
              //声明主题交换机
              channel.exchangeDeclare(TOPIC_EXCHANGE, TOPIC,false);
              channel.basicPublish(TOPIC_EXCHANGE,"apple.orange.banana",false, null,"q1".getBytes(StandardCharsets.UTF_8));
              channel.basicPublish(TOPIC_EXCHANGE,"dog.pig.rabbit",false, null,"q21".getBytes(StandardCharsets.UTF_8));
              channel.basicPublish(TOPIC_EXCHANGE,"lazy.lazy1",false, null,"q22".getBytes(StandardCharsets.UTF_8));
          } catch (IOException | TimeoutException e  ) {
              log.error("出现异常: ",e);
          }
      }
  }
  ```

- 消费者准备

```java
@Slf4j
public class ConsumerOne implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            channel.queueDeclare(Q1_QUEUE,false,false,false,null);
            channel.queueBind(Q1_QUEUE,TOPIC_EXCHANGE,Q1_ROUTE_KEY);
            channel.basicConsume(Q1_QUEUE,true,new DefaultConsumer(channel){
                @SneakyThrows
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}

```

```java
@Slf4j
public class ConsumerTwo implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            channel.queueDeclare(Q2_QUEUE,false,false,false,null);
            //绑定两个路由key到Q2上
            channel.queueBind(Q2_QUEUE,TOPIC_EXCHANGE,Q2_ROUTE_KEY1);
            channel.queueBind(Q2_QUEUE,TOPIC_EXCHANGE,Q2_ROUTE_KEY2);
            channel.basicConsume(Q2_QUEUE,true,new DefaultConsumer(channel){
                @SneakyThrows
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}
```

![image-20220526191122177](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261911262.png)

![image-20220526191051986](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910072.png)

- 测试

```java
        Thread consumer1 = new Thread(new ConsumerOne(),"消费者1");
        Thread consumer2 = new Thread(new ConsumerTwo(),"消费者2");
        Thread publisher = new Thread(new Publisher(),"生产者");
        consumer1.start();
        consumer2.start();
        publisher.start();
```

![image-20220526191045087](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910173.png)





****



### RPC模式

[RabbitMQ tutorial - Remote procedure call (RPC) — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-six-java.html)

RPC就是跨进程通信，之前列举的模式，都是生产者发送消息，消费者进行消息的模式，即单向通信模式。

如果想要实现消费者接受到消息后，还能将结果返回给生产者这种双向通信模式该怎么办呢？

不怕，Rabbitmq已经为我们准备好了，下面来看看吧:

![img](https://www.rabbitmq.com/img/tutorials/python-six.png)



在RabbitMQ上做RPC很容易, 客户端发送请求消息，服务器使用响应消息进行回复。为了接收响应，我们需要随请求一起发送“回调”队列地址。我们可以使用默认队列。

```java
            //生成一个临时队列
            String queue = channel.queueDeclare().getQueue();
            AMQP.BasicProperties basicProperties = new AMQP.BasicProperties.Builder()
                    //指明回调队列的地址
                    .replyTo(queue).build();
            channel.basicPublish("","rpc_queue",basicProperties,"rpc调用".getBytes(StandardCharsets.UTF_8));
```

> 消息属性:
>
> AMQP 0-9-1 协议预定义了一组与消息一起提供的 14 个属性。大多数属性很少使用，但以下属性除外：
>
> - deliveryMode：将消息标记为持久性（值为 2）或非持久化（任何其他值）。
> - contentType：用于描述编码的 mime 类型。例如，对于常用的 JSON 编码，最好将此属性设置为：application/json。
> - replyTo：通常用于命名回调队列。
> - correlationId：用于将 RPC 响应与请求相关联。

****

##### Correlation Id

为每个 RPC 请求创建一个回调队列。这是非常低效的，更好的方法是为每个客户端创建一个回调队列。

这引发了一个新问题，在该队列中收到响应后，不清楚响应属于哪个请求。这就是使用 correlationId 属性的时候。我们将为每个请求将其设置为唯一值。稍后，当我们在回调队列中收到消息时，我们将查看此属性，并基于此，我们将能够将响应与请求进行匹配。如果我们看到一个未知的 correlationId 值，我们可以安全地丢弃该消息 - 它不属于我们的请求。

您可能会问，为什么我们应该忽略回调队列中的未知消息，而不是失败并出现错误？这是由于服务器端可能存在争用条件。尽管不太可能，但 RPC 服务器可能会在向我们发送答案后立即死亡，但在为请求发送确认消息之前。如果发生这种情况，重新启动的 RPC 服务器将再次处理该请求。这就是为什么在客户端上，我们必须优雅地处理重复的响应，并且 RPC 在理想情况下应该是幂等的。



****

**Rpc具体工作流程如下:**

- 对于 RPC 请求，客户端发送一条具有两个属性的消息：replyTo（设置为仅为请求创建的匿名独占队列）和 correlationId（设置为每个请求的唯一值）。
- 请求将发送到rpc_queue队列。
- RPC 工作线程（也称为：服务器）正在等待该队列上的请求。当出现请求时，它会执行作业，并使用 replyTo 字段中的队列将包含结果的消息发送回客户端。
- 客户端等待回复队列上的数据。当出现一条消息时，它会检查 correlationId 属性。如果它与请求中的值匹配，则会将响应返回到应用程序。

****

**实际使用演示:**

- client端

```java
/**
 * 发送请求到服务端,然后接受到服务器发回的响应数据
 */
@Slf4j
public class Client implements Runnable{
    final BlockingQueue<String> response = new ArrayBlockingQueue<>(1);
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel client = rabbitmqUtil.createChannel();
            String rpcQueue = client.queueDeclare().getQueue();
            log.info("rpc队列名为: {}",rpcQueue);
            String uid = UUID.randomUUID().toString();
            AMQP.BasicProperties props = new AMQP.BasicProperties().builder()
                    //发送的消息关联的唯一ID
                    .correlationId(uid)
                    //消息发送到哪个队列
                    .replyTo(rpcQueue).
                    build();
             //使用默认交换机,路由key为rpc,路由失败会调用消息回退接口
            client.basicPublish("","rpc_queue",true,props,"rpc".getBytes(StandardCharsets.UTF_8));
            //监听rpc队列中回复的消息--返回的是当前消费者的标签
            String ctag = client.basicConsume(rpcQueue, true, (consumerTag, delivery) -> {
                if (delivery.getProperties().getCorrelationId().equals(uid)) {
                    response.offer(new String(delivery.getBody()));
                }
            }, consumerTag -> {
            });

            //阻塞取,直到队列中存在消息
            String res = response.take();
            //取消当前消费者---临时队列会被删除
            client.basicCancel(ctag);
            log.info("拿到rpc返回的结果: {}",res);
        } catch (IOException | TimeoutException | InterruptedException e) {
            log.error("出现异常: ",e);
        }
    }
}
```

- server端

```java
@Slf4j
public class Server implements Runnable{

    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel server = rabbitmqUtil.createChannel();
            //声明客户端和服务器之间通信的队列,这个不是回调队列
            server.queueDeclare(getRpc_queue(), false, false, false, null);
            //清空队列中的消息
            server.queuePurge(getRpc_queue());
            //最多只接收一条消息
            server.basicQos(1);
            //消息rpc_queue队列中客户端发送来的请求
            server.basicConsume(getRpc_queue(),false,
                    //消费队列中的消息
                    (consumerTag, delivery) ->{
                //准备一会要回复的属性
                AMQP.BasicProperties props = new AMQP.BasicProperties().builder()
                                //发送的消息关联的唯一ID
                                .correlationId(delivery.getProperties().getCorrelationId())
                                .build();
                        String reply = new String(delivery.getBody());
                        log.info("客户端回复消息为: {}",reply);
                        if(reply.equals("rpc")){
                            //进行回复
                            server.basicPublish("",delivery.getProperties().getReplyTo(),props,"respect!!!".getBytes(StandardCharsets.UTF_8));
                            //对客户端发送的消息做出回应
                            server.basicAck(delivery.getEnvelope().getDeliveryTag(),false);
                        }
                        },consumerTag -> {});
        } catch (IOException | TimeoutException e) {
            e.printStackTrace();
        }
    }

    private String getRpc_queue() {
        return "rpc_queue";
    }
}
```

- 测试

```java
        Thread client = new Thread(new Client());
        Thread server=new Thread(new Server());
        client.start();
        server.start();
```

![image-20220526191034156](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910242.png)

****

##### 队列排他性问题

**如果仔细阅读了上面文章的童鞋，此时可能会有一个疑问，明明临时队列是当前客户端独占的，为什么server这边还能向该临时队列放入消息呢？**

- 排他队列是基于连接可见的，同一个连接的不同信道是可以同时访问同一连接创建的排他队列的。
- RabbitMQ会自动删除这个队列，而不管这个队列是否被声明成持久性的（Durable =true)。 也就是说即使客户端程序将一个排他性的队列声明成了Durable的，只要调用了连接的Close方法或者客户端程序退出了，RabbitMQ都会删除这个队列。注意这里是连接断开的时候，而不是通道断开。这个其实前一点保持一致，只区别连接而非通道。

**之所以会有这个疑问，是因为没搞清楚rabbitmq的整体涉及，上面我们声明的是默认交换机，即server端向默认交换机发送了消息，然后携带了routekey,由默认交换机将消息转交给这个临时队列**



****



### 发布确认模式

**更多可以细节可以参考官方文档:**

[RabbitMQ tutorial - Reliable Publishing with Publisher Confirms — RabbitMQ](https://www.rabbitmq.com/tutorials/tutorial-seven-java.html)

[发布者确认 ](https://www.rabbitmq.com/confirms.html)

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消息都将会被指派一个唯一的 ID(从 1 开始)，**一旦消息被投递到所有匹配的队列之后，broker就会发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了。**

> 这里是消息被成功发送到交换机后，就会告诉生产者，消息发送成功了，而不是消费者确认消息后再进行回调

如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，**如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用程序同样可以在回调方法中处理该 nack 消息**

> 这种确认机制可以类比tcp的消息确认思想，异步回调是一种常用的实现异步方式的思想

****

#### 开启发布确认

发布确认默认是没有开启的，如果要开启需要调用方法 confirm.Select，每当你要想使用发布角认，都需要在channel上调用该方法

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```

****

#### 单个发布确认

- 这是一种简单的确认方式，它是一种同步确认发布的方式，`也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布`, `waitForConfirmsOrDie(long)这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常`

- 这种确认方式有一个最大的缺点就是：`发布速度特别的慢`.

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            //mandatory为true时,消息路由失败,会回调消息回退接口
            channel.addReturnListener(new RouteFailListener());
            //开启发布确认
            channel.confirmSelect();
            long begin = System.currentTimeMillis();
            //单个发布确认
            for (int i = 0; i < 100; i++) {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,true, null,("序号"+i).getBytes(StandardCharsets.UTF_8));
                // 单个消息马上进行发布确认
                boolean flag = channel.waitForConfirms();
                if (flag){
                   log.info("消息发送成功");
                }
            }
           log.info("总耗时: {}",System.currentTimeMillis()-begin);
        } catch (IOException | TimeoutException | InterruptedException e  ) {
            log.error("出现异常: ",e);
        }
    }
}
```



![image-20220526191027062](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910143.png)



****

#### 批量发布确认

上面那种方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的缺点就是:当发生故障导致发布出现问题时，不知道是哪个消息出现问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            //mandatory为true时,消息路由失败,会回调消息回退接口
            channel.addReturnListener(new RouteFailListener());
            //开启发布确认
            channel.confirmSelect();
            long begin = System.currentTimeMillis();
            //单个发布确认
            for (int i = 0; i < 100; i++) {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,true, null,("序号"+i).getBytes(StandardCharsets.UTF_8));
                if (i%10 == 0){
                    channel.waitForConfirms();
                }
            }
           log.info("总耗时: {}",System.currentTimeMillis()-begin);
        } catch (IOException | TimeoutException | InterruptedException e  ) {
            log.error("出现异常: ",e);
        }
    }
}
```



![image-20220526191019366](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910442.png)



****



#### 异步确认

异步确认是通过其他线程回调确认接口实现的，实现也比较简单，并且不会阻塞当前正在运行的线程，推荐使用。

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            //mandatory为true时,消息路由失败,会回调消息回退接口
            channel.addReturnListener(new RouteFailListener());
            //开启发布确认
            channel.confirmSelect();

            // 消息确认成功回调函数
            /*
             * 参数1：消息的标记
             * 参数2：是否为批量确认
             * */
            ConfirmCallback ackCallback = (deliveryTag, multiply) -> {
                System.out.println("确认的消息："+deliveryTag);
                System.out.println("是否为批量确认：  " + (multiply?"YES":"NO"));
            };

            // 消息确认失败回调函数--nack-ed的消息(客户端拒绝接受，或者路由失败的消息)
            ConfirmCallback nackCallback = (deliveryTag, multiply) -> {
                System.out.println("未确认的消息："+deliveryTag);
                System.out.println("是否为批量确认：  " + (multiply?"YES":"NO"));
            };

            // 准备消息的监听器，监听哪些消息成功，哪些消息失败
            /*
             * 参数1：监听哪些消息成功
             * 参数2：监听哪些消息失败
             * */
            channel.addConfirmListener(ackCallback,nackCallback);

            long begin = System.currentTimeMillis();
            //单个发布确认
            for (int i = 0; i < 100; i++) {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,true, null,("序号"+i).getBytes(StandardCharsets.UTF_8));
            }
           log.info("总耗时: {}",System.currentTimeMillis()-begin);
        } catch (IOException | TimeoutException e  ) {
            log.error("出现异常: ",e);
        }
    }
}
```





![image-20220526191011227](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910309.png)



****



#### 如何处理异步未确认消息

**这里给出的是官方文档中介绍的一种解决方法，大家可以参考，也可以自己大开脑洞**

最好的解决方案就是把未确认的消息放到一个基于内存的能被发布线程访问的队列，比如说用`ConcurrentLinkedQueue这个队列在confirm callbacks与发布线程之间进行消息的传递`

消息发布的同时，将消息都记录到一个高并发的哈希表中----->在监听器的成功回调函数中，从哈希表中删除成功发送的消息---->在监听器的失败回调函数中，可以通过指定key,得到发送失败的消息



```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.prepareChannel();
            //mandatory为true时,消息路由失败,会回调消息回退接口
            channel.addReturnListener(new RouteFailListener());
            //开启发布确认
            channel.confirmSelect();

            /*
             * 线程安全有序的一个哈希表 适用于高并发的情况下
             * 1、轻松地将序号与消息进行关联
             * 2、轻松地批量删除，只要给到序号
             * 3、支持高并发
             * */
            ConcurrentSkipListMap<Long,String> outstandingConfirms = new ConcurrentSkipListMap<>();

            // 消息确认成功回调函数
            /*
             * 参数1：消息的标记
             * 参数2：是否为批量确认
             * */
            ConfirmCallback ackCallback = (deliveryTag, multiply) -> {
                // 删除到已经确认的消息，剩下的就是未确认的消息

                //如果消息是批量发送的情况下，这里就要采用批量删除
                if(multiply)
                {
                    ConcurrentNavigableMap<Long, String> confiremed = outstandingConfirms.headMap(deliveryTag);
                    confiremed.clear();
                }else {
                    //否则删除单个即可
                    outstandingConfirms.remove(deliveryTag);
                }
                System.out.println("确认的消息："+deliveryTag);
                System.out.println("是否为批量确认：  " + (multiply?"YES":"NO"));
            };

            // 消息确认失败回调函数
            ConfirmCallback nackCallback = (deliveryTag, multiply) -> {
                // 打印一下未确认的消息都有哪些
                String message = outstandingConfirms.get(deliveryTag);
                System.out.println("未确认的消息是：" + message +"未确认的消息tag：" + deliveryTag);
            };

            // 准备消息的监听器，监听哪些消息成功，哪些消息失败
            /*
             * 参数1：监听哪些消息成功
             * 参数2：监听哪些消息失败
             * */
            channel.addConfirmListener(ackCallback,nackCallback);

            long begin = System.currentTimeMillis();
            //单个发布确认
            for (int i = 0; i < 100; i++) {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,true, null,("序号"+i).getBytes(StandardCharsets.UTF_8));

                // 此处记录下所有要发送的消息的总和
                outstandingConfirms.put(channel.getNextPublishSeqNo(),"序号"+i);
            }
           log.info("总耗时: {}",System.currentTimeMillis()-begin);
        } catch (IOException | TimeoutException e  ) {
            log.error("出现异常: ",e);
        }
    }
}

```

> 避免在失败的回调接口中尝试通过通道对消息进行重新发布，因为回调是在另外的 I/O 线程中调度的，其中通道不应该执行操作。更好的解决方案是将消息排队到由发布线程轮询的内存中队列中。像 ConcurrentLinkedQueue 这样的类是确认回调和发布线程之间传输消息的良好候选者。

****

#### 确认成功和失败的依据

首先给出结论:

- 当消息发送到交换机就会通知生产者消息发送成功,下面给出验证:

![image-20220526191001228](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261910351.png)

- 当交换机不存在时，向交换机发消息会直接抛出异常，或者网络拥塞情况下，当指定时间内，未能成功将消息送到交换机手里，收到交换机的确认信息，也会抛出异常。

![image-20220526190955457](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261909540.png)



- 异步成功是直接回调成功接口，这里不进行测试了，异步失败回调接口会在消息未能被成功发送到交换机时触发，一般不会触发。

> 如果出现交换机不存在这种情况，是属于通道级异常，当前通道直接被关闭，并不会回调失败接口

![image-20220526190948623](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261909706.png)

发送通道级异常，服务器端会发送Basic.Cancel，会回调相关cancel接口

![image-20220526190942731](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261909835.png)

****



## 进阶玩法

进阶玩法这里只会讲一部分，更多的特性大家可以参考官方文档中协议扩展部分内容:

[Protocol Extensions — RabbitMQ](https://www.rabbitmq.com/extensions.html)

****

### 消息过期时间

[Time-To-Live and Expiration — RabbitMQ](https://www.rabbitmq.com/ttl.html)

RabbitMQ 允许您为消息和队列设置 TTL（生存时间）。这由[可选的队列参数控制](https://www.rabbitmq.com/queues.html)，最好使用[策略](https://www.rabbitmq.com/parameters.html)来完成。

消息 TTL 可以应用于单个队列、一组队列或逐条消息应用。

TTL 设置也可以由[操作员策略](https://www.rabbitmq.com/parameters.html#operator-policies)强制执行。

> 策略设置是指通过命令行设置或者web界面进行设置，这里我不会进行介绍，感兴趣可以自己点开链接，看一下官方文档的相关介绍

****

#### 单队列消息ttl

通过使用[策略](https://www.rabbitmq.com/parameters.html#policies)设置 message-ttl 参数或在队列声明时指定相同的参数，可以为给定队列设置消息 TTL。

在队列中停留的时间比配置的 TTL 更长的时间的消息称为死信消息。请注意，路由到多个队列的消息可以在它所在的每个队列中的不同时间死亡，或者根本不死亡。一个队列中消息的死亡不会影响其他队列中同一消息的生存期。

服务器保证死消息不会使用 basic.deliver 传递（传递给使用者）或包含在 basic.get-ok 响应中（用于一次性提取操作）。此外，服务器将尝试在基于 TTL 的消息到期时或之后不久删除邮件。

TTL 参数或策略的值必须是非负整数 （0 <= n），以**毫秒为单位描述 TTL 周期**。因此，值为 1000 表示添加到队列的消息将在队列中保留 1 秒，或者直到它被传递给使用者。参数可以是 AMQP 0-9-1 类型的short-short-int, short-int, long-int, 或者 long-long-int.

- **声明一个队列，其中的消息最多可以保存1秒**

```java
            HashMap<String, Object> arguments = new HashMap<>();
            arguments.put("x-message-ttl",1000);
            channel.queueDeclare(Q2_QUEUE,false,false,false,arguments);
```

可以将消息 TTL 策略应用于其中已有消息的队列，但这涉及[一些注意事项](https://www.rabbitmq.com/ttl.html#per-message-ttl-caveats)。

如果消息重新排队（例如，由于使用了具有重新排队参数的 AMQP 方法，或者由于通道关闭），则会保留消息的原始到期时间。

将 TTL 设置为 0 会导致消息在到达队列时过期，除非它们可以立即传递给使用者。因此，如果设置了死信交换，则消息将是死信。

****

#### 单条消息ttl

通过在发送 [basic.publish](https://www.rabbitmq.com/amqp-0-9-1-reference.html#basic.publish) 时设置过期字段，可以基于每条消息指定 TTL。

过期字段的值描述 TTL 周期（以毫秒为单位）。适用与 x-message-ttl 相同的约束。由于过期字段必须是字符串，因此代理将（仅）接受数字的字符串表示形式。

**当同时指定了每队列和每条消息 TTL 时，将选择两者之间的较低值。**

- 发布一条消息，该消息最多可以驻留在队列中 60 秒：

```java
byte[] messageBodyBytes = "Hello, world!".getBytes();
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                                   .expiration("60000")
                                   .build();
channel.basicPublish("my-exchange", "routing-key", properties, messageBodyBytes);
```

****



#### 注意事项

- 只有当过期的消息到达队列的头部时，它们才会被丢弃（或死信）
- 消息过期和使用者传递之间可能存在自然的争用条件，例如，消息在写入套接字后但在到达使用者之前可能会过期。
- 设置每条消息时，TTL 过期的消息可能会在未过期的消息后面排队，直到后者被使用或过期。因此，此类过期消息使用的资源将不会被释放，并且它们将被计入队列统计信息（例如队列中的消息数）。
- 如果不设置 TTL，表示消息永远不会过期。
- 如果将 TTL 设置为 0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被死信或丢弃。



> 如果设置了单队列消息的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)，
>
> 设置单条消息的ttl,消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间；



****



#### 队列ttl

TTL 也可以在队列上设置，而不仅仅是队列内容。队列只有在未使用（例如，没有使用者）时才会在一段时间后过期。此功能可与[自动删除队列属性](https://www.rabbitmq.com/queues.html)一起使用。

通过在队列声明时，通过x-expires设置 或设置 expires [策略](https://www.rabbitmq.com/parameters.html#policies)，可以为给定队列设置过期时间。这控制队列在被自动删除之前可以未使用多长时间。

> “未使用”表示队列没有使用者，队列最近未被重新声明（重新声明会续订租约），并且至少在过期期限内未调用 basic.get。
>
> 例如，这可以用于 RPC 样式的应答队列，其中可以创建许多可能永远不会被耗尽的队列。

服务器保证队列将被删除，如果队列未被使用时长超过过期时间设置。

但是不保证在过期后队列将以多快的速度被删除。

当服务器重新启动时，持久队列的租约将重新启动。

> 过期时间单位为毫秒，并且它必须是正整数（与消息 TTL 不同，它不能为 0）



- 该队列在未使用 30 分钟后过期。

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-expires", 1800000);
channel.queueDeclare("myqueue", false, false, false, args);
```

****

### 死信队列

[Dead Letter Exchanges — RabbitMQ](https://www.rabbitmq.com/dlx.html)

-  死信，顾名思义就是无法被消费的消息，字面意思可以这样理解，一般来说，producer将消息投递到 broker或者直接到queue里了,consumer 从 queue取出消息进行消费，但某些时候由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的处理，就变成了死信，有死信自然就有了死信队列。
- 应用场景：为了保证订单业务的消息数据不丢失，需要使用到RabbitMQ的死信队列机制，当消息消费发生异常时，将消息投入死信队列中.还有比如说:用户在商城下单成功并点击去支付后在指定时间未支付时自动失效

****

#### 死信的来源

1. 消息TTL过期
2. 队列达到最大长度（队列满了，无法再添加数据到mq中）
3. 消息被拒绝（basic.reject或basic.nack）并且requeue=false

> 请注意，队列过期不会对其中的消息进行死信。

****

#### 死信处理的方式

- 丢弃，如果不是很重要，可以选择丢弃
-  记录死信入库，然后做后续的业务分析或处理
-  通过死信队列，由负责监听死信的应用程序进行处理

![image-20220526190926955](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261909070.png)

****

#### 配置死信队列

-  配置业务队列，绑定到业务交换机上
-  为业务队列配置死信交换机和路由key
-  为死信交换机配置死信队列

**`注意，并不是直接声明一个公共的死信队列，然后所有死信消息就自己跑到死信队列里去了`**。而是为每个需要使用死信的业务队列配置一个死信交换机，这里同一个项目的死信交换机可以共用一个，然后为每个业务队列分配一个单独的路由key。

**`有了死信交换机和路由key后，接下来，就像配置业务队列一样，配置死信队列，然后绑定在死信交换机上。`**

也就是说，死信队列并不是什么特殊的队列，只不过是绑定在死信交换机上的队列。

死信交换机也不是什么特殊的交换机，只不过是用来接受死信的交换机，所以可以为任何类型【Direct、Fanout、Topic】。

一般来说，会为每个业务队列分配一个独有的路由key，并对应的配置一个死信队列进行监听，也就是说，一般会为每个重要的业务队列配置一个死信队列。

> 死信交换机和死信队列就是普通的交换机和普通的队列，只不过会在声明普通队列时，指定该队列上的死信消息会转发给哪个交换机进行处理器，而该交换机，我们就称为死信交换机
>
> 然后死信交换机再将消息转交给绑定到其上的死信队列进行消费

****

#### 实战

- publisher准备

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            declare(channel);
            //发送一条消息到队列中
            for (int i = 0; i < 10; i++) {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY,null,("dead message"+i).getBytes(StandardCharsets.UTF_8));
            }
        } catch (IOException | TimeoutException e  ) {
            log.error("出现异常: ",e);
        }
    }

    public void declare(Channel channel) throws IOException {
       //声明死信交换机
        channel.exchangeDeclare(DEAD_EXCHANGE,DIRECT,false,true,null);
        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE,false,false,true,null);
        //绑定死信交换机和死信队列
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,DEAD_KEY);

        //普通队列属性设置
        HashMap<String, Object> arguments = new HashMap<>();
        //设置当前普通队列关联的死信交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);
        //设置死信RoutingKey
        arguments.put("x-dead-letter-routing-key",DEAD_KEY);
        //设置队列中消息的存活时间--5s
        arguments.put("x-message-ttl",5000);

        //声明普通交换机
        channel.exchangeDeclare(EXCHANGE_NAME,DIRECT,false,true,null);
        //声明普通队列
        channel.queueDeclare(QUEUE_NAME,false,false,true,arguments);
        //绑定普通交换机和普通队列
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,ROUTING_KEY);
    }
}

```

- 死信队列的consumer准备

> 这里我们不提供普通队列的消费者，因为我们的目的是让普通队列中的消息无人消费，然后过期被放入死信队列中

```java
@Slf4j
public class DeadConsumer implements Runnable{
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            //直接进行消费的消息--所有声明操作已经在生产者部分完成
            channel.basicConsume(DEAD_QUEUE,false,(consumerTag, message) -> {
                log.info("消息为: {}",new String(message.getBody()));
                channel.basicAck(message.getEnvelope().getDeliveryTag(),true);
            },consumerTag -> {});
        } catch (IOException | TimeoutException e  ) {
            log.error("出现异常: ",e);
        }
    }
}
```

- 测试

```java
        Thread deadConsumer = new Thread(new DeadConsumer());
        Thread publisher = new Thread(new Publisher());
        publisher.start();
       //预留时间声明和创建交换机和队列
        Thread.sleep(2000);
        deadConsumer.start();
```

消息过期前:

![image-20220526190833950](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261908027.png)

消息过期后:

![image-20220526190827998](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261908084.png)

死信消息会携带x-death头信息，指明这个死信消息是如何产生的:

```java
log.info("头信息打印: {}",message.getProperties().getHeaders());
```

输出内容如下:

```java
头信息打印: {
    x-first-death-exchange=dhy-exchange, 
    x-death=[{
        reason=expired, count=1, exchange=dhy-exchange, time=Sat May 21 21:38:45 CST 2022, 
        routing-keys=[dhy], queue=dhy-queue
             }], 
    x-first-death-reason=expired, 
    x-first-death-queue=dhy-queue
     }
```

**关于reson有四个取值,分别如下:**

- rejected: the message was rejected with requeue parameter set to false
- expired: the [message TTL](https://www.rabbitmq.com/ttl.html) has expired
- maxlen: the [maximum allowed queue length](https://www.rabbitmq.com/maxlength.html) was exceeded
- delivery_limit: the message has been returned more times than the limit (set by policy argument [delivery-limit](https://www.rabbitmq.com/quorum-queues.html#poison-message-handling) of quorum queues).

****



#### AMQP.BasicProperties

**这里主要是做一个说明:**

```java
String contentType, //消息内容的类型
String contentEncoding, //消息内容的编码格式
Map<String,Object> headers,//header类型的交换机可以用到
Integer deliveryMode,//消息持久化 1 不持久化 2 持久化
Integer priority,//优先级
String correlationId, //关联id
String replyTo,//通常用于命名回调队列
String expiration,//设置过期消息过期时间
String messageId, //消息id
Date timestamp, //消息的时间戳
String type,  //类型
String userId, //用户ID
String appId, //应用程序id
String clusterId //集群id	
```

****

#### 队列消息最大个数限制

[Queue Length Limit — RabbitMQ](https://www.rabbitmq.com/maxlength.html)

我们可以使用命令行或者编码方式设置队列消息的最大个数限制，这里我只讲编码，命令行大家可以参考官方文档，那里写的非常详细。



**默认队列消息个数溢出的行为：**

默认队列是没有消息个数的限制的，但是如果我们设置了，那么一但队列滞留消息个数超过限制，会将队列头部的消息丢弃或者死信(选择的依据在于是否存在死信交换机)

> 丢弃头部消息的原因是因为头部消息是最旧的，新消息会被放入到队列尾部
>
> 在所有情况下，都使用**处于就绪**状态的消息数;[使用者未确认的消息](https://www.rabbitmq.com/confirms.html)不计入限制。
>
> 我们可以自定义溢出策略，这里没讲，大家可以看上面给出的官方文档链接

- 设置当前队列最多同时保存10个消息

```java
Map<String, Object> args = new HashMap<String, Object>();
//max-length-bytes可以设置当前队列最多保存的消息数量总共字节数--具体参考官方文档
//overflow可以用来设置溢出策略--具体参考官方文档
args.put("x-max-length", 10);
channel.queueDeclare("myqueue", false, false, false, args);
```

****

### 优先级队列

[Priority Queue Support — RabbitMQ](https://www.rabbitmq.com/priority.html)

> 优先级队列会对队列中的消息，按照优先级大小进行排序，因此被称为优先级队列

任何队列都可以使用客户端提供的[可选参数](https://www.rabbitmq.com/queues.html#optional-arguments)转换为优先级队列(不能使用策略动态调整，下面会讲),默认支持的最大优先级为255，官方推荐范围是0-10。

****

#### 设置队列优先级

- x-max-priority: 参数应为介于 1 和 255 之间的正整数，指示队列应支持的最大优先级

```java
Channel ch = ...;
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-max-priority", 10);
ch.queueDeclare("my-priority-queue", true, false, false, args);
```

然后发布者在发布消息的时候，可以通过下面的方式指定消息的优先级:

```java
            Channel channel = rabbitmqUtil.createChannel();
            AMQP.BasicProperties basicProperties = new AMQP.BasicProperties.Builder()
                   //指定当前发送的消息的优先级
                    .priority(5)
                    .build();
            channel.basicPublish("",QUEUE_NAME,basicProperties,"hello".getBytes(StandardCharsets.UTF_8));
```

****



#### 注意事项

- 每个队列的每个优先级都有一些内存中和磁盘上成本。还有额外的CPU成本，尤其是在进行消费时，因此最好不要把队列优先级范围设置的太大。
- 没有优先级属性的消息将被视为优先级为 0。优先级高于队列最大值的消息将被视为以最大优先级发布。



官方推荐:

> 如果需要优先级队列，我们建议使用介于 1 和 10 之间的队列。目前使用更多的优先级会通过使用更多的 Erlang 进程来消耗更多的 CPU 资源。[运行时计划](https://www.rabbitmq.com/runtime.html)也会受到影响。

****

#### 消费者与优先级队列的交互事项

默认情况下，使用者在确认任何消息之前可能会收到大量消息，仅受网络背压的限制。

因此，如果这样一个饥肠辘辘的使用者连接到一个空队列，消息随后将发布到该队列中，则消息可能根本不会在队列中等待任何时间。在这种情况下，优先级队列将没有任何机会对它们进行优先级排序。

在大多数情况下，您希望在使用者的手动确认模式下使用 basic.qos 方法，以限制可以随时发出以进行传递的消息数，从而允许对消息进行优先级排序。

****

#### 消息过期和队列最大长度

- 消息只有到达消息头部队列才会知道其是否过期，因此普通队列不同，即使是是设置了单队列消息 TTL 也可能导致过期的低优先级消息卡在未过期的高优先级消息后面。这些消息将永远不会传递，但它们将显示在队列统计信息中。

- [像往常一样，设置了最大长度的队列](https://www.rabbitmq.com/maxlength.html)将从队列的头部丢弃消息以强制实施限制。这意味着可能会丢弃较高优先级的消息，以便为优先级较低的消息让路。

****

#### 为什么不支持策略动态调整

为队列定义可选参数的最方便方法是通过[策略](https://www.rabbitmq.com/parameters.html)。策略是配置 [TTL](https://www.rabbitmq.com/ttl.html)、[队列长度限制](https://www.rabbitmq.com/maxlength.html)和其他[可选队列参数的](https://www.rabbitmq.com/queues.html)推荐方法。

但是，策略不能用于配置优先级，因为策略是动态的，可以在声明队列后进行更改。优先级队列在队列声明后永远无法更改它们支持的优先级数，因此使用策略不是一个安全的选项。

****

### 消费者优先级

[Consumer Priorities — RabbitMQ](https://www.rabbitmq.com/consumer-priority.html)

使用者优先级允许您确保高优先级使用者在处于活动状态时接收消息，而当高优先级使用者阻塞时，消息才会发送给较低优先级的使用者。

通常，连接到队列的活动使用者以轮循机制方式从该队列接收消息。当使用使用者优先级时，如果存在多个具有相同高优先级的活动使用者，则以轮循机制传递消息.

****

#### 活跃消费者

活跃消费者是无需等待即可接收消息的消费者。如果消费者无法接收消息，则消费者将被阻止 - 因为其通道在发出 basic.qos 后已达到未确认消息的最大数量，或者仅仅是因为网络拥塞。

****

#### 设置消费者优先级

```java
Channel channel = ...;
Consumer consumer = ...;
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-priority", 10);
channel.basicConsume("my-queue", false, args, consumer);
```

****

### 备份交换机

[Alternate Exchanges — RabbitMQ](https://www.rabbitmq.com/ae.html)

**mandatory**

- TRUE: 消息路由到队列失败,调用消息return接口
- FALSE: 消息路由到队列失败,尝试将消息转发给兜底交换机

mandatory为false的时候，尝试将消息转发给兜底交换机，这里兜底交换机也就是下面我们要聊的备份交换机。

- 为什么要有备份交换机存在?

有了mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。

但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。

而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。

而且设置mandatory参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。

如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢?

**`前面在设置死信队列时我们提到，可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。`**

**`在RabbitMQ.中，有一种备份交换机的机制存在，可以很好的应对这个问题。`**

什么是备份交换机呢?

备份交换机可以理解为 RabbitMQ中交换机的“备胎”，当我们**为某一个交换机声明一个对应的备份交换机时**，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为Fanout，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进入这个队列了。

> 当备份交换机接收到无法路由的消息后，我们可以通过给该备份交换机绑定一个备份队列和一个报警队列，分别负责保存和向运维人员报警这些消息

****

#### 使用

> 备份交换机相当于某一个交换机的备份，因此当某个交换机需要一个备份交换机的时候，我们需要在该交换机声明的时候，通过参数告诉该交换机，与之关联的备份交换机的名字。

- publisher

```java
@Slf4j
public class Publisher implements Runnable {
    @Override
    public void run() {
        try {
            RabbitmqUtil rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();
            Map<String, Object> args = new HashMap<String, Object>();
            //通过属性指定关联的备份交换机名字为my-ae
            args.put("alternate-exchange", "my-ae");
            //my-direct的备份交换机名字为my-ae
            channel.exchangeDeclare("my-direct", "direct", false, false, args);
            //my-ae一般为扇形交换机
            channel.exchangeDeclare("my-ae", "fanout");
            //给备份交换机绑定一个队列
            channel.queueDeclare("temp",false,false,true,null);
            channel.queueBind("temp","my-ae","temp");
            channel.addReturnListener(new RouteFailListener());
            //设置manatory为true
            channel.basicPublish("my-direct", "no", true, null, "hello".getBytes(StandardCharsets.UTF_8));
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ", e);
        }
    }
}
```

- temp队列的consumer

```java
@Slf4j
public class ConsumerOne implements Runnable{
    @Override
    public void run() {
        RabbitmqUtil rabbitmqUtil = null;
        try {
            rabbitmqUtil = new RabbitmqUtil("application.yml");
            Channel channel = rabbitmqUtil.createChannel();

            channel.basicConsume("temp",true,new DefaultConsumer(channel){
                @SneakyThrows
                @Override
                public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                    log.info("消息为: {}",new String(body));
                }
            });
        } catch (IOException | TimeoutException e) {
            log.error("出现异常: ",e);
        }
    }
}
```

- 测试

![image-20220526190814184](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261908250.png)



备份交换机拿到了无法路由的消息

****

#### 工作流程

当交换机遇到路由失败的消息时，会首先尝试去需求备份交换机的帮助，如果存在就将消息转交给备份交换机，否则记录警告日志或者调用消息回退接口(manatory为true).

如果备份交换机也无法完成消息路由，那么会去寻找当前备份交换机的备份交换机。

****

#### mandatory和备份交换机听谁的?

上面的测试已经告诉我们答案了，听备份交换机的。

****

### 消息追踪

在使用任何消息中间件的过程中，难免会出现某条消息异常丢失的情况。对于RabbitMQ而言，可能是因为生产者或消费者与RabbitMQ断开了连接，而它们与RabbitMQ又采用了不同的确认机制；也有可能是因为交换器与队列之间不同的转发策略；甚至是交换器并没有与任何队列进行绑定，生产者又不感知或者没有采取相应的措施；另外RabbitMQ本身的集群策略也可能导致消息的丢失。这个时候就需要有一个较好的机制跟踪记录消息的投递过程，以此协助开发和运维人员进行问题的定位。

****

#### FireHose

在RabbitMQ中可以使用Firehose功能来实现消息追踪，Firehose可以记录每一次发送或者消费消息的记录，方便使用RabbitMQ的使用者进行调试、排错等。

Firehose的机制是将生产者投递给RabbitMQ的消息，或者是RabbitMQ投递给消费者的消息按照指定的格式发送到默认的交换器上。这个默认的交换器的名称为amq.rabbitmq.trace，它是一个topic类型的交换器。发送到这个交换器上的消息的routingKey为publish.exchangename和deliver.queuename。其中exchangename和queuename为实际的交换器和队列的名称，分别对应生产者投递到交换器的消息和消费者从队列中获取的消息。

- 注意: 打开trace会影响消息写入性能,适当打开后请关闭

```sh
#开启
rabbitmqctl trace_on [-p vhost] 
#关闭
rabbitmqctl trace_off [-p vhost]
```

> Firehose默认情况处于关闭状态，并且Firehose的状态也是非持久化的，会在RabbitMQ服务重启的时候还原成默认的状态。Firehose开启之后多少会影响服务的性能，因为它会引起额外的消息生成、路由和存储。

- 一旦我们开启了FireHose功能,那么我们每发送一条消息,默认的trace交换机便会记录一条日志消息,然后把该条日志消息发送到绑定了该交换机的队列上去(默认该交换机没有绑定任何队列)

![image-20220526190804413](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261908485.png)

![image-20220526190739665](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261907738.png)

​                                                          开启FireHose功能:

![image-20220526190725496](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261907566.png)

​                                                       自己创建一个logQueue绑定到内部使用的trace交换机上,队列路由key为#

![image-20220526190717944](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261907015.png)

![image-20220526190711169](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261907259.png)

****

#### rabbitmq_tracing插件

rabbitmq_tracing和Firehose在实现上一样，只不过rabbitmq_tracing的方式比firehose多了一层GUI的包装,更容易使用和管理

启用插件:

```sh
rabbitmq-plugins enable rabbitmq_tracing
```

![image-20220526190521645](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261905771.png)

- “Format”表示输出的消息日志格式，有Text和JSON两种，Text格式的日志方便人类阅读，JSON的方便程序解析。
- JSON格式的payload（消息体）默认会采用Base64进行编码，如上面的“trace test payload.”会被编码成“dHJhY2UgdGVzdCBwYXlsb2FkLg==”。
- “Max payload bytes”表示每条消息的最大限制，单位为B。比如设置了了此值为10，那么当有超过10B的消息经过RabbitMQ流转时，在记录到trace文件的时候会被截断。如上text日志格式中“trace test payload.”会被截断成“trace test”。
- "Pattern"用来设置匹配的模式，和Firehose的类似。如“#”匹配所有消息流入流出的情况，即当有客户端生产消息或者消费消息的时候，会把相应的消息日志都记录下来；“publish.#”匹配所有消息流入的情况；“deliver.#”匹配所有消息流出的情况。

****

### 惰性队列

RabbitMQ从 3.6.0版本开始引入了惰性队列的概念。惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多的消息存储。当消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了。

默认情况下，当生产者将消息发送到RabbitMQ的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当RabbitMQ需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的操作，进而无法接收新的消息。虽然 RabbitMQ的开发者们一直在升级相关的算法，但是效果始终不太理想，尤其是在消息量特别大的时候。

****

#### two mode

队列具备两种模式：default 和 lazy。默认的为 default 模式，在 3.6.0 之前的版本无需做任何变更。lazy模式即为惰性队列的模式，可以通过调用 channel.queueDeclare 方法的时候在参数中设置，也可以通过Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

在队列声明的时候可以通过“x-queue-mode”参数来设置队列的模式，取值为“default”和“lazy”。下面示例中演示了一个惰性队列的声明细节：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode","lazy");
channel.queueDeclare( "myqueue", false, false, false,args);
```

**内存对比**

在发送1百万务消息，每条消息大概占1KB的情况下，普通队列占用内存是1.2GB,而惰性队列仅仅占用1.5MB

****

## 整合springboot

- 创建Springboot项目，引入rabbitmq整合springboot的starter

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

- 配置连接信息

  ```yaml
  spring:
    rabbitmq:
      host: 服务器地址
      #通信端口
      port: 5672
      username: guest
      password: guest
      #如果没有修改的话,默认为/,可以不写
      virtual-host: /
  ```

- 添加队列

  ```java
  @Configuration
  public class RabbitConfig {
     public static final String QUEUE_NAME="order_queue";
  
      /**
       * 参数介绍:
       *   1.队列名
       *   2.队列持久化
       *   3.队列是否具有排他性,有排他性的队列只能被创建的Connection处理
       *   4.如果队列没有消费者,那么是否自动删除队列
       */
     @Bean
     public Queue queue(){
        return new Queue(QUEUE_NAME,true,false,false);
     }
  }
  ```

  

- 添加消费者监听队列

```java
@Component
@Slf4j
public class RabbitConsumerListener {
    @RabbitListener(queues = RabbitConfig.QUEUE_NAME)
    public void handlerMsg(String msg){
      log.info("接收到队列发送的消息: {}",msg);  
    }
}
```

- 启动项目

![image-20220526190322083](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261903168.png)

**客户端通过channel和RabbitMQ建立连接**

![image-20220526190316048](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261903136.png)

**测试放入消息到队列**

![image-20220526190250104](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261902209.png)

**查看消费者输出**

![image-20220526190218107](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261902177.png)

****



### 延迟队列

延时队列,队列内部是有序的，最重要的特性就体现在它的延时属性上，延时队列中的元素是希望在指定时间到了以后或之前取出和处理，简单来说，延时队列就是用来存放需要在指定时间被处理的元素的队列

****

#### 应用场景

1.订单在十分钟之内未支付则自动取消

2.新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。

3.用户注册成功后，如果三天内没有登陆则进行短信提醒。

4.用户发起退款，如果三天内没有得到处理则通知相关运营人员。

5.预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务

****

#### 实战

创建两个队列QA和QB，两者队列TTL分别设置为10S和40S，然后在创建一个交换机X和死信交换机Y，它们的类型都是direct，创建一个死信队列QD，它们的绑定关系如下：

![image-20220526190206227](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261902318.png)



- 常量准备

```jade
public class RabbitmqConstants {

    //--------------------EXCHANGE--------------------------

    public static final String X_EXCHANGE = "X";
    public static final String Y_DEAD_LETTER_EXCHANGE = "Y";

    //--------------------QUEUE--------------------------

    public static final String QUEUE_A = "QA";
    public static final String QUEUE_B = "QB";
    public static final String DEAD_LATTER_QUEUE = "QD";

    //------------------ROUTE_KEY-----------------------------

    public static final String ROUTE_KEY_A="XA";
    public static final String ROUTE_KEY_B="XB";
    public static final String ROUTE_KEY_DEAD="YD";
}
```

- 交换机和队列准备

```java
@Configuration
public class RabbitConfig {

   //--------------------EXCHANGE--------------------------

   @Bean("xExchange")
   public DirectExchange xExchange()
   {
      return new DirectExchange(X_EXCHANGE);
   }

   @Bean("yExchange")
   public DirectExchange yExchange()
   {
      return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
   }

   //--------------------QUEUE--------------------------

   @Bean("queueA")
   public Queue queueA(){
      Map<String, Object> arguments = new HashMap<>(3);
      //设置死信交换机
      arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
      //设置死信Routing-key
      arguments.put("x-dead-letter-routing-key",ROUTE_KEY_DEAD);
      //设置TTL 单位是ms ----该队列里面消息过期时间为10s
      arguments.put("x-message-ttl",10000);
      return QueueBuilder.nonDurable(QUEUE_A).withArguments(arguments).build();
   }


   @Bean("queueB")
   public Queue queueB(){
      Map<String, Object> arguments = new HashMap<>(3);
      //设置死信交换机
      arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
      //设置死信Routing-key
      arguments.put("x-dead-letter-routing-key",ROUTE_KEY_DEAD);
      //设置TTL 单位是ms ----该队列里面消息过期时间为40s
      arguments.put("x-message-ttl",40000);
      return QueueBuilder.nonDurable(QUEUE_B).withArguments(arguments).build();
   }


   @Bean("queueD")
   public Queue queueD(){
      return QueueBuilder.nonDurable(DEAD_LATTER_QUEUE).build();
   }

   //--------------------bind--------------------------

   @Bean
   public Binding queueABindingX(@Qualifier("queueA") Queue queueA,
                                 @Qualifier("xExchange") DirectExchange xExchange){
      //队列A和X交换机绑定
      return BindingBuilder.bind(queueA).to(xExchange).with(ROUTE_KEY_A);
   }


   @Bean
   public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                 @Qualifier("xExchange") DirectExchange xExchange){
      //队列B和x交换机绑定
      return BindingBuilder.bind(queueB).to(xExchange).with(ROUTE_KEY_B);
   }


   @Bean
   public Binding queueDBindingX(@Qualifier("queueD") Queue queueD,
                                 @Qualifier("yExchange") DirectExchange yExchange){
      //死信队列D和死信交换机绑定
      return BindingBuilder.bind(queueD).to(yExchange).with(ROUTE_KEY_DEAD);
   }

}
```

- 消费者准备

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = DEAD_LATTER_QUEUE)
    public void receiveD(Message message, Channel channel) throws Exception {
        String msg = new String(message.getBody());
        log.info("当前时间：{}，收到死信队列的消息：{}", new Date().toString(), msg);
    }
}
```



- 生产者准备

```java
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/ttl")
public class RabbitmqController {
    private final RabbitTemplate rabbitTemplate;

    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable String message){
        log.info("当前时间：{}，发送一条信息给两个TTL队列：{}",new Date().toString(),message);
        rabbitTemplate.convertAndSend(X_EXCHANGE,ROUTE_KEY_A,"消息来自TTL为10s的队列：" + message);
        rabbitTemplate.convertAndSend(X_EXCHANGE,ROUTE_KEY_B,"消息来自TTL为40s的队列：" + message);
    }
}
```

- 测试

![image-20220526190153426](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901507.png)

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息，然后被消费掉，这样一个延时队列就打造完成了

不过，如果这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有 10S 和 40S两个时间选项，如果需要一个小时后处理，那么就需要增加 TTL 为一个小时的队列，如果是预定会议室然后提前通知这样的场景，岂不是要增加无数个队列才能满足需求？

****

#### 优化

上面我们设置的是单队列消息ttl,其实为了方便自定义，可以设置每一条消息的过期时间。

**这里我们新增一个队列，是在生产者发出消息的时候，动态设置每条消息的ttl时间**

![image-20220526190145144](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901243.png)



- 新增常量

```java
  public static final String QUEUE_C = "QC";
  public static final String ROUTE_KEY_C = "XC";
```

- 新增配置

```java
   @Bean("queueC")
   public Queue queueC(){
      Map<String, Object> arguments = new HashMap<>(3);
      //设置死信交换机
      arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
      //设置死信Routing-key
      arguments.put("x-dead-letter-routing-key",ROUTE_KEY_DEAD);
      return QueueBuilder.nonDurable(QUEUE_C).withArguments(arguments).build();
   }

   @Bean
   public Binding queueCBindingX(@Qualifier("queueC") Queue queueC,@Qualifier("xExchange") DirectExchange xExchange){
      return BindingBuilder.bind(queueC).to(xExchange).with(ROUTE_KEY_C);
   }
```

- 增加一个单独自定义过期时间的请求

```java
    @GetMapping("sendExpirationMsg/{message}/{ttlTime}")
    public void sendMsg(@PathVariable String message,@PathVariable String ttlTime){
        log.info("当前时间：{}，发送一条时长{}毫秒TTL信息给队列QC：{}",
                TimeUtil.getCurTime(),ttlTime,message);
        rabbitTemplate.convertAndSend(X_EXCHANGE,ROUTE_KEY_C,message,msg->{
            msg.getMessageProperties().setExpiration(ttlTime);
            return msg;
        });
    }
```

- 测试

![image-20220526190137535](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901596.png)

看起来似乎没什么问题，但是在最开始的时候，就介绍过如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时“死亡“，因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列，如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行。



****

#### 插件实现延迟队列

上文中提到的问题，确实是一个问题，如果不能实现在消息粒度上的 TTL，并使其在设置的 TTL 时间及时死亡，就无法设计成一个通用的延时队列。那如何解决呢，接下来我们就去解决该问题。

在官网上下载 https://www.rabbitmq.com/community-plugins.html，下载rabbitmq_delayed_message_exchange 插件，然后解压放置到 RabbitMQ 的插件目录。



- 下载插件到linux服务器,然后解压

```sh
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.10.0/rabbitmq_delayed_message_exchange-3.10.0.ez
#如果下载速度比较慢,大家可以尝试使用下面的镜像连接进行下载
wget http://110.40.155.17/download/rabbitmq_delayed_message_exchange-3.10.0.ez
#解压
tar -zxvf 3.10.0.tar.gz
```

- 将插件拷贝到docker容器内rabbitmq安装目录下的 plgins 目录下

```sh
          #宿主机文件                                  #容器ID(可缩写)或者容器名:容器内插件目录
docker cp rabbitmq_delayed_message_exchange-3.10.0.ez cf:/plugins
```

- 执行下面命令让该插件生效，然后重启 RabbitMQ

```sh
docker exec -it cf rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

![image-20220526190128027](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901103.png)

- 重启rabbitmq服务

```sh
docker restart cf
```

![image-20220526190121762](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901838.png)

****

#### 延迟交换机使用

- 在没有使用延迟交换机之前:

![image-20220526190114609](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901693.png)

- 使用延迟交换机后:

![image-20220526190105069](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261901160.png)



> 如果运用延迟交换机的官方文档链接如下:

[rabbitmq/rabbitmq-delayed-message-exchange: Delayed Messaging for RabbitMQ (github.com)](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)

这里先简单给出原生API如何使用延迟交换机,然后下面会再给出springboot如何实现

> 延迟交换机的干啥的: 通过给每个消息指定延迟发送时间，延迟交换机拿到这些消息后，不会立刻将其路由到某个队列，而是先保存起来，然后等待消息的延迟时间结束后，再将消息发送到指定的队列中去

- 要使用延迟交换机，先将交换机类型声明为：`x-delayed-message`

```java
Map<String, Object> args = new HashMap<String, Object>();
//通过一个头信息去声明延迟交换机的具体类型--延迟交换机只是实现了一个消息延迟发送的机制
//具体是哪一种交换机的实现类型，还需要我们自己去选择好
args.put("x-delayed-type", "direct");
//x-delayed-message是延迟交换机的类型
channel.exchangeDeclare("my-exchange", "x-delayed-message", true, false, args);
```

- 发送消息时，通过头信息告诉插件延迟多长时间，然后发送我们的消息

```java
byte[] messageBodyBytes = "delayed payload".getBytes("UTF-8");
Map<String, Object> headers = new HashMap<String, Object>();
//通过在头信息中加入x-delay字段，告诉插件延迟5秒后,将这个消息路由到指定的队列去
headers.put("x-delay", 5000);
AMQP.BasicProperties.Builder props = new AMQP.BasicProperties.Builder().headers(headers);
channel.basicPublish("my-exchange", "", props.build(), messageBodyBytes);
```

> 注意: 如果发送消息的时候，头信息里面没有x-delay字段，那么交换机拿到消息后,会立刻发送该消息
>
> 通过x-delayed-type字段，我们可以在声明延迟交换机的时候指定其路由模式，并且该头信息是必须携带而且指向存在的交换机类型
>
> 延迟交换机可以理解为代理交换机，他额外提供对消息延迟发送的支持，然后会对已经存在的交换机进行包装，但是因为添加了额外的功能支持，必定会存在性能消耗。
>
> 因此如果不需要延迟消息发送的功能，那么建议不要使用延迟交换机。

- 该交换机是将消息持久化到硬盘中，因此使用该交换机会导致性能偏低,这点需要考虑。
- 该交换机只会尝试去发送一次消息，可能会导致发送失败的原因有消息路由的失败
- 该交换机不支持mandatory属性

****

**实战演示:**

在这里新增了一个队列 delayed.queue,一个自定义交换机 delayed.exchange，绑定关系如下:

![image-20220526190053458](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261900533.png)

- 常量准备

```java
public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
public static final String DELAYED_QUEUE_NAME = "delayed.queue";
public static final String DELAYED_ROUTING_KEY = "DELAY";
```

- 队列和交换机准备

```java
   @Bean("delayedQueue")
   public Queue delayedQueue(){
      return new Queue(DELAYED_QUEUE_NAME);
   };

   @Bean
   public CustomExchange delayedExchange(){

      Map<String, Object> arguments = new HashMap<>();
      //自定义交换机的类型
      arguments.put("x-delayed-type","direct");
      /*
       * 1.交换机的名称
       * 2.交换机的类型
       * 3.是否需要持久化
       * 4.是否需要自动删除
       * 5.其他的参数
       * */
      return new CustomExchange(DELAYED_EXCHANGE_NAME,
                                //延迟交换机类型
                                "x-delayed-message",
              false,false,arguments);
   }
   
   @Bean
   public Binding delayedQueueBindingDelayedExchange(@Qualifier("delayedQueue") Queue delayedQueue,
                                                     @Qualifier("delayedExchange") CustomExchange delayedExchange){
      return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
   }
```

- 消费者

```java
@Slf4j
@Component
public class DelayQueueConsumer {
    @RabbitListener(queues = DELAYED_QUEUE_NAME)
    public void recieveDelayQueue(Message message) {
        String msg = new String(message.getBody());
        log.info("当前时间：{}，收到延迟队列的消息：{}", TimeUtil.getCurTime(), msg);
    }
}
```

- 生产者

```java
    @GetMapping("/delay/{message}/{delayTime}")
    public void sendMsg(@PathVariable String message, @PathVariable Integer delayTime){
        log.info("当前时间：{}，发送一条时长{}毫秒信息给延迟队列delayed.queue：{}",
                TimeUtil.getCurTime(),delayTime,message);
        rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME
                ,DELAYED_ROUTING_KEY,message,msg -> {
                    // 发送消息的时候 延迟时长 单位ms
                    msg.getMessageProperties().setDelay(delayTime);
                    return msg;
                });
    }
```

- 测试

![image-20220526185954978](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859051.png)

可以看到，并不会因为先发送的消息延迟时间长，就阻塞后面短延迟消息的接收



****

### 发布确认模式

> 这里演示如何用springboot优雅实现发布确认模式

- 常量准备

```java
public class RabbitmqConstants {

    //--------------------EXCHANGE--------------------------
    
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";

    //--------------------QUEUE--------------------------

    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";

    //------------------ROUTE_KEY-----------------------------

    public static final String CONFIRM_ROUTING_KEY = "key1";

}
```

- 交换机和队列声明

```java
@Configuration
public class RabbitConfig {

   //--------------------EXCHANGE--------------------------

   @Bean
   public DirectExchange confirmExchange()
   {
       return new DirectExchange(CONFIRM_EXCHANGE_NAME);
   }

   //--------------------QUEUE--------------------------

    @Bean
    public Queue confirmQueue()
    {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

   //--------------------bind--------------------------

    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue confirmQueue,
                                        @Qualifier("confirmExchange")DirectExchange confirmExchange)
    {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }

}
```

- 消费者(次要,确认与否和消费者没关系，不清楚回看上面发布确认模式)

```java
@Slf4j
@Component
public class Consumer {
    
    @RabbitListener(queues = CONFIRM_QUEUE_NAME)
    public void receiveConfirmMessage(Message message) {
        String msg = new String(message.getBody());
        log.info("接受到的队列confirm.queue消息：{}", msg);
    }
}
```

- 生产者

```java
@Slf4j
@RequiredArgsConstructor
@RequestMapping("/msg")
public class RabbitmqController {
    private final RabbitTemplate rabbitTemplate;

    @PostMapping("/{message}")
    public void sendMessage(@PathVariable String message){
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME
                ,CONFIRM_ROUTING_KEY
                ,message,                
                //消息唯一标识
                new CorrelationData("1"));
        log.info("发送消息内容：{}",message);
    }
}
```

- 回调接口准备

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //将当前实现类，注入到rabbitTemplate中的接口上
        //不然到时候rabbitTemplate调用接口时，找不到我们的实现类
        rabbitTemplate.setConfirmCallback(this);
    }

    /*
     * 交换机确认回调方法,发消息后，交换机接收到了就回调
     *   1.1 correlationData：保存回调消息的ID及相关信息
     *   1.2 b:交换机收到消息，为ack=true
     *   1.3 s:失败原因，成功为null
     *
     * 发消息，交换机接受失败，也回调
     *   2.1 correlationData：保存回调消息的ID及相关信息
     *   2.2 b:交换机没收到消息，为ack=false
     *   2.3 s:失败的原因
     *
     * */

    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {
        String id = correlationData!=null ? correlationData.getId():"";
        if (b){
            log.info("交换机已经收到ID为：{}的信息",id);
        }else {
            log.info("交换机还未收到ID为：{}的消息，由于原因：{}",id,s);
        }
    }
}

```

- 开启发布确认模式

```java
spring.rabbitmq.publisher-confirm-type=correlated
```

⚫ **NONE** 禁用发布确认模式，是默认值
⚫ **CORRELATED** 发布消息成功到交换器后会触发回调方法
⚫ **SIMPLE** 发布一条消息确认一条消息



> SIMPLE模式注意: 使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker

- 测试

![image-20220526185943321](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859387.png)

****

### 消息回退接口

上面讲过，这里讲讲springboot中如何使用

- ### mandatory

  - TRUE: 消息路由到队列失败,调用消息return接口
  - FALSE: 消息路由到队列失败,尝试将消息转发给兜底交换机

- 配置文件中开启设置mandatory为true

```yaml
spring.rabbitmq.publisher-returns=true
```

- 设置回退接口

```java
@Component
@Slf4j
public class MyCallBack implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //将当前实现类，注入到rabbitTemplate中的接口上
        //不然到时候rabbitTemplate调用接口时，找不到我们的实现类
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }

    /*
     * 交换机确认回调方法,发消息后，交换机接收到了就回调
     *   1.1 correlationData：保存回调消息的ID及相关信息
     *   1.2 b:交换机收到消息，为ack=true
     *   1.3 s:失败原因，成功为null
     *
     * 发消息，交换机接受失败，也回调
     *   2.1 correlationData：保存回调消息的ID及相关信息
     *   2.2 b:交换机没收到消息，为ack=false
     *   2.3 s:失败的原因
     *
     * */

    @Override
    public void confirm(CorrelationData correlationData, boolean b, String s) {
        String id = correlationData!=null ? correlationData.getId():"";
        if (b){
            log.info("交换机已经收到ID为：{}的信息",id);
        }else {
            log.info("交换机还未收到ID为：{}的消息，由于原因：{}",id,s);
        }
    }
    
    @Override
    public void returnedMessage(ReturnedMessage returnedMessage) {
        log.error("消息{}，被交换机{}退回，退回的原因：{},路由Key：{}",
                new String(returnedMessage.getMessage().getBody())
                , returnedMessage.getExchange()
                , returnedMessage.getReplyText()
                , returnedMessage.getRoutingKey());
    }
}

```

- 测试

```java
    @PostMapping("/msg/{message}")
    public void sendMessage(@PathVariable String message){
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME
                ,CONFIRM_ROUTING_KEY
                ,message+"key1",new CorrelationData("1"));
        log.info("发送消息内容：{}",message+"key1");


        CorrelationData correlationData2 = new CorrelationData("2");
        rabbitTemplate.convertAndSend(CONFIRM_EXCHANGE_NAME
                ,CONFIRM_ROUTING_KEY+"2"
                ,message+"key12",correlationData2);
        log.info("发送消息内容：{}",message+"key12");
    }
```

![image-20220526185936547](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859618.png)

****

### 备份交换机

> 备份交换机是rabbitmq对AMPQ协议的扩展，在rabbitmq中被称为Alternate Exchanges

[Alternate Exchanges — RabbitMQ](https://www.rabbitmq.com/ae.html)

声明备份交换机代码如下:

```java
@Configuration
public class RabbitConfig {

    //交换机
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
    //队列
    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";
    //RoutingKey
    public static final String CONFIRM_ROUTING_KEY = "key1";
    //备份交换机
    public static final String BACKUP_EXCHANGE_NAME = "backup_exchange";
    //备份队列
    public static final String BACKUP_QUEUE_NAME = "backup_queue";
    //报警队列
    public static final String WARNING_QUEUE_NAME = "warning_queue";

    //声明交换机
    @Bean
    public DirectExchange confirmExchange()
    {
        return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME).durable(true)
                //指定备份交换机
                .withArgument("alternate-exchange",BACKUP_EXCHANGE_NAME).build();
    }

    //声明队列
    @Bean
    public Queue confirmQueue()
    {
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }


    //备份交换机---一般是扇出类型
    @Bean
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    //备份队列
    @Bean
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    //报警队列
    @Bean
    public Queue warningQueue(){
        return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }

    //绑定
    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue confirmQueue,
                                        @Qualifier("confirmExchange")DirectExchange confirmExchange)
    {
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }


    @Bean
    public Binding backupQueueBindingBackupExchange(@Qualifier("backupQueue") Queue backupQueue,
                                                    @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    @Bean
    public Binding warningQueueBindingBackupExchange(@Qualifier("warningQueue") Queue backupQueue,
                                                     @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }
    
}

```

其他过程和普通队列，交换机使用没有区别



****

## 生产注意

### 消息补偿

![image-20220526185924491](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859638.png)

- 核心思路: 数据库记录用户确认过的消息,然后通过比对生产者已经发出的消息和消费者确认的消息,就可以找出丢失或未确认的消息,接着通知消费者重发
- 上面还用到了一个延迟消息发送机制,以及定时比对服务,双重确保消息补偿一定到位

****

### 如何保持消息幂等性

幂等性是啥:

- 用户对于同一操作发起的一次请求或者多次请求的结果是一致的,不会因为多次点击而产生了副作用。

举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。

在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等

实现消息幂等性的方法有下面几种:

****

#### 全局唯一ID

![image-20220526185915502](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859592.png)

该方法最大问题在于每次查询数据库这个过程非常浪费性能,因此不推荐采用

****



#### redis的setnx

> 使用setnx的原因在于其唯一性,即如果key已经存在了,再进行设置为失败

![image-20220526185907237](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859296.png)

![image-20220526185900982](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261859086.png)

****

#### 乐观锁

![image-20220526185842199](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261858289.png)

****

## Rabbitmq基本通信过程小结

![image-20220526185821945](https://raw.githubusercontent.com/dhyly/picBed/main/rabbitmq/202205261858067.png)

还有很多额外参数来控制通信流程，这里没有列举出来，大家可以参考我给出的过程，自己继续添加和完善。

****


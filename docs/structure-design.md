
# sumary
在日常的开发和设计过程中，大家对技术设计上的一些问题往往会面临很多的选择，不同的人会有不同的选择，每每如此，我都会尝试着问自己：我做出选择和判断背后的原则是什么？

经过这么多年的发展，在软件设计过程，目前沉淀下来的原则有很多，但很多情况下，很多原则为了普适性，总结得会比较抽象，一旦太过抽象，对原则的解释和理解就会因人而异，譬如：高内聚低耦合原则，大家都懂，但是如何落地和执行却是很难说完全达成一致。因此，需要针对一些实际的场景中的问题去总结和补充，在大的原则下具化形成大家容易理解一致的相对明确原则。

本文介绍的就是我在工作中遇到的一些问题而总结和使用到的一些常用原则。

# 一  常用原则总结

## 1  分层设计相关原则

### 单向依赖原则

原则上只允许较高层次依赖较低层次，不允许反向依赖。

我们部门是为B类企业提供金融解决方案的技术部门，针对我们部门，在金融平台层系统不能反向依赖业务产品层系统。同一层的金融平台层系统之间的依赖不进行限制，但会尽量减少同层依赖。

另外，我们在解决底层依赖的高层中沉淀了几种基本方式：

系统依赖转换为数据依赖；

接口依赖，通过底层定义SPI，业务层实现，这种做法其实是不得已为之，同时，我们在设计过程中还是尽可能避免走这条路；

通过事件机制解耦依赖。


### 无循环依赖原则

系统设计时，尽量减少系统之间的依赖，同时需要避免系统之间出现循环调用。

这是微服务场景下最容易出现的一个问题，尤其是同层的领域系统之间的调用，导致系统容易出现循环调用，循环依赖带来的一个严重的问题是影响系统的发布和部署问题。

### 避免跨层调用原则

较高层次不允许之间跨层调用底层。

软件设计中进行分层的一个重要目的是通过分层屏蔽底层的实现细节，如果出现跨层相当于把底层的实现直接暴露了。譬如门面服务层，绕过领域服务层，直接调用DAO层进行数据读写操作，一旦需要重构修改原有的DAO层接口，就发现升级改造成本巨大，我不知道有多少个团队也面临过这种痛苦。

### 单一职责原则

该原则由罗伯特·C·马丁（Robert C. Martin）于《敏捷软件开发：原则、模式和实践》一书中提出的。这里的职责是指类变化的原因，单一职责原则规定一个类应该有且仅有一个引起它变化的原因，否则类应该被拆分（There should never be more than one reason for a class to change）。

这个原则虽然提出时是解决类的职责定义问题，但实际上在对模块的划分上也有指导意义。该原则虽然很简单，但是往往也容易被忽视。

在最近的项目中，我充分体会到这个原则的作用，我们部门的金融网络系统主要解决机构标准化对接问题，我们将系统分为了上下两层，下层通过标准化的接口对接机构，提升机构跨产品的复用能力；上层是产品扩展层，通过提供标准接口给到上游的业务产品层，支持同一个产品接入多家机构，屏蔽机构差异。我们判断一个功能到底属于机构对接层，还是产品扩展层的一个简单的原则是：如果新增一家机构，能否做到只影响机构对接层，而保持产品扩展层代码不改；反过来，如果新增一个产品，是否能做到只修改产品扩展层，机构层能否不改代码。同时，为了避免这个原则被突破，我们甚至在机构对接层的代码中，去除了所有和产品有关的参数，这样，根据产品定制的逻辑天然无法放到这一层。

### 数据冗余

架构设计应该使得系统中数据的冗余最小。

譬如我们在实践过程中，接口设计时，在Javadoc上强制指定接口的必传参数，尽量做到最小集，减少上游系统使用接口的成本。另外要求在接口实现时，提前进行参数校验，不让不满足要求的数据冗余到系统中。

为了提高系统性能，备份节点和子系统/模块必要时需要对数据进行缓存，当发生变化时，必须有相应的机制保证缓存数据的一致性和有效性。

## 2  质量属性相关原则

### 数据安全

这块在我们金融业务部门中尤其突出，金融由于其特殊性，往往需要收集大量的客户真实和隐私数据，数据安全是设计中需要重点考虑的问题，通常我们会主要关注以下三个方面的问题：

#### 数据存储安全：
敏感数据加密、日志输出脱敏。


#### 数据传输安全：
包括加密、传输通道规范，最少字段传输（够用原则），尤其是我们金融部门，需要将数据输出给到外部第三方机构情况比较多，这块上面会控制比较严格。


#### 数据输出展示：
前端展示需要防止水平越权，另外，前端的展示可以埋点和方便数据采集。


## 3  资损防控

### 可核对和可监控：
上下游系统的数据模型核对关联关系简单、稳定（具备通用性，和产品无关）。


### 可熔断：
对关键资损链路需要做到可熔断。


对金融技术部门而言，资损防控是第一位，而我们在实际过程中发现，由于前期的一些系统在设计之初没有考虑资损的防控，导致核对或者监控的成本很高，因此，在后来的系统数据模型设计时重点会去review是否具备可核对。

## 4  并发控制

### 悲观锁：   
代码编码规范——一锁二查三更新。

### 乐观锁：    
必须在事务内更新。


## 5  热点问题

避免流量倾斜，导致单台机器/单个数据表/数据库集中读写。

这个需要在设计时充分提前预判业务的发展规模和系统的容量问题。在实际实施过程中，我们会提前按照3~5年左右的业务规模来设计。

## 6  数据倾斜

分表分库规则在设计时需要考虑数据分布均匀，避免单库或者单表数据倾斜。

数据倾斜这个在之前踩过比较大的坑，在系统设计之初没有结合业务场景去考虑系统的数据存储层设计，导致数据出现严重倾斜，数据库操作出现瓶颈，现在是我们在设计存储层方案时必须要考虑的一个原则。

##  7  性能原则

#### 可压测：
对性能要求高的链路，需要做到可以压测。

这个主要是由于每到大促就需要重新梳理和改造压测链路，耗时费力，苦不堪言。

##  8  事务控制相关原则

#### 优先使用编程式事务：
为了更好的控制事务，一般要求使用编程式事务，避免潜在的跨事务问题。
#### 事务更新需要保证顺序一致性：
强一致要求还是最终一致，强一致是否会涉及到跨库，事务操作时需要相同记录的更新顺序保证一致。

事务中不进行远程调用。


##  9  一致性相关原则

#### 区分系统调用错误和业务失败：
远程调用失败，不代表下游系统没有接收请求，更不能做为业务失败依据，需要严格区分系统调用错误和业务失败。

#### 可重试：
任何一行代码执行时都有可能因系统重启而中断，所以需要支持可重试。

#### 异步处理必须增加核对：
最终一致性离不开恢复重试策略，也需要有系统间数据核对用于及时发现数据不一致，同时在核对时需要增加处理时效的监控，及时发现长时间未处理成功的数据。


# 二  API设计相关设计原则

## 1  水平越权控制

API设计时需要考虑防范水平越权。

目前我们的做法是，从前端到后端，每层都需要进行越权校验。通过从接口设计层面防控，避免某层出现疏忽导致越权的事件发生。

## 2  接口幂等控制

调用方必须提供用于幂等控制的参数，为了控制幂等，同一个请求的幂等参数不变。

在血泪史上，由于接口不幂等导致的问题太多了，这个目前基本上已经成了部门在接口设计上的共识。

## 3  兼容性原则

API升级和调整，需要兼容老的版本。

为了保证接口可以升级，我们对接口的设计就会存在比较高的要求，譬如接口参数中不能使用枚举，不能使用Java基础类型等，同时也要求接口设计需要具备一定前瞻性和通用性，尤其对于面向业务领域的接口设计，更要求对该领域的业务知识有比较多的了解。

当然还有一些原则在《Java开发手册》中已有叙述，这里就不在赘述。

# 三  总结

本文介绍了我们在系统设计和开发实际场景中总结出的一些原则，通过这些原则的总结和沉淀，可以在后续出现同类问题时做出相对正确的选择，避免重蹈覆辙。另外，通过在大的原则下进行具体化和明确化，能够让大家容易达成一致，让架构方案更容易落地，不走偏。

另外，无论是在生活上还是工作上，建议多从成功的经验或者失败的教训中去总结，形成自己的原则，丰富自己的决策系统。这是《原则》这本书给我带来的一个比较大的启发。
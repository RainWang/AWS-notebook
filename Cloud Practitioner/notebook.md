# 1.云的基本知识
## 1.1 区域（Regional）
选择区域时主要考虑因素：
1. 国家政策，安全性。
2. 地理位置，低延迟。
3. 区域可提供的服务。
4. 价格。

[每个区域可使用服务查询链接](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)

## 1.2 可用区（Availability Zone）
一个区域中，可用区通常3个（最少3个，最大6个）。<br>
可用区命名规则：
1. 区域名称+数字+字母
2. 同一个“可用区”在不同账号中，可能显示不同的名称。

![可用区命名规则](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/2.png)

## 1.3 边缘站点（Edge Locations）和区域边缘缓存（Regional Edge Caches）
1. “区域边缘缓存”的容量通常大于“边缘站点”。
2. 访问高的数据存放于“边缘站点”，访问低的数据存放于“区域边缘缓存”。

例：<br>
用户向S3请求数据时，会在CloudFront中缓存一份数据，CloudFront会在“区域边缘缓存”中缓存一份数据，“区域边缘缓存”又会在“边缘站点”缓存一份数据。<br>
“边缘站点”会定期清理不常用数据，释放空间给经常访问的数据。<br>
当用户在“边缘站点”访问不到数据时，会再向“区域边缘缓存”请求数据，逐级往上。

![边缘站点与区域边缘缓存的关系例图](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/1.png)

# 2.核心服务
## 2.1 IAM（Identity and Access Management）-Global Service
### 2.1.1 用户（Users）&组（Groups）&角色（Roles）&策略（Policies）
1. 不建议用Root用户登录AWS，建议通过IAM用户（Users）登录AWS。
2. 组（Groups）只能包含用户，不能包含其它组。
3. 角色（Roles）不能用于登录，一般被赋予某个资源，你可以在任何时候将角色赋予EC2实例，**并且不需要重启而能够马上生效。**
4. 策略（Policies）是种JSON格式文件，它可以给组，用户，角色赋予权限。

策略结构：
![策略](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/3.png)

IAM使用案例：
![IAM授权使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/4.png)

### 2.1.2 登录AWS方式
1. AWS Managment Console
2. AWS CLI（Access Key）
3. AWS SDK（Access Key）

### 2.1.3 安全组件
1. 凭证报告（IAM Credentials Report）-账号级别
2. 访问顾问（IAM Access Advisor）-用户级别

[MFA下载页面](https://aws.amazon.com/cn/iam/features/mfa/)

## 2.2 EC2
1. 实例重启后，公用IP会变化，弹性IP地址（Elastic IP）可以使实例重启后，公用IP不发生变化，但是需要支付一定费用，私有IP在实例重启后不会变化。
2. 默认实例用户：ec2-user。

### 2.2.1 安全组（Security Groups）
1. 防火墙，默认允许任何流量流出，阻止任何流量流入。
2. **如果访问实例收到“time out”错误，就是防火墙错误；** 如果实例收到“connection refused”错误，那可能是访问服务没有开启等原因。
3. 安全组可以被ip或者安全组引用。

### 2.2.2 存储
#### 2.2.2.1 EBS（Elastic Block Store）
1. 网络驱动器，通过网络链接实例，类似U盘，**一次只能绑定一个实例，实例终止，数据依旧能保留在EBS上。**
2. **EBS跟可用区（AZ）绑定**，us-east-1a可用区的EBS不能链接到us-east-1b的实例，但是通过网络快照（snapshot），则可以在不同的可用区之间移动卷。
3. 需要提前定义容量和IOPS。

#### 2.2.2.2 EFS（Elastic File System）
1. 这是个网络文件系统，即NFS（network file system），它可以同时链接到多个EC2实例，又称为共享网络文件系统。
2. **只能用于EC2，可以跨多个可用区（AZ）。** 一个AZ中的实例可能与另一个AZ中的实例共享一个EFS。
3. 价格是EBS的3倍，但是不需要提前定义容量，用多少付多少。
4. 通过启用EFS Infrequent Access（EFS-IA）策略，可以让EFS中不常访问的文件（大于60天），自动移动到EFS-IA中，EFS-IA能提供高达92%的折扣。

#### 2.2.2.3 FSx
1. Amazon FSx for Windows：Windows本机文件共享系统。
2. Amazon FSx for Lustre：Lustre是众所周知的开源的并行文件系统，主要用于高性能（HPC）的场景。

### 2.2.3 负载均衡-ELB（Elastic Load Balancing）
ELB的类型：
1. 应用负载均衡ALB（Application Load Balancer）：Http/Https/gRPC协议，位于网络协议第七层。
2. 网络负载均衡NLB（Network Load Balancer）：TCP/UDP协议，位于网络协议第四层。**它非常高性能，每秒能处理几百万请求。**
3. 网关负载均衡GLB（Gateway Load Balancer）：可以路由流量到防火墙，以便你执行入侵检测等操作，位于网络协议第三层。

### 2.2.4 ASG（Auto Scaling Groups）
1. 当ASG被删除，所创建的实例也会被自动删除。
2. ASG的收缩策略：手动策略，动态策略，预测策略。

**高可用性（High Availability）** 指的是ASG把实例创建到不同AZ，ELB可以访问当前可用AZ的实例，从而在特殊情况（自然灾害等）导致某个AZ不可用时，用户访问实例依旧不受影响。

## 2.3 S3
1. S3的桶（Buckets）类似文件夹，存在桶（Buckets）里的用户数据叫对象（Object），即文件。
2. S3的桶（Buckets）只能取全球唯一的名字。
3. S3的桶（Buckets）是在区域（Regional）里创建的。
4. 每个对象（Object）都有个Key，对象Key是文件的完整路径，即**Key**是以下加粗部分。
    - s3://1006493605/**notebook/Cloud_Practitioner/test.png**
    - s3://1006493605/**test.png**
5. S3内部没有文件夹的概念，是通过对象的Key去找到对象的。
6. S3最大可以上传5TB（5000GB）的对象，如果上传的对象大于5GB，则要通过“multi-part”上传，即5TB对象需要分成1000份上传。
7. 可以通过S3桶策略或者IAM策略去定义S3中对象（Object）的访问权限。
8. S3的Versioning功能可以开启S3的版本控制功能。
9. S3的复制（Replication）机制：
    - Cross-Region Replication（CRR）跨区域拷贝
    - Same-Region Replication（SRR）同区域拷贝
10. S3的复制（Replication）机制是一个异步复制机制，**而且必须开启版本控制（Versioning）功能**。
11. S3的存储类别（S3 Storage Classes），可以手动修改对象的存储类别，也可以通过设置桶的“生命周期规则”去让桶自动归类对象的存储类别：
![IAM授权使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/8.png)
12. S3默认开启服务器端加密，即上传的对象由AWS加密，也可以修改为客户端加密，即由客户自己加密对象后上传。
13. S3 Snow Family：
    - Snowcone：
         - Snowcone：<br>
        提供了8TB的HDD。
        - Snowcone SSD：<br>
        提供了16TB的SDD。   
    - Snowball Edage：
        - Snowball Edge Storage Optimized：<br>
        提供了80TB的可用数据块。
        - Snowball Edge Compute Optimized：<br>
        提供了42TB的可用数据块。
    - Snowmobile：提供100PB的存储空间。
14. 通过存储网关（Storage Gateway）可以把客户的文件系统与AWS的云桥接起来，用于混合云。
15. 属于**无服务（serverless）** ，即不需要创建实例。

## 2.4 数据库
### 2.4.1 RDS-关系型数据库
1. **不能SSH到数据库实例。**
2. AWS自带的Aurora数据库比普通的RDS数据库效率更高，但是是收费的。
3. 可以创建数据库只读副本（Read Replicas）去提高数据库的访问速度，最大只能创建15个，**只读副本可以跨区域创建**。
4. 可以在其它AZ创建故障转移副本（Failover DB）去防止突发情况，只能在其它AZ中选一个AZ创建故障转移副本，故障转移副本平常不可访问，只有等主DB发生故障时才可访问。
5. 可以使用**ElastiCache**对数据库的读取速度进行优化，ElastiCache是一种数据库缓存技术，支持Redis和Memcached。

### 2.4.2 NoSql-非关系型数据库
#### 2.4.2.1 DynamoDB
1. 一种**无服务（serverless）** 的数据库，即不需要创建实例。
2. 每秒可以处理几百万请求，数百TB的存储，延迟非常低。
3. DynamoDB加速器叫DAX。
4. DynamoDB无法像RDS一样，把表与表之间联系起来。
5. 全球表（Global Tables）服务可以使DynamoDB在跨区域使用时，延迟降低。各个区域的DynamoDB都可以读写，且区域之前互相复制。

#### 2.4.2.2 DocumentDB
1. 一种**无服务（serverless）** 的数据库，即不需要创建实例。
2. 类似于MongoDB。

### 2.4.3 Redshift-数据仓库
1. 数据库主要用于事务处理，即OLTP（Transaction），数据仓库（warehousing）主要用于数据分析，即OLAP（Analytics）。
2. 建立OLAP应用之前，我们要想办法把各个独立系统的数据抽取出来，经过一定的转换和过滤，存放到一个集中的地方，成为数据仓库。这个抽取，转换，加载的过程叫ETL（Extract， Transform，Load）。
3. 数据清洗工具有EMR（Elastic MapReduce）和Glue等，EMR创建**Hadoop clusters**去处理大数据。<br><br>
例：<br>
通过EC2，Lambda，Beantalk等工具将数据收集到S3，在S3通过数据清洗工具EMR（Elastic MapReduce）,Glue等将结果上传至Redshift，Redshift通过报告显示给用户。<br>
![Redshift使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/6.png)

### 2.4.4 Athena-SQL查询工具
1. Athena是**无服务（serverless）** 的，可以用SQL语句对存储在S3里的数据进行查询。
2. 还可以用Athena对各种LOG进行查询。

### 2.4.5 QuickSign-创建图表工具
可以为数据库创建分析的图表。

### 2.4.6 Neptune-图表数据库
为高度链接的数据集应用服务，如SNS，wiki。

### 2.4.7 QLDB-金融交易分类账
数据不可变，多用于金融交易，非去中心化。

### 2.4.8 DMS（Database Migration Service）-数据迁移服务
用于数据库之间的数据迁移，支持不同种类数据库之间的迁移。

# 3.其它计算服务
## 3.1 Docker运行工具
### 3.1.1 ECS（Elastic Container Service）
1. 弹性容器服务，用来启动Docker。
2. **必须预先创建EC2实例。**

### 3.1.2 Fargate
1. 弹性容器服务，用来启动Docker。
2. **不用预先创建EC2实例，属于无服务（serverless）服务。**

### 3.1.3 ECR（Elastic Container Registry）
是一个AWS上的私有Docker注册表，用来存储Docker映像，以便在ECS和Fargate上运行Docker。

## 3.2 Lambda
1. **不用预先创建EC2实例，属于无服务（serverless）服务。**
2. **事件驱动**，事件触发后才会调用函数进行计算。
3. 集成在所有AWS的服务中，支持多种代码语言。
4. Lambda有时间限制，最多只能运行15分钟。
5. Lamdba有磁盘空间限制。
6. **API Gateway**可以把Lambda函数以“HTTP API”的方式暴露出来。

![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/9.png)

## 3.3 Batch
1. Batch是个托管批处理服务，**Batch Job是一个运行在ECS上的Docker映像**，这意味着任何可以在ECS上运行的都可以在Batch上运行。
2. 无时间空间限制。

# 4.大规模部署和管理
## 4.1 CloudFormation
1. 通过模版文件自动化创建资源组件，模版文件分为Json，Yaml两种格式。
2. 在模版文件中，“Resources”是不能缺少的。
3. 删除堆栈时，通过模版创建的资源组件（比如EC2，安全组等）都会被删除。
4. 通过CDK（AWS Cloud Development Kit），可以用熟悉的编程语言定义云架构模版。

[CloudFormation 的官方模板下载链接](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html#w2ab1c28c58c13c17)

## 4.2 Beanstalk
1. 属于PaaS，平台即服务。
2. Beanstalk通过CloudFormation搭建环境。

## 4.3 SSM（AWS Systems Manager）
1. 属于**混合云服务（Hybrid AWS）**。
2. **SSM Session Manager** 服务允许你在EC2或者自己主机上开启一个安全的shell，无需通过SSH访问。
3. 实例使用时，需要给实例赋予可以访问SSM服务的角色。

# 5.网络
## 5.1 Route53-域名解析服务
1. 主要的三个功能：域名注册，DNS路由，运行状况检查。
2. DNS路由策略：
    - 简单路由
    - 地理位置路由
    - 故障转移路由
    - 加权路由
    - 基于延迟的路由
    - 多值应答路由
3. DNS的记录类型：
    - A记录：是用来指定主机名（或者说是域名）对应的IP地址记录，可以通过A记录来给网站的域名指向到自己的web 服务器上之后，那么访问域名的内容就会来自于自己的web服务器。
    - CNAME：通常称为别名解析，可以将注册的不同域名都转到一个域名记录上，由这个域名记录统一解析管理，与A记录不同的是，CNAME别名记录设置的可以是一个域名。

## 5.2 CloudFront-内容分发服务
1. **可以通过Shield或者WAF（AWS Web Application Firewall）对DDoS攻击形成保护。**
2. 内容源（Origin），CloudFront分发的内容来源：
    - S3存储桶：可以通过OAC（Origin Access Control）去增加安全性。
    - 自定义内容源（HTTP）：可以是EC2实例，ALB（Application Load Balancer），S3 website，或者任何HTTP。

## 5.3 VPC（Virtual Private Cloud）
### 5.3.1 子网（subnets）
1. 在AZ内有多个子网，子网内部的主机可以互相联通。
2. 公有子网可以直接访问外部网络，私有子网不能直接访问外部网络。
3. **Internet Gateways**使VPC可以访问外网，在VPC内的公有子网可以直接连接网络，但是私有子网需要通过连接建立在公有子网内部的**NAT Gateways**去连接网络。
4. 需要通过路由表（Route Tables）定义子网之间的访问。

路由表如下所示，0.0.0.0/0代表外网。
![路由表](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/11.png)

### 5.3.2 NACL（Network ACL）
NACL和安全组的区别：
![NACL和安全组的区别](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/12.png)

### 5.3.3 Peering
可以建立两个VPC之间的联系，但是得确保两个VPC之间没有重复的IP，并且没有传输性，即A同时和B与C连接后，B与C之间也无法连接，必须在B与C之间用Peering再连接一次。

### 5.3.4 Endpoints
1. 可以使两个AWS服务之间通过创建私有网络进行连接，更加的安全和快速。
2. VPC Endpoint Gateway只能连接S3和DynamoDB。
3. VPC Endpoint Interface可以连接除了S3和DynamoDB以外的其它服务。
4. PrivateLink服务提供AWS服务和第三方VPC之间的私有连接，更加安全和具有扩展性。
5. **Peering是VPC级别的，Endpoints是服务应用级别的。**

### 5.3.5 混合云连接服务
#### 5.3.5.1 Site to Site VPN（Virtual Private Network）
1. 可以把自己的数据中心VPN网络连接到AWS，加密后的信息通过共有网络进行连接，存在共有网络带宽限制和一定的安全问题。
2. 自己的数据中心一侧必须使用一个**CGW（Customer Gateway）**，AWS一侧需要使用**VGW（Virtual Private Gateway）**。

#### 5.3.5.2 Direct Connect
在自己的数据中心和AWS之间建立一条专用的私人网络，相对安全快速。

#### 5.3.5.3 ClientVPN
在本地电脑上安装ClientVPN后，可以直接通过私有网络访问AWS的实例。

### 5.3.6 Transit Gateway
可以把多个VPC连接起来。

# 6.消息
## 6.1 SQS（Simple Queue Service）-简单队列服务
消息被读取后会从队列中删除。

## 6.2 SNS（Simple Notification Service）-简单提示服务
不保留消息，一旦触发，会发送给所有订阅者。

# 7.安全与监控
## 7.1 监控服务
### 7.1.1 CloudWatch
1. CloudWatch不仅能够监控AWS的服务，还能监控非AWS产品的服务。
2. CloudWatch的警报可以触发SNS，ASG，SMM和EC2的action。
3. 在EC2或者自己的主机上安装“CloudWatch Logs Agent”，可以获得其日志，但是得确保EC2有合适的IAM权限访问CloudWatch。

### 7.1.2 EventBridge
1. 通过“Schedule”可以制定一个Cron jobs，即定时任务，触发一些功能，如Lambda函数的调用等。
2. 通过“Event Pattern”，可以在一些AWS服务做某些动作时，发送一个事件，如Root用户登录时，或者EC2状态为pending时等等。
3. 事件总线（Event Bus）除了可以接受来自AWS服务的事件，还可以接受来自AWS SaaS合作伙伴或者自定义APP的事件。

### 7.1.3 CloudTrail
1. CloudTrail可以查看所有用户的控制台操作，以及CLI，AWS开发工具包和API的执行操作。
2. 为了长期保存日志，可以把日志发送到CloudWatch或者S3中。

### 7.1.4 X-Ray
能够对应用程序进行一个跟踪和可视化分析。

### 7.1.5 CodeGuru
1. CodeGuru会查看（review）和运行你提交到代码，找出漏洞等，目前支持Java和Python。
2. CodeGuru Profiler可以根据运行你提交的代码，显示代码运行中哪部份cpu消耗得多等。

### 7.1.6 Health Dashboard
是一个全球服务，显示所有区域所有服务的安全状况，也会对用户账号的安全状况进行监管。

## 7.2 安全服务
### 7.2.1 WAF和Shield
1. 防止DDos攻击。
1. WAF（Web Appliction Firewall，web应用防火墙）位于第七层。
2. Shield分为标准版和高级版，标准版免费。

### 7.2.2 KMS（Key Management Service）-密钥管理服务
1. AWS中的数据通常在两种情况下需要数据加密，一种是存储的时候，一种是传输的时候。
2. KMS管理加密软件，CloudHSM（Hardware Security Module）管理加密硬件。
3. 客户主密钥CMK（Customer Master Keys）类型：
    - 客户自己管理的CMK
    - AWS管理的CMK
    - AWS自己的CMK
    - CloudHSM的Keys

### 7.2.3 ACM（AWS Certificate Manager）-AWS证书管理
管理生成SSL/TLS证书，用来HTTPS服务加密用。

### 7.2.4 Secrets Manager
可以与RDS集成，定期更新密钥。

### 7.2.5 Artiface
作为一个咨询服务，可以让客户按需访问合规性报告和AWS协议。

### 7.2.6 GuardDuty
通过机器学习，异常检测和第三方数据去发现威胁，**可以使你免受加密货币的攻击**。

### 7.2.7 Inspector
对EC2实例和ECR容器映像和Lambda函数做一个扫描，找出漏洞，按等级列出风险评估列表。

## 7.3 高级认证


# 8.账单和支持

# 9.完善的构架
## 9.1 Well-Architected Framework
一个用于设计基础设施的指南，一种评估和实施架构的系统的方法，从众多经验教训中建立起来的最佳实践。

框架的支柱：
1. 卓越运营（）
    - 利用代码执行操作
    - 注释文档
    - 定期进行一些增量的微小变更
    - 经常优化程序
    - 预见故障
    - 从操作事件及故障中总结经验
2. 安全性
    - 身份验证机制
    - 启用追踪性
    - 在所有层应用安全性
    - 自动实施安全性最佳实践
    - 保护传输中的数据和静态数据
    - 做好安全性事件应对准备
3. 可靠性
    - 测试恢复流程
    - 自动故障恢复
    - 横向扩展提升总体系统可用性
    - 无需猜测容量
    - 自动管理变更
4. 性能效率
    - 普及先进技术
    - 数分钟内实现全球化部署
    - 使用无服务架构
    - 更频繁进行试验
    - 制度化选择
5. 成本优化
    - 采用消费模型
    - 衡量总体效率
    - 无需再为数据中心运营投入资金
    - 分析支出和确定支出属性
    - 使用托管服务降低拥有成本

# 10.其它服务
## 10.1 机器学习
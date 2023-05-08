# 1.云的基本知识
## 1.1 区域（Regional）
选择区域时主要考虑因素：
1. 国家政策，安全性。
2. 地理位置，低延迟。
3. 区域可提供的服务。
4. 价格。

[每个区域可使用服务查询链接](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/)

## 1.2 AZ（Availability Zone）-可用区
一个区域（Regional）中，AZ通常3个（最少3个，最大6个）。<br>
可用区命名规则：
1. 区域名称+数字+字母
2. 同一个AZ在不同账号中，可能显示不同的名称。

![AZ命名规则](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/2.png)

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
2. **组（Groups）只能包含用户，不能包含其它组。**
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

### 2.1.3 IAM安全组件
1. 凭证报告（IAM Credentials Report）-账号级别<br>
生成一个报告，报告列出所有账户中用户的各种凭证的状态。
2. 访问顾问（IAM Access Advisor）-用户级别<br>
显示用户的服务权限，以及用户上次的访问时间。
3. 访问分析器（IAM Access Analyzer）<br>
对AWS资源IAM访问权限的分析，需要定义一个信任域（Zone of Trust），信任区域之外的任何访问都将作为调查结果报告。

### 2.1.4 MFA（Multi Factor Authentication）
为了账号安全性，尤其是Root用户，除了用密码登录AWS账号，最好再增加MFA设备验证。<br>
MFA验证方式：
1. 虚拟MFA设备（Virtual MFA device）<br>
[虚拟MFA设备下载页面](https://aws.amazon.com/cn/iam/features/mfa/)
2. 物理设备：<br>
    - U2F（Universal 2nd Factor）Security Key
    - Hardware Key Fob MFA Device
    - Haedware Key Fob MFA Device for AWS GovCloud（US）

## 2.2 EC2
1. **实例重启后，公用IP会变化。** 弹性IP地址（Elastic IP）可以使实例重启后，公用IP不发生变化，但是需要支付一定费用，私有IP在实例重启后，不会变化。
2. 默认实例用户：ec2-user。

### 2.2.1 安全组（Security Groups）
1. 防火墙，**默认允许任何流量流出，阻止任何流量流入。**
2. **如果访问实例收到“time out”错误，就是防火墙错误；** 如果实例收到“connection refused”错误，那可能是访问服务没有开启等原因。
3. 安全组可以被单个EC2实例，或者其它安全组引用。
4. 在设置流量流入流出规则时，只有**allow**规则，没有**deny**。

安全组如下所示，可以设置流量访问的端口号和IP：
![IAM授权使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/13.png)

### 2.2.2 存储
#### 2.2.2.1 EBS（Elastic Block Store）
1. 网络驱动器，通过网络链接实例，类似U盘，**一次只能绑定一个实例，实例终止，数据依旧能保留在EBS上。**
2. **EBS跟可用区（AZ）绑定**，us-east-1a可用区的EBS不能链接到us-east-1b的实例，但是通过网络快照（snapshot），就可以在不同的可用区之间移动卷。
3. 需要提前定义容量和IOPS，**后期可以扩展容量。**

#### 2.2.2.2 EFS（Elastic File System）
1. 这是个网络文件系统，即NFS（network file system），**它可以同时链接到多个EC2实例，又称为共享网络文件系统。**
2. **只能用于EC2，可以跨多个可用区（AZ）。** 一个AZ中的实例可能与另一个AZ中的实例共享一个EFS。
3. 价格是EBS的3倍，但是**不需要提前定义容量，用多少付多少。**
4. 通过启用EFS Infrequent Access（EFS-IA）策略，可以让EFS中不常访问的文件（大于60天），自动移动到EFS-IA中，EFS-IA能提供高达92%的折扣。
5. **只能用于Linux实例。**

#### 2.2.2.3 FSx
用于AWS的第三方**高性能（HPC）文件系统**，可以让EC2实例和客户自己的数据中心共享文件。
1. Amazon FSx for Windows：Windows本机文件共享系统。
2. Amazon FSx for Lustre：Linux的并行文件系统，主要用于高性能（HPC）的场景。

### 2.2.3 负载均衡-ELB（Elastic Load Balancing）
ELB的类型：
1. 应用负载均衡ALB（Application Load Balancer）：Http/Https/gRPC协议，位于网络协议第七层。
2. 网络负载均衡NLB（Network Load Balancer）：TCP/UDP协议，位于网络协议第四层。**它非常高性能，每秒能处理几百万请求。**
3. 网关负载均衡GLB（Gateway Load Balancer）：位于网络协议第三层。

### 2.2.4 ASG（Auto Scaling Groups）
1. 当ASG被删除，所创建的实例也会被自动删除。
2. ASG的收缩策略：手动策略，动态策略，预测策略。

### 2.2.5 AMI（Amazon Machine Image）
类似与Docker的Image，通过**EC2 Image Builder**可以自动创建，维护，验证和测试AMI。

## 2.3 S3
1. S3的桶（Buckets）类似文件夹，存在桶（Buckets）里的用户数据叫对象（Object），即文件。
2. **S3的桶（Buckets）只能取全球唯一的名字。**
3. S3的桶（Buckets）是在区域（Regional）里创建的。
4. 每个对象（Object）都有个Key，对象Key是文件的完整路径，即**Key**是以下加粗部分。
    - s3://1006493605/**notebook/Cloud_Practitioner/test.png**
    - s3://1006493605/**test.png**
5. S3内部没有文件夹的概念，是通过对象的Key去找到对象的。
6. S3最大可以上传5TB（5000GB）的对象，如果上传的对象大于5TB，则要通过“multi-part”上传，即5TB对象需要分成1000份上传。
7. 可以通过S3桶策略或者IAM策略去定义S3中对象（Object）的访问权限。
8. S3的Versioning功能可以开启S3的版本控制功能。
9. S3的复制（Replication）机制：
    - Cross-Region Replication（CRR）跨区域拷贝
    - Same-Region Replication（SRR）同区域拷贝
10. S3的复制（Replication）机制是一个异步复制机制，**而且必须开启版本控制（Versioning）功能**。
11. S3的存储类别（S3 Storage Classes），可以手动修改对象的存储类别，也可以通过设置桶的“生命周期规则”去让桶自动归类对象的存储类别：
![IAM授权使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/8.png)
12. **S3默认开启服务器端加密**，即上传的对象由AWS加密，也可以修改为客户端加密，即由客户自己加密对象后上传。
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
14. 通过**存储网关（Storage Gateway）** 可以把客户的文件系统与AWS的云桥接起来，用于混合云。
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
3. **DynamoDB加速器叫DAX。**
4. DynamoDB无法像RDS一样，把表与表之间联系起来。
5. **全球表（Global Tables）** 服务可以使DynamoDB在跨区域使用时，延迟降低。各个区域的DynamoDB都可以读写，且区域之前互相复制。

#### 2.4.2.2 DocumentDB
1. 一种**无服务（serverless）** 的数据库，即不需要创建实例。
2. 类似于**MongoDB**。

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

### 2.4.5 QuickSight-创建图表工具
可以为数据库创建分析的图表。

### 2.4.6 Neptune-图表数据库
为高度链接的数据集应用服务，如SNS，wiki。

### 2.4.7 QLDB-金融交易分类账
数据不可变，多用于金融交易，非去中心化。

### 2.4.8 Amazon Managed Blockchain-区块链
去中心化的**区块链技术。**

### 2.4.9 DMS（Database Migration Service）-数据迁移服务
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
4. Lambda有时间限制，**最多只能运行15分钟。**
5. Lamdba有磁盘空间限制。

![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/9.png)

## 3.3 API Gateway
无服务创建API的服务，例：客户通常无法直接访问Lambda函数，需要通过API Gateway把Lambda函数以“REST HTTP API”的方式暴露给用户。

## 3.4 Batch
1. Batch是个托管批处理服务，**Batch Job是一个运行在ECS上的Docker映像**，这意味着任何可以在ECS上运行的都可以在Batch上运行。
2. 无时间空间限制。

## 3.5 Lightsail
新手入门工具，帮助新手快速创建一个实例。

# 4.大规模部署和管理
## 4.1 CloudFormation
1. 通过模版文件自动化创建资源组件，模版文件分为Json，Yaml两种格式。
2. 在模版文件中，“Resources”是不能缺少的。
3. 删除堆栈时，通过模版创建的资源组件（比如EC2，安全组等）都会被删除。
4. 通过**CDK（AWS Cloud Development Kit）**，可以用熟悉的编程语言定义云架构模版。

[CloudFormation 的官方模板下载链接](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html#w2ab1c28c58c13c17)

## 4.2 Beanstalk
1. 属于PaaS，平台即服务。
2. Beanstalk通过CloudFormation搭建环境。

## 4.3 Code代码构建服务
### 4.3.1 CodeCommit
类似github，代码管理仓库。

### 4.3.2 CodeBuild
自动构建代码。

### 4.3.3 CodeDeploy
自动部署应用程序。

### 4.3.4 CodePipeline
持续集成和持续部署CI/CD（Continuous Intergration & Continuous Delivery）工具。

### 4.3.5 CodeArtifact
一项完全托管式工件存储库服务，可让您轻松安全地存储、发布和共享软件开发过程中使用的软件包。

### 4.3.6 CodeStar
一个UI系统，展示构建的各种可视化数据。

### 4.3.7 Cloud9
在云中的代码编辑编译IDE系统。

## 4.4 SSM（AWS Systems Manager）
1. 属于**混合云服务（Hybrid AWS）**。
2. **SSM Session Manager** 服务允许你在EC2或者自己主机上开启一个安全的shell，无需通过SSH访问。
3. 实例使用时，需要给实例赋予可以访问SSM服务的角色。
4. 默认的AWS AMI中，安装了SSM，但是在客户自己的机器上，需要安装SSM agent才能使用。

## 4.5 OpsWorks
即**CHEF&puppet**，是两个可以帮助你自动执行服务器配置，或者重复执行操作的软件，SSM的替代选项。

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
1. CDN加速，原理就是在全球建立CDN服务器，即边缘站点（Edge Locations）和区域边缘缓存（Regional Edge Caches），依靠缓存加速网页访问速度。
2. 内容源（Origin），CloudFront分发的内容来源：
    - S3存储桶：可以通过OAC（Origin Access Control）去增加安全性。
    - 自定义内容源（HTTP）：可以是EC2实例，ALB（Application Load Balancer），S3 website，或者任何HTTP。
3. **集成了Shield，防止DDos攻击。**

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
可以建立两个VPC之间的联系，**但是得确保两个VPC之间没有重复的IP。**<br>
**没有传输性**，即A同时和B与C连接后，B与C之间也无法连接，必须在B与C之间用Peering再连接一次。

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
基于客户端的VPN服务，在本地电脑上安装ClientVPN后，可以直接通过私有IP访问AWS的实例。

### 5.3.6 Transit Gateway
可以把多个VPC连接起来。

## 5.4 网络加速器
### 5.4.1 S3 Transfer Acceleration
可以加速S3的传输速度，原理是用户上传文件到最近的边缘位置，边缘位置直接通过专门的网络传输到S3。

### 5.4.2 AWS Global Accelerator
可以加速网络传输速度，原理和S3 Transfer Acceleration一样，直接在边缘位置和AWS服务直接有一条专门的网络，可以省去路由的时间。<br>
与CloudFront的区别就是CloudFront主要是利用缓存，AWS Global Accelerator不会缓存数据。<br>
 **集成了Shield，防止DDos攻击。**

## 5.5 Outposts
Outposts是部署在客户机房的一体化机柜，在逻辑上作为AWS Region的延展。<br>
本身具有低延迟，数据可以存在本地，机器由AWS维护等优势。

## 5.6 WaveLength
5G网络。

## 5.7 Local Zones
是将服务器部署在靠近客户的数据中心，在逻辑上作为AWS Region的延展。<br>
例：A区域中没有某个AZ，客户希望把服务放置在这个AZ，则需要把这个AZ设置为Local Zones，加入到A区域中。

# 6.消息
## 6.1 SQS（Simple Queue Service）-简单队列服务
消息被读取后会从队列中删除，消息不会被推送到接收者，接收方必须轮询或从SQS提取消息，并且消息不能同时被多个接收者接收。

## 6.2 SNS（Simple Notification Service）-简单提示服务
不保留消息，一旦触发，会发送给所有订阅者。

## 6.3 Kinesis
Amazon Kinesis可以让你轻松收集、处理和分析**实时流数据**。利用Amazon Kinesis，你可以在收到数据的同时对数据进行处理和分析，无需等到数据全部收集完成才进行处理。

## 6.4 MQ
是一种托管消息代理服务，适用于RabbitMQ和ActiveMQ这两种技术，扩展性没有SQS/SNS好。

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
通过机器学习，异常检测和第三方数据（日志分析）去发现威胁，**可以使你免受加密货币的攻击**。

### 7.2.7 Inspector
对EC2实例和ECR容器映像和Lambda函数做一个扫描，找出漏洞，按等级列出风险评估列表。

### 7.2.8 Config
Config负责跟踪所有创建、删除或管理的资源，通过记录配置及其随时间的变化来帮助审核和记录资源的合规性。

### 7.2.9 Macie
一项管理数据安全和数据隐私的服务，可以帮你把存储在S3的数据归类为哪些是个人身份信息，即**PII（personally identifiable information）**。

### 7.2.10 Security Hub
这是一种让你拥有中央安全工具来**管理不同AWS账号（Multi Account）** 之间的安全性并自动执行安全检查的方法。<br>
多重安全检查的工具，将跨多个账号，把所有这些警报发送到一个称为安全中心的位置。

### 7.2.11 Detective
Detective可以帮助你找到安全警报发生的根本原因。

### 7.2.12 Abuse
如果你发现AWS的某些资源或者IP存在滥用或者违法的行为，可以举报至这个abuse@amazonaws邮箱。

### 7.2.13 Root用户特权
根用户可以访问所有的AWS服务和资源，以下是只有根用户才有的特权（只是列出了可能会考试的一部分，没有完全列出）：
1. 改变账号设置（账号的名字邮箱密码和用户access key）
2. 关闭AWS账号
3. 改变或者取消AWS Support Plan。
4. 在Reserved Instance Marketplace注册为一个卖方。

## 7.3 身份认证
### 7.3.1 AWS STS（Security Token Service）-安全令牌服务
它可以让你创建临时有限的权限去访问你的AWS资源。

### 7.3.2 Cognito
为AWS以外的外部用户提供身份验证，比如Web和移动应用程序的用户，也可以与谷歌，facebook等账号连用。

### 7.3.3 Directory Services
Directory Services是Microsoft Active Directory（AD）集成到AWS的服务。

### 7.3.4 IAM Identity Center
账户中心，登录后，可以访问所有账户的控制台，管理多个AWS账户和云应用程序的访问。

# 8.账单和支持
## 8.1 定价基础知识
### 8.1.1 定价模式
1. 按需付费（Pay as you go）
2. 预留容量，付费更少
3. 用量越大，费用越少
4. AWS规模越大，价格越低

### 8.1.2 免费服务
1. VPC
2. Elastic Beanstalk
3. CloudFormation
4. IAM
5. Auto Scaling
6. Consolidated Biling
7. 新手免费套餐

## 8.2 成本计算器
### 8.2.1 估算成本
**Pricing Calculator**根据选定的服务估算价格。

### 8.2.2 追踪成本
#### 8.2.2.1 Billing Dashboard
计费仪表板，会显示当月的所有成本。

#### 8.2.2.2 Cost Allocation Tags
成本分配标签，它允许我们跟踪我的成本在一个详细的等级或者组（Resource Group）中。

#### 8.2.2.3 Cost and Usage Reports
可以生成每月甚至每小时的成本和用户报告，**可以获得最全面，最细粒度的报告**，以深入了解你的成本和使用情况。

#### 8.2.2.4 Cost Explorer
1. 成本浏览器，我们可以可视化的理解和管理成本。<br>
2. 可以获得每月甚至每小时的成本数据。<br>
3. 可以选择一个最优的**Saving Plan**去降低成本。<br>
4. **可以预测长达12个月的使用成本。**

### 8.2.3 监控成本
#### 8.2.3.1 Billing Alarms
可以在Billing Alarms设置账单阀值，如果超过阀值，就会收到通知。

#### 8.2.3.2 Budgets
创建预算，当超过预算，就会发出警报。

#### 8.2.3.3 Cost Anomaly Detection
持续监控成本，利用机器学习，通过对历史数据的分析，去预测未来成本，在超出阀值时发出警报。

## 8.3 组织和合并支付
### 8.3.1 组织（Organizations）
1. 一个全球服务，其理念是通过创建一个组织，你可以管理多个AWS账户，通过共同支付，共享资源等降低成本。
2. 可以使用SCP（Service Control Policies）限制账户权限，SCP对主账号（Master Account）没有影响，被限制的AWS账号，即使用户是Root用户，也会被限制权限。
3. 可以按业务单位OU（Organizational Units）组织账号，类似用户组管理用户，一个OU可能被包含在其它OU中，并且一个账号可以属于多个OU。

### 8.3.2 Control Tower
通过最佳实践，帮助用户设置和管理安全，合规，多账户的AWS环境。Control Tower位于组织（Organizations）的顶层，它将自动为您设置组织以组织您的账户。

### 8.3.3 Service Catalog
在很多组织中，研发部门可能都要启动一些AWS资源来进行内部测试。如果不同的研发人员都去启动EC2实例，然后选择AMI，选择实例类型等等，那么最终的资源的规格肯定是五花八门。
<br><br>
这样对于组织后续的管理将会带来很大的麻烦，无法进行一致性的监管，也无法满足合规性要求，甚至可能会自行选择一些配置比较高而用不到或需要使用付费的实例用于测试，这样还会导致成本浪费。<br><br>
所以Service Catalog服务，就可以解决这些问题。通过**预先定义好允许启动的资源，做好相应的配置，放到一个集中的地方提供给研发进行快速启动和使用。** 这样的话，研发同学只要页面点击几次就可以进行部署，而且也可以遵循组织的一致性标准。

## 8.4 支持计划（Support Plan）
支持计划类型：
1. 基础支持计划
2. 开发人员支持计划
3. 商业支持计划
4. 企业支持计划

[AWS不同的支持计划对比](https://us-east-1.console.aws.amazon.com/support/plans/home?region=us-east-1#/?refid=how-to-take-advantage-of-the-aws-free-tier_cfm-blog_link)

[AWS Acceptable Use Policy 链接](https://aws.amazon.com/cn/aup/)

## 8.5 其它服务
### 8.5.1 Trusted Advisor 
通过优化AWS环境降低成本，提高性能并提高安全性的在线工具。<br>
提供实时指导，帮助按照AWS最佳实践配置资源。

### 8.5.2 Compute Optimizer
通过优化AWS资源去提高性能和减少开支。<br>
支持的资源有：
1. EC2
2. ASG
3. EBS
4. Lamdba

### 8.5.3 AWS Service Quotas
通过设置服务的限额去管理服务的额度，超过限额会发送通知到CloudWatch。

# 9.构架和生态
## 9.1 完善的构架（Well-Architected Framework）
一个用于设计基础设施的指南，一种评估和实施架构的系统的方法，从众多经验教训中建立起来的最佳实践。

框架的支柱：
1. 卓越运营（Operational Excellence）
    - 利用代码执行操作
    - 注释文档
    - 定期进行一些增量的微小变更
    - 经常优化程序
    - 预见故障
    - 从操作事件及故障中总结经验
2. 安全性（Security）
    - 身份验证机制
    - 启用追踪性
    - 在所有层应用安全性
    - 自动实施安全性最佳实践
    - 保护传输中的数据和静态数据
    - 做好安全性事件应对准备
3. 可靠性（Reliability）
    - 测试恢复流程
    - 自动故障恢复
    - 横向扩展提升总体系统可用性
    - 无需猜测容量
    - 自动管理变更
4. 性能效率（Performance Efficiency）
    - 普及先进技术
    - 数分钟内实现全球化部署
    - 使用无服务架构
    - 更频繁进行试验
    - 制度化选择
5. 成本优化（Cost Optimization）
    - 采用消费模型
    - 衡量总体效率
    - 无需再为数据中心运营投入资金
    - 分析支出和确定支出属性
    - 使用托管服务降低拥有成本
6. 可持续性（Sustainability）
    - 了解你的影响
    - 达到可持续目标
    - 最大限度利用你的服务
    - 预测并采用新的，更高效的硬件和软件
    - 使用AMS（AWS Managed Service）
    - 减少云工作的下游负载

## 9.2 生态（Ecosystem）
AWS Blogs：https://aws.amazon.com/blogs/aws<br>
AWS Forums（community）：https://forums.amazon.com/index.jspa<br>
AWS Whitepapers & Guides：https://aws.amazon.com/whitepapers<br>
AWS Quick Starts：https://aws.amazon.com/quickstart<br>
AWS Solutions：https://aws.amazon.com/solutions<br>

# 10.其它服务
## 10.1 机器学习
### 10.1.1 Rekognition
物体识别，人脸识别，文本检测，Pathing（例如运动中的路径选择）。

### 10.1.2 Transcribe
将语言转成文本。

### 10.1.3 Polly
将文本转成语言。

### 10.1.4 Translate
翻译。

### 10.1.5 Comprehend
一种自然语言处理，用于文字分析，提取文字中的情绪主题等。

### 10.1.6 SageMaker
创建机器学习模型。

### 10.1.7 Forecast
把数据上传到S3，Forecast会利用数据预测结果。

### 10.1.8 Kendra
文档搜索服务。

### 10.1.9 Personalize
提供个性化推荐。

### 10.1.10 Textract
用于提取文本，扫描文本证件等，提取信息。

### 10.1.11 Lex & Connect
Lex类似于Alexa，像小米机器人那种，可以智能聊天，Connect可以创建智能客服中心。

## 10.2 WorkSpaces
DaaS（Desktop as a Service）服务，让你可以轻松的调用Linux或者Windows桌面。<br>
它将是一个允许你消除所谓内部**虚拟桌面VDI（Virtual Desktop Infrastructure）** 的解决方案。

## 10.3 Amazon AppStream 2.0
可以把应用通过浏览器展示。

## 10.4 AWS IoT Core
物联网，可以把IoT设备轻松的连接到AWS云中。

## 10.5 Amazon Elastic Transcoder
把存放在S3中的视频等流媒体文件进行转码，转成合适播放的格式。

## 10.6 AppSync
实时的同步数据，利用了facebook的**GraphQL**技术

## 10.7 Amplify
帮助你开发部署web和手机应用，目标是构建移动程序后端。

## 10.7 Device Farm
可以在Device Farm上测试不同的设备。

## 10.8 Backup
备份服务，通过制定备份计划，可以把数据备份到S3上。

## 10.9 Disaster Recovery Strategies
备份服务，通过制定备份计划，可以把数据备份到S3上。

## 10.10 DataSync
把大量数据同步到S3，EFS，FSx for Windows上。

## 10.11 AWS Fault Injection Simulator（FIS）
错误注入模拟器，基于**Chaos Engineering**，通过对基础设施的破坏，确保应用程序是真正可靠的。

## 10.12 AWS Step Funtions
可视化构建工作流。

## 10.13 Ground Station
控制卫星通信，处理数据和扩展卫星操作。

## 10.14 Pinpoint
消息推送，营销传播服务。
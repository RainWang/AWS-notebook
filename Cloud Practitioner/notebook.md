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
1. 实例重启后，公用IP会变化，私有IP不会变化。
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

### 2.4.4 Athena-S3查询工具
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

![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/7.png)

## 3.3 Batch
1. Batch是个托管批处理服务，**Batch Job是一个运行在ECS上的Docker映像**，这意味着任何可以在ECS上运行的都可以在Batch上运行。
2. 无时间空间限制。

# 4.部署管理服务
## 4.2 CloudFormation
1. 通过模版文件自动化创建资源组件，模版文件分为Json，Yaml两种格式。
2. 在模版文件中，“Resources”是不能缺少的。
3. 删除堆栈时，通过模版创建的资源组件（比如EC2，安全组等）都会被删除。

[CloudFormation 的官方模板下载链接](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html#w2ab1c28c58c13c17)

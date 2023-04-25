# 1.云的基本知识
## 1.1 边缘站点（Edge Locations）和区域边缘缓存（Regional Edge Caches）

1. “区域边缘缓存”的容量通常大于“边缘站点”。
2. 访问高的数据存放于“边缘站点”，访问低的数据存放于“区域边缘缓存”。

例：<br>
用户向S3请求数据时，会在CloudFront中缓存一份数据，CloudFront会在“区域边缘缓存”中缓存一份数据，“区域边缘缓存”又会在“边缘站点”缓存一份数据。<br>
“边缘站点”会定期清理不常用数据，释放空间给经常访问的数据。<br>
当用户在“边缘站点”访问不到数据时，会再向“区域边缘缓存”请求数据，逐级往上。

![边缘站点与区域边缘缓存的关系例图](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/1.png)

## 1.2 可用区（Availability Zone）命名规则

1. 区域名称+数字+字母
2. 同一个“可用区”在不同账号中，可能显示不同的名称。

![可用区命名规则](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/2.png)

# 2.核心服务
## 2.1 IAM（Identity and Access Management）
不建议用Root用户登录AWS，建议通过IAM登录AWS。
![IAM](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/3.png)

![IAM授权使用案例](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/4.png)

[MFA下载页面](https://aws.amazon.com/cn/iam/features/mfa/)

## 2.2 CloudFormation
1. 通过模版文件自动化创建资源组件，模版文件分为Json，Yaml两种格式。
2. 在模版文件中，“Resources”是不能缺少的。
3. 删除堆栈时，通过模版创建的资源组件（比如EC2，安全组等）都会被删除。

[CloudFormation 的官方模板下载链接](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-2.html#w2ab1c28c58c13c17)

## 2.3 Lambda-无服务计算
核心概念：事件驱动和函数，支持多种代码语言。
![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/5.png)

## 2.4 Redshift-数据仓库
1. 快速并且完全托管的云端大规模并行PB级数据仓库，可以让用户以SQL和现有的商业智能工具对PB级的结构化数据进行大规模并行查询，并且在几秒内返回结果。

2. 数据库和数据仓库的区别：
>（1） 用途<br>
数据库：主要用于事务处理，即OLTP（Transaction），也就是我们常用的面向业务的增删改查操作。常用的数据库有Mysql，Oracle，PostgreSQL。<br><br>
数据仓库：主要用于数据分析，即OLAP（Analytics），供上层决策，常见于一些查询性的统计数据。常见的数仓有Greenplum，Hive。基于MYISAM存储引擎的MySQL也是可以用来做数据仓库的。<br><br>
>（2）特性<br>
数据库：因为是事务性操作，所以一般是读写优化的。读写相对简单，一次只是对少量数据进行操作。<br><br>
数据仓库：因为是数据分析，需要对大量数据进行查询，所以一般仅仅是读优化的。查询相对复杂，一次要对大量数据进行操作。

3. 建立OLAP应用之前，我们要想办法把各个独立系统的数据抽取出来，经过一定的转换和过滤，存放到一个集中的地方，成为数据仓库。这个抽取，转换，加载的过程叫ETL（Extract， Transform，Load）。

![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/6.png)

![Lambda运行机制](https://1006493605.s3.ap-northeast-1.amazonaws.com/notebook/Cloud_Practitioner/7.png)

## 2.5 CloudWatch-监控服务

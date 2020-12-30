### 前言
写文档设计完，之后写代码简直就是行云流水。

### 什么是开放平台？
京东云，淘宝开放平台，腾讯开放平台，微信，顺丰，阿里巴巴
### 流程
![](https://tva1.sinaimg.cn/large/0081Kckwly1gm68nibtjcj30wi04iaet.jpg)
很少去做详细设计
### 如何编写概要设计说明书
![](https://tva1.sinaimg.cn/large/0081Kckwly1gm68qx3jwyj30da0cater.jpg)
<br/>写文档不仅仅是为了自己明白，是为了让所有人都明白
<br/>
### 概要设计说明书
<br/>

**1.需求分解**
<br/>开发者中心：查看api文档，注册称为开发者，新增app
<br/>网关：让第三方应用，以http的方式调用进来，流控中心,计费模块，熔断机制
<br/>授权中心：业务授权，提供用户授权的流程
<br/>控制后台：审核操作
<br/>
<br/>

**2. 架构设计**
<br/>
2.1架构图<br/>

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm6a9gdmllj30o60viqg8.jpg)<br/>
1. 搞清楚整个业务中参与的节点有哪些？<br/>
2. 各个节点如何进行通信<br/>
3. 各个节点有哪些子模块<br/>

2.2关系图
![](https://tva1.sinaimg.cn/large/0081Kckwly1gm6ajr8akrj30y80jih1i.jpg)

2.3 交互图

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm6b3kw8kyj31cs0u07wh.jpg)

2.4 系统划分

|系统模块 | 功能概要描述 |
|---|---|
|API网关|1. 实现具体的API调用，以HTTP的协议 <br/> 2.调用的同时对权限、用户授权、流控、账户验证 <br/> 3.用业务服务完          成业务完成调用 |
|开发者中心|1.提供api文档<br>2.注册成为开发者<br> 3.创建申请APP调用<br> 4.付费充值|
|授权中心|1.为第三方的应用提供授权，生成访问Token（对外）<br> 2.提供授权验证服务（内部）|
|控制后台|1.管理者审核开发资质，审核应用<br>2.权限设置<br>3.流控设置|

**3.技术选型**

网关：http协议，流控：Redis， mysql（不建议）

授权中心： OAuth2.0, 自己实现这个OAuth2.0

开发者中心：动态生成API文档结构。java反射实现文档结构的自动生成。

控制后台：动态配置的推送，zookeeper 

**4.授权中心**

![](https://tva1.sinaimg.cn/large/0081Kckwly1gm6btiwyd3j31360nkkb4.jpg)








# SpringCloud优惠券系统
1.    项目介绍及架构
2.  注册中心
3.  网关
    
4.  service模块
	* 公共模块
	* 优惠券模板模块
	* 优惠券分发模块
	* 优惠券结算模块
    
5.  项目难点与亮点

## 项目介绍
本项目将优惠券系统拆分，实现了优惠券分发，结算，核销，考虑了面对高并发的场景。
技术栈：SpringCloud + Redis + Caffeine + Kafka + Eureka +Hystrix
整体的架构：
微信图片_20230304170830.png
![输入图片说明](/image/1_20230304170830.png)

## 注册中心
使用Eureka作为注册中心，Eureka Server的配置，其中application.yml 中是单机作为注册中心的配置，bootstrap.yml中是多台设备作为注册中心。


## 网关

使用Zuul作为网关，实现了限流、token的过滤。
TODO 在网关异步实现请求日志的保存
主要实现了四个过滤器：

 1. 记录请求开始的时间戳过滤器：PreRequestFilter
 2. token过滤器：校验请求传递的token
 3. 限流过滤器：在token之后执行
 4. post过滤器：记录一次正常请求的时间


## service模块

### 优惠券模板

![输入图片说明](/image/2_20230304170059.png)
**核心：运营人员设定的条件构造优惠券模板，-》异步+优惠券码**

优惠券码（18位）：产品线 + 类型（前4位）+ 随机日期 （6位）+ 0~9随机数（后8位）
> 优惠券与优惠券模板有大量相同的信息，为什么还有优惠券模板

-   只需要输入一遍优惠券模板信息，就可以批量生成优惠券信息
    
-   创建优惠券模板和领取优惠券，是不同的人完成的，服务需要隔离

优惠券模板规定有使用期限，有两种过期策略：

1.  template模块使用自己定期清理策略
    
2.  使用template模块的其他模块自己校验（因为1中会有延迟）

### 优惠券分发
![输入图片说明](/image/3_20230304170813.png)

> 根据用户id和优惠券状态查找用户优惠券记录

1.  由于不涉及用户系统，用户id不做校验，需要fake
    
2.  优惠券状态有三类：可用的，已使用的，过期的
    
3.  用户数据存储于Redis之中
    
4.  除了获取用户的优惠券之外，还有延迟的过程处理策略
    

> 根据用户id查找当前可以领取的优惠券模板

1.  优惠券模板从template模块中获取（熔断兜底策略）
    
2.  根据优惠券的领取限制，比对当前用户所拥有的优惠券做出判断
    


> 领取优惠券

1.  优惠券模板从template模块获取（熔断兜底策略）
    
2.  根据优惠券的领取限制，比对当前用户所拥有的优惠券，判断是否可以领取
    
3.  从Redis中获取优惠券码
    
4.  优惠券写入MySQL和Redis中
    
### 结算模块
![输入图片说明](/image/4_20230304170729.png)
> 结算（核销）优惠券

1.  校验需要结算的优惠券是否是合法的
    
2.  利用settlement模块计算结算数据
    
3.  如果是核销，需要写入数据库
    

**结算模块**

> 根据优惠券类型结算优惠券

1.  优惠券是分类的，不同的优惠券是有不同的计算方法
    
2.  不同类的优惠券是可以组合，所以，也需要不同的计算方法
    
## 项目难点

<!--stackedit_data:
eyJoaXN0b3J5IjpbNDE2MzMwNDJdfQ==
-->

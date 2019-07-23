### dubbo学习

#### 1.分布式基础理论

```
“分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”

分布式系统（distributed system）是建立在网络之上的软件系统。

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进。
```

#### 2.发展演变

##### 单一应用架构

```
当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。
适用于小型网站，小型管理系统，将所有功能都部署到一个功能里，简单易用。
缺点： 1、性能扩展比较难 
       2、协同开发问题
       3、不利于升级维护
```

##### 垂直应用架构

```
当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。
通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展也更方便，更有针对性。
缺点： 公用模块无法重复利用，开发性的浪费
```

##### 分布式服务架构

```
当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。
```

##### 流动计算架构

```
当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)[ Service Oriented Architecture]是关键。
```

#### RPC

什么是RPC

```
RPC【Remote Procedure Call】是指远程过程调用，是一种进程间通信方式，他是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同。
```

#### dubbo核心概念

```
Apache Dubbo (incubating) |ˈdʌbəʊ| 是一款高性能、轻量级的开源Java RPC框架，它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。
```

![1563894250364](C:\Users\DH\AppData\Roaming\Typora\typora-user-images\1563894250364.png)

##### **服务提供者（Provider）**：

```
暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。
```

##### **服务消费者（Consumer）**: 

```
调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
```

##### **注册中心（Registry）**：

```
注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
```

##### **监控中心（Monitor）**：

```
服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心
```

##### 调用关系说明

```
	服务容器负责启动，加载，运行服务提供者。
	服务提供者在启动时，向注册中心注册自己提供的服务。
	服务消费者在启动时，向注册中心订阅自己所需的服务。
	注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
	服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
	服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。
```

#### dubbo环境搭建

##### 1、下载zookeeper

```
网址 https://archive.apache.org/dist/zookeeper/zookeeper-3.4.13/
```

##### 2、解压zookeeper

```
解压运行zkServer.cmd ，初次运行会报错，没有zoo.cfg配置文件
```

##### 3、修改zoo.cfg配置文件

```
将conf下的zoo_sample.cfg复制一份改名为zoo.cfg即可。
注意几个重要位置：
dataDir=./   临时数据存储的目录（可写相对路径）
clientPort=2181  
zookeeper的端口号
修改完成后再次启动zookeeper
```

##### 4、使用zkCli.cmd测试

```
ls /：列出zookeeper根下保存的所有节点
create –e /atguigu 123：创建一个atguigu节点，值为123
get /atguigu：获取/atguigu节点的值
```

#### zookeeper

##### 1.下载zookeeper的安装包

```
下载地址为：https://mirrors.cnnic.cn/apache/zookeeper/
```

##### 2.解压下载的压缩包，修改配置文件

```
修改目录下zookeeper-3.4.10\conf\zoo_sample.cfg文件内容，并将文件名修改为zoo.cfg。

        实际上只需要修改dataDir和dataLogDir两个地方即可，dataDir为Zookeeper 保存数据的目录，dataLogDir为Zookeeper 保存日志文件的目录。
```

##### 3.启动zookeeper

```
找到目录\zookeeper-3.4.10\bin，双击zkServer.cmd，直接运行；这种方法遇到问题会闪退。
打开cmd,进入到\zookeeper-3.4.10\bin目录下，运行zkServer.cmd。
```

#### dubbo简单实例

##### 1.一共分三个模块，每个模块都有独立的pom文件

api内为公用接口，consumer调用远程服务，provider提供远程服务。

##### 2.在consumer和provider的pom中添加依赖

```
<dependencies>
    	<dependency>
    		<groupId>com.alibaba</groupId>
    		<artifactId>dubbo</artifactId>
    		<version>2.6.0</version>
    	</dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo-demo-api</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.9</version>
        </dependency>
    </dependencies>
```

##### 3.在dubbo-demo-api内添加接口

```
public interface DemoService {
 		String sayHello(String name);
}
```

##### 4..在dubbo-demo-provider内实现接口

```
public class DemoServiceImpl implements DemoService {
 
    public String sayHello(String name) {
        System.out.println("[" + new SimpleDateFormat("HH:mm:ss").format(new Date()) + "] Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name + ", response form provider: " + RpcContext.getContext().getLocalAddress();
    }
 
}
```

##### 5.修改配置provider文件的内容为：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">


    <!--指定当前服务/应用的名字（同样的服务名字相同，不要和别的服务同名）-->
    <dubbo:application name="user-service-provider" owner="programmer" organization="dubbo"></dubbo:application>
    <!--指定注册中心的位置-->
    <!--<dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>-->
    <dubbo:registry protocol="zookeeper" address="127.0.0.1:2181"></dubbo:registry>

    <!--制定通信规则(通信协议？通信端口)-->
    <dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>
    <!--暴露服务,ref实现真正的实现对象-->

    <dubbo:service interface="com.hhf.gmall.service.UserService" ref="userServiceImpl" protocol="dubbo"></dubbo:service>

    <bean id="userServiceImpl" class="com.hhf.gmall.service.impl.UserServiceImpl"></bean>
</beans>
```

##### 6.启动服务，注意要先启动zookeeper

```
public class MainApplication {

    public static void main(String args[]) throws IOException {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("provider.xml");
        ioc.start();
        System.in.read();
    }
}
```

##### 7.在dubbo-demo-consumer内调用服务
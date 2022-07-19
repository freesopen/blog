> ELK即Elasticsearch、Logstash、Kibana,组合起来可以搭建线上日志系统，本文主要讲解使用ELK来收集SpringBoot应用产生的日志。

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#学习前需要了解的内容) 学习前需要了解的内容

- [开发者必备Docker命令](https://www.macrozheng.com/mall/reference/docker_command.html)
- [使用Docker Compose部署SpringBoot应用](https://www.macrozheng.com/mall/reference/docker_compose.html)
- [SpringBoot应用中使用AOP记录接口访问日志](https://www.macrozheng.com/mall/reference/aop_log.html)

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#elk中各个服务的作用) ELK中各个服务的作用

- Elasticsearch:用于存储收集到的日志信息；
- Logstash:用于收集日志，SpringBoot应用整合了Logstash以后会把日志发送给Logstash,Logstash再把日志转发给Elasticsearch；
- Kibana:通过Web端的可视化界面来查看日志。

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#使用docker-compose-搭建elk环境) 使用Docker Compose 搭建ELK环境

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#需要下载的docker镜像) 需要下载的Docker镜像

```bash
docker pull elasticsearch:6.4.0
docker pull logstash:6.4.0
docker pull kibana:6.4.0
```

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#搭建前准备) 搭建前准备

- elasticsearch 启动成功需要特殊配置，具体参考[mall在Linux环境下的部署（基于Docker Compose）](https://mp.weixin.qq.com/s/JYkvdub9DP5P9ULX4mehUw)

- 中的elasticsearch部分；
- docker-compose.yml文件地址:https://github.com/macrozheng/mall-learning/blob/master/mall-tiny-elk/src/main/docker/docker-compose.yml
- logstash-springboot.conf配置文件地址：https://github.com/macrozheng/mall-learning/blob/master/mall-tiny-elk/src/main/docker/logstash-springboot.conf

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#开始搭建) 开始搭建

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#创建一个存放logstash配置的目录并上传配置文件) 创建一个存放logstash配置的目录并上传配置文件

##### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#logstash-springboot-conf文件内容) logstash-springboot.conf文件内容

```text
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "es:9200"
    index => "springboot-logstash-%{+YYYY.MM.dd}"
  }
}
```

##### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#创建配置文件存放目录并上传配置文件到该目录) 创建配置文件存放目录并上传配置文件到该目录

```bash
mkdir /mydata/logstash
```

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#使用docker-compose-yml脚本启动elk服务) 使用docker-compose.yml脚本启动ELK服务

##### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#docker-compose-yml内容) docker-compose.yml内容

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:6.4.0
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #设置使用jvm内存大小
    volumes:
      - /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /mydata/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
      - 9300:9300
  kibana:
    image: kibana:6.4.0
    container_name: kibana
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - "elasticsearch.hosts=http://es:9200" #设置访问elasticsearch的地址
    ports:
      - 5601:5601
  logstash:
    image: logstash:6.4.0
    container_name: logstash
    volumes:
      - /mydata/logstash/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
```

##### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#上传到linux服务器并使用docker-compose命令运行) 上传到linux服务器并使用docker-compose命令运行

```bash
docker-compose up -d
```

注意：Elasticsearch启动可能需要好几分钟，要耐心等待。 ![img](assets/tech_screen_01.52b03edd.png)

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#在logstash中安装json-lines插件) 在logstash中安装json_lines插件

```bash
# 进入logstash容器
docker exec -it logstash /bin/bash
# 进入bin目录
cd /bin/
# 安装插件
logstash-plugin install logstash-codec-json_lines
# 退出容器
exit
# 重启logstash服务
docker restart logstash
```

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#开启防火墙并在kibana中查看) 开启防火墙并在kibana中查看

```bash
systemctl stop firewalld
```

访问地址：http://192.168.3.101:5601

![img](assets/tech_screen_02.14052f31.png)

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#springboot应用集成logstash) SpringBoot应用集成Logstash

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#在pom-xml中添加logstash-logback-encoder依赖) 在pom.xml中添加logstash-logback-encoder依赖

```xml
<!--集成logstash-->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>5.3</version>
</dependency>
```

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#添加配置文件logback-spring-xml让logback的日志输出到logstash) 添加配置文件logback-spring.xml让logback的日志输出到logstash

> 注意appender节点下的destination需要改成你自己的logstash服务地址，比如我的是：192.168.3.101:4560 。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <!--应用名称-->
    <property name="APP_NAME" value="mall-admin"/>
    <!--日志文件保存路径-->
    <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
    <contextName>${APP_NAME}</contextName>
    <!--每天记录日志到文件appender-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${APP_NAME}-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    <!--输出到logstash的appender-->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <!--可以访问的logstash日志收集端口-->
        <destination>192.168.3.101:4560</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#运行springboot应用) 运行Springboot应用

![img](assets/tech_screen_03.6e5e33d4.png)

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#在kibana中查看日志信息) 在kibana中查看日志信息

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#创建index-pattern) 创建index pattern

![img](assets/tech_screen_04.392f4d69.png)

![img](assets/tech_screen_05.038f1c0a.png)

![img](assets/tech_screen_06.b5ff7766.png)

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#查看收集的日志) 查看收集的日志

![img](assets/tech_screen_07.49481d40.png)

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#调用接口进行测试) 调用接口进行测试

![img](assets/tech_screen_08.0208beae.png)

![img](assets/tech_screen_09.688d0645.png)

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#制造一个异常并查看) 制造一个异常并查看

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#修改获取所有品牌列表接口) 修改获取所有品牌列表接口

![img](assets/tech_screen_10.ad07daa0.png)

#### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#调用该接口并查看日志) 调用该接口并查看日志

![img](assets/tech_screen_11.63b5329a.png)

### [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#总结) 总结

搭建了ELK日志收集系统之后，我们如果要查看SpringBoot应用的日志信息，就不需要查看日志文件了，直接在Kibana中查看即可。

## [#](https://www.macrozheng.com/mall/reference/mall_tiny_elk.html#项目源码地址) 项目源码地址

https://github.com/macrozheng/mall-learning/tree/master/mall-tiny-elk



关于kibana 汉化

1.2 进入容器

```java
docker exec -it Kibana容器id bash
```

1.3 进入config 目录下

```java
cd config/
```

1.4 编辑 kibana.yml 文件

```java
vi kibana.yml 
```

```java
i18n.locale: "zh-CN"
```

![在这里插入图片描述](assets/20210219092736140.png)

```yml
docker restart Kibana容器id
```
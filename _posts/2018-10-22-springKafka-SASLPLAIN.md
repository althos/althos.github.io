---
layout:     post
title:      Springboot+Kafka+SASL/PLAIN配置
subtitle:   spring-kafka+SASL/PLAIN配置
date:       2018-10-22
author:     Cyber
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
- 微信
- springboot
- java
- kafka
- 开发
---



>  基于上一篇kafka配置SASL/PLAIN之后。。。



### 引入环境需要的jar

>```
><dependencies>
>    <dependency>
>        <groupId>org.springframework.boot</groupId>
>        <artifactId>spring-boot-starter-web</artifactId>
>        <exclusions><!-- 去除 -->
>            <exclusion>
>                <groupId>org.springframework.boot</groupId>
>                <artifactId>spring-boot-starter-tomcat</artifactId>
>            </exclusion>
>        </exclusions>
>    </dependency>
>
>    <dependency><!--我用的jetty-->
>        <groupId>org.springframework.boot</groupId>
>        <artifactId>spring-boot-starter-jetty</artifactId>
>        <version>1.2.2.RELEASE</version>
>       <!-- <scope>provided</scope>-->
>    </dependency>
>    <dependency>
>        <groupId>org.springframework.boot</groupId>
>        <artifactId>spring-boot-starter-aop</artifactId>
>    </dependency>
>    <dependency>
>        <groupId>org.springframework</groupId>
>        <artifactId>spring-context-support</artifactId>
>    </dependency>
>    <dependency>
>        <groupId>org.springframework.kafka</groupId>
>        <artifactId>spring-kafka</artifactId>
>    </dependency>
>      <dependency>
>          <groupId>javax.servlet</groupId>
>          <artifactId>javax.servlet-api</artifactId>
>          <version>3.1.0</version>
>          <!--<scope>provided</scope>-->
>      </dependency>
>    <dependency>
>        <groupId>org.springframework</groupId>
>        <artifactId>spring-context-support</artifactId>
>    </dependency>
></dependencies>
>```

### spring-kafka配置

> ```
> server:    #容器端口
>   port: 8089
>   servlet:
>     context-path:
> 
> spring:
>   kafka:
>     bootstrap-servers: 127.0.0.1:9092
>     producer:
>       key-serializer: org.apache.kafka.common.serialization.StringSerializer
>       value-serializer: org.apache.kafka.common.serialization.StringSerializer
>       properties:    #如果不是SASL/PLAIN就不要这个属性
>         sasl.mechanism: PLAIN
>         security.protocol: SASL_PLAINTEXT
>     consumer:
>       group-id: test-consumer-group  ##这个是系统默认的，自己也可以去自定义
>       enable-auto-commit: true
>       # auto-offset-reset: latest
>       auto-commit-interval: 1000
>       key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
>       value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
>       properties:  #如果不是SASL/PLAIN就不要这个属性
>         sasl.mechanism: PLAIN
>         security.protocol: SASL_PLAINTEXT
> ```

### 读取配置文件

>```
>/**
> * Author:junru.pan
> * Email: panyiboy@gamil.com
> */
>@SpringBootApplication
>public class Applicatcion extends SpringBootServletInitializer {
>
>    public static void main(String[] args) {
>        SpringApplication.run(Applicatcion.class, args);
>    }
>    static {
>        //E:/sf_project/mgmt-itsc-hn-rlzp/Sf-api/src/main/resources
>        System.setProperty("java.security.auth.login.config",   "G:\\kafka_client_jaas.conf");  ##这个文件是上篇博客配的那个鬼文件
>    }
>
>    @Override
>    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
>        return application.sources(Applicatcion.class);
>    }
>}
>```

### producer

> ```
> @RestController
> @RequestMapping("kafka")
> public class ProducerController {
> 
>     @Autowired
>     private KafkaTemplate<String, String> kafkaTemplate;
> 
>     @RequestMapping("send")
>     public String send(String msg){
>         kafkaTemplate.send("test", msg);
>         return "success";
>     }
> 
> }
> ```

### comsumer

> ```
> /**
>  * Author:junru.pan
>  * Email: panyiboy@gmail.com
>  */
> @Component
> public class Listener {
>         @KafkaListener(topics = "test")
>         public void listen (ConsumerRecord<?, ?> record) throws Exception {
>             System.out.printf("topic = %s, offset = %d, value = %s \n", record.topic(), record.offset(), record.value());
>         }
> }
> ```

### 测试

> 启动时会展示信息：
>
> 	acks = 1
> 	batch.size = 16384
> 	bootstrap.servers = [127.0.0.1:9092]
> 	buffer.memory = 33554432
> 	client.id = 
> 	compression.type = none
> 	connections.max.idle.ms = 540000
> 	enable.idempotence = false
> 	interceptor.classes = null
> 	key.serializer = class org.apache.kafka.common.serialization.StringSerializer
> 	linger.ms = 0
> 	max.block.ms = 60000
> 	max.in.flight.requests.per.connection = 5
> 	max.request.size = 1048576
> 	metadata.max.age.ms = 300000
> 	metric.reporters = []
> 	metrics.num.samples = 2
> 	metrics.recording.level = INFO
> 	metrics.sample.window.ms = 30000
> 	partitioner.class = class org.apache.kafka.clients.producer.internals.DefaultPartitioner
> 	receive.buffer.bytes = 32768
> 	reconnect.backoff.max.ms = 1000
> 	reconnect.backoff.ms = 50
> 	request.timeout.ms = 30000
> 	retries = 0
> 	retry.backoff.ms = 100
> 	sasl.jaas.config = null
> 	sasl.kerberos.kinit.cmd = /usr/bin/kinit
> 	sasl.kerberos.min.time.before.relogin = 60000
> 	sasl.kerberos.service.name = null
> 	sasl.kerberos.ticket.renew.jitter = 0.05
> 	sasl.kerberos.ticket.renew.window.factor = 0.8
> 	sasl.mechanism = PLAIN
> 	security.protocol = SASL_PLAINTEXT
> 	send.buffer.bytes = 131072
> 	ssl.cipher.suites = null
> 	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
> 	ssl.endpoint.identification.algorithm = null
> 	ssl.key.password = null
> 	ssl.keymanager.algorithm = SunX509
> 	ssl.keystore.location = null
> 	ssl.keystore.password = null
> 	ssl.keystore.type = JKS
> 	ssl.protocol = TLS
> 	ssl.provider = null
> 	ssl.secure.random.implementation = null
> 	ssl.trustmanager.algorithm = PKIX
> 	ssl.truststore.location = null
> 	ssl.truststore.password = null
> 	ssl.truststore.type = JKS
> 	transaction.timeout.ms = 60000
> 	transactional.id = null
> 	value.serializer = class org.apache.kafka.common.serialization.StringSerialize

> 用postman调后台的producer接口,传msg值，listener展示收到的信息（我传的cyber）：
>
> topic = test, offset = 202, value = cyber
> topic = test, offset = 203, value = cyber

就这样，实作代码demo在[码云](https://gitee.com/junruPan/common-tools/tree/master/kafka-demo)


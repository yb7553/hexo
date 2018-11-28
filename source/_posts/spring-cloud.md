---
title: spring cloud微服务
date: 2018-11-28 10:37:21
tags:
---

![image.png](https://upload-images.jianshu.io/upload_images/5189695-fb5af178b28c633a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Spring Cloud包含多个子项目，Spring Cloud Config（可扩展配置服务）、Spring Cloud Netflix、Spring Cloud CloudFoundry（开源PaaS云平台）、Spring Cloud AWS（亚马逊云服务平台）、Spring Cloud Security、Spring Cloud Commons、spring Cloud Zookeeper、Spring Cloud CLI等项目。



## 一、服务注册中心

1、eureka服务注册中心

       Spring Cloud Eureka是Spring Cloud Netflix项目下的服务治理模块。而Spring Cloud Netflix项目是Spring Cloud的子项目之一，主要内容是对Netflix公司一系列开源产品的包装，它为Spring Boot应用提供了自配置的Netflix OSS整合。通过一些简单的注解，开发者就可以快速的在应用中配置一下常用模块并构建庞大的分布式系统。它主要提供的模块包括：服务发现（Eureka），断路器（Hystrix），智能路由（Zuul），客户端负载均衡（Ribbon）等。

      创建服务注册中心eureka-server，引入eureka依赖：



```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



通过@EnableEurekaServer注解启动一个服务注册中心提供给其他应用进行对话。



```
@EnableEurekaServer
@SpringBootApplication
public class Application {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```


在默认设置下，该服务注册中心也会将自己作为客户端来尝试注册它自己，所以我们需要禁用它的客户端注册行为，只需要在application.properties配置文件中增加如下信息：

#应用名
spring.application.name=eureka-server
#tomcat监听端口
server.port=1001
 
#eureka主机地址
eureka.instance.hostname=localhost
#禁用eureka客户端的注册行为
eureka.client.register-with-eureka=false
#如果为true，启动时报警.  
eureka.client.fetch-registry=false
在http://127.0.0.1:1001/ 可看到eureka的服务管理页面
      

 2、Consul服务治理


       Spring Cloud Consul项目是针对Consul的服务治理实现。Consul是一个分布式高可用的系统，它包含多个组件，但是作为一个整体，在微服务架构中为我们的基础设施提供服务发现和服务配置的工具。它包含了下面几个特性：

服务发现
健康检查
Key/Value存储
多数据中心
    Consul自身提供了服务端，所以我们不需要像之前实现Eureka的时候创建服务注册中心，直接通过下载consul的服务端程序就可以使用。

   consul服务器启动命令：

consul agent -dev
服务管理界面：http://127.0.0.1:8500



## 二、服务提供方

1、向eureka注册

      服务的客户端，并向服务注册中心注册自己

     创建一个基本的Spring Boot应用。命名为eureka-client，在pom.xml中，加入如下配置：


```
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-dependencies</artifactId>
           <version>Dalston.SR1</version>
           <type>pom</type>
           <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

   配置文件配置：

#指定微服务的名称后续在调用的时候只需要使用该名称就可以进行服务的访问
spring.application.name=eureka-client
#修改tomcat端口为2001
server.port=2001
#指定服务注册中心的位置
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/

并在服务提供方中尝试着提供一个接口来获取当前所有的服务信息。

```
@RestController
public class DcController {
	
	private final Logger log=Logger.getLogger(getClass());
	
	@Autowired
	private DiscoveryClient discoveryClient;//通过DiscoveryClient对象，在日志中打印出服务实例的相关内容。
    
	@RequestMapping(value="/dc")
	public String dc(){
		String services="eclipse services:"+discoveryClient.getServices();
		log.info(services);
		return services;
	}
}
```


最后在应用主类中通过加上@EnableDiscoveryClient注解，该注解能激活Eureka中的DiscoveryClient实现，这样才能实现Controller中对服务信息的输出。
@EnableEurekaClient
@SpringBootApplication
public class Application {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}


2、向consul注册

 接口和之前一样，只要改变依赖和配置信息就行

   添加Spring Cloud Consul依赖如下：
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
修改application.properties配置文件如下：


#consul服务注册与发现

```
spring.application.name=consul-client
spring.cloud.consul.host=127.0.0.1
spring.cloud.consul.port=8500
```




## 三、服务消费方

1、使用Spring Cloud提供的负载均衡器客户端接口LoadBalancerClient来实现服务的消费

创建一个服务消费者工程，命名为：eureka-consumer。并在pom.xml中引入依赖:



```
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
  
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
</dependencies>
```

配置application.properties，指定eureka注册中心的地址：


spring.application.name=eureka-consumers
server.port=2101
#服务注册中心地址
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
创建应用主类。初始化RestTemplate，用来真正发起REST请求。@EnableDiscoveryClient注解用来将当前应用加入到服务治理体系中。



```
@EnableDiscoveryClient//注解用来将当前应用加入到服务治理体系中。
@SpringBootApplication
public class Application {
 
	@Bean
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}
	
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```


创建一个接口用来消费eureka-client提供的接口：


```
@RestController
public class DcController {
 
	@Autowired
	private RestTemplate restTemplate;//利用RestTemplate对象实现对服务提供者接口的调用
	@Autowired 
	private LoadBalancerClient loadBalancerClient;
	
	@GetMapping("/consumer")//注解get请求
	public String dc(){
		//通过loadBalancerClient的choose负载均衡的选出一个eureka-client的服务实例
		ServiceInstance serviceInstance=loadBalancerClient.choose("eureka-client");
		String url="http://"+serviceInstance.getHost()+":"+serviceInstance.getPort()+"/dc";
		System.out.println(url);
		return restTemplate.getForObject(url, String.class);
	}
}
```

     

        可以看到这里，我们注入了LoadBalancerClient和RestTemplate，并在/consumer接口的实现中，先通过loadBalancerClient的choose函数来负载均衡的选出一个eureka-client的服务实例，这个服务实例的基本信息存储在ServiceInstance中，然后通过这些对象中的信息拼接出访问/dc接口的详细地址，最后再利用RestTemplate对象实现对服务提供者接口的调用。



2、Spring Cloud Ribbon

       Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。它是一个基于HTTP和TCP的客户端负载均衡器。它可以通过在客户端中配置ribbonServerList来设置服务端列表去轮询访问以达到均衡负载的作用。



导入依赖：



```
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
   
    <!-- 负载均衡的工具 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-ribbon</artifactId>
      <version>1.3.1.RELEASE</version>
    </dependency>
  </dependencies>
```

application.properties配置文件的配置不变，还是从eureka中拿服务。
修改应用主类。为RestTemplate增加@LoadBalanced注解：



```
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
 
	@Bean
	@LoadBalanced
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}
	
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```


修改Controller。去掉原来通过LoadBalancerClient选取实例和拼接URL的步骤，直接通过RestTemplate发起请求。


```
@RestController
public class DcController {
 
	@Autowired
	private RestTemplate restTemplate;
	
	@GetMapping("/consumer")
	public String dc(){
		return restTemplate.getForObject("http://eureka-client/dc", String.class);
	}
	/*
	 * Spring Cloud Ribbon有一个拦截器，它能够在这里进行实际调用的时候，自动的去选取服务实例，
	 * 并将实际要请求的IP地址和端口替换这里的服务名，从而完成服务接口的调用。
	 */
}
```


3、Spring Cloud Feign
       Spring Cloud Feign是一套基于Netflix Feign实现的声明式服务调用客户端。它使得编写Web服务客户端变得更加简单。我们只需要通过创建接口并用注解来配置它既可完成对Web服务接口的绑定。它具备可插拔的注解支持，包括Feign注解、JAX-RS注解。它也支持可插拔的编码器和解码器。Spring Cloud Feign还扩展了对Spring MVC注解的支持，同时还整合了Ribbon和Eureka来提供均衡负载的HTTP客户端实现。

pom.xml中的依赖：



```
<parent> 
 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
      <version>1.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-feign</artifactId>
      <version>1.3.1.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
```

修改应用主类。通过@EnableFeignClients注解开启扫描Spring Cloud Feign客户端的功能：

@EnableFeignClients//注解开启扫描Spring Cloud Feign客户端的功能
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}

创建一个Feign的客户端接口定义。使用@FeignClient注解来指定这个接口所要调用的服务名称，接口中定义的各个函数使用Spring MVC的注解就可以来绑定服务提供方的REST接口，比如下面就是绑定eureka-client服务的/dc接口的例子：

/*
 * 通过@FeignClient定义的接口来统一的生成我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可
 */
@FeignClient("eureka-client")//使用@FeignClient注解来指定这个接口所要调用的服务名称
public interface DcClient {
 
	@GetMapping("/dc")//接口中定义的各个函数使用Spring MVC的注解就可以来绑定服务提供方的REST接口
	public String consumer();
}

修改Controller。通过定义的feign客户端来调用服务提供方的接口：


```
@RestController
public class DcController {
 
	@Autowired
	private DcClient dcClient;
	
	@GetMapping("/consumer")
	public String consumer(){
		return dcClient.consumer();
	}
}
```




## 四、配置中心

       Spring Cloud Config为服务端和客户端提供了分布式系统的外部化配置支持。配置服务器为各应用的所有环境提供了一个中心化的外部配置。配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。



1、构建服务端Config-Server

     pom.xml中引入spring-cloud-config-server依赖，完整依赖配置如下：


 
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
    <relativePath/>
  </parent>
<dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    
    <!-- config-server也注册为服务，这样所有客户端就能以服务的方式进行访问 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    
  </dependencies>
  <dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Brixton.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```


application.properties中配置服务信息以及git信息，例如：

#服务器应用名
spring.application.name=config-server
#远程仓库地址
spring.cloud.config.server.git.uri=https://github.com/jlzl123/config-repo-demo.git
spring.cloud.config.server.git.username=jlzl123
spring.cloud.config.server.git.password=*******
          
server.port:1201
 
#eureka注册中心配置
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
spring.cloud.config.server.git.uri是git远程配置文件仓库地址



创建Spring Boot的程序主类，并添加@EnableConfigServer注解，开启Config Server:


package org.config.server.git;
 
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;
 
@EnableDiscoveryClient//用来将config-server注册到上面配置的服务注册中心上去。
@EnableConfigServer//开启Spring Cloud Config的服务端功能。
@SpringBootApplication
public class Application {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}

Spring Cloud Config也提供本地存储配置的方式。我们只需要设置属性spring.profiles.active=native，Config Server会默认从应用的src/main/resource目录下检索配置文件。也可以通过spring.cloud.config.server.native.searchLocations=file:F:/properties/属性来指定配置文件的位置。虽然Spring Cloud Config提供了这样的功能，但是为了支持更好的管理内容和版本控制的功能，还是推荐使用git的方式。


完成上面后我们就可以通过URL访问远程配置信息了：

http://localhost:1201/config-client-dev/dev/master




URL与配置文件的映射关系如下：
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
上面的url会映射{application}-{profile}.properties对应的配置文件，{label}对应git上不同的分支，默认为master。profile是配置文件对应的不同开发环境。
比如：要访问config-label-test分支，didispace应用的prod环境，可以通过这个url：http://localhost:7001/didispace/prod/config-label-test。



2、构建客户端config-client

pom.xml中引入spring-cloud-starter-config依赖，完整依赖关系如下：


```
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
			<exclusions><!-- 排除对[spring-boot-starter-logging]的依赖，解决log4j依赖jar包冲突 -->
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-log4j</artifactId>
			<version>1.3.8.RELEASE</version>
		</dependency>
 
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
	</dependencies>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Brixton.RELEASE</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
     
    <!--这个元素描述了项目相关的所有资源路径列表，例如和项目相关的属性文件，这些资源被包含在最终的打包文件里。-->
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
```



创建Spring Boot启动主类，并利用@EnableDiscoveryClient注解将客户端以服务的形式注册到eureka再服务注册中心。



```
package org.config.client;
 
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
 
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```


创建bootstrap.properties配置，来指定config server，例如：
#远程配置仓库配置文件名
spring.application.name=config-client-dev,test,config-test,datasource
#配置中心服务器地址
spring.cloud.config.uri=http://localhost:1201/
#当前配置环境，相同配置文件的不同环境，通过profile区分
spring.cloud.config.profile=dev
#git仓库分支，默认为master
spring.cloud.config.label=master
server.port=2003
#git的yml配置文件的格式好象有问题，最好用properties
#eureka注册中心配置
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/

这里需要格外注意：上面这些属性必须配置在bootstrap.properties中，config部分内容才能被正确加载。因为config的相关配置会先于application.properties，而bootstrap.properties的加载也是先于application.properties。
即配置客户端启动，先加载bootstrap.properties配置文件,然后从远程配置仓库获取配置，所以这些配置会优于application.properties的配置被容器加载，即远程仓库的配置会覆盖掉application的配置。这样，就可以直接把项目配置文件写在远程仓库。

注意，springboot并不加载bootstrap.properties，只用config client的jar封装ConfigServerBootstrapConfiguration先加bootstrap.properties。



创建一个Rest Api来返回配置中心的from属性，具体如下：


```
@RefreshScope
@RestController
class TestController {
    @Value("${from}")
    private String from;
    @RequestMapping("/from")
    public String from() {
        return this.from;
    }
}
```



也可以通过@ConfigurationProperties注解来绑定配置属性，创建如下配置类：



```
package org.config.client.dataSourceConfig;
 
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;
 
@Configuration
//@ConfigurationProperties注解绑定属性,prefix为属性名的前缀，如:jdbc.url
@ConfigurationProperties(prefix="jdbc",ignoreUnknownFields=false)
public class DataSourceProperties {
 
	private String driver;
	private String url;
	private String username;
	private String password;
	
	public String getDriver() {
		return driver;
	}
	public void setDriver(String driver) {
		this.driver = driver;
	}
	public String getUrl() {
		return url;
	}
	public void setUrl(String url) {
		this.url = url;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}	
}
```



## 五、断路器

        在分布式架构中，当某个服务单元发生故障（类似用电器发生短路）之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个错误响应，而不是长时间的等待。这样就不会使得线程因调用故障服务被长时间占用不释放，避免了故障在分布式系统中的蔓延。

        在Spring Cloud中使用了Hystrix 来实现断路器的功能。Hystrix是Netflix开源的微服务框架套件之一，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。Hystrix具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。

1、eureka-consumer-ribbon添加断路器

pom.xml中引入依赖hystrix依赖:



```
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-hystrix</artifactId>
      <version>1.3.1.RELEASE</version>
    </dependency>
```


在eureka-consumer-ribbon的主类Application中使用@EnableCircuitBreaker注解开启断路器功能：


```
package org.eureka.consumer.ribbon;
 
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;
 
@EnableDiscoveryClient
@SpringBootApplication
@EnableCircuitBreaker//@EnableCircuitBreaker注解开启断路器功能
public class Application {
 
	@Bean
	@LoadBalanced//加载负载均衡
	public RestTemplate restTemplate(){
		return new RestTemplate();
	}
	
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```


改造原来的服务消费方式，新增ComputeService类，在使用ribbon消费服务的函数上增加@HystrixCommand注解来指定回调方法。


```
package org.eureka.consumer.ribbon.service;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
 
@Service
public class ComputeService {
 
	@Autowired
	private RestTemplate restTemplate;
	
	//注解实现断路器回调，当调用服务失败后，调用addServiceFallback方法
	@HystrixCommand(fallbackMethod="addServiceFallback")
	public String addService(){
		return restTemplate.getForObject("http://eureka-client/add?a=10&b=20", String.class);
	}
	
	@SuppressWarnings("unused")
	private String addServiceFallback(){
		return "error";
	}
}
```


提供rest接口的Controller改为调用ComputeService的addService
	@RequestMapping(value="/addService",method=RequestMethod.GET)
	public String addService(){
		return computeService.addService();
	}
这样添加断路器后，调用/addService接口服务，当服务提供者断线或获取服务超时，会调用fallback。


2、eureka-consumer-feign

fegin集成了hystrix断路器，只要在配置文件中配置下面参数开启就可以了，不用导入hystrix依赖。


feign.hystrix.enabled=true

使用@FeignClient注解中的fallback属性指定回调类：


```
package org.eureka.consumer.feign.service;
 
import org.eureka.consumer.feign.fallback.DcClientHystrix;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
/*
 * 通过@FeignClient定义的接口来统一的生成我们需要依赖的微服务接口。而在具体使用的时候就跟调用本地方法一点的进行调用即可
 */
//使用@FeignClient注解来指定这个接口所要调用的服务名称,如过开启了断路器，通过fallback参数指定熔断后的回调类，该回调类实现当前接口
@FeignClient(value="eureka-client",fallback=DcClientHystrix.class)
public interface DcClient {
 
	@GetMapping("/dc")//接口中定义的各个函数使用Spring MVC的注解就可以来绑定服务提供方的REST接口
	public String consumer();
}
```


创建回调类ComputeClientHystrix，实现@FeignClient的接口，此时实现的方法就是对应@FeignClient接口中映射的fallback函数。


```
package org.eureka.consumer.feign.fallback;
 
import org.eureka.consumer.feign.service.DcClient;
import org.springframework.stereotype.Component;
 
/*
 * 回调类，实现@FeignClient的接口，此时实现的方法就是对应@FeignClient接口中映射的fallback函数。
 */
@Component
public class DcClientHystrix implements DcClient{
 
	@Override
	public String consumer() {
		// TODO Auto-generated method stub
		return "调用失败，执行回调函数";
	}
 
}
```




## 六、服务网关

    服务网关对微服务架构的作用：

不仅仅实现了路由功能来屏蔽诸多服务细节，更实现了服务级别、均衡负载的路由。
实现了接口权限校验与微服务业务逻辑的解耦。通过服务网关中的过滤器，在各生命周期中去校验请求的内容，将原本在对外服务层做的校验前移，保证了微服务的无状态性，同时降低了微服务的测试难度，让服务本身更集中关注业务逻辑的处理。
实现了断路器，不会因为具体微服务的故障而导致服务网关的阻塞，依然可以对外服务。

使用Spring Cloud Netflix中的Zuul做服务网关，引入依赖spring-cloud-starter-zuul、spring-cloud-starter-eureka，如果不是通过指定serviceId的方式，eureka依赖不需要，但是为了对服务集群细节的透明性，还是用serviceId来避免直接引用url的方式吧。



```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

应用主类使用@EnableZuulProxy注解开启Zuul

package org.api.gateway;
 
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.cloud.client.SpringCloudApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
 
@EnableZuulProxy//注解开启Zuul服务网关
@SpringCloudApplication//它整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker等
public class GatewayApplication {
 
	public static void main(String[] args) {
		new SpringApplicationBuilder(GatewayApplication.class).web(true).run(args);
	}
}
```



application.properties配置文件配置：

spring.application.name=api-gateway
server.port=5555
 
#配置注册服务中心地址，如果下面路由不用serviceId就不用配置，也不用导入eureka依赖
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
 
#路由配置
#配置属性zuul.routes.api-a-url.path中的api-a-url部分为路由的名字，可以任意定义.
zuul.routes.api-a.path=/eureka-client/**
zuul.routes.api-a.url=http://127.0.0.1:2001/
 
zuul.routes.ribbon.path=/ribbon/**
zuul.routes.ribbon.serviceId=eureka-ribbon-consumer
 
zuul.routes.api-b.path=/feign/**
zuul.routes.api-b.serviceId=eureka-feign-consumer

这样/api-a/**的访问都映射到http://localhost:2001/上，也就是说当我们访问http://localhost:5555/api-a/add?a=1&b=2的时候，Zuul会将该请求路由到：http://localhost:2222/add?a=1&b=2上。


       服务网关还可以做服务过滤，在完成了服务路由之后，我们对外开放服务还需要一些安全措施来保护客户端只能访问它应该访问到的资源。所以我们需要利用Zuul的过滤器来实现我们对外服务的安全控制。在服务网关中定义过滤器只需要继承ZuulFilter抽象类实现其定义的四个抽象函数就可对请求进行拦截与过滤。



```
package org.api.gateway.filter;
 
import javax.servlet.http.HttpServletRequest;
 
import org.springframework.stereotype.Component;
 
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
 
@Component//需要实例化该过滤器
public class AccessFilter extends ZuulFilter{
 
	/*
	 * shouldFilter：返回一个boolean类型来判断该过滤器是否要执行
	 */
	@Override
	public boolean shouldFilter() {
		// TODO Auto-generated method stub
		return true;
	}
 
	/*
	 * filterOrder：通过int值来定义过滤器的执行顺序
	 */
	@Override
	public int filterOrder() {
		// TODO Auto-generated method stub
		return 0;
	}
 
	/*
	 * filterType:返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：   
             pre：可以在请求被路由之前调用
             routing：在路由请求时候被调用
             post：在routing和error过滤器之后被调用
             error：处理请求时发生错误时被调用
	 */
	@Override
	public String filterType() {
		// TODO Auto-generated method stub
		return "pre";
	}
 
	/*
	 * run：过滤器的具体逻辑。
	 */
	@Override
	public Object run() {
		// TODO Auto-generated method stub
		RequestContext ctx=RequestContext.getCurrentContext();
		HttpServletRequest request=ctx.getRequest();
		String token=request.getParameter("token");
		if(token==null){
			ctx.setSendZuulResponse(false);
//			ctx.getResponse().setCharacterEncoding("utf-8");
			//设置浏览器解码
			ctx.getResponse().setContentType("text/html;charset=utf-8");
			ctx.setResponseBody("非法访问!!");
			ctx.setResponseStatusCode(401);
			return null;
		}
		return null;
	}
}
```


## Spring Cloud Config (Spring's client/server approach for storing and serving distributed configurations across multiple applications and environments.)

##### Server
- Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```
- App \
@EnableConfigServer

- properties

server.port=8888
spring.cloud.config.server.git.uri=ssh://localhost/config-repo
spring.cloud.config.server.git.clone-on-start=true

##### Client
- Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

- properties
```
server.port=8889
spring.application.name=config-client
spring.profiles.active=development
spring.cloud.config.uri=http://localhost:8888
```
## Spring Cloud Security (token-based security in Spring Boot applications  it makes OAuth2-based SSO easie)
## Server
#### Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

#### Application
@EnableAuthorizationServer

#### Properties:
server.port=7070

## Cleint
#### Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
```

#### Config
```java
@Configuration
@EnableOAuth2Sso
public class OAuthSecurityConfig extends WebSecurityConfigurerAdapter {
	protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/")
                .permitAll()
                .anyRequest()
                .authenticated();
    }
}
```

#### Properties:
```yml
security:
  oauth2:
    client:
      accessTokenUri: http://localhost:7070/authserver/oauth/token
      userAuthorizationUri: http://localhost:7070/authserver/oauth/authorize
      clientId: authserver
      clientSecret: passwordforauthserver
    resource:
      userInfoUri: http://localhost:9000/user
 server:
   servlet:
     session:
       cookie.name=KSESSION
```

 ## Spring Cloud Stream is a framework built on top of Spring Boot and Spring Integration that helps in creating event-driven or message-driven microservices    
## Producer
#### Dependency same for all services

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-kafka</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```
- properites
spring.cloud.stream.bindings.output.destination=testTopic

- Source App Service
```java
@SpringBootApplication
@EnableBinding(Source.class)
public class MyLoggerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyLoggerServiceApplication.class, args);
    }
 
    @StreamListener(Processor.INPUT)
    @SendTo(Processor.OUTPUT)
    public LogMessage enrichLogMessage(LogMessage log) {
        return new LogMessage(String.format("[1]: %s", log.getMessage()));
    }
}

@RestController
public class PublishController {
	
    @Autowired
    private MessageChannel output;

	@PostMapping("/publish")
	public Book publishEvent(@RequestBody Book book) {
		output.send(MessageBuilder.withPayload(book).build());
		return book;
	}

}
```

- Processor App Servie
```java
@SpringBootApplication
@EnableBinding(Processor.class)
public class DIscountApp {

	Logger logger=LoggerFactory.getLogger(DIscountApp.class);

	public static void main(String[] args) {
		SpringApplication.run(DIscountApp.class, args);
	}


	@Transformer(inputChannel = Processor.INPUT,outputChannel = Processor.OUTPUT)
	public Object addDiscountToProduct(List<Product> products){
		List<Product> productList = new ArrayList<>();
		
		logger.info("Product actual price is {} , after discount total price is {} ");
//		for(Product product:products){
//			if(product.getPrice()>8000){
//				productList.add(calculatePrice(product,10));
//			}
//			else if(product.getPrice()>5000){
//				productList.add(calculatePrice(product,5));
//			}
//		}
		return  "discount service";
	}


	private Product calculatePrice(Product product, int percentage) { 
		double actualPrice = product.getPrice();
		double discount = actualPrice * percentage / 100;
		product.setPrice(actualPrice - discount);
		logger.info("Product actual price is {} , after discount total price is {} ");
		return product;
	}
}
````
- Sink App Service

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class CourierApp {
	
	Logger logger = LoggerFactory.getLogger(CourierApp.class);

    @StreamListener(Sink.INPUT)	@StreamListener("input") @StreamListener(Sink.INPUT)
    public void orderDispatched(String products) {
		List<Product> test = products.get(0).toString();
		List<Product> prod = products;
		logger.info("Order dispatched to your mailing address : " + products);
            logger.info("Order dispatched to your mailing address : " + product.toString());
        });
    }

    public static void main(String[] args) {
        SpringApplication.run(CourierApp.class, args);
    }

}
```
- properties
spring.cloud.stream.bindings.input.destination=myTopic

##  Spring Cloud Gateway
#### dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-gateway</artifactId>
    <version>2.0.0.RC2</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```
- properties:
```	spring:
  cloud:
    gateway:
      routes:
      - id: baeldung_route
        uri: http://baeldung.com
        predicates:
        - Path=/baeldung/
management:
  endpoints:
    web:
      exposure:
        include: "*'
```

## Spring Cloud OpenFeign ( a declarative REST client for Spring Boot apps, Feign makes writing web service clients easier with pluggable annotation support, which includes Feign annotations and JAX-RS annotations)
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
#### application
```java
@SpringBootApplication
@EnableFeignClients
public class ExampleApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(ExampleApplication.class, args);
    }
}
```
-- feign client
```java
@FeignClient(value = "jplaceholder", url = "https://jsonplaceholder.typicode.com/")
public interface JSONPlaceHolderClient {
 
    @RequestMapping(method = RequestMethod.GET, value = "/posts")
    List<Post> getPosts();
 
    @RequestMapping(method = RequestMethod.GET, value = "/posts/{postId}", produces = "application/json")
    Post getPostById(@PathVariable("postId") Long postId);
}
```
```yml
feign:
  client:
    config:
      default:
        connectTimeout: 5000
        readTimeout: 5000
        loggerLevel: basic
```
##  Spring Cloud Task (The goal of Spring Cloud Task is to provide the functionality of creating short-lived microservices for Spring Boot application.)
- Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-task</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-task-core</artifactId>
</dependency>
```
#### Application
```java
@SpringBootApplication
@EnableTask
public class TaskDemo {
    // ...
}
```
#### Task
```java
@Component
public static class HelloWorldApplicationRunner 
  implements ApplicationRunner {
 
    @Override
    public void run(ApplicationArguments arg0) throws Exception {
        System.out.println("Hello World from Spring Cloud Task!");
    }
}
```
## Spring Cloud Zookeeper provides Apache Zookeeper integration for Spring Boot apps through autoconfiguration and binding to the Spring Environment.
- Dependency
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
	<version>1.0.3.RELEASE</version>
</dependency>
```
- Application
```java
@SpringBootApplication
@EnableDiscoveryClient
public class HelloWorldApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldApplication.class, args);
    }
}
```
- properties:
```yml
spring:
  application:
    name: HelloWorld
  cloud:
    zookeeper:
      discovery:
        enabled: true
logging:
  level:
    org.apache.zookeeper.ClientCnxn: WARN
spring:
  cloud:
    zookeeper:
      connect-string: localhost:2181
```
Note: Discover Service With Feign Client can also be done

## Spring Cloud CLI (The tool provides a set of command line enhancements to the Spring Boot CLI that helps in further abstracting and simplifying Spring Cloud deployments)
- install cli from zip
https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.3.1.RELEASE/spring-boot-cli-2.3.1.RELEASE-bin.zip
unzip spring-boot-cli-2.3.1.RELEASE-bin.zip
export PATH==$PATH:/e

## Spring Cloud CLI (The tool provides a set of command line enhancements to the Spring Boot CLI that helps in further abstracting and simplifying Spring Cloud deployments)
- install cli from zip
> wget https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.3.1.RELEASE/spring-boot-cli-2.3.1.RELEASE-bin.zip
> unzip spring-boot-cli-2.3.1.RELEASE-bin/bin
export PATH=$PATH:/home/nasar/Documents/spring-boot-cli/spring-2.3.1.RELEASE/bin
> spring --version

#### OR If you have (software development kit)
> sudo apt install unzip zip
> curl -s https://get.sdkman.io | bash
> source "/root/.sdkman/bin/sdkman-init.sh"
> sdk install springboot
> spring --version
> spring install org.springframework.cloud:spring-cloud-cli:2.2.0.BUILD-SNAPSHOT
> spring cloud --version
> spring cloud --list
> nao app.groovy
```java
@RestController
class ThisWillActuallyRun {
    @RequestMapping("/")
    String home() {
        "Hello World!"
    }
}
```
> spring run app.groovy
> curl localhost:8080
   Hello World
- Others
> spring cloud configserver @ localhost:8888
> spring cloud eureka @ localhost:8761
> spring cloud h2 @ localhost:9095
> spring cloud kafka @ localhost:9091
> spring cloud zipkin @ localhost:9411
> spring cloud dataflow @ localhost:9393
> spring cloud hystrixdashboard @ localhost:7979
- update yml file
> nano configserver.yml
```yml
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
	  
```
> nano cloud.yml
```yml
spring:
  cloud:
    launcher:
      deployables:
        - name: configserver
          coordinates: maven://...:spring-cloud-launcher-configserver:1.3.2.RELEASE
          port: 8888
          waitUntilStarted: true
          order: -10
        - name: eureka
          coordinates: maven:/...:spring-cloud-launcher-eureka:1.3.2.RELEASE
          port: 8761
```
## Spring Cloud Contract(contract between a Producer and a Consumer, in a distributed system – for both HTTP-based and message-based interactions)
- Dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-contract-verifier</artifactId>
    <version>2.1.1.RELEASE</version>
    <scope>test</scope>
</dependency>
```
#### Producer and consumer call each other with RestTemplate with a contract.

## Spring Zuul Proxy (Zuul is a JVM based router and server side load balancer by Netflix)
- Dependency
``xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```
- properties
```yml
zuul:
  routes:
    foos:
      path: /foos/**
      url: http://localhost:8081/spring-zuul-foos-resource/foos
```

## Spring Cloud Bus (Spring Cloud Bus uses lightweight message broker to link distributed system nodes. The primary usage is to broadcast configuration changes or other management information)
- Dependecy Client
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```
- properties:
```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
  cloud:
    bus:
      enabled: true
      refresh:
        enabled: true
```
- Dependency Server
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-monitor</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
    <version>3.0.4.RELEASE</version>
</dependency>
```
- properties
```
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
- in git properties
```
user.role=Programmer
change
user.role=Developer
```

## Spring Cloud Consul (Consul is a tool that provides components for resolving some of the most common challenges in a micro-services architecture like Service Discovery, Health Checking, Distributed Configuration, KV Store, Secure Service Communication, Multi Datacenter)
- Install consul
> curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
> sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
> sudo apt-get update && sudo apt-get install consul
> consul
http://localhost:8500/ui
> consul agent -dev
> consul members
> curl localhost:8500/v1/catalog/nodes
> dig @127.0.0.1 -p 8600 Judiths-MBP.node.consul
> consul leave
> mkdir ./consul.d
> echo '{
  "service": {
    "name": "web",
    "tags": [
      "rails"
    ],
    "port": 80
  }
}' > ./consul.d/web.json
> consul agent -dev -enable-script-checks -config-dir=./consul.d
> dig @127.0.0.1 -p 8600 web.service.consul
> curl http://localhost:8500/v1/catalog/service/web

- dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-all</artifactId>
    <version>1.3.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-consul-config</artifactId>
    <version>1.3.0.RELEASE</version>
</dependency>
```
- properties:
```yml
spring:
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        healthCheckPath: /my-health-check
        healthCheckInterval: 20s
      config:
        enabled: true
```

## Cloud App Starter (provide bootstrapped and ready-to-go applications – that can serve as starting points for future development)
> sudo docker run -p 50070:50070 sequenceiq/hadoop-docker:2.4.1
> git clone https://github.com/spring-cloud-stream-app-starters/twitter.git
> mvnw clean install -PgenerateApps
> java -jar twitter_stream_source.jar --consumerKey=<CONSUMER_KEY> --consumerSecret=<CONSUMER_SECRET> --accessToken=<ACCESS_TOKEN> --accessTokenSecret=<ACCESS_TOKEN_SECRET>
> git clone https://github.com/spring-cloud-stream-app-starters/hdfs.git
> mvnw clean install -PgenerateApps
> java -jar hdfs-sink.jar --fsUri=hdfs://127.0.0.1:50010/

- dependency
```xml
<dependency>
	<groupId>org.springframework.cloud.stream.app</groupId>
	<artifactId>spring-cloud-starter-stream-source-twitterstream</artifactId>
	<version>2.1.2.RELEASE</version>
</dependency>
<dependency>
	<groupId>org.springframework.cloud.stream.app</groupId>
	<artifactId>spring-cloud-starter-stream-sink-hdfs</artifactId>
	<version>2.1.2.RELEASE</version>
</dependency>
```
- Applciation along with SourceApp, ProcessorApp, SinkApp
```java
@SpringBootApplication
public class AggregateApp {
    public static void main(String[] args) {
        new AggregateApplicationBuilder()
          .from(SourceApp.class).args("--fixedDelay=5000")
          .via(ProcessorApp.class)
          .to(SinkApp.class).args("--debug=true")
          .run(args);
    }
}
```
> java -jar twitterhdfs.jar

## Spring Cloud – Bootstrapping (these apps is going to have a file called bootstrap.properties. It will contain properties just like application.properties but with a twist, A parent Spring ApplicationContext loads the bootstrap.properties first. This is critical so that Config Server can start managing the properties in application.properties)

- bootstrap.properties
```yml
spring.cloud.config.name=discovery
spring.cloud.config.uri=http://localhost:8081
```
- discovery.properties in our Git repository
```yml
spring.application.name=discovery
server.port=8082
 
eureka.instance.hostname=localhost
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```
- bootstrap.properties  for Gateway Zull
```yml
spring.cloud.config.name=gateway
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```
- gateway.properties in our Git repository
```
spring.application.name=gateway
server.port=8080
 
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
 
zuul.routes.book-service.path=/book-service/**
zuul.routes.book-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.book-service.execution.isolation.thread.timeoutInMilliseconds=600000
 
zuul.routes.rating-service.path=/rating-service/**
zuul.routes.rating-service.sensitive-headers=Set-Cookie,Authorization
hystrix.command.rating-service.execution.isolation.thread.timeoutInMilliseconds=600000
 
zuul.routes.discovery.path=/discovery/**
zuul.routes.discovery.sensitive-headers=Set-Cookie,Authorization
zuul.routes.discovery.url=http://localhost:8082
hystrix.command.discovery.execution.isolation.thread.timeoutInMilliseconds=600000
```
- bootstrap.properties
```
spring.cloud.config.name=book-service
spring.cloud.config.discovery.service-id=config
spring.cloud.config.discovery.enabled=true
 
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```
- book-service.properties in our Git repository
```
spring.application.name=book-service
server.port=8083
 
eureka.client.region = default
eureka.client.registryFetchIntervalSeconds = 5
eureka.client.serviceUrl.defaultZone=http://localhost:8082/eureka/
```

## Spring Cloud – Securing Services (Spring security)
- dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
- Application
```java
@EnableRedisHttpSession
public class SessionConfig
  extends AbstractHttpSessionApplicationInitializer {
}
```
- eureka/gateway/book-service.properties files in our git repository
```
spring.redis.host=localhost 
spring.redis.port=6379
```
- application.properties
```
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
security.user.name=configUser
security.user.password=configPassword
security.user.role=SYSTEM
```
- bootstrap.properties for config server with security
```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
  ```
- bootstrap.properties for bookservice
```
spring.cloud.config.username=configUser
spring.cloud.config.password=configPassword
eureka.client.serviceUrl.defaultZone=
  http://discUser:discPassword@localhost:8082/eureka/
```
- book-service.properties
```
management.security.sessions=never
```

## Spring Cloud Vault (for security sensitive information like database credentials, api gateway toke, api keys etc)
- dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-vault-config-databases</artifactId>
</dependency>
```
- bootstrap.yml
```yml
spring:
  cloud:
    vault:
      uri: https://localhost:8200
      ssl:
        trust-store: classpath:/vault.jks
        trust-store-password: changeit
```
OR
- bootstrap.yml generic secret
```yml
spring:
  cloud:
    vault:
      # other vault properties omitted ...
      generic:
        enabled: true
        application-name: fakebank
```
- bootstrap.yml database secret
```yml
spring:
  cloud:
    vault:
      # ... other properties omitted
      database:
        enabled: true
        role: fakebank-accounts-rw
```

## Spring Cloud Data Flow ( cloud-native programming and operating model for composable data microservices, developers can create and orchestrate data pipelines for common use cases such as data ingest, real-time analytics, and data import/export.)
- Source: is the application that consumes events
- Processor: consumes data from the Source, does some processing on it, and emits the processed data to the next application in the pipeline
- Sink: either consumes from a Source or Processor and writes the data to the desired persistence layer
- Data Flow Shell - The Data Flow Shell is a client for the Data Flow Server
- Message Broker
-- Apache Kafka
-- RabbitMQ

- Local Data Flow Server
- dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-dataflow-server-local</artifactId>
</dependency>
```
- Applciation
```java
@EnableDataFlowServer
@SpringBootApplication
public class SpringDataFlowServerApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(
          SpringDataFlowServerApplication.class, args);
    }
}
```
> mvn spring-boot:run

- The Data Flow Shell
-dependency
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dataflow-shell</artifactId>
</dependency>
```
- Application
```java
@EnableDataFlowShell
@SpringBootApplication
public class SpringDataFlowShellApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(SpringDataFlowShellApplication.class, args);
    }
}
```
> mvn spring-boot:run

- create client service source, processing and sink

- Register a Stream App
maven://<groupId>:<artifactId>[:<extension>[:<classifier>]]:<version>

```
app register --name time-source --type source --uri maven://com.baeldung.spring.cloud:spring-data-flow-time-source:jar:0.0.1-SNAPSHOT
app register --name time-processor --type processor --uri maven://com.baeldung.spring.cloud:spring-data-flow-time-processor:jar:0.0.1-SNAPSHOT
app register --name logging-sink --type sink --uri maven://com.baeldung.spring.cloud:spring-data-flow-logging-sink:jar:0.0.1-SNAPSHOT
```
- Create and Deploy the Stream
```
stream create --name time-to-log 
  --definition 'time-source | time-processor | logging-sink'
```

## ETL(extract, transform and load) with Spring Cloud Data Flow (Spring Cloud Data Flow is a cloud-native toolkit for building real-time data pipelines and batch processes. Spring Cloud Data Flow is ready to be used for a range of data processing use cases like simple import/export, ETL processing, event streaming, and predictive analytics)
- create data pipelines in two flavors
Long-lived real-time stream applications using Spring Cloud Stream
Short-lived batched task applications using Spring Cloud Task

> docker run --name dataflow-rabbit -p 15672:15672 -p 5672:5672 -d rabbitmq:3-management
-  install and run the PostgreSQL RDBMS on the default port 5432
> CREATE DATABASE dataflow;
> java -Dloader.path=lib -jar spring-cloud-dataflow-server-local-1.6.3.RELEASE.jar \
    --spring.datasource.url=jdbc:postgresql://127.0.0.1:5432/dataflow \
    --spring.datasource.username=postgres_username \
    --spring.datasource.password=postgres_password \
    --spring.datasource.driver-class-name=org.postgresql.Driver \
    --spring.rabbitmq.host=127.0.0.1 \
    --spring.rabbitmq.port=5672 \
    --spring.rabbitmq.username=guest \
    --spring.rabbitmq.password=gues

http://localhost:9393/dashboard

> java -jar spring-cloud-dataflow-shell-1.6.3.RELEASE.jar
> dataflow config server http://{host}
> dataflow:>app import --uri http://bit.ly/Darwin-SR1-stream-applications-rabbit-maven
> dataflow:> app list
- Composing an ETL Pipeline

- Transform – Mapping JDBC fields to the MongoDB fields structure
- dependency
```xml
dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
</dependency>
```
- processing App in Postgress
```java
@EnableBinding(Processor.class)
public class CustomerProcessorConfiguration {
 
    @Transformer(inputChannel = Processor.INPUT, outputChannel = Processor.OUTPUT)
    public Customer convertToPojo(Customer payload) {
 
        return payload;
    }
}
```
- Sink in MongoDB
```java
@Document(collection="customer")
public class Customer {
 
    private Long id;
    private String name;
 
    // Getters and Setters
}
@EnableBinding(Sink.class)
public class CustomerListener {
 
    @Autowired
    private CustomerRepository repository;
 
    @StreamListener(Sink.INPUT)
    public void save(Customer customer) {
        repository.save(customer);
    }
@Repository
public interface CustomerRepository extends MongoRepository<Customer, Long> {
 
}
```
- Stream Definition Register
```
app register --name customer-transform --type processor --uri maven://com.customer:customer-transform:0.0.1-SNAPSHOT
app register --name customer-mongodb-sink --type sink --uri maven://com.customer:customer-mongodb-sink:jar:0.0.1-SNAPSHOT
```

## Batch Processing with Spring Cloud Data Flow (a batch process makes it easy to create short-lived services where tasks are executed on demand.)
- dependency
```xml
dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
- Application
```java
@EnableTask
@EnableBatchProcessing
@SpringBootApplication
public class BatchJobApplication {
 
    public static void main(String[] args) {
        SpringApplication.run(BatchJobApplication.class, args);
    }
}

@Configuration
public class JobConfiguration {
 
    private static Log logger
      = LogFactory.getLog(JobConfiguration.class);
 
    @Autowired
    public JobBuilderFactory jobBuilderFactory;
 
    @Autowired
    public StepBuilderFactory stepBuilderFactory;
 
    @Bean
    public Job job() {
        return jobBuilderFactory.get("job")
          .start(stepBuilderFactory.get("jobStep1")
          .tasklet(new Tasklet() {
            
              @Override
              public RepeatStatus execute(StepContribution contribution, 
                ChunkContext chunkContext) throws Exception {
                
                logger.info("Job was run");
                return RepeatStatus.FINISHED;
              }
        }).build()).build();
    }
}
```
> mvn clean install
- Register
> app register --name batch-job --type task --uri maven://com.baeldung.spring.cloud:batch-job:jar:0.0.1-SNAPSHOT
> task create myjob --definition batch-job
> task launch myjob
> task execution list







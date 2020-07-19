
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

## Zuul Proxy (Zuul is a JVM based router and server side load balancer by Netflix)
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









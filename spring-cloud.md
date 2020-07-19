
## Spring Cloud Config (Spring's client/server approach for storing and serving distributed configurations across multiple applications and environments.)

##### Server
--Dependency
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>

-App
@EnableConfigServer

-properties
server.port=8888
spring.cloud.config.server.git.uri=ssh://localhost/config-repo
spring.cloud.config.server.git.clone-on-start=true

##### Client
--Dependency
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>

-properties
server.port=8889
spring.application.name=config-client
spring.profiles.active=development
spring.cloud.config.uri=http://localhost:8888

## Spring Cloud Security (token-based security in Spring Boot applications  it makes OAuth2-based SSO easie)
#### Dependency
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
#### Application
@EnableOAuth2Sso

#### Properties:
security:
  oauth2:
    client:
      accessTokenUri: http://localhost:7070/authserver/oauth/token
      userAuthorizationUri: http://localhost:7070/authserver/oauth/authorize
      clientId: authserver
      clientSecret: passwordforauthserver
    resource:
      userInfoUri: http://localhost:9000/user
      





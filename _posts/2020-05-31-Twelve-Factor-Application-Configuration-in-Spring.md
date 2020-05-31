---
title: Twelve-Factor Application&#58; Configuration in Spring
author: Hamza Belmellouki
categories: [Spring]
tags: [spring-cloud, cloud-native]
comments: true
---

## Introduction

The Twelve-Factor application methodology is a collection of best practices that are designed to enable applications to be developed with portability and resilience when deployed. In this post, we’ll cover the third factor, [Config](https://12factor.net/config). We’ll also see how the spring ecosystem contributes to helping developers achieve this factor.

The Twelve-Factor application manifesto advocates on externalizing configs(passwords, hostnames…) as [environment variables](https://en.wikipedia.org/wiki/Environment_variable):

> **The twelve-factor app stores config in environment variables** (often shortened to *env vars* or *env*). Env vars are easy to change between deploys without changing any code

Open-sourcing your codebase without compromising any credentials is the test that your app needs when you want to check if you’ve honored the factor.

## Using @Value

This example demonstrates a quite simple example on how to load up properties from sample.properties file using regular spring:

```
message="Blogging is cool"
```

Here I am creating a command-line runner to print the value that is loaded from the sample.properties. Note that this file exists in the classpath, meaning in `target/classes` directory:

```java
@SpringBootApplication
@Slf4j
@PropertySource("classpath:sample.properties")
public class ConfigsApplication implements CommandLineRunner {
    @Value("${message}")
    private String msg;

    public static void main(String[] args) {
        SpringApplication.run(ConfigsApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        log.info(msg);
    }
}
```

## Profiles

If you’re working in the enterprise chances are your projects have to go into many different environments(Dev \| Test \| Stage \|UAT /Pre-Prod \| Prod). Each of these environments may have different settings that are specific to them. How can we load the appropriate settings in its appropriate environment? The simplest answer to this question is to work with Spring Profiles. You’ll map each environment to its configuration that’s defined as a properties or YAML file.

You can leverage environment variables in your YAML/properties files as this factor advocate. In fact, you should do that since you don’t want to check out sensitive info like db credentials into your source control management system.

In this example, we’ll work with two environments: dev and prod. These two environments will differ in the database setup. For the dev environment, we’ll define a file called `application-dev.yml`. For prod, we’ll define it as `application-prod.yml`.

application-dev.yml:

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
    password: sa
  jpa:
    hibernate:
      ddl-auto: create
```

The above configuration didn’t use environment variables because there’s no need, this profile is used solely for a dev environment(Don’t do that in prod).

application-prod.yml:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: none
```

Here I’ve used environment variables to load db credentials to the database. Now if you want you can publish your project to the public without worrying about db credentials being leaked.

You can activate a profile using one of these methods:

### JVM system parameter

Passing a JVM system parameter is a way to activate a profile. For example, if you build your project using the spring-boot plugin you can pass the parameter like so:

```
mvn spring-boot:run -Dspring.profiles.active=prod
```

### Environment Variable

Activate a profile by setting up an environment variable. For example, in Unix environment you would set your env in .bashrc file like so:

```
export spring_profiles_active=dev
```

### Maven profile

You can leverage Maven profiles to activate Spring Profiles. This is my favorite way to activate a profile. For example, in your `profiles` section we’ll define two maven profiles `dev` and `prod`, each of these profiles will set a JVM system parameter:

```xml
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
        </properties>
    </profile>
</profiles>
```

After setting up your maven profiles. You’ll need to set up resource filtering in your pom.xml to trigger variable resolution.

```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    ...
</build>
</properties>
```

All that’s left is to specify which profile will be active by running a maven command with `-P` option:

```
mvn spring-boot:run -Pdev
```

## Spring Cloud

If you're using Spring Cloud, we can centralize the configuration in "Spring Cloud Config Server" and consume the configuration that exists in a configuration git repository which stores all the necessary property configuration. The Config Server acts like a proxy between our configuration repository and clients. Note clients will consume the Spring Cloud Config Server via REST API.

Leveraging the above approach if configuration is changed the application will need to restart. Spring Cloud Config Server solves that problem by introducting a new scope: The **refresh scope**, this will allow us to reconfigure components without a restart. In this example we'll see how to configure a Config Server and how clients can consume it using its REST API.

### Build a config server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

Our Config Server will load configuration from a [git repo](https://github.com/Hamzablm/configuration-demo) hosted on GitHub. The service is a regular Spring Boot application which expose .(properties / yml) configuration as a REST API. Now, all you have to do to add Spring Clould Config Server support is to annotate your Spring Boot app with `@EnableConfigServer` annotation:

```java
@SpringBootApplication
@EnableConfigServer
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

This annotation will tell Spring Boot provide all plumbing you app need to configure beans and expose the API.

This is the configuration for the Config Server located in `application.yml`:

```yaml
server:
  port: 8181
  
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Hamzablm/configuration-demo
```

The property `spring.cloud.config.server.git.uri` tells spring to look for configuration files located this [git repository](https://github.com/Hamzablm/configuration-demo). Every Spring Boot application that leverage Spring Cloud should provide a unique name which is defined in `bootstrap.yml`:

```yml
spring:
  application:
    name: config-server
```

Keep in mind that "Spring Could Config Server" will bring all configuration from [configuration-demo](https://github.com/Hamzablm/configuration-demo) repository and expose the appropriate configuration to clients by matching client's `spring.application.name` to the configuration filename that exists in the git repo. 

Note that you can even expose a specific configuration file for a specific profile; say if you run in a `dev` profile then `configuration-client-dev.properties` will be leveraged if it exists.

Also, If an `application.(properties/yml)` file exists in the git repo the config server will expose its properties to all clients, other configuration files will either define new properties or override the ones from `application.(properties/yml)`

### Buid the client

Add Spring Cloud Config Client dependency to connect to Spring Cloud Config Server to fetch the app's configuration. Also, you'll need to bing Spring Actuator to access the `refresh` endpoint (more on that later) which will enable you to update the configuration on the fly in the client side: 

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-config</artifactId>
</dependency>

<dependency>
 	 <groupId>org.springframework.boot</groupId>
	 <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

The client which is a Spring Cloud based service will look for `bootstrap.yml` to bootstrap the service. `bootstrap.yml` start up before any `application.(properties/yml)`. Here `bootstrap.yml` defines the name of the service which should match the name of the configuration filename in the git repository so the client know what need to be consumed.

```yaml
spring:
  application:
    name: configuration-client
  cloud:
    config:
      uri: http://localhost:8181
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

`cloud.config.uri` specifies the endpoint where your Config Server is deployed. `management.endpoints.web.exposure.include=*` is used to tell spring to enable actuator endpoint; we'll need them if we want to make our configuration refreshable.

Here we can access our config properties using traditional approaches like `Environment` abstraction, `@PropertySource`, `@ConfigurationProperties`:

```java
@RestController
@RefreshScope
class MessageRestController {
    @Autowired
    private Environment env;

    @GetMapping("/msg")
    String getMessage() {
        return env.getProperty("project.message");
    }
}
```

This class is annotated with `@RefreshScope` therefore any changes in the configuration git repo could take effect without restarting the client app; to refresh your configs all you have to do is to perform a post request to this endpoint:

```bash
curl localhost:8080/actuator/refresh -d {} -H "Content-Type: application/json"
```

**PS**: Imagine having lots of microservices consuming your Config Server. If you change the configuration repository then everyone of those clients will have to refresh its configuration by performing a post request to the Actuator refresh endpoint and there's no need to tell you that's not a scalable solution. To solve this problem we can leverage **Spring Cloud Bus**, which links nodes of a distributed system with a lightweight message broker (e.g RabbitMQ) to broadcast state changes. Maybe I'll do a whole blog to talk about Spring Cloud Bus.

### Running the code

First run the config server by running ``mvn spring-boot run`` and make sure you get the property defined in configuration-client in the json payload by accessing this url: http://localhost:8181/configuration-client/master. You should get something like:

```json
{
   "name":"configuration-client",
   "profiles":[
      "master"
   ],
   "label":null,
   "version":"e467e3598c0509da56a15731a1f01e0851c46307",
   "state":null,
   "propertySources":[
      {
         "name":"https://github.com/Hamzablm/configuration-demo/configuration-client.properties",
         "source":{
            "project.message":"Blogging is awesome"
         }
      }
   ]
}
```

Notice now I got my property `project.message` from the git repo into my config server. 

Now after running the config server, go ahead and run the config client by running `mvn spring-boot run`. Then go ahead and access this endpoint which just expose the configuration: `http://localhost:8080/msg`:

`````
Blogging is awesome
`````

Now that you've saw how to centralize your configuration in the configuration git repo and expose them using Spring Cloud Config Server make sure to clone the [repository](https://github.com/Hamzablm/spring-cloud-configs) if you want to run the demo.

## Wrap Up

In this post I attempted to show you how to leverage the Spring ecosystem capabilities to honor Config factor in a Twelve-Factor application. 

In the next article, I'll blog about writing integration tests using spring boot.
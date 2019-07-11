# Todo(s) Workshop

Howdy and welcome!  

This is the [one-and-only doc](#todos-workshop) for how to use the Todo apps together as a sample set.  Each project listed below has it's own README that details app specifics, this doc is an exhaustive list of information on how to use them together on PCF.

## Intended Audience

This content is intended for anyone wanting a sound understanding of Spring Boot on PCF or for anyone wanting to hack around a simple model.

## Need to know

### Domain Model

All the samples in this workshop center around the `Todo` model and hence most are named `todos-blah-blah-blah`.  Each project scratches a particular Spring Boot + PCF itch but all share this simple `Todo` type.

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
class Todo implements Serializable {
    private String id;
    private String title;
    private Boolean complete = Boolean.FALSE;
}
```

*However* it should be stated that each project maintains it's own implementation of this tiny model.  This gist is a Todo is a String id, String title and Boolean complete flag and that's it.  String as id perhaps is a good talking point around how id(s) should be typed and managed for certain use-cases.  Also considerations around what really needs an id anyway, who should own the id and such.  This sample set uses String (UUID) as an id and app code "owns" dealing out new ones...database id generation isn't used.

**Apropos**

* [`java.util.UUID`](https://docs.oracle.com/javase/8/docs/api/java/util/UUID.html)
* [Mongo ObjectId](https://docs.mongodb.com/manual/reference/method/ObjectId/)
* [Mongo Blog Post](https://www.mongodb.com/blog/post/generating-globally-unique-identifiers-for-use-with-mongodb)
* [Sonyflake distributed ids](https://github.com/sony/sonyflake)

### Prereqs

#### Development

Each project can be cloned, built and pushed to PCF individually.  All that's required to work with the projects locally are listed below.

1. Java 8 JDK ([sdkman](https://sdkman.io/sdks#java) is a nice dev option)
1. Maven 3 (again [sdkman](https://sdkman.io/sdks#maven))
1. Make (for quick clone)
1. [Cloud Foundry CLI](https://sdkman.io/sdks#java)
1. [Curl](https://curl.haxx.se/) or [Httpie](https://httpie.org/)
1. An editor and internet connection

#### Pivotal Cloud Foundry

1. [Pivotal Application Service 2.5 and up](https://docs.pivotal.io/pivotalcf/2-5)
1. [MySQL for PCF 2.5 and up](https://docs.pivotal.io/p-mysql/2-5/index.html)
1. [RabbitMQ for PCF 1.15 and up](https://docs.pivotal.io/rabbitmq-cf/1-15/index.html)
1. [Redis for PCF 2.0 and up](https://docs.pivotal.io/redis/2-0/index.html)
1. [Spring Cloud Services for PCF 2.0.x](https://docs.pivotal.io/spring-cloud-services/2-0/common/index.html)
1. An account on PCF with Space Developer access
1. This sample set consumes at a minimum 1 MySQL SI, 1 Redis SI, 1 RabbitMQ SI, 1 Config Server SI and 1 Service Registry SI for [5 total](https://www.youtube.com/watch?v=k6OWSf8_5J4).

### Projects Setup

Project setup is pretty straightforward as each project is [standard spring boot](https://start.spring.io) with maven. Follow the steps below for a quick way to clone and build.  However you can easily clone each project listed [here](#projects) and manually run the maven builds (`mvnw clean package`).  The recommendation is that projects are siblings on the file-system.

```
${someDir}/todos-api
${someDir}/todos-edge
${someDir}/todos-webui
${someDir}/todos-mysql
${someDir}/todos-redis
${someDir}/todos-app
${someDir}/todos-shell
```

1. Clone this project (the one you're reading) as `todos-shell`
1. cd into `todos-shell`
1. Run `make clone` to clone all the projects
1. Afterwards you should have these projects in `todos-shell/apps`
    * todos-api
    * todos-edge
    * todos-webui
    * todos-mysql
    * todos-redis
    * todos-app
    * todos-shell
1. Run `make build` to kickoff a maven build for each project.  By default this requires internet access to Maven Central and Spring Repositories.
1. The build places artifacts in each project's `${project}/target` directory.  By default the artifact version is set to `1.0.0.SNAP` and all apps expose this info over `/actuator/info`.

### Projects

The samples listed below are used throughout the workshop, each repository goes into more depth on the app but here's a quick summary of each.  

**Note** - Each project contains a master and cloud branch.  The master branch contains plain-ole Spring Boot apps while the cloud branch adds [Spring Cloud](https://spring.io/projects/spring-cloud) features to each.

#### todos-api

Todo(s) API is a sample Spring Boot service that uses `spring-boot-starter-web` to implement a REST API for `Todo(s)`.  Not listed in this set but similar is todos-webflux which is same API implemented using `spring-boot-starter-webflux` and hence non-blocking (and uses Netty container).

**Talking Points**

* Spring Boot and its advantages
* Spring Boot Starter POM
* Spring Boot Auto Configuration
* Spring Boot Property Sources
* Spring Boot embedded Containers
* Spring Boot logging
* Spring Boot Web stacks (Spring WebMVC and Spring WebFlux) 
* [Spring Framework](https://spring.io/)
* [Spring Boot](https://spring.io/projects/spring-boot)
* [Java Buildpack](https://github.com/cloudfoundry/java-buildpack)
* Spring Auto Reconfiguration - key is understanding the levels of auto-configuration at play on PCF

#### todos-edge

Todo(s) Edge is an edge abstraction for other Todo apps and serves as a client entry-point into app functionality and is implemented using [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway).  Todo(s) Edge is implemented using Kotlin however there's remarkably little code as the gateway is virtually all configuration.  The edge is configured to route traffic sent to `/todos/` to a backing API of some sort while root traffic contains the UI.  During certain sections of the workshop we'll use Spring Cloud to control where the edge sends traffic.

**Talking Points**

* Kotlin
* Kotlin support in Spring
* Spring WebFlux
* Spring Cloud Gateway
* Netty

#### todos-webui

A sample frontend Vue.js app wrapped in Spring Boot

* [Spring Boot](https://spring.io/projects/spring-boot) for app bits, using webflux runtime
* [Vue.js](https://vuejs.org/) for frontend, inspired by [TodoMVC Vue App](http://todomvc.com/examples/vue/), difference is this one is vendored as a Spring Boot app and calls a backing endpoint (``/todos``)

This application assumes the ``/todos`` endpoint is exposed from the same "origin".  Because of this its best to use this application behind the [todos-edge](https://github.com/corbtastik/todos-edge) which will serve as a gateway and single origin to the client for both loading ``todos-webui`` and for proxying API calls to ``/todos``.

![Todo(s) WebUI](img/todos-webui.png "Todo(s) WebUI")

#### todos-mysql

Inevitably you'll need to implement Microservices to talk with Databases. This repository contains a Microservice implemented in Kotlin using Spring Boot, Spring Cloud, Spring Data JPA anf Flyway.  It can be used as a standalone data service or as a backend for a Todo app.

[Spring Framework 5](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0) added support for [Kotlin](https://kotlinlang.org/). Developers can now implement Microservices in [Java](https://en.wikipedia.org/wiki/Java_(programming_language)), [Groovy](https://groovy-lang.org/) and [Kotlin](https://kotlinlang.org/). Kotlin is slowly starting to show up in development communities given most like the simplified syntax. If you're a Java developer you'll get up to speed on Kotlin quickly because it's very Java like but with [lots of simplifications](https://kotlinlang.org/docs/reference/comparison-to-java.html).

Todo(s) Data uses Flyway to handle database schema creation and versioning. Using Flyway from Spring Boot starts with declaring a dependency on `org.flywaydb:flyway-core` in `pom.xml` and Spring Boot will Auto Configure on startup.  Spring profiles are used to boot into different database context, the default profile initiates H2 while the "cloud" profile used MySQL.

**Talking Points**

* Kotlin
* Spring Data
* Spring Data JPA
* Flyway
* MySQL for PCF

#### todos-redis

Todo(s) Redis is a simple Spring Data Redis backend for a Todo API.  It's used as the default "caching" layer in the workshop sample set.

**Talking Points**

* Redis
* Redis Hashes
* Redis cli
* Redis for PCF
* Spring Data Redis
* Caching Use Cases and Patterns

#### todos-app

Todo(s) App is a composite backend for a fully functional Todo app, it contains look-aside caching and business logic and uses Spring Cloud for app to app connectivity.  We'll see how to configure a fully functional Todo application using this backend which integrates with a System of Record (Sor) service and a Cache service. All enabled with Spring Cloud, MySQL for PCF and Redis for PCF.

**Talking Points**

* Spring Cloud Services for PCF
* MySQl for PCF
* Redis for PCF
* DiscoveryService
* RestTemplate
* [Pivotal Blog - Caching Patterns](https://content.pivotal.io/blog/an-introduction-to-look-aside-vs-inline-caching-patterns)

#### todos-shell

Todo(s) shell is a [Spring Shell](https://projects.spring.io/spring-shell/) application used to automate configuration and deployment of Todo apps to PCF.  It uses the [CF Java Client](https://github.com/cloudfoundry/cf-java-client), your CF credentials and your locally compiled `todos-*` jars to automate app deployment to PCF.

The list of shell commands

```bash
push-app: todos-edge,todos-api,todos-webui NO spring-cloud
push-scs: push-app with spring-cloud    
push-internal: push-app with private app networking
push-lookaside: todos-edge,todos-app,todos-webui with spring-cloud
push-my-sql: todos-edge,todos-mysql,todos-webui NO spring-cloud
push-scs-my-sql: todos-edge,todos-mysql,todos-webui with spring-cloud
push-redis: todos-edge,todos-redis,todos-webui NO spring-cloud
push-scs-redis: todos-edge,todos-redis,todos-webui with spring-cloud    
```

**Talking Points**

* [Spring Shell](https://projects.spring.io/spring-shell/)
* [CF Java Client](https://github.com/cloudfoundry/cf-java-client)
* Automation
* Spring Cloud connectivity
* Stamping out Todo apps...now what?
* Next steps...Users, tenancy and SSO

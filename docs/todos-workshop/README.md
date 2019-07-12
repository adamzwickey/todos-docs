# Todo(s) Workshop

Howdy and welcome!  

This is the [one-and-only doc](#todos-workshop) for how to use the Todo apps together as a sample set.  Each project listed below has it's own README that details app specifics, this doc is an exhaustive list of information on how to use them together on PCF.

## Intended Audience

This content is intended for anyone wanting a sound understanding of Spring Boot on PCF or for anyone wanting to hack around a simple model and get to know salient features of both.  

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

### PreReqs

#### Development

Each project can be cloned, built and pushed to PCF individually.  All that's required to work with the projects locally are listed below.

1. Java 8 JDK ([sdkman](https://sdkman.io/sdks#java) is a nice dev option)
1. [Git client](https://git-scm.com/downloads)
1. Maven 3 (again [sdkman](https://sdkman.io/sdks#maven))
1. Make (for quick clone)
1. [Cloud Foundry CLI](https://github.com/cloudfoundry/cli)
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

1. Clone the `todos-shell` project
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

---

## Shop Sample Set

![Todos Samples](img/shop-samples.png "Todos Samples")


## Shop 0

### Setting up projects, PCF accounts and cf push ice-breaker

This Shop is all about setting up the projects and doing a CF push ice-breaker.  The best way to get started it to clone [Todo(s) Shell](https://github.com/corbtastik/todos-edge) (or download zip) and run the following make command to clone all the projects.

1. Complete [Projects Setup](#projects-setup) and initial build
1. You should have these projects cloned to the same directory on your machine

    ```bash
    ${someDir}/todos-api
    ${someDir}/todos-edge
    ${someDir}/todos-webui
    ${someDir}/todos-mysql
    ${someDir}/todos-redis
    ${someDir}/todos-app
    ${someDir}/todos-shell
    ```
1. Make sure you've installed the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli) as mentioned in [PreReqs](#prereqs)
1. Login to Pivotal Cloud Foundry with CLI - *Note* you will need your username and password for PCF.

    ```bash
    # -a use your API endpoint,
    # skip SSL cert validation if you're using self-signed certificates
    $ cf login -a api.sys.retro.io --skip-ssl-validation
    ```
1. [Login to Pivotal Apps Manager](https://docs.pivotal.io/pivotalcf/customizing/console-login.html), typically this is hosted at `apps.YOUR-SYSTEM-DOMAIN`, in this sample my System Domain is `sys.retro.io`, yours will be different.
1. If needed you can use Pivotal Apps Manager to download the cf-cli for your OS as well as copy the required `cf login` command for your PCF endpoint.

![Apps Man Tools](img/apps-man-tools.png "Apps Man Tools")

1. Grok the cf-cli - run the following commands
    1. `cf -v`
    1. `cf target`
    1. `cf domains`
    1. `cf buildpacks`
    1. `cf marketplace`

1. Compile and Push ice-breaker [App](#todos-api) - Now is a good time to talk about what PCF did to containerize and bring the app online.

Edit the manifest.yml in `todos-api` to your liking.  Use a unique name for your application name, recommend to prefix with your name.

**manifest.yml**

```yaml
---
applications:
# use something unique, for example corbs-todos-api    
- name: todos-api 
memory: 1G
path: target/todos-api-1.0.0.SNAP.jar
buildpack: java_buildpack
env:
    TODOS_API_LIMIT: 1024
```

**compile and push**

```bash
> cd ${someDir}/todos-api
> git checkout master # (should already be on master branch)
> mvnw clean package
> cf push
```

PCF is capable of containerizing many different types of applications, take a look at these samples to get a feel for what PCF supports.

1. [Java examples](https://github.com/cloudfoundry/java-buildpack#examples)
1. [CF Sample Apps](https://github.com/cloudfoundry-samples)
1. [SSO Samples](https://github.com/pivotal-cf/identity-sample-apps)

---

## Shop 1

### Introduce Todo sample set (whiteboard or slide with picture)

In this Shop we're going to build a simple 3 service App that consist of a backing API implemented in Spring Boot, a WebUI which is a Spring Boot vendored Vue.js app and an Edge implemented with Spring Cloud Gateway that helps with application level routing.  This Shop is plain-ole Spring Boot without Spring Cloud integration (which is later) and relies simply on core PCF features.

At the end of this Shop you'll have 3 apps running in PCF that have been manually configured to work together.

* Intro to Spring Boot and [Sample Set](#shop-sample-set)
* Code or inspect [Todo Edge](#todos-edge), [Todos API](#todos-api) and [Todos WebUI](#todos-webui) locally
* Run a maven build on todos-edge, todos-api, todos-webui
* Manually configure manifests for your todos-edge, todos-api and todos-webui apps
* In particular manually configure the route endpoints on todos-edge
    * `TODOS_API_ENDPOINT: <your-todos-api-url>`
    * `TODOS_UI_ENDPOINT: <your-todos-webui-url>`
* cf push each app
* You'll have a route to your Todo Edge app...for example `https://corbs-todos-edge.apps.retro.io`.  Open using Chrome to access UI.
* Extra mile - Create a custom route in cf and map to your Todos Edge
* Extra mile - Use Todo Shell to automate pushing the apps
    * `shell:>push-app --tag corbs`

---

## Shop 2

### Introduce Spring Cloud for Todo sample set

* Intro to Spring Cloud
* Configure git repository for application configs
* Provision p-config-server, walk through configuration options
* Code and/or inspect Todos API locally and cf push with Spring Cloud enabled
* Make config change
* Refresh Todos API
* Show updated configuration and walk through how the refresh works
* Next steps - refresh bus, encrypted values

---

## Shop 3

### Introduce Edge (Spring Cloud Gateway) and WebUI (Spring Boot + Vue.js) apps

* Refer back to the picture we're building
* Introduce Spring Cloud Gateway as an application edge and routing tool
* Inspect Todo(s) Edge code and configuration and walk through how routes are handled
* Inspect both master and cloud branches for differences, note cloud branch uses Spring Cloud semantics for connectivity so we can stop maintaining URIs and such.  
* Introduce WebUI and simply show it's a Spring Boot app vendoring a frontend Javascript/HTML/CSS app.  
* Manually configure (master branch) todos-edge,todos-api,todos-webui and cf push
* Confirm the app is functioning, users should now have a UI similar to [this](#todos-webui).
* Slap a high-five or something as you've manually completed pushing the app
* Extra mile stuff - use [Todo(s) Shell](#todos-shell) to automate the deployment of the same three apps as one functioning "Todo app" on PCF.  The shell will use your PCF creds and push configured apps to PCF.  Each deploy results in 3 apps running (todos-edge,todos-api,todos-webui) each with a user provided "tag" which will prefix the running app instances.

---

## Shop 4

### Internal Routes on PCF

We can restrict access to the Backend API and UI by removing public routes for those apps and then mapping them to an internal domain (``apps.internal``).  Once the apps have an internal route we can add a network policy that allows the Edge to call them.

Push the Edge, API and UI with the ``manifest-internal.yml`` from each project and the result will be a deployment where the Edge is the access point for the Todo app and the API and UI apps aren't over-exposed on the network.

1. Push todos-api and todos-webui with the ``manifest-internal.yml`` in each project
1. Push the todos-edge and configure the API and UI internal routes in ``manifest-internal.yml``
1. Configure `todos.api.endpoint` and `todos.ui.endpoint` in `manifest-internal.yml`
    ```yaml
    ---
    applications:
    - name: todos-edge
      memory: 1G
      routes:
      - route: todos-edge.apps.retro.io
      - route: todos.apps.retro.io  
      path: target/todos-edge-1.0.0.SNAP.jar
      env: # internal routes
        TODOS_UI_ENDPOINT: http://todos-ui.apps.internal:8080
        TODOS_API_ENDPOINT: http://todos-api.apps.internal:8080
    ```
1. Add network policy to allow access to API and UI from Edge
    ```bash
    $ cf add-network-policy todos-edge --destination-app todos-api
    $ cf add-network-policy todos-edge --destination-app todos-webui
    $ cf network-policies
    Listing network policies in org retro / space arcade as corbs...
    source       destination   protocol   ports
    todos-edge   todos-api     tcp        8080
    todos-edge   todos-webui   tcp        8080    
    ```
1. Push todo-edge with internal routes ``cf push -f manifest-internal.yml`` (awwwweee yeah)

---

## Shop 5

### Introduce backing services for MySQL and Redis

* Refer back to the picture we're building...it would be nice to swap out todos-api which is just keeping an internal map of the data with something more apropos.  For instance with a database like MySQL or NoSQL store like Redis.
* Introduce
    * Spring Data at large
    * Spring Data Rest
    * Spring Data JPA
    * Spring Data Repositories, Crud and Paging
    * Spring Data Redis non-reactive and reactive
    * MySQL for PCF
    * Redis for PCF
* Inspect todos-mysql and todos-redis
* Compile, configure and cf push both
* Configure todos-edge with todos-mysql backend in git repo
* Refresh todos-edge
* Access your Todo(s) app and take note of data persistence in backing MySQL db
* Repeat the process again, configuring your todos-edge to use todos-redis instead
* Extra mile stuff - Using todos-shell deploy a ready made Todo(s) App with a MySQL backed API by running `shell:>push-scs-my-sql --tag myscs`
* Discuss pros and cons of both types of stores
* Start to introduce caching use-cases, patterns and position Pivotal Cloud Cache

        What have you done up to this point?  ...At this point in the shop each attendee should have coded, inspected, modified Spring Boot source code to at least 1 of 5 repositories, or perhaps just implemented some of these samples by hand as some have actually done.  At any rate attendees have seen and/or originated code for an Edge, API and/or UI app with and without Spring Cloud.  Been introduced to Java life on PCF (i.e. Java Buildpack and its features and perhaps with how to control), developers always want to know about JAVA_OPS, vm args and debugging java apps), how containers are built and the benefits of platform baked containers.  Folks should also have been introduced to backing services on PCF and how such services are consumed from Spring Boot apps.  This brings about an opportunity to discuss Spring Auto Reconfiguration in the Java Buildpack.  Developers at this point will typically ask questions around "Connection Pooling and Management", where does the DataSource come from?  

---

## Shop 6 

### Lookaside Caching Backend

This Shop puts together a backend for our Todo(s) app that implements Lookaside caching at the app level and leverages previously deployed todos-mysql and todos-redis instances as the System of Record and Cache respectively.  This Shop is focused on using cloud-native connectivity baked into Spring Cloud apps, for example the Lookaside Caching app (todos-app) integrates with the Sor and Cache using DiscoveryService and hence is able to leverage internal name resolution.  IPs and URIs come and go, Spring Cloud can go a long way to insulate application code from physical network properties.

*Note* that todos-app also fires events that Streams deployed in Shop 7 will use.

1. Inspect code for todos-app, discuss how DiscoveryService and RestTemplate gets created and name resolution in Spring Cloud
1. Configure VIPs for the Sor and Cache on Lookaside app
1. Push Lookaside Caching Backend
1. Configure Edge to use Lookaside Caching Backend
1. Refresh Edge
1. Access the UI

---

## Shop 7

### Spring Cloud Streams integration

In this Shop we code, inspect and deploy 3 Spring Cloud Stream apps and integrate using RabbitMQ for PCF to handle messaging.  

1. Code, Inspect and Build todos-sink and todos-processor
1. Create an on-demand rabbitmq service if one has not already been created, call it `todos-messaging` or `${yourname}-todos-messaging`.
1. Configure todos-sink and todos-processor with the rabbitmq service
1. Configure todos-sink and todos-processor with Spring Cloud services
1. Push the 2 apps to PCF
1. Note two things
    1. Now events being sent from todos-app are being written to the Sor
    1. Any Todo entered with a "hashtag" will be indexed (i.e. "Get some peanut butter #groceries")
1. Where to go from here?  Spring Cloud Tasks then Spring Cloud Data Flow
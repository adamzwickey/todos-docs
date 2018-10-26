## Todos Part 1  

Howdy! - what is this?  A sample set of apps using Spring Boot to handle Todo(s) various ways.  The point is not to build an awesome Todo app but to have a canonical model to use as we kick the tires on Spring Boot, Spring at large and complimentary frameworks such as Vue.js.  The idea comes from the wonderful and awesome work of ``todomvc.com`` which uses the Todo model to baseline and compare frontend stacks.

This section focuses on building and running the Part 1 set of Todo(s) apps, which consists of

1. [Todo(s) API Gateway](https://github.com/corbtastik/todos-cloud-gateway) - Spring Cloud Gateway
2. [Todo(s) WebMVC API](https://github.com/corbtastik/todos-api) - Spring WebMVC based Todos API
3. [Todo(s) WebFlux API](https://github.com/corbtastik/todos-webflux) - Spring WebFlux based Todos API
4. [Todo(s) UI](https://github.com/corbtastik/todos-ui) - todomvc.com Vue.js Todos UI

### PrePrep

If you've not completed [PrePrep](PREPREP.md) now would be a good time to do that :smile:

### Clone Part 1

```bash
> git clone https://github.com/corbtastik/todos-cloud-gateway.git
> git clone https://github.com/corbtastik/todos-api.git
> git clone https://github.com/corbtastik/todos-webflux.git
> git clone https://github.com/corbtastik/todos-ui.git
```  

### Build each project

Build each maven project then run from the command line or your IDE.  Todos UI is a non-JVM app and has a few dependencies to run locally, most notably Node.js/npm.  We'll see when we push to CF we don't need to worry about that setup.

You can also run the provided ``build.sh`` script from the directory that contains each project.  For example the ``WORKSPACE`` directory shown below.

```bash
# where WORKSPACE is any directory you have write access
$WORKSPACE/todos-api
$WORKSPACE/todos-cloud-gateway
$WORKSPACE/todos-ui
$WORKSPACE/todos-webflux
$WORKSPACE/clone.sh
$WORKSPACE/build.sh
$WORKSPACE/deploy.sh
```

```bash
> cd $WORKSPACE/todos-cloud-gateway
> mvnw clean package
```

```bash
> cd $WORKSPACE/todos-api
> mvnw clean package
```

```bash
> cd $WORKSPACE/todos-webflux
> mvnw clean package
```

```bash
> cd $WORKSPACE/todos-ui
> npm install
```

### Run locally

Start API Gateway

```bash
> cd $WORKSPACE/todos-cloud-gateway
> java -jar ./target/todos-cloud-gateway-1.0.0.SNAP.jar --eureka.client.enabled=false
# Started HttpServer on /0:0:0:0:0:0:0:0:9999
# Netty started on port(s): 9999
```

Start API

```bash
> cd $WORKSPACE/todos-api
> java -jar ./target/todos-api-1.0.0.SNAP.jar --eureka.client.enabled=false
# Tomcat started on port(s): 8080 (http) with context path ''
```

Start UI

```bash
> cd $WORKSPACE/todos-ui
> npm run dev
# Starting up http-server, serving .
# Available on:
#   http://127.0.0.1:4040
#   http://10.0.1.13:4040
# Hit CTRL-C to stop the server
```

Open a browser to the [Todos UI](`http://localhost:9999`)

### Run on CF

First make sure you're targeting a CF installation.  See info [here](PREPREP.md).

### Deploy  

Set these properties to suit your needs, if you cloned using default repository names the artifact properties should be fine, just add the route you want for the apps.  You can also override these variables on the cli when we push (see below).

```yml
app:
  # route: https://todosapp.cfapps.io
  route: https://YOUR-APP-ROUTE
api:
  # route: https://todos-api.cfapps.io
  route: https://YOUR-TODOS-API-ROUTE
  artifact: todos-api/target/todos-api-1.0.0.SNAP.jar
ui:
  # route: https://todos-ui.cfapps.io
  route: https://YOUR-TODOS-UI-ROUTE
  artifact: todos-ui/
gateway:
  # route: https://todos-cloud-gateway.cfapps.io
  route: https://YOUR-TODOS-CLOUD-GATEWAY-ROUTE
  artifact: todos-cloud-gateway/target/todos-cloud-gateway-1.0.0.SNAP.jar
```

```bash
# modify part-1-vars.yml with your values and push
> cf push -f part-1-manifest.yml --vars-file part-1-vars.yml
# OR override on cli
> cf push -f part-1-manifest.yml --vars-file part-1-vars.yml \
  --var app.route=https://YOUR-APP-ROUTE \
  --var api.route=https://YOUR-TODOS-API-ROUTE \
  --var ui.route=https://YOUR-TODOS-UI-ROUTE \
  --var gateway.route=https://YOUR-TODOS-CLOUD-GATEWAY-ROUTE
```

### Verify

Open a browser to `https://YOUR-TODOS-API-ROUTE`.

![Image of Todos UI](todos-part1.png)

Call the Spring Boot Todos API directly.

```bash
> http corbs-todosapp.cfapps.io/todos/
HTTP/1.1 200 OK
X-Todos-Cloud-Gateway-Route-Id: todos_all_api
X-Vcap-Request-Id: 8039d428-2260-430d-6b89-a2033f8057fd

[
    {
        "completed": true,
        "id": 5,
        "title": "make bacon pancakes"
    },
    {
        "completed": true,
        "id": 6,
        "title": "eat bacon pancakes"
    },
    {
        "completed": true,
        "id": 7,
        "title": "clean up bacon pancake mess"
    }
]
```
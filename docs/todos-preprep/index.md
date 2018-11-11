# README

## Pre-Prep  

Create a top-level directory to house all apps.  [Todo(s) CICD](https://github.com/corbtastik/todos-ui) requires all Todo(s) projects to exist on the same-level.

## Create and cd into project directory

We want to create a top-level directory and clone all repositories into this location.  Think of this directory as our ``TODOS_HOME``.

```bash
> mkdir todos-apps
> cd todos-apps
> export TODOS_HOME=`pwd`
```

## PAS Properties  

Before running deployment scripts edit ``todos-cicd/deploy.conf`` and add your PAS properties.  ``deploy.conf`` has presets for [Pivotal Web Services](https://run.pivotal.io).

```bash
#!/bin/bash
cf_api="https://api.run.pivotal.io"
cf_domain="cfapps.io"
cf_user=""
cf_pass=""
cf_org=""
cf_space=""
```

## PAS Servives

* MySQL
* RabbitMQ
* Redis
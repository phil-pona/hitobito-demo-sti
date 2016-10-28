# Hitobito Source to Image example

This example shows how hitobito can be deployed on APPUiO

## Architecture

We use the source to image workflow to build the application images. The Application consists of the following components_

* Ruby on Rails application, backend and frontend
* Delayed Job, same image but with different entrypoint
* Mysql Database
* Memcached
* SphinxSearch

## Dependencies

The Hitobito application uses the following services:

* mysql
* sphinx
* memcached 

therefore they are deployed as well and are available in the application template which is used to create the complete hitobito infrastructure

## Deployment

```
oc new-project hitobito
```

Create a the services incl. Persistent Storage

```
oc process -n openshift -f hitobito-demo-persistent.json \
-v RAILS_ROOT_USER_EMAIL=example@hitobito.ch \
-v RAILS_MAIL_DELIVERY_CONFIG="address: localhost, port: 25" \
-v RAILS_HOST_NAME=demo.hitobito.ch |\
oc create -f -
```

then install lets encrypt cert

### Manual Deployment

```
oc new-project hitobito
```

create Database
```
oc new-app mysql-ephemeral \
     -pMYSQL_USER=hitobito -pMYSQL_PASSWORD=hitobito \
     -pMYSQL_DATABASE=hitobito -pDATABASE_SERVICE_NAME=mysql
```

Create application (s2i ruby standards)
```
oc new-app centos/ruby-22-centos7~https://github.com/phil-pona/hitobito-demo-sti.git --name=hitobito
oc expose service hitobito 
oc env dc hitobito \
      -e RAILS_ENV=production \
      -e RAILS_SERVE_STATIC_FILES=1  \
      -e RAILS_DB_NAME=hitobito \
      -e RAILS_DB_HOST=mysql \
      -e RAILS_DB_USERNAME=hitobito \
      -e RAILS_DB_PASSWORD=hitobito \
      -e RAILS_DB_ADAPTER=mysql2 \
      -e RAILS_HOST_NAME=hitobito-pitc-hitobito-test.ose3.puzzle.ch \
      -e RAILS_SPHINX_HOST=sphinx \
      -e MEMCACHE_SERVERS=memcached:11211 \
      -e RAILS_ROOT_USER_EMAIL=hitobito@puzzle.ch 
```

Create application (pitc rails)
```
oc new-app pitc-rails-bi-prod/ose3-rails~https://github.com/phil-pona/hitobito-demo-sti.git --name=hitobito-rails
oc expose service hitobito-rails 
oc env dc hitobito-rails \
      -e RAILS_ENV=production \
      -e RAILS_SERVE_STATIC_FILES=1  \
      -e RAILS_DB_NAME=hitobito \
      -e RAILS_DB_HOST=mysql \
      -e RAILS_DB_USERNAME=hitobito \
      -e RAILS_DB_PASSWORD=hitobito \
      -e RAILS_DB_ADAPTER=mysql2 \
      -e RAILS_HOST_NAME=hitobito-pitc-hitobito-rails-test.ose3.puzzle.ch \
      -e RAILS_SPHINX_HOST=sphinx \
      -e MEMCACHE_SERVERS=memcached:11211 \
      -e RAILS_ROOT_USER_EMAIL=hitobito@puzzle.ch 
```

Mail config

```
-
              name: RAILS_MAIL_DELIVERY_CONFIG
              value: 'address: mail.example.com, port: 25'
```

#### Recreate deployment strategy

edit in dc

```
spec:
  strategy:
    type: Recreate
    recreateParams:
      timeoutSeconds: 600
```

#### incremental Build

edit in bc

```
sourceStrategy:
      from:
        kind: ImageStreamTag
        name: 'ruby-22-centos7:latest'
      incremental: true
```

#### DB Migrations

Livecycle hock
```
      pre:
        failurePolicy: Abort
        execNewPod:
          command:
            - /usr/bin/bash
            - '-c'
            - 'bundle exec rake db:migrate db:seed wagon:setup'
          containerName: hitobito
```

### Memcached

oc new-app https://github.com/appuio/memcached.git  --strategy=docker --name=memcached

### Sphinx

oc new-app https://github.com/appuio/sphinxsearch.git  --strategy=docker --name=sphinx

## HTTPS Route

Letsencrypt

## Configuration


## Build local

download s2i binary from https://github.com/openshift/source-to-image

run
```
s2i build --scripts-url=file://.s2i/bin . centos/ruby-22-centos7 hitobito-s2i-exmaple
```


## TODO

* Logging to standard out
** memcached
** sphinx
** application
* switch to puzzle rails base image
* sphinx configuration with envs replace values in configfile from env, and application part from rails
* Delay jobs must run in foreground and log to the console
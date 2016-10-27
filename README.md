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

Create application
```
oc new-app openshift/ruby-22-centos7~https://github.com/phil-pona/hitobito-demo-sti.git --name=hitobito
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
      -e MEMCACHE_SERVERS=memchache:11211
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

## Configuration


## Build local

download s2i binary from https://github.com/openshift/source-to-image

run
```
s2i build --scripts-url=file://.s2i/bin . openshift/ruby-22-centos7 hitobito-s2i-exmaple
```


## TODO

Logging to standard out
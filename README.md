YANGCATALOG
===========

You can find official yangcatalog website in [here](https://yangcatalog.org).

# Deployment
Main repository to start up all the pieces of yang-catalog

## Requirements

* Docker, including docker-compose
* IPv4 Internet connectivity (Docker Hub doesn't do IPv6 yet)
* Sufficient space for container images (approx 10 GB)
* Sufficient space for all the modules data
  - approx 45 GB for local development
  - up to 250 GB for production
* Sufficient RAM
  - approx 8 GB for local development
  - up to 24 GB for production

### External requirements

The deployment uses several services via container images that are
distributed by third parties, e.g. via DockerHub

* elasticsearch
* rabbitmq
* mariadb
* nginx (as the base image of the frontend container, which includes
  static content)

## Basic Usage

Create `conf/yangmodules.conf` from `conf/yangmodules.conf.sample`,
replacing values enclosed in `@...@` with your own.

```
git submodule init
git submodule update
docker-compose build # not strictly necessary
docker-compose up
```

The `docker-compose build` should build container images for the
various components of the Yang Catalog.

The `docker-compose up` will start containers from these images, as
well as from some third-party container images (e.g. RabbitMQ,
MariaDB) and combine them into a functional local deployment
of the Yang Catalog, which should be accessible on
http://localhost

Make sure that you will start the elasticsearch Dockerfile as well
since in production environment this is not used and an AWS elasticsearch
instance is used instead

## Status

### Elasticsearch

Elasticsearch instance can be started in two different ways. Locally
you can set "es-aws" in your yangcatalog.conf file to `False` and start
the elasticsearch manually using `docker run` command. Also es-host
has to be set to elasticsearch instance IP (k8s setup) or
`yc-elasticsearch` (docker-compose setup) and port to `9200`. In production
we use AWS elasticsearch service. This instance runs on different server
and are connected to yangcatalog server. For this the "es-aws" needs to be
set to `True` and es-host set to whatever URL AWS provides for elasticsearch.
In AWS make sure that this URL is not opened for internet but is opened for
the other instance that is running yangcatalog.org

### Logging

All the logging is done to different files that should be located in
one directory which is specified in yangcatalog.conf config file.
Normally you would find this in /var/yang/logs. This directory is
a volume shared with host system. All the logs can be checked and
filtered using yangcatalog admin UI if you have access to that.

### ConfD

The Yang Catalog requires ConfD, which is not open source and cannot
be included here in the same way as the other components. In order
for yangcatalog to be working you need to acquire confd since this
is a beating heart of yangcatalog and contains all the metadata
about each module. Confd binary is copied to resources and each
docker service that will need this will install it in its own
container.

### Kubernetes support

It is possible to deploy YANGCATALOG on Kubernetes by following this [guide](./k8s/README.md).

### OpenShift support

OpenShift is, at its core, a multi-tenant variant of Kubernetes.  It
imposes some limitations on the containers it runs; for example,
processes run under a non-root UID.  Many container images are build
under the assumption that they _will_ run as root.

## Structure of the Repository

Various open-source components of the Yang Catalog are imported as
submodules:

* [backend](https://github.com/YangCatalog/backend) - the Yang Catalog
  API server
* [search](https://github.com/YangCatalog/search) - the Yang search
  Web application - repo DISCONTINUED
* [web_root](https://github.com/YangCatalog/web_root) - static HTML,
  CSS etc. content
* [yangre-gui](https://github.com/plewyllie/yangre-gui) - Peter
  Lewyllie's Yang Regular Expression checker
* [yangvalidator](https://github.com/YangCatalog/bottle-yang-extractor-validator) - Carl
  Moberg's Yang validation application
* [sdo_analysis](https://github.com/YangCatalog/sdo_analysis) - Scripts
  to analyze and validate yang files
* [admin_ui](https://github.com/YangCatalog/admin_ui) - Admin frontend
  to monitor and manage yangcatalog.org
* [yangcatalog-ui](https://github.com/YangCatalog/yangcatalog-ui) - Whole Yang Catalog frontend - Angular app

`docker-compose.yml` is the actual "orchestration" that attempts to
describe a complete (modulo ConfD) deployment of the Yang Catalog.

The `conf` directory contains a few files that are passed to the
containers using bind mounts.

### .env variables

Some of the following variables if changed here has to corespond with yangcatalog.conf file

`COMPOSE_PROJECT_NAME=yc` - When running a docker-compose up command it will create
a docker containers that have some specific name (like backend, frontend, yang-search...).
This will add a prefix to these names so in this example we would have yc-backend, yc-frontend...

`MYSQL_ROOT_PASSWORD=<ROOT PASSWORD>` - Password to get into mysql cli as a root or to make any
changes as root. This password has to be used in any of these cases

`MYSQL_DATABASE=yang_catalog` - Database name

`MYSQL_USER=yang` - yangcatalog specific user for mysql

`MYSQL_PASSWORD=<MYSQL PASSWORD>` - Password to get into mysql cli as a yang user or to make any
changes as yang user. This password has to be used in any of these cases. It can be same as root
password although this is not recommended.

`MYSQL_VOLUME=/var/yang/mysql` - where are all the mysql database files located

`RABBITMQ_USER=<RABBITMQ_USER>` - rabbitmq username.

`RABBITMQ_PASSWORD=<RABBITMQ PASSWORD>`  - rabbitmq username.

`ELASTICSEARCH_DATA=/var/lib/elasticsearch` - This is only used with local elasticsearch. If AWS
elasticsearch is used this variables can be set to anything. If local elasticsearch is used this
variable is used to specify where elasticsearch indexed data should be saved

`ELASTICSEARCH_LOG=/var/log/elasticsearch`  - This is only used with local elasticsearch. If AWS
elasticsearch is used this variables can be set to anything. If local elasticsearch is used this
variable is used to specify where elasticsearch logs should be saved

`NGINX_LOG=/var/log/nginx` - Where do we save nginx logs

`KEY_FILE=/home/yang/deployment/resources/yangcatalog.org.key` - certificate key file for HTTPS protocol.
If None exists just use any path

`CERT_FILE=/home/yang/deployment/resources/yangcatalog.org.crt` - certificate file for HTTPS protocol.
If None exists just use any path

`CA_CERT_FILE=/home/yang/deployment/resources/yangcatalog.org.crt` - ca_certificate file for HTTPS protocol.
If None exists just use any path

`ELASTICSEARCH_ID=1001` - This is only used with local elasticsearch. If AWS
elasticsearch is used this variables can be set to anything. If local elasticsearch is used this
variable is used to set permissions to specific user ID. It`s safe to use same ID as YANG_ID

`ELASTICSEARCH_GID=1001` - This is only used with local elasticsearch. If AWS
elasticsearch is used to set permissions to specific user GID. It`s safe to use same GID as YANG_GID

`YANG_ID=1016` - ID created for yang user

`YANG_GID=1016` - GID created for yang user

`MYSQL_ID=1001` - ID created for mysql. It`s safe to use same ID as YANG_GID

`MYSQL_GID=1001` - GID created for mysql. It`s safe to use same GID as YANG_GID

`YANG_RESOURCES=/var/yang` - specify path where all the yangcatalog data will be saved.

`NGINX_FILES=yangcatalog-nginx*.conf` - NGINX config files used for NGINX. There is testing config file
or production one with HTTPS. Read [documentation](./DOCUMENTATION) file to find out how to use this variable

`GIT_USER_NAME=foo` - yangcatalog is pushing some data to github repository and it needs to
set usename before any commit is done.

`GIT_USER_EMAIL=bar@foo.com` - yangcatalog is pushing some data to github repository and it needs to
set email address before any commit is done.

`CRON_MAIL_TO=miroslav.kovac@pantheon.tech` - comma separated list of emails which are used
with cron jobs. If any cron job will fail it will send it to this comma separated list of email addresses.
This is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md fileThis is added to the end of README.md file
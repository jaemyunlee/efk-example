# EFK stack example

## How to set up and test

Environment : Mac

### step 1. Install Docker and init swarm mode.

`docker swarm init`

### step 2. Create a docker image.

`$ docker build -t jayground/fluentd -f ./fluentd/Dockerfile .`

you can check images you just made.

`$ docker image ls`

### step 3. stack deploy.

`$ docker stack deploy -c docker-compose.yml efk`

You can check service status.

`$ docker service ls`

You can also check with visualizer.

hit 'http://localhost:8080' on your browser.

All services should be running.

### step 4. Kibana setup.

Management/Index Patterns

- local-swarm-log-*

Discover logs in Kibana

### step 5. access httpd and leaves logs.

`$ curl http://localhost:80`

you can see new log in Kibana.

### step 6. deletes logs with Elastic Curator.

```$ pip install elasticsearch-curator
$ curator --config /usr/share/elasticsearch/curator.yml /usr/share/elasticsearch/action.yml

INFO      Deleting selected indices: [u'local-swarm-log-20180507']
INFO      ---deleting index local-swarm-log-20180507
INFO      Action ID: 1, "delete_indices" completed.
INFO      Job completed.
```

If you don't have logs more than 2 days, It doesn't delete any logs.
Because It sets up to delete all logs made 2 days prior.

```
# elasticsearch/curator/action.yml

unit: days
unit_count: 2
``` 

7. setup cron job to empty logs regularly.

```
$ crontab -e
```

## Elasticsearch 6.2.4

refer to [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

The images are available in three different configurations or "flavors".
 
- basic: ships with X-Pack Basic features pre-installed and automatically activated with a free license.
- platinum: features all X-Pack functionally under a 30 day trial license.
- __oss: does not include X-Pack and contains only open-source Elasticsearch__.

We doesn't need to install X-Pack.

```
# elasticsearch/Dockerfile

FROM docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.4
```

It uses Elasticsearch Curator and Cron to empty log data every two days.

Elasticsearch Curator helps you curate, or manage, your Elasticsearch indices and snapshots.

## Fluentd 1.1.x

To reduce the size, It installs Alpine version.

refer to [fluent/fluentd Docker Hub documentation]

```
# In the document

RUN apk add --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install \
        fluent-plugin-elasticsearch \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* \
           /home/fluent/.gem/ruby/2.3.0/cache/*.gem
```

It adds fluent-plugin-concat to concat stderr and install with options to skip documents installation.

gem install —no-rdoc —no-ri

By default, RubyGems will install documents (ri, rdoc) together with gem when you install gem. 
—no-rdoc —no-ri options skip documents installation.

```
# fluentd/Dockerfile

RUN apk add --update --virtual .build-deps \
        sudo build-base ruby-dev \
 && sudo gem install \
        fluent-plugin-elasticsearch --no-rdoc --no-ri --version 2.9.2\
 && sudo gem install \
        fluent-plugin-concat --no-rdoc --no-ri --version 2.2.2\
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* \
           /home/fluent/.gem/ruby/2.3.0/cache/*.gem
```

fluent-plugin-concat

It concats stderr per traceback. If you don't set this, it leaves every single line as log for stderr.

```
<filter **>
  @type concat
  key log
  multiline_start_regexp /^Traceback/
</filter>
```

fluent-plugin-elasticsearch

It sets prefix as 'local-swarm-log'. So You can find index pattern with 'local-swarm-log-*' in Kibana.
```
<match **>
  @type copy
  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix local-swarm-log
    ...
  </store>
  <store>
    @type stdout
  </store>
</match>
```

## Kibana 6.2.4

Like Elasticsearch, install Kibana without X-Pack

```
# docker-compose.yml

image: docker.elastic.co/kibana/kibana-oss:6.2.4
```

## docker-compose.yml

### Network

```
networks:
  intraservice:
    driver: overlay
    ipam:
      driver: default
```

### Logging

you can set logging option for service.

```
logging:
  driver: "fluentd"
     options:
        tag: httpd.access
```
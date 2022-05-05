# Kafka


![kafka.scheme](https://github.com/DevEnv-94/Logs/blob/master/Kafka/images/scheme.png)


#### server.properties
```bash
...
delete.topic.enable = true

listeners=PLAINTEXT://{{ ansible_eth1.ipv4.address }}:9092
...

```

#### zookeeper.properties
```bash

dataDir=/var/zookeeper

clientPort=2181

maxClientCnxns=0

admin.enableServer=false

```




#### Fluent.app
```bash

<source>
  @type tail
  path /var/log/nginx/app2_access.log
  pos_file /var/log/td-agent/nginx.access.app2.pos
  tag nginx.access.app2
  <parse>
    @type nginx
  </parse>
</source>


<filter **>
  @type record_transformer
  enable_ruby true

  <record>
    @tag ${tag}
  </record>
</filter>


<match nginx.access.**>
  @type kafka2

  brokers {{ hostvars[groups['log_server'][0]]['ansible_eth1']['ipv4']['address'] }}:9092

  default_topic test-logs

  <format>
    @type json
  </format>

  <buffer>
    @type file
    path /var/log/td-agent/logs_buf/
    flush_mode interval
    flush_interval 10s
    flush_at_shutdown true  
  </buffer>

</match>

```


#### Fluent.log
```bash

<source>
  @type kafka

  brokers {{ ansible_eth1.ipv4.address }}:9092
  topics test-logs
  format json
  add_prefix from_kafka

</source>

<match from_kafka**>

  @type elasticsearch
  host {{ ansible_eth1.ipv4.address }}
  port 9200
  index_name apps_logs
  include_tag_key true
  tag_key @log_name
  <buffer>

    @type file
    path /var/log/td-agent/buf
    flush_at_shutdown true
    flush_interval 10s

  </buffer>

</match>

```
# Send logs to elasticsearch

#### elasticksearch.yml config file

```bash

cluster.name: elastic-test

node.name: server

path.data: /var/lib/elasticsearch

path.logs: /var/log/elasticsearch

http.port: 9200

xpack.security.enabled: false

xpack.security.enrollment.enabled: false


xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12


xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12

cluster.initial_master_nodes: ["server"]


http.host: [_local_, _site_]

```

#### fluentd config file

```bash

#td-agent --dry-run -c /etc/td-agent/td-agent.conf 
#curl -XGET http://localhost:9200/_search?size=100 | jq

<source>

  @type tail
  path /var/log/*.log
  pos_file /var/log/td-agent/pos_file.log.pos
  tag elastic.*
  
  <parse>
    @type none
  </parse>

</source>

<match elastic.**>

  @type elasticsearch
  host 127.0.0.1
  port 9200
  logstash_format true
  logstash_prefix task07.${tag[3]}
  logstash_dateformat %Y-%m-%d
  include_tag_key true
  tag_key @log_name
  flush_interval 10s

  <buffer tag>

    @type file
    path /var/log/td-agent/buf
    flush_at_shutdown true
    flush_interval 10s

  </buffer>

</match>

```
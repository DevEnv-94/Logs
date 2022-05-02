# Elasticsearch cluster


### Best Practise

* The Elasticsearch cluster runs at the speed of the slowest data node, so it is recommended to use servers with the same parameters for data nodes.

* ES thinks not in terms of disk space, but in shards, and tries to evenly distribute them among Data nodes. If there is more space on one of the data nodes than on the others, then it will be uselessly idle.

* In Elasticsearch, a new day begins at zero hours UTC, not local time at all. Critical for automatic index creation.

* Elasticsearch will not store two copies of a shard on the same node, so it makes no sense to require a replica to be created in the case of a single node.


```yaml

# increase file descriptors

      - name: Increase max open files to 65536
        community.general.pam_limits:
          domain: '@elasticsearch'
          limit_type: '-'
          limit_item: nofile
          value: 65536


# Kernel parametres
      - name: Decrease swappines to '1'
        ansible.posix.sysctl:
          name: vm.swappiness
          value: '1'
          state: present
          reload: yes

      - name: Increase vm.max_map_count to '262144'
        ansible.posix.sysctl:
          name: vm.max_map_count
          value: '262144'
          state: present
          reload: yes

```

```bash

curl -XGET 'http://VM_IP:9200/_cluster/state?pretty' #info about node in cluster
sudo curl -sS -XGET 'http://VM_IP:9200/_cat/master?pretty' #who is master
curl -XGET 'http://VM_IP:9200/_cluster/health?pretty' #cluster state
curl -XGET http://VM_IP:9200/_search?size=100 | jq 

```


```bash 

As a general rule, the maximum heap size should be no more than 50% of RAM and no more than 32766m 

-Xms32766m 
-Xmx32766m

```


![Elastic_scheme](https://github.com/DevEnv-94/Logs/blob/master/Elasticsearch_cluster/images/scheme.png)


#### Rsyslog config file

```bash

syslog.* {
	action(type="omfwd" target="127.0.0.1" port="55514" protocol="udp")
}

```

#### elasticsearch.yml cluster config file in ninja

```bash

cluster.name: test_es_cluster

node.name: test-es-node-{{node_number}}

{% if node_number == 2 -%}
node.roles: [ data, master, voting_only ]
{% else -%}
node.roles: [ data, master ]
{% endif %}

network.host: {{ ansible_eth1.ipv4.address }}

http.port: 9200

discovery.seed_hosts: ["{{ hostvars[groups['elasticsearch'][0]]['ansible_eth1']['ipv4']['address'] }}", "{{ hostvars[groups['elasticsearch'][1]]['ansible_eth1']['ipv4']['address'] }}","{{ hostvars[groups['elasticsearch'][2]]['ansible_eth1']['ipv4']['address'] }}"]

cluster.initial_master_nodes: ["test-es-node-1", "test-es-node-2","test-es-node-3"] 


path.data: /var/lib/elasticsearch

path.logs: /var/log/elasticsearch

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

```

#### Fluent.Apps config file in ninja

```bash

<source>
  @type http
  port 8080
  bind 127.0.0.1
  tag applogs.{{ ansible_hostname }}
</source>

<source>
  @type syslog
  port 55514
  tag syslogs.{{ ansible_hostname }}
  bind 127.0.0.1
</source>

<filter **>
  @type record_transformer
  enable_ruby true

  <record>
    tag ${tag}.${hostname}
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match {applogs.**,syslogs.**}>
  @type forward

  <server>
    host {{ hostvars[groups['fluent'][0]]['ansible_eth1']['ipv4']['address'] }}
    port 24224
  </server>

  <buffer>
    @type file
    path /var/log/td-agent/logs_buf/
    flush_mode interval
    flush_interval 10s
    flush_at_shutdown true  
  </buffer>
</match>

#<filter **>
# @type record_transformer
#  enable_ruby true
#  <record>
#    tag ${tag}.${hostname}
#    hostname "#{Socket.gethostname}"
#    @timestamp ${time.strftime('%Y-%m-%dT%H:%M:%S')} #"2021-04-16T18:52:03"
#  </record>
#</filter>

```

#### Fluent.aggregator config file in ninja

```bash

<source>
  @type forward
  port 24224
  bind {{ hostvars[groups['fluent'][0]]['ansible_eth1']['ipv4']['address'] }}
</source>

<source>
  @type syslog
  port 55514
  tag syslogs
  bind 127.0.0.1
</source>

<match {applogs.**,syslogs.**}>
  @type elasticsearch

  hosts {{ hostvars[groups['elasticsearch'][0]]['ansible_eth1']['ipv4']['address'] }}:9200,{{ hostvars[groups['elasticsearch'][1]]['ansible_eth1']['ipv4']['address'] }}:9200,{{ hostvars[groups['elasticsearch'][2]]['ansible_eth1']['ipv4']['address'] }}:9200
  index_name ${tag[0]}
  include_timestamp true
  time_key_format %Y-%m-%dT%H:%M:%S
    <buffer>
      @type file
      path /var/log/td-agent/log_buf/
      flush_mode interval
      flush_interval 10s
      flush_at_shutdown true
    </buffer>
</match>

```

#### Application(log_generator)

```bash

#!/bin/bash
for (( i=1; i <= 12; i++ ))
do
  curl -i -X POST -d 'json={"action":"login","user":"'"$((RANDOM%100))"'", "current_timestamp": '$(date +%s)'}' http://localhost:8080/applogs
  sleep 5
done

```
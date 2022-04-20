# Fluentd config files


```bash

#td-agent --dry-run -c /etc/td-agent/td-agent.conf 


# Fluentd apps config file
<source>

  @type tail
  path /var/log/*.log
  pos_file /var/log/td-agent/pos_file.log.pos
  tag {{ ansible_hostname }}.*
  
  <parse>
    @type none
  </parse>

</source>


<filter {{ ansible_hostname }}.**>

  @type record_transformer
  <record>
    tag "${tag}"
  </record>

</filter>

<match {{ ansible_hostname }}.**>
  @type forward

  <buffer tag>

    @type file
    path /var/log/td-agent/file
    timekey 1m

  </buffer>

  <server>
    host {{ hostvars[groups['logs'][0]]['ansible_eth1']['ipv4']['address'] }}
    port 24225
  </server>

</match>

```

```bash

# Fluentd log config file
#td-agent --dry-run -c /etc/td-agent/td-agent.conf 
#aws s3 ls s3://{{ s3_bucket }} --recursive

<source>

  @type forward
  port 24225
  bind 0.0.0.0
  add_tag_prefix host

</source>


<match host.**>

  @type copy

  <store>

  @type file
  path /tmp/logs/${tag[1]}/${tag[4]}
  append true

    <buffer tag>

      @type file
      path /var/log/td-agent/file
      timekey 1m

    </buffer>

  </store>

  <store>

  @type s3
    aws_key_id {{ aws_key_id }}
    aws_sec_key {{ aws_sec_key }}
    s3_bucket {{ s3_bucket }}
    s3_region {{ s3_region }}
    path logs/${tag[1]}/${tag[4]}.${tag[5]}

    <buffer tag>

      @type file
      path /var/log/td-agent/s3
      timekey 1m

    </buffer>

  </store>

</match>

```

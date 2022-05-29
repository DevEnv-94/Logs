# Logs from docker


```bash

#Flunetd can monitor all containers logs on node.
#Containers does not require specific journal driver or write to cpecific tom.
#Fluentd needs read permissions to log files

<source>
    @type tail
    @id in_tail_container_logs
    path /var/lib/docker/containers/*/*-json.log
    pos_file /fluentd/log/containers.log.pos
    time_format "%Y-%m-%dT%H:%M:%S.%L%Z"
    keep_time_key true
    read_from_head true
    tag "docker.*"
    format json
</source>

```



```bash

#Configure default fluentd driver for docker
#Flunetd is required been configured before this changes of log driver

$ cat /etc/docker/daemon.json 

{
   "log-driver": "fluentd",
   "log-opts": {
     "fluentd-address": "localhost:24224"
   }
}

```



####Config files in my case

```bash

<source>

  @type forward
  port 24224
  bind 0.0.0.0
  add_tag_prefix docker

</source>

<match docker.**>

  @type file

  path /var/log/fluentd


    <buffer tag>

      @type file
      path /var/log/td-agent/buf/
      chunk_limit_size 500
      chunk_limit_records 10
      flush_mode interval
      flush_interval 30s
      flush_at_shutdown true
      
    </buffer>

</match>

```


```bash

$ cat /etc/docker/daemon.json 

{
   "log-driver": "fluentd",
   "log-opts": {
     "fluentd-address": "{{ hostvars[groups['fluentd'][0]]['ansible_eth1']['ipv4']['address'] }}:24224"
   }
}

```


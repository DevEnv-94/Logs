# Fluentd config file


```bash

#td-agent --dry-run -c /etc/td-agent/td-agent.conf 

<source>

  @type tail
  read_from_head true
  path /var/log/nginx/access.log
  pos_file /var/log/td-agent/nginx.access.log.pos
  tag nginx.access
  <parse>
    @type nginx
  </parse>

</source>


# rewrite nginx.acces tag to nginx.* where '*' status code of nginx log
<match nginx.access>
  @type rewrite_tag_filter

  <rule>
    key code
    pattern /([1-5][0-9]{2})/
    tag nginx.$1
  </rule>
</match>


<match {nginx.1**,nginx.2**,nginx.3**}>
  @type file
  path /tmp/1xx-3xx
</match>

<match {nginx.4**,nginx.5**}>
  @type file
  path /tmp/4xx-5xx
</match>

```


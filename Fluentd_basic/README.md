# Fluentd config file


```bash

#td-agent --dry-run -c /etc/td-agent/td-agent.conf  check config

# /etc/rsyslog.d/50-default.conf
#module(load="imudp")
#input(type="imudp" port="514")

# check logs sending https://github.com/mingrammer/flog

<source>

  @type tail
  read_from_head true
  path /var/log/app.log
  pos_file /var/log/td-agent/app.log.pos
  tag fluentd
  <parse>
    @type apache2
  </parse>

</source>

<match fluentd>

  @type remote_syslog
  host {{ ansible_ssh_host }}
  port 514
  program fluentd

</match>

```
# Rsyslogd config files


```bash

#/etc/rsyslog.d/20-app.conf

syslog.=crit @{{ hostvars[groups['rsyslog'][0]]['ansible_eth1']['ipv4']['address'] }};RSYSLOG_SyslogProtocol23Format  # sends only crytical  priority syslog messages 
cron.*;cron.!=debug @{{ hostvars[groups['rsyslog'][0]]['ansible_eth1']['ipv4']['address'] }}:515;RSYSLOG_SyslogProtocol23Format # sends all cron messages except debug priority

```

```bash

#/etc/rsyslog.d/22-rsyslog.conf

module(load="pmciscoios")

module(load="imudp")
input(type="imudp" port="514" ruleset="remote")
input(type="imudp" port="515" ruleset="remote2")

ruleset(name="remote"){
        if prifilt("syslog.=crit") then {
        action(type="omfile" file="/var/log/rsyslog/syslog")
                stop      
        }
}
ruleset(name="remote2"){
        if prifilt("cron.*;cron.!=debug") then {
        action(type="omfile" file="/var/log/rsyslog/cron")
                stop        
        }
}

#syslog.crit /var/log/rsyslog/syslog
#cron.*;cron.!=debug /var/log/rsyslog/syslog



#$template DynamicFile,"/var/log/rsyslog/%syslogfacility-text%"
#cron.*;cron.!=debug ?DynamicFile

#$template DynamicFile1,"/var/log/rsyslog/%syslogfacility-text%"
#syslog.=crit ?DynamicFile1

#$template RemoteHost,"/var/syslog/hosts/%HOSTNAME%/syslog
#syslog.=crit RemoteHost

```
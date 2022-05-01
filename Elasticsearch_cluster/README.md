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

Max Heap size not 32g

-Xms32766m 
-Xmx32766m

```


![Elastic_scheme](https://github.com/DevEnv-94/logs/blob/master/images/grafana.png)
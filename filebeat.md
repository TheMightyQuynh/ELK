## Configure Filebeat
Edit `/etc/filebeat/filebeat.yml`
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/ubuntu/data/*.json

filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 10s
  
setup.template.settings:
  index.number_of_shards: 1

setup.kibana:
  host: "10.255.255.252:5601"

output.logstash:
  hosts: ["10.255.255.252:5044"]

seccomp:
  default_action: allow
  syscalls:
  - action: allow
    names:
    - rseq
```

## Handy stuff
Log location: `/var/log/filebeat`

Data path (configurable): `/home/ubuntu/data`

Command to test configuration:
```
filebeat test config /etc/filebeat/filebeat.yml
```

Command to test output:
```
filebeat test output
```

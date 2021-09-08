# ELK
Notice echnology stack Elastiksearch Logstash Kibana
## Storage menegment with curator & ILM policy
### Exmple with use curator if not using ILM
```
Request to elasticserch to get indexes.
curl -s -XGET -u <user>:<password> 127.0.0.1:9200/_cat/indices
delete some indexes with temlplete
curl -XDELETE -u <user>:<password> http://127.0.0.1:9200/prod-*
```
then must be instaled consul
config use yml linter config.yml
```
---
client:
  hosts:
    - 127.0.0.1
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  username: <user>
  password: <password>
  timeout: 30
  master_only: False

logging:
  loglevel: INFO
  logfile:  /var/log/curator/curator.log
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```
action.yml # this is ACTION what we will do with reqest to host in config file.
delete indexes with ttl above 30 days template = value (value: prod*)
```
---
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 30 days (based on index creation date)
    options:
      ignore_empty_list: True
    filters:
    - filtertype: pattern
      kind: prefix
      value: prod*
    - filtertype: age
      source: creation_date
      direction: older
      unit: days
      unit_count: 30
```
All configuration can find in oficial documentation
### ILM policy
if with have indexes without flags used policy
then we delete indexes with consul
ILM polycy can tune with kibana or using api requst to elastic search like POST request.
```
api request curl XGET 127.0.0.1:9200/_ilm/policy/Deleting_old_indexes. | jq

{
  "Deleting_old_indexes.": {
    "version": 3,
    "modified_date": "2021-08-13T04:32:45.885Z",
    "policy": {
      "phases": {
        "hot": {
          "min_age": "0ms",
          "actions": {
            "set_priority": {
              "priority": 50
            }
          }
        },
        "delete": {
          "min_age": "30d",
          "actions": {
            "delete": {
              "delete_searchable_snapshot": true
            }
          }
        }
      }
    }
  }
}
```
then we can check indexes 
with ILM polycy flags
```
request GET prod-*/_ilm/explain

    "prod-2021.08.30" : {
      "index" : "prod-2021.08.30",
      "managed" : true,
      "policy" : "Deleting_old_indexes.",
      "lifecycle_date_millis" : 1630281611148,
      "age" : "9.08d",
      "phase" : "hot",
      "phase_time_millis" : 1630281622752,
      "action" : "complete",
      "action_time_millis" : 1630281624186,
      "step" : "complete",
      "step_time_millis" : 1630281624186,
      "phase_execution" : {
        "policy" : "Deleting_old_indexes.",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "set_priority" : {
              "priority" : 50
            }
          }
        },
```
without ILM flags
```
"prod-08.10" : {
      "index" : "prod-mc-service-facts-2021.08.10",
      "managed" : false
    },

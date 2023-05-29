# APINotice
A service-level alarm platform based on kibana watcher / 一个基于Kibana Wathcer的服务分级报警平台

## Intro

About more news, please click here.
- [设计与实现基于Kibana Wathcer的服务分级报警平台](https://github.com/WGrape/Blog/issues/220)

## Configure
<details>

<summary>configure template</summary>

```json
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": [
          "<log-yourbusiness-{now/d}>"
        ],
        "rest_total_hits_as_int": true,
        "body": {
          "query": {
            "bool": {
              "must": [],
              "filter": [
                {
                  "bool": {
                    "should": [
                      {
                        "match": {
                          "level": "FATAL"
                        }
                      }
                    ],
                    "minimum_should_match": 1
                  }
                },
                {
                  "match_phrase": {
                    "level": "FATAL"
                  }
                },
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-1m"
                    }
                  }
                }
              ],
              "should": [],
              "must_not": []
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.hits.total": {
        "gte": 1
      }
    }
  },
  "actions": {
    "my_webhook": {
      "webhook": {
        "scheme": "http",
        "host": "webhook.yousite.com",
        "port": 80,
        "method": "post",
        "path": "/alerts/trigger/",
        "headers": {
          "Content-Type": "application/json"
        },
        "body": "{\"status\":\"alert\",\"labels\":{\"alertname\":\"business-watcher-level\"},\"annotations\":{\"summary\":\"【业务服务S级报警】{{ctx.payload.hits.hits.0._source.service_id}} ({{ctx.payload.hits.hits.0._source.uri}}) 一分钟 {{ctx.payload.hits.total}} 条\",\"description\":\"[{{ctx.payload.hits.hits.0._source.service_id}}] 一分钟内产生 {{ctx.payload.hits.total}}  条错误日志 , human_time: {{ctx.payload.hits.hits.0._source.human_time}} , traceid: {{ctx.payload.hits.hits.0._source.traceid}} , service_id: {{ctx.payload.hits.hits.0._source.service_id}} , message: {{ctx.payload.hits.hits.0._source.message}}\",\"detail_url\":\"\"}}"
      }
    }
  }
}
```

</details>

## Example

<img width="1726" alt="截屏2023-05-29 21 23 53" src="https://github.com/WGrape/runview/assets/35942268/673ad66f-a74e-44c3-98a2-dd8d00260cb7">

<img width="1717" alt="截屏2023-05-29 21 24 46" src="https://github.com/WGrape/runview/assets/35942268/034b26df-c29e-44a2-bc60-f1ab64bcc277">

# ELK Stack on Docker

## 진행상황

- coin-pipeliner 네트워크 내에서 kafka-cluster, producer, elk stack 모두 연결 완료
- vega 문법을 이용하여 candle chart visualize
- 현재는 데이터를 읽어들이는 대로 저장하고 시각화 하기 때문에 생명주기 관리가 필요하다. 데이터가 쌓이기만 하는 상황

## 앞으로의 과제

- flick 의 분석결과 시각화
- 여러 인덱스를 같이 시각화
- 인덱스 생명주기 관리 hot-warm-cold cycle (못할 가능성 농후)

## 실행 방법

1. 커스텀 네트워크 생성 (이미 있으면 Skip)

```bash
docker network create coin-pipeliner
```

1. (Optional) 비밀번호 변경

내장된유저 Elastic 의 비밀번호를 변경하고 싶다면

.env 파일의

ELASTIC_PASSWORD 값을 변경

그 외에 KIBANA_SYSTEM_PASSWORD 등도 같이 변경해준다.

```bash
ELASTIC_VERSION=8.1.2

## Passwords for stack users
#

# User 'elastic' (built-in)
#
# Superuser role, full access to cluster management and data indices.
# https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
ELASTIC_PASSWORD='changeme'

# User 'logstash_internal' (custom)
#
# The user Logstash uses to connect and send data to Elasticsearch.
# https://www.elastic.co/guide/en/logstash/current/ls-security.html
LOGSTASH_INTERNAL_PASSWORD='changeme'

# User 'kibana_system' (built-in)
#
# The user Kibana uses to connect and communicate with Elasticsearch.
# https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
KIBANA_SYSTEM_PASSWORD='changeme'
```

1. Kafka 실행

[https://github.com/coin-pipeliner/Coin-Producer/tree/dev](https://github.com/coin-pipeliner/Coin-Producer/tree/dev) 참고

1. ELK 빌드

```bash
docker compose build
```

1. ELK 실행

```bash
docker compose up # -d
```

1. Elastic Search 에 logstash 와 kibana 가 접근할 수 있는 권한 부여

- Elastic Search 가 정상적으로 bootup 되고 나서 바로 권한을 주어야 logstash 나 kibana 에서 접속할 때 오류가 없다
- logstash 권한 부여 (terminal 에 입력)
- 비밀번호를 바꿨다면 changeme 를 변경해주고 email 도 본인 email

```bash
curl -k -u elastic:changeme -X POST "http://localhost:9200/_security/user/logstash_internal?pretty" -H 'Content-Type: application/json' -d'
{
  "password" : "changeme",
  "roles" : [ "superuser" ],
  "full_name" : "logstash internal",
  "email" : "이메일@yonsei.ac.kr"
}
'
```

- kibana 권한 부여

```bash
curl -k -u elastic:changeme -X POST "http://localhost:9200/_security/user/kibina_system?pretty" -H 'Content-Type: application/json' -d'
{
  "password" : "changeme",
  "roles" : [ "superuser" ],
  "full_name" : "kibana system",
  "email" : "이메일@yonsei.ac.kr"
}
'
```

1. kibana 웹 접속 - localhost:5601

user: elastic

password: changeme (초기)

1. coin-producer 로 데이터 적재

- 현재는 btc-krw-날짜 로 index가 만들어짐
- 웹 콘솔 좌측 메뉴바 맨 하단 stack management 에서 저장된 인덱스 확인

![스크린샷 2022-05-15 오후 9.48.39.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/2fa29466-1187-48f8-9228-93a327ba9f4d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.48.39.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125611Z&X-Amz-Expires=86400&X-Amz-Signature=b32035cfd1030c7f4b43b9f8431d35400161ded92f7e749d3aa0b230d48a9a05&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.48.39.png%22&x-id=GetObject)

![스크린샷 2022-05-15 오후 9.49.08.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/b5bf6a7a-5c1c-46b5-9763-eac7b8f3a915/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.49.08.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125646Z&X-Amz-Expires=86400&X-Amz-Signature=916f5b78af052cd9476f09d19c7b108870976d9b7ec2b0026a2b5f7d3df076a8&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.49.08.png%22&x-id=GetObject)

데이터 자체를 확인하기 위해서 Discover 탭 클릭

data view 를 만들어야 볼 수 있는데 index 이름 그대로 적어주면 된다

![스크린샷 2022-05-15 오후 9.49.35.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/935a0514-3750-4405-814f-883cb01aca11/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.49.35.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125702Z&X-Amz-Expires=86400&X-Amz-Signature=f66cd0d170ac40d3f04a42b96e5a0e6ea25dffff4f768688f1a9ea2f41b0932d&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.49.35.png%22&x-id=GetObject)

우측 상단에 시간을 적절히 설정하면 데이터를 확인할 수 있다.

![스크린샷 2022-05-15 오후 9.50.05.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/7d5e73ae-66be-40e9-a567-f556a27808c7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.50.05.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125737Z&X-Amz-Expires=86400&X-Amz-Signature=18d0dba93858010cf911994e4933779f8238a6adee74fbf715f2c05351b307cf&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.50.05.png%22&x-id=GetObject)

시각화를 위해서 Visualize Library 클릭

![스크린샷 2022-05-15 오후 9.50.33.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/330ecb57-a954-4998-9b0d-2a9256a1e366/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.50.33.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125752Z&X-Amz-Expires=86400&X-Amz-Signature=8ed47a98df432c88243e7463b77e413d7fa766fc60dd97e07856d2f02eab2c75&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.50.33.png%22&x-id=GetObject)

Custom visualiztion 으로 visualiztion 을 생성한다 (candle 차트 생성을 위해)

![스크린샷 2022-05-15 오후 9.51.03.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/05b57bf2-be4e-496f-ae2f-d18db86e7b17/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.51.03.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125807Z&X-Amz-Expires=86400&X-Amz-Signature=2e046f4deaa914774caa432b939bbf909aacc335e291b4d9b3ad89596d133128&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.51.03.png%22&x-id=GetObject)

우측 에디터에 vega 코드 입력 (vega 는 그래프를 위한 라이브러리)

![스크린샷 2022-05-15 오후 9.51.56.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f7345b7d-a67c-4aba-89a6-cb21ba4774f2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-05-15_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.51.56.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20220515%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20220515T125820Z&X-Amz-Expires=86400&X-Amz-Signature=acd06b2ce19a1509ef8bff733eb54445840283ad9b86d423b69d6d011678779c&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA%25202022-05-15%2520%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE%25209.51.56.png%22&x-id=GetObject)

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v4.json",
  "width": 800,
  "autosize": {
    "type": "fit",
    "contains": "padding"
  },
  "description": "A candlestick chart inspired by an example in Protovis (http://mbostock.github.io/protovis/ex/candlestick.html)",
  "data": {
    "url": {
      "%context%": true,
      "%timefield%": "@timestamp",
      "index": "_all",
      "body": {
        "aggs": {
          "time_buckets": {
            "date_histogram": {
              "field": "@timestamp",
              "fixed_interval": "1s",
              "time_zone": "Asia/Seoul",
              "min_doc_count": 0
            },
            "aggs": {
              "high": {
                "max": {
                  "field": "high_price"
                }
              },
              "open": {
                "max": {
                  "field": "opening_price"
                }
              },
              "close": {
                "min": {
                  "field": "trade_price"
                }
              },
              "low": {
                "max": {
                  "field": "low_price"
                }
              }
            }
          }
        },
        "size": 0
      }
    },
    "format": { "property": "aggregations.time_buckets.buckets" }
  },

  "encoding": {
    "x": {
      "field": "key",
      "type": "temporal",
      "title": "Date"
    },
    "y": {
      "type": "quantitative",
      "scale": { "zero": false },
      "axis": { "title": "Price" }
    },
    "color": {
      "condition": {
        "test": "datum.open < datum.close",
        "value": "#06982d"
      },
      "value": "#ae1325"
    }
  },
  "layer": [
    {
      "mark": "rule",
      "encoding": {
        "y": { "field": "high.value" },
        "y2": { "field": "low.value" }
      }
    },
    {
      "mark": "bar",
      "encoding": {
        "y": { "field": "close.value" },
        "y2": { "field": "open.value" }
      }
    }
  ]
}
```

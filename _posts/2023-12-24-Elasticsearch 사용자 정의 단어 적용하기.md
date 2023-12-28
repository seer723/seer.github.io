---
title: "Elasticsearch 사용자 정의 단어 적용하기"
last_modified_at: 2023-12-24
categories:
  - ElasticSearch
tags:
  - ElasticSearch
  - Tokenizing
  - UserDictionaryRules
---

> 엘라스틱 서치에서 지역명을 검색할 때 특정지역이 검색되지 않는 문제가 있었어요.  
> 이번 글에서는 검색어 토크나이징이 잘못되었을 때 이를 해결하는 방법에 대해 알아볼게요.

##### 1. 문제의 시작, 토크나이징

엘라스틱 서치에서 "방이" 라는 지역단어로 검색했을 때 아무런 결과도 나오지 않는 문제가 있었어요.  

문제의 원인은 바로 토크나이징 처리였어요.  

현재 사용하고 있는 토크나이저는 한글 형태소 분석에서 많이 사용되는 nori_tokenizer 인데요.

nori_tokenizer로 시스템에서 사용하는 지역 단어를 토크나이징하면 아래와 같은 결과가 나와요.

```json
GET /_analyze
{
  "tokenizer" : {
    "type": "nori_tokenizer"
  },
  "text": "잠실/방이/석촌"
}
```

```json
{
  "tokens": [
    {
      "token": "잠실",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 0
    },
    {
      "token": "방",
      "start_offset": 3,
      "end_offset": 4,
      "type": "word",
      "position": 1
    },
    {
      "token": "이",
      "start_offset": 4,
      "end_offset": 5,
      "type": "word",
      "position": 2
    },
    {
      "token": "석촌",
      "start_offset": 6,
      "end_offset": 8,
      "type": "word",
      "position": 3
    }
  ]
}
```

"방이" 라는 단어가 "방"과 "이"로 나뉘어지는 것을 확인할 수 있어요.

바로 이 부분이 문제였어요.

InvertedIndex 에 토큰이 저장될 때 각각 "방"과 "이"로 한 글자씩 토큰화되어 저장되기 때문에 "방이" 로 검색했을 때에는 아무런 결과가 나오지 않은 것이었죠.

그래서 어떻게 이 문제를 해결할 수 있는지 찾아봤어요.

##### 2. 해결의 가능성, 사용자 정의 단어

[엘라스틱 서치 가이드북 사이트](https://esbook.kimjmin.net/06-text-analysis/6.7-stemming/6.7.2-nori){:target="_blank"} 에서 user_dictionary_rules 라는 옵션을 발견했어요.

user_dictionary_rules 는 토크나이징시에 설정된 사용자 단어를 최우선으로 적용하는 옵션인데요.

"방이" 처럼 토크나이징으로 쪼개지지 않기를 원하는 단어를 등록해두면 토크나이저가 이를 고려해서 원단어를 유지해주는 방식이예요.

그러면 한 번 user_dictionary_rules 를 적용해보도록 할게요.

```json
GET /_analyze
{
  "tokenizer": {
    "type": "nori_tokenizer",
    "user_dictionary_rules": [
      "방이"
    ]
  },
  "text": "잠실/방이/석촌"
}
```

```json
{
  "tokens": [
    {
      "token": "잠실",
      "start_offset": 0,
      "end_offset": 2,
      "type": "word",
      "position": 0
    },
    {
      "token": "방이",
      "start_offset": 3,
      "end_offset": 5,
      "type": "word",
      "position": 1
    },
    {
      "token": "석촌",
      "start_offset": 6,
      "end_offset": 8,
      "type": "word",
      "position": 2
    }
  ]
}
```

"방이" 단어가 토크나이징으로 쪼개지지 않고 원단어가 그대로 토큰이 된 것을 확인할 수 있어요.

이제 user_dictionary_rules 가 적용된 인덱스를 생성하고 엘라스틱 서치에서 검색에 되는지 확인해볼게요.

##### 3. 테스트의 끝, 실제 적용

```json
PUT location_with_rules
{
  "settings": {
    "analysis": {
      "tokenizer": {
        "location_tokenizer": {
          "type": "nori_tokenizer",
          "decompound_mode": "mixed",
          "user_dictionary_rules": [
            "방이"
          ]
        }
      },
      "analyzer": {
        "location_analyzer": {
          "tokenizer": "location_tokenizer",
          "filter": [
            "lowercase",
            "stop",
            "trim"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "district": {
        "type": "text",
        "analyzer": "location_analyzer",
        "search_analyzer": "location_analyzer"
      }
    }
  }
}
```

인덱스를 추가했으니 지역 데이터를 넣어볼게요.

```json
PUT location_with_rules/_doc/1
{
  "district": "잠실/방이/석촌"
}
```

이제 "방이" 라는 단어로 검색을 해볼게요.

```json
GET location_with_rules/_search
{
  "query": {
    "match": {
      "district": "방이"
    }
  }
}
```

그러면 아래와 같이 방이가 포함된 지역 데이터가 조회되는 것을 확인해볼 수 있어요.

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.6931471,
    "hits": [
      {
        "_index": "location_with_rules",
        "_id": "1",
        "_score": 0.6931471,
        "_source": {
          "district": "잠실/방이/석촌"
        }
      }
    ]
  }
}
```

user_dictionary_rules 를 적용해서 토크나이징이 잘 되는 것을 확인하였어요.

더 자세한 정보는 [엘라스틱 서치 가이드북 사이트](https://esbook.kimjmin.net/06-text-analysis/6.7-stemming/6.7.2-nori){:target="_blank"}에서 확인하실 수 있으니 참고해보시면 될 것 같아요.


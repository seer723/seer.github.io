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

"방이" 라는 지역단어로 검색했을 때 아무런 결과도 나오지 않는 문제가 있었어요.  

문제의 원인으로 가장 의심이 되는 부분은 토크나이징 처리였어요.  

현재 사용하고 있는 토크나이저는 한글 형태소 분석에서 많이 사용되는 nori_tokenizer 인데요.

nori_tokenizer로 "방이" 라는 단어를 토크나이징 해봤어요.

```json
GET /_analyze
{
    "tokenizer": "nori",
    "text": "방이"
}
```

그랬더니 "방이" 라는 단어가 "방"과 "이"로 나뉘어지는 것이 아니겠어요.

```json
{
    "tokens" : [
        {
            "token": "방",
            "start_offset": 0,
            "end_offset": 1,
            "type": "word",
            "position": 0
        },
        {
            "token": "이",
            "start_offset": 1,
            "end_offset": 2,
            "type": "word",
            "position": 1
        }
    ]
}
```

바로 이 부분이 문제였어요.

InvertedIndex 에 토큰이 저장될 때 각각 "방"과 "이"로 한 글자씩 토큰화되어 저장되기 때문에 "방이" 로 검색했을 때에는 아무런 결과가 나오지 않은 것이었죠.

문제의 원인을 찾은 저는 어떻게 문제를 해결할 수 있는지 열심히 구글링을 시작했어요.

##### 2. 해결의 가능성, 사용자 정의 단어

user_dictionary_rules 에서 사용자 단어를 정의해두면 토크나이징 적용단계에서 최우선으로 고려된다.

https://esbook.kimjmin.net/06-text-analysis/6.7-stemming/6.7.2-nori

동해물과 백두산이 예제

##### 3. 테스트의 끝, 실제 적용


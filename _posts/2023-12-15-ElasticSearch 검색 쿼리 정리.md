---
title: "ElasticSearch 검색 쿼리 정리"
last_modified_at: 2023-12-15
categories:
  - ElasticSearch
tags:
  - ElasticSearch
  - Kibana
  - DevTools
---

> 최근 회사에서 엘라스틱 서치를 사용하게 되면서 공부한 내용을 정리해보았어요.  
> 기초부터 다지는 ElasticSearch 운영 노하우 책을 보고 공부했어요.
> 실습은 Kinaba 에서 DevTools 기능을 이용해서 진행했어요.

## 1. 엘라스틱 서치 버전을 비롯한 정보 확인
```json
GET /
```

## 2. 인덱스 목록 확인
```json
GET _cat/indices
```
    
## 3. 특정 인덱스 확인
```json
GET _cat/indices/[인덱스명]
```

## 4. 특정 문서 조회
```json
GET [인덱스명]/_doc/[id]
```
    
## 5. 인덱스내 문서 개수 조회
```json
GET [인덱스명]/_count
```
    
## 6. 인덱스내 모든 문서 조회
```json
GET [인덱스명]/_search
{
  "query" : {
    "match_all": {}
  }
}
```
    
## 7. 분석기 동작 확인
```json
GET /_analyze
{
  "analyzer" : "standard",
  "text": "아버지가방에들어가셨다"
}
```
```json
GET /_analyze
{
  "analyzer" : "nori",
  "text": "아버지가방에들어가셨다"
}
```
```json
GET /_analyze
{
  "tokenizer" : "nori",
  "text": "Quick Brown Foxes!"
}
```

## 8. 데이터 벌크 추가
```json
POST [인덱스명:serach_sample]/_bulk
{"index":{"_id":"1"}}
{ "title": "ElasticSearch Training Book", "publisher": "insight", "ISBN": "9788966264849", "release_date": "2020/09/30", "description": "ElasticSearch is cool open source search engine" }
{"index":{"_id":"2"}}
{ "title" : "Kubernetes: Up and Running", "publisher": "O'Reilly Media, Inc.", "ISBN": "9781491935675", "release_date": "2017/09/03", "description" : "What separates the traditional enterprise from the likes of Amazon, Netflix, and Etsy? Those companies have refined the art of cloud native development to maintain their competitive edge and stay well ahead of the competition. This practical guide shows Java/JVM developers how to build better software, faster, using Spring Boot, Spring Cloud, and Cloud Foundry." }
{"index":{"_id":"3"}}
{ "title" : "Cloud Native Java", "publisher": "O'Reilly Media, Inc.", "ISBN": "9781449374648", "release_date": "2017/08/04", "description" : "What separates the traditional enterprise from the likes of Amazon, Netflix, and Etsy? Those companies have refined the art of cloud native development to maintain their competitive edge and stay well ahead of the competition. This practical guide shows Java/JVM developers how to build better software, faster, using Spring Boot, Spring Cloud, and Cloud Foundry." }
{"index":{"_id":"4"}}
{ "title" : "Learning Chef", "publisher": "O'Reilly Media, Inc.", "ISBN": "9781491944936", "release_date": "2014/11/08", "description" : "Get a hands-on introduction to the Chef, the configuration management tool for solving operations issues in enterprises large and small. Ideal for developers and sysadmins new to configuration management, this guide shows you to automate the packaging and delivery of applications in your infrastructure. You’ll be able to build (or rebuild) your infrastructure’s application stack in minutes or hours, rather than days or weeks." }
{"index":{"_id":"5"}}
{ "title" : "Elasticsearch Indexing", "publisher": "Packt Publishing", "ISBN": "9781783987023", "release_date": "2015/12/22", "description" : "Improve search experiences with ElasticSearch’s powerful indexing functionality – learn how with this practical ElasticSearch tutorial, packed with tips!" }
{"index":{"_id":"6"}}
{ "title" : "Hadoop: The Definitive Guide, 4th Edition", "publisher": "O'Reilly Media, Inc.", "ISBN": "9781491901632", "release_date": "2015/04/14", "description" : "Get ready to unlock the power of your data. With the fourth edition of this comprehensive guide, you’ll learn how to build and maintain reliable, scalable, distributed systems with Apache Hadoop. This book is ideal for programmers looking to analyze datasets of any size, and for administrators who want to set up and run Hadoop clusters." }
{"index":{"_id":"7"}}
{ "title": "Getting Started with Impala", "publisher": "O'Reilly Media, Inc.", "ISBN": "9781491905777", "release_date": "2014/09/14", "description" : "Learn how to write, tune, and port SQL queries and other statements for a Big Data environment, using Impala—the massively parallel processing SQL query engine for Apache Hadoop. The best practices in this practical guide help you design database schemas that not only interoperate with other Hadoop components, and are convenient for administers to manage and monitor, but also accommodate future expansion in data size and evolution of software capabilities. Ideal for database developers and business analysts, the latest revision covers analytics functions, complex types, incremental statistics, subqueries, and submission to the Apache incubator." }
{"index":{"_id":"8"}}
{ "title": "NGINX High Performance", "publisher": "Packt Publishing", "ISBN": "9781785281839", "release_date": "2015/07/29", "description": "Optimize NGINX for high-performance, scalable web applications" }
{"index":{"_id":"9"}}
{ "title": "Mastering NGINX - Second Edition", "publisher": "Packt Publishing", "ISBN": "9781782173311", "release_date": "2016/07/28", "description": "An in-depth guide to configuring NGINX for your everyday server needs" }
{"index":{"_id":"10"}}
{ "title" : "Linux Kernel Development, Third Edition", "publisher": "Addison-Wesley Professional", "ISBN": "9780672329463", "release_date": "2010/06/09", "description" : "Linux Kernel Development details the design and implementation of the Linux kernel, presenting the content in a manner that is beneficial to those writing and developing kernel code, as well as to programmers seeking to better understand the operating system and become more efficient and productive in their coding." }
{"index":{"_id":"11"}}
{ "title" : "Linux Kernel Development, Second Edition", "publisher": "Sams", "ISBN": "9780672327209", "release_date": "2005/01/01", "description" : "The Linux kernel is one of the most important and far-reaching open-source projects. That is why Novell Press is excited to bring you the second edition of Linux Kernel Development, Robert Love's widely acclaimed insider's look at the Linux kernel. This authoritative, practical guide helps developers better understand the Linux kernel through updated coverage of all the major subsystems as well as new features associated with the Linux 2.6 kernel. You'll be able to take an in-depth look at Linux kernel from both a theoretical and an applied perspective as you cover a wide range of topics, including algorithms, system call interface, paging strategies and kernel synchronization. Get the top information right from the source in Linux Kernel Development." }
``` 

## 9. 검색하기
```json
GET search_sample/_search
{
  "query": {
    "match": {
      "description": "nginx guide"
    }
  }
}
```

## 10. 검색 결과 정렬하기
```json
GET search_sample/_search
{
  "sort": [
    {"ISBN.keyword": "desc"}
  ],
  "query": {
    "match": {
      "title": "nginx"
    }
  }
}
```

Sort는 Text 필드가 아닌 Keyword나 Integer와 같은 Not Analyzed가 기본인 필드를 기준으로 해야되요.

## 11. 원하는 필드만 출력하기
```json
GET search_sample/_search
{
  "_source": ["title", "description"],
  "query": {
    "match": {
      "title": "nginx"
    }
  }
}
```

## 12. 검색된 단어 강조하기
```json
GET search_sample/_search
{
  "query": {
    "term": {
      "title": "nginx"
    }
  },
  "highlight": {
    "fields": {
      "title": {}
    }
  }
}
```

## 13. match_phrase
```json
GET search_sample/_search
{
  "query": {
    "match": {
      "description": "Kernel Linux"
    }
  }
}
```
match 쿼리는 analyzer를 통해 분석한 토큰을 기준으로 검색하고 검색어 토큰 순서는 고려하지 않아요.  
nginx guide 검색어나 guide nginx 검색어는 동일한 검색 결과를 보여줘요.
```json
GET search_sample/_search
{
  "query": {
    "match_phrase": {
      "description": "Kernel Linux"
    }
  }
}
```
```json
GET search_sample/_search
{
  "query": {
    "match_phrase": {
      "description": "Linux Kernel"
    }
  }
}
```
match_phrase는 검색토큰 순서를 고려해요.  
match_phrase도 match와 마찬가지로 검색어 토큰을 만들지만 검색어 토큰이 정확한 순서로 포함된 문서를 찾기 때문에 일부 토큰만 포함된 문서는 보여주지 않아요.
    
## 14. Query Context 와 Filter Context
1. Query Context
  - Full Text Search 를 의미하며 검색어가 문서와 얼마나 매칭되는가를 표현하는 score 값을 가져요.
  - match, match_phrase, multi_match, query_string 이 있어요.
2. Filter Context
  - 검색어가 문서에 존재하는지 여부를 Yes나 No 형태의 검색 결과로 보여줘요. Query Context 와는 다르게 score 값을 가지지 않아요.
  - term, terms, range, wildcard 가 있어요.
3. Query Context 와 Filter Context 의 가장 큰 차이점은 검색어를 Analyze 하느냐의 여부예요.
  - Filter Context 는 Analyze 를 하지 않기 때문에 대소문자를 구분해요.

```json
GET search_sample/_search
{
  "query": {
    "term": {
      "title": "Linux"
    }
  }
}
```
```json
GET search_sample/_search
{
  "query": {
    "term": {
      "title": "linux"
    }
  }
}
```

## 15. Range 쿼리 사용
```json
GET search_sample/_search
{
  "query": {
    "range": {
      "release_date": {
        "gte": "2015/01/01",
        "lte": "2015/07/23"
      }
    }
  }
}
```
wildcard query는 모든 inverted index를 하나하나 확인하기 때문에 검색 속도가 매우 느리기 때문에 사용을 지양하는 것이 좋아요.
    
## 16. Bool Query
1. Bool Query 항목들
  - must : 항목 내 쿼리에 일치하는 문서를 검색, 스코어링 O
  - filter : 항목 내 쿼리에 일치하는 문서를 검색, 캐싱 O
  - should : 항목 내 쿼리에 일치하는 문서를 검색, 스코어링 O
  - must_not : 항목 내 쿼리에 일치하지 않는 문서를 검색, 캐싱 O
2. 사용
  - must, should 는 스코어링을 하기 때문에 Query Context 에서 실행되요.
  - filter, must_not 은 Filter Context 에서 실행되요.
  - Bool Query는 Query/Filter Context 를 혼합해서 사용할 수 있어요.

```json
GET search_sample/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "nginx"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "release_date": {
              "gte": "2016/01/01",
              "lte": "2017/12/31"
            }
          }
        }
      ]
    }
  }
}
```
filter context 에 포함되는 쿼리들은 filter 절에 넣는 것이 좋아요.  
must 절에 range를 넣어도 되지만 filter 절을 쓰는게 속도가 더 빨라요.  
왜냐하면 must 절에 포함된 Filter Context 들은 score 를 계산하는 데 활용되기 때문에 불필요한 연산들이 들어가기 때문이예요.
    
## 17. must_not
```json
GET search_sample/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "nginx"
          }
        },
        {
          "range": {
            "release_date": {
              "gte": "2014/01/01",
              "lte": "2017/12/31"
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "description": "performance"
          }
        }
      ]
    }
  }
}
```
description 필드에 performance 라는 문자열이 포함되지 않는 문서를 검색하는 쿼리예요.
        
## 18. should
```json
GET search_sample/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "nginx"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "release_date": {
              "gte": "2014/01/01",
              "lte": "2017/12/31"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "title": "performance"
          }
        },
        {
          "match": {
            "description": "scalable web"
          }
        }
      ]
    }
  }
}
```
should 절 내에 일치하는 부분이 있는 문서는 스코어가 올라가게 되요.
        
## 19. minimum_should_match 옵션
```json
GET search_sample/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": "nginx"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "release_date": {
              "gte": "2014/01/01",
              "lte": "2017/12/31"
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "title": "performance"
          }
        },
        {
          "match": {
            "description": "scalable web"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}
```
minimum_should_match 옵션의 값(1)은 should 절 쿼리중 적어도 하나는 일치해야 결과를 리턴한다는 의미예요.        

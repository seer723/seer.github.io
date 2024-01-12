---
title: "2024-01-12-SpringBoot @Async 사용법"
last_modified_at: 2024-01-12
categories:
  - Java
tags:
  - Java
  - SpringBoot
  - @Async
  - 비동기
---

> 기존에 사용하고 있던 학교 검색 API 의 성능을 개선해야하는 일이 생겼는데요.  
> Elasticsearch 를 직접적으로 수정하지는 않고 @Async 를 써서 SprintBoot 상에 구현된 로직을 개선하여 성능을 높이는 작업을 진행했어요.

##### 1. @Async 란?

AsyncApplication.java
```java
package org.example.async;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync
public class AsyncApplication {

    public static void main(String[] args) {
        SpringApplication.run(AsyncApplication.class, args);
    }

}
```

AsyncController.java
```java
package org.example.async.controller;

import lombok.RequiredArgsConstructor;
import org.example.async.service.AsyncService;
import org.example.async.service.SyncService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class AsyncController {
    private static final int COUNT = 1000;

    private final SyncService syncService;
    private final AsyncService asyncService;

    @GetMapping("/sync")
    public String goSync() {
        long start = System.currentTimeMillis();
        System.out.println("### sync start ###");
        System.out.println("start: " + start);
        for (int i = 0; i < COUNT; i++) {
            syncService.sync(i + 1);
        }
        long end = System.currentTimeMillis();
        System.out.println("end: " + end);
        System.out.println("elapsed millis: " + (end - start) + " millis");
        System.out.println("### sync end ###");
        return "sync";
    }

    @GetMapping("/async")
    public String goAsync() throws InterruptedException {
        long start = System.currentTimeMillis();
        System.out.println("### async start ###");
        System.out.println("start: " + start);
        for (int i = 0; i < COUNT; i++) {
            asyncService.async(i + 1);
        }
        long end = System.currentTimeMillis();
        System.out.println("end: " + end);
        System.out.println("elapsed millis: " + (end - start) + " millis");
        System.out.println("### async end ###");
        return "async";
    }

}
```

AsyncService.java
```java
package org.example.async.service;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class AsyncService {
    @Async
    public void async(int index) throws InterruptedException {
        Thread.sleep(100);
        System.out.println("async - " + index);
    }
}
```

SyncService.java
```java
package org.example.async.service;

import org.springframework.stereotype.Service;

@Service
public class SyncService {
    public void sync(int index) {
        System.out.println("sync - " + index);
    }
}
```

##### 2. 실제 코드 @Async 적용 및 유의사항

> ### 기존 동기식 코드

```java
public ListResponseDto<SearchDto.Event> searchEvents(String query, String type, String city, Set<String> district, String sort, int page, int size)
        throws IOException {
    addSearchTerm(SearchDto.SearchTerm.create(query));

    List<Query> queries = new ArrayList<>();
    queries.add(multiMatchQuery(query, ElasticSearch.FIELDS_NAME, ElasticSearch.FIELDS_HOSPITAL_NAME, ElasticSearch.FIELDS_CITY,
            ElasticSearch.FIELDS_DISTRICT));
    if (isExist(city)) {
        queries.add(matchQuery(ElasticSearch.FIELDS_CITY, city));
    }
    if (isExist(district)) {
        queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
    }
    queries.add(term(ElasticSearch.FIELDS_TYPE, type));
    queries.add(term(ElasticSearch.FIELDS_STATE, "ACTIVE"));

    SearchRequest searchRequest = SearchRequest.of(s -> s
            .index(ElasticSearch.INDEX_EVENT)
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(st -> st.field(f -> f.field("_score").order(SortOrder.Desc)))
            .sort(st -> st.field(f -> f.field(SortType.getValue(sort)).order(SortType.getOrder(sort))))
    );

    SearchResponse<SearchDto.Event> searchResponse = elasticsearchClient.search(searchRequest, SearchDto.Event.class);
    List<SearchDto.Event> list = searchResponse.hits().hits().stream().map(Hit::source).filter(Objects::nonNull)
            .peek(i -> i.setThumbnail((cloudFrontService.generateSignedUrl(i.getThumbnail())))).toList();

    return ListResponseDto.of(list, total(searchResponse));
}

public void addSearchTerm(SearchDto.SearchTerm searchTerm) throws IOException {
    elasticsearchClient.index(i -> i
            .index(ElasticSearch.INDEX_QUERY)
            .id(searchTerm.getDateUpdated().toString())
            .document(searchTerm)
    );
}
```

> ### @Async 를 적용한 비동기식 코드

```java
public ListResponseDto<SearchDto.Event> searchEvents(String query, String type, String city, Set<String> district, String sort, int page, int size) throws IOException {
    List<Query> queries = new ArrayList<>();
    queries.add(multiMatchQuery(query, ElasticSearch.FIELDS_NAME, ElasticSearch.FIELDS_HOSPITAL_NAME, ElasticSearch.FIELDS_CITY,
            ElasticSearch.FIELDS_DISTRICT));
    if (isExist(city)) {
        queries.add(matchQuery(ElasticSearch.FIELDS_CITY, city));
    }
    if (isExist(district)) {
        queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
    }
    queries.add(term(ElasticSearch.FIELDS_TYPE, type));
    queries.add(term(ElasticSearch.FIELDS_STATE, "ACTIVE"));

    SearchRequest searchRequest = SearchRequest.of(s -> s
            .index(ElasticSearch.INDEX_EVENT)
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(st -> st.field(f -> f.field("_score").order(SortOrder.Desc)))
            .sort(st -> st.field(f -> f.field(SortType.getValue(sort)).order(SortType.getOrder(sort))))
    );

    SearchResponse<SearchDto.Event> searchResponse = elasticsearchClient.search(searchRequest, SearchDto.Event.class);
    List<SearchDto.Event> list = searchResponse.hits().hits().stream().map(Hit::source).filter(Objects::nonNull)
            .peek(i -> i.setThumbnail((cloudFrontService.generateSignedUrl(i.getThumbnail())))).toList();

    return ListResponseDto.of(list, total(searchResponse));
}

@Async
public void addSearchTerm(SearchDto.SearchTerm searchTerm) throws IOException {
    System.out.println("### addSearchTerm Start###");
    elasticsearchClient.index(i -> i
            .index(ElasticSearch.INDEX_QUERY)
            .id(searchTerm.getDateUpdated().toString())
            .document(searchTerm)
    );
    System.out.println("### addSearchTerm End###");
}
```

@Async 를 붙인 서비스를 다른 서비스 내에서 호출하면 동작하지 않음

@Async 를 붙인 서비스를 컨트롤러로 빼줘야 동작함

##### 3. 성능 개선 확인

포스트맨 응답속도

1. 동기
   785ms, 310ms, 212ms, 247ms, 240ms, 233ms, 115ms, 229ms, 266ms, 261ms > 289.8ms

2. 비동기
   421ms, 108ms, 120ms, 94ms, 96ms, 16ms, 95ms, 112ms, 1112ms, 105ms > 227.9 ms

오늘도 미션 클리어! 👍
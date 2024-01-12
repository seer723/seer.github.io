---
title: "2024-01-12-SpringBoot @Async 사용법"
last_modified_at: 2024-01-09
categories:
  - Java
tags:
  - Java
  - SpringBoot
  - @Async
---

> 모던 자바 인 액션(한빛미디어) 책을 보면서 동작 파라미터화에 대해 공부한 내용을 정리했어요.  
> 농장재고목록 애플리케이션 예제 코드를 개선하면서 자바 8의 장점을 느껴볼 수 있는 내용이예요.  

##### 1. 농장재고목록 애플리케이션

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

@Async 를 붙인 서비스를 다른 서비스 내에서 호출하면 동작하지 않음

@Async 를 붙인 서비스를 컨트롤러로 빼줘야 동작함


```java
[동기 - 컨트롤러로 분리]

        ### addSearchTerm Start###
        ### addSearchTerm End###
        ### searchHospitals start ###
start: 1705020297771
        ### searchHospitals set queries ###
        ### searchHospitals set sorts ###
        ### searchHospitals set request ###
        ### searchHospitals send request ###
end: 1705020297889
elapsed millis: 118 millis
### searchHospitals end ###

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

```java
[비동기 - 컨트롤러로 분리]

        ### searchHospitals start ###
start: 1705020332746
        ### searchHospitals set queries ###
        ### addSearchTerm Start###
        ### searchHospitals set sorts ###
        ### searchHospitals set request ###
        ### searchHospitals send request ###
end: 1705020332933
elapsed millis: 187 millis
### searchHospitals end ###
        ### addSearchTerm End###

public ListResponseDto<SearchDto.Event> searchEvents(String query, String type, String city, Set<String> district, String sort, int page, int size)
        throws IOException {
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


오늘도 미션 클리어! 👍
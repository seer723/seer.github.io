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
> 개선 방안에 대해 고민을 해보면서 Elasticsearch 를 직접적으로 수정하기는 아직 어려울 것 같다는 판단이 들었어요.
> 그래서 우선은 SprintBoot 상에 구현된 로직에 @Async 를 적용하여 성능을 높여보자는 방향을 잡고 작업을 진행했어요.

##### 1. @Async 란?

@Async 는 메서드를 비동기식으로 실행하도록 지정하는 어노테이션인데요.

저는 아직 @Async 를 제대로 써보지를 못해서 우선 실제코드에 적용하기 전에 간단한 예제를 만들어보았어요.

프로젝트 코드는 [spring-boot-async-example](https://github.com/seerlog/spring-boot-async-example){:target="_blank"}에서 확인해보실 수 있어요.

> #### AsyncApplication.java
> ```java
> package org.example.async;
>
> import org.springframework.boot.SpringApplication;
> import org.springframework.boot.autoconfigure.SpringBootApplication;
> import org.springframework.scheduling.annotation.EnableAsync;
>
> @SpringBootApplication
> @EnableAsync
> public class AsyncApplication {
> 
>     public static void main(String[] args) {
>         SpringApplication.run(AsyncApplication.class, args);
>     }
> 
> }
> ```

> #### AsyncController.java
> ```java
> package org.example.async.controller;
> 
> import lombok.RequiredArgsConstructor;
> import org.example.async.service.AsyncService;
> import org.example.async.service.SyncService;
> import org.springframework.web.bind.annotation.GetMapping;
> import org.springframework.web.bind.annotation.RestController;
> 
> @RestController
> @RequiredArgsConstructor
> public class AsyncController {
>     private static final int COUNT = 1000;
> 
>     private final SyncService syncService;
>     private final AsyncService asyncService;
> 
>     @GetMapping("/sync")
>     public String goSync() {
>         long start = System.currentTimeMillis();
>         System.out.println("### sync start ###");
>         System.out.println("start: " + start);
>         for (int i = 0; i < COUNT; i++) {
>             syncService.sync(i + 1);
>         }
>         long end = System.currentTimeMillis();
>         System.out.println("end: " + end);
>         System.out.println("elapsed millis: " + (end - start) + " millis");
>         System.out.println("### sync end ###");
>         return "sync";
>     }
> 
>     @GetMapping("/async")
>     public String goAsync() throws InterruptedException {
>         long start = System.currentTimeMillis();
>         System.out.println("### async start ###");
>         System.out.println("start: " + start);
>         for (int i = 0; i < COUNT; i++) {
>             asyncService.async(i + 1);
>         }
>         long end = System.currentTimeMillis();
>         System.out.println("end: " + end);
>         System.out.println("elapsed millis: " + (end - start) + " millis");
>         System.out.println("### async end ###");
>         return "async";
>     }
> 
> }
> ```

> #### AsyncService.java
> ```java
> package org.example.async.service;
> 
> import org.springframework.scheduling.annotation.Async;
> import org.springframework.stereotype.Service;
> 
> @Service
> public class AsyncService {
>     @Async
>     public void async(int index) throws InterruptedException {
>         Thread.sleep(100);
>         System.out.println("async - " + index);
>     }
> }
> ```

> #### SyncService.java
> ```java
> package org.example.async.service;
> 
> import org.springframework.stereotype.Service;
> 
> @Service
> public class SyncService {
>     public void sync(int index) {
>         System.out.println("sync - " + index);
>     }
> }
> ```

sync 요청과 async 요청을 만들고 각각 대응되는 서비스를 구현한 예제예요.

async 서비스의 경우에는 Thread.sleep(100) 을 사용해서 0.1초의 지연을 발생시켰어요.

이제 스프링 부트 프로젝트를 실행하고 http://localhost:8080/sync 요청을 호출해보면 바로 응답이 오는 걸 확인할 수 있어요.

다음으로 http://localhost:8080/async 요청을 호출해보면 응답은 바로 오지만 로그를 보면 여전히 요청 처리중인 모습을 확인할 수 있습니다.




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
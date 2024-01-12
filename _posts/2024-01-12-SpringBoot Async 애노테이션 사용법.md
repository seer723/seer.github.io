---
title: "SpringBoot Async 애노테이션 사용법"
last_modified_at: 2024-01-12
categories:
  - Java
tags:
  - Java
  - SpringBoot
  - Async
  - 비동기
---

> 기존에 사용하고 있던 학교 검색 API 의 성능을 개선해야하는 일이 생겼는데요.  
> 개선 방안에 대해 고민을 해보면서 Elasticsearch 를 직접적으로 수정하기는 아직 어려울 것 같다는 판단이 들었어요.
> 그래서 우선은 SprintBoot 상에 구현된 로직에 @Async 를 적용하여 성능을 높여보자는 방향을 잡고 작업을 진행했어요.

##### 1. @Async란?

@Async는 메서드를 비동기로 실행하도록 지정하는 어노테이션인데요.

저는 아직 @Async를 제대로 써보지 못해서 우선 실제코드에 적용하기 전에 간단한 예제를 만들어보았어요.

프로젝트 코드는 [spring-boot-async-example](https://github.com/seerlog/spring-boot-async-example){:target="_blank"}에서 보실 수 있어요.

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
> import lombok.extern.slf4j.Slf4j;
> import org.apache.logging.log4j.LogManager;
> import org.apache.logging.log4j.Logger;
> import org.example.async.service.AsyncService;
> import org.example.async.service.SyncService;
> import org.springframework.web.bind.annotation.GetMapping;
> import org.springframework.web.bind.annotation.RestController;
> 
> @Slf4j
> @RestController
> @RequiredArgsConstructor
> public class AsyncController {
> Logger logger = LogManager.getLogger(this.getClass());
> private static final int COUNT = 1000;
> 
>     private final SyncService syncService;
>     private final AsyncService asyncService;
> 
>     @GetMapping("/sync")
>     public String goSync() {
>         long start = System.currentTimeMillis();
>         logger.info("### sync start ###");
>         logger.info("start: {}", start);
>         for (int i = 0; i < COUNT; i++) {
>             syncService.sync(i + 1);
>         }
>         long end = System.currentTimeMillis();
>         logger.info("end: {}", end);
>         logger.info("elapsed millis: {} millis", (end - start));
>         logger.info("### sync end ###");
>         return "sync";
>     }
> 
>     @GetMapping("/async")
>     public String goAsync() throws InterruptedException {
>         long start = System.currentTimeMillis();
>         logger.info("### sync start ###");
>         logger.info("start: {}", start);
>         for (int i = 0; i < COUNT; i++) {
>             asyncService.async(i + 1);
>         }
>         long end = System.currentTimeMillis();
>         logger.info("end: {}", end);
>         logger.info("elapsed millis: {} millis", (end - start));
>         logger.info("### sync end ###");
>         return "async";
>     }
> 
> }
> ```

> #### AsyncService.java
> ```java
> package org.example.async.service;
> 
> import lombok.extern.slf4j.Slf4j;
> import org.apache.logging.log4j.LogManager;
> import org.apache.logging.log4j.Logger;
> import org.springframework.scheduling.annotation.Async;
> import org.springframework.stereotype.Service;
> 
> @Service
> @Slf4j
> public class AsyncService {
> Logger logger = LogManager.getLogger(this.getClass());
> 
>     @Async
>     public void async(int index) throws InterruptedException {
>         Thread.sleep(100);
>         logger.info("async - {}", index);
>     }
> }
> ```

> #### SyncService.java
> ```java
> package org.example.async.service;
> 
> import lombok.extern.slf4j.Slf4j;
> import org.apache.logging.log4j.LogManager;
> import org.apache.logging.log4j.Logger;
> import org.springframework.stereotype.Service;
> 
> @Service
> @Slf4j
> public class SyncService {
> Logger logger = LogManager.getLogger(this.getClass());
> 
>     public void sync(int index) {
>         logger.info("sync - {}", index);
>         System.out.println("sync - " + index);
>     }
> }
> ```

sync 요청과 async 요청 API를 각각 만들고 대응되는 서비스를 구현한 예제예요.

async 서비스의 경우에는 Thread.sleep(100) 을 사용해서 0.1초의 지연을 발생시켰어요.

이제 스프링 부트 프로젝트를 실행하고 http://localhost:8080/sync 요청을 호출해보면 바로 응답이 오는 걸 확인할 수 있어요.

다음으로 http://localhost:8080/async 요청을 호출해보면 응답은 바로 오지만 로그를 보면 요청 처리중인 모습을 확인할 수 있는데요.

@Async로 핵심 로직 이외의 부분을 비동기로 처리해주면 응답속도를 높일 수 있을 것 같아요.

##### 2. 실제 코드 @Async 적용 및 유의사항

> ### 기존 코드 - 동기식
```java
public List<SchoolDTO> searchSchool(String query, Set<String> district, String sort, int page, int size) throws IOException {
   addSearchTerm(SearchDto.SearchTerm.create(query));
   
   List<Query> queries = new ArrayList<>();
   if (isExist(district)) {
   queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
   }
   queries.add(multiMatchQuery(query, "name", "city"));

    List<SortOptions> sortOptions = new ArrayList<>();
    sortOptions.add(getSortOptions("_score", SortOrder.Desc));
    if(SortType.getValue(sort).equals(SortType.LATEST.fieldValue())) {
        sortOptions.add(getSortOptions("rank", SortOrder.Asc));
        sortOptions.add(getSortOptions("update_dt", SortOrder.DESC));
    } else {
        sortOptions.add(getSortOptions(SortType.getValue(sort), SortType.getOrder(sort)));
        sortOptions.add(getSortOptions("rank", SortOrder.Asc));
    }
    
    SearchRequest searchRequest = SearchRequest.of(s -> s
            .index("schoolIndex")
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(sortOptions)
    );

    return elasticsearchClient.search(searchRequest, SchoolDTO.class).hits().hits()
            .stream().map(Hit::source).filter(Objects::nonNull).collect(Collectors.toList());
}

public void addSearchTerm(SearchDto.SearchTerm searchTerm) throws IOException {
    elasticsearchClient.index(i -> i
            .index(ElasticSearch.INDEX_QUERY)
            .id(searchTerm.getDateUpdated().toString())
            .document(searchTerm)
    );
}
```

> ### 수정 코드 - 비동기식
```java
@GetMapping(value = "/hospitals")
@Operation(summary = "병원 검색")
public List<SchoolDTO> searchHospitals(
    @RequestParam String query,
    @RequestParam(required = false) String city,
    @RequestParam(required = false) Set<String> district,
    @RequestParam(defaultValue = "LATEST") String sort,
    @RequestParam(defaultValue = "0") @Min(value = 0, message = "from 은 0 이하일 수 없습니다.") int from,
    @RequestParam(defaultValue = "10") @Min(value = 0, message = "size 는 0 이하일 수 없습니다.") int size
) {
   searchService.addSearchTerm(SearchDto.SearchTerm.create(query));
   List<SchoolDTO> schools = searchService.searchSchool(query, city, district, sort, from, size);
   return Response.of(schools);
}
```

```java
public List<SchoolDTO> searchSchool(String query, Set<String> district, String sort, int page, int size) throws IOException {
   List<Query> queries = new ArrayList<>();
   if (isExist(district)) {
      queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
   }
   queries.add(multiMatchQuery(query, "name", "city"));

   List<SortOptions> sortOptions = new ArrayList<>();
   sortOptions.add(getSortOptions("_score", SortOrder.Desc));
   if(SortType.getValue(sort).equals(SortType.LATEST.fieldValue())) {
      sortOptions.add(getSortOptions("rank", SortOrder.Asc));
      sortOptions.add(getSortOptions("update_dt", SortOrder.DESC));
   } else {
      sortOptions.add(getSortOptions(SortType.getValue(sort), SortType.getOrder(sort)));
      sortOptions.add(getSortOptions("rank", SortOrder.Asc));
   }

   SearchRequest searchRequest = SearchRequest.of(s -> s
           .index("schoolIndex")
           .from(page)
           .size(size)
           .query(boolQuery(queries))
           .sort(sortOptions)
   );

   return elasticsearchClient.search(searchRequest, SchoolDTO.class).hits().hits()
           .stream().map(Hit::source).filter(Objects::nonNull).collect(Collectors.toList());
}

@Async
public void addSearchTerm(SearchDto.SearchTerm searchTerm) throws IOException {
    elasticsearchClient.index(i -> i
            .index(ElasticSearch.INDEX_QUERY)
            .id(searchTerm.getDateUpdated().toString())
            .document(searchTerm)
    );
}
```

실제로 logger 를 찍어보면 비동기로 동작하는 것을 확인할 수 있어요.

테스트를 해보면서 한 가지 알게된 부분은 @Async를 붙인 서비스를 다른 서비스 내에서 호출하면 동작하지 않는다는 것이였어요.

@Async 서비스를 컨트롤러에서 호출해야 비동기로 실행되는 것을 확인했어요.

@Async 를 쓰실 때 이 부분을 유의하시면 좋을 것 같아요.

##### 3. 성능 개선 확인

원래대로라면 Jmeter나 nGrinder 와 같은 부하 테스트 툴을 이용해서 성능 개선을 검증해봐야하는데요.

아쉽게도 개발 환경에서 엘라스틱 서치가 쓰이고 있어서 부하 테스트는 진행하지 못했어요.

대신에 Postman을 이용해서 addSearchTerm 함수를 동기/비동기로 호출 했을 때 응답시간을 측정해보았어요.

| 항목 | 동기 | 비동기 |
|----|------|------|
| 1회 | 785ms | 421ms |
| 2회 | 310ms | 108ms |
| 3회 | 212ms | 120ms |
| 4회 | 247ms | 94ms |
| 5회 | 240ms | 96ms |
| 6회 | 233ms | 16ms |
| 7회 | 115ms | 95ms |
| 8회 | 229ms | 112ms |
| 9회 | 266ms | 1112ms |
| 10회 | 261ms | 105ms |
| 평균 | 289.8ms | 227.9ms |

비록 API 10회 호출에 대한 테스트 결과이지만 동기보다 비동기일 떄 응답속도가 조금 더 빠른 것을 확인해볼 수 있어요.

앞으로 개발하면서 로직에 대해 고민하면서 비동기를 적용할 수 있는 부분이 있다면 적극적으로 적용해보려고 해요.

그럼, 오늘도 미션 클리어! 👍
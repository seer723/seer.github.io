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

@Async 를 붙인 서비스를 다른 서비스 내에서 호출하면 동작하지 않음

@Async 를 붙인 서비스를 컨트롤러로 빼줘야 동작함


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

```java
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
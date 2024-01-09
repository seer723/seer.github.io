---
title: "2024-01-09-ElasticsearchClient 정렬기준 조건화 방법"
last_modified_at: 2024-01-09
categories:
  - Java
tags:
  - Java
  - Elasticsearch
  - ElasticsearchClient
---

> 모던 자바 인 액션(한빛미디어) 책을 보면서 동작 파라미터화에 대해 공부한 내용을 정리했어요.  
> 농장재고목록 애플리케이션 예제 코드를 개선하면서 자바 8의 장점을 느껴볼 수 있는 내용이예요.  

##### 1. 농장재고목록 애플리케이션

```java
public ListResponseDto<SearchDto.Hospital> searchHospitals(String query, Set<String> district, String sort, int page, int size)
        throws IOException {
        addSearchTerm(SearchDto.SearchTerm.create(query));

        List<Query> queries = new ArrayList<>();
        if (isExist(district)) {
            logger.debug(district.toString());
            queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
        }
        queries.add(multiMatchQuery(query, ElasticSearch.FIELDS_NAME, ElasticSearch.FIELDS_CITY, ElasticSearch.FIELDS_DISTRICT,
            ElasticSearch.FIELDS_MEDICAL_CATEGORY));

        SearchRequest searchRequest = SearchRequest.of(s -> s
            .index(ElasticSearch.INDEX_HOSPITAL)
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(st -> st.field(f -> f.field("_score").order(SortOrder.Desc)))
            .sort(st -> st.field(f -> f.field(SortType.getValue(sort)).order(SortType.getOrder(sort))))
            .sort(st -> st.field(f -> f.field(ElasticSearch.FIELDS_EXPOSED_RANK).order(SortOrder.Asc)))
        );

        SearchResponse<SearchDto.Hospital> searchResponse = elasticsearchClient.search(searchRequest, SearchDto.Hospital.class);
        List<SearchDto.Hospital> list = searchResponse.hits().hits().stream().map(Hit::source).filter(Objects::nonNull)
            .peek(i -> i.setThumbnail((cloudFrontService.generateSignedUrl(i.getThumbnail())))).toList();

        return ListResponseDto.of(list, total(searchResponse));
    }
```

```java
public ListResponseDto<SearchDto.Hospital> searchHospitals(String query, Set<String> district, String sort, int page, int size)
        throws IOException {
    addSearchTerm(SearchDto.SearchTerm.create(query));

    List<Query> queries = getQueries(query, district);
    List<SortOptions> sortOptions = getSortOptionsList(sort);

    SearchRequest searchRequest = SearchRequest.of(s -> s
            .index(ElasticSearch.INDEX_HOSPITAL)
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(sortOptions)
    );

    SearchResponse<SearchDto.Hospital> searchResponse = elasticsearchClient.search(searchRequest, SearchDto.Hospital.class);
    List<SearchDto.Hospital> list = searchResponse.hits().hits().stream().map(Hit::source).filter(Objects::nonNull)
            .peek(i -> i.setThumbnail((cloudFrontService.generateSignedUrl(i.getThumbnail())))).toList();

    return ListResponseDto.of(list, total(searchResponse));
}

private List<Query> getQueries(String query, Set<String> district) {
    List<Query> queries = new ArrayList<>();

    if (isExist(district)) {
        logger.debug(district.toString());
        queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
    }
    queries.add(multiMatchQuery(query, ElasticSearch.FIELDS_NAME, ElasticSearch.FIELDS_CITY, ElasticSearch.FIELDS_DISTRICT,
            ElasticSearch.FIELDS_MEDICAL_CATEGORY));

    return queries;
}

private List<SortOptions> getSortOptionsList(String sort) {
    List<SortOptions> sortOptions = new ArrayList<>();

    if(SortType.getValue(sort).equals(SortType.LATEST.fieldValue())) {
        sortOptions.add(getSortOptions("_score", SortOrder.Desc));
        sortOptions.add(getSortOptions(ElasticSearch.FIELDS_EXPOSED_RANK, SortOrder.Asc));
        sortOptions.add(getSortOptions(SortType.LATEST.fieldValue(), SortType.LATEST.orderValue()));
    } else {
        sortOptions.add(getSortOptions("_score", SortOrder.Desc));
        sortOptions.add(getSortOptions(SortType.getValue(sort), SortType.getOrder(sort)));
        sortOptions.add(getSortOptions(ElasticSearch.FIELDS_EXPOSED_RANK, SortOrder.Asc));
    }

    return sortOptions;
}

private SortOptions getSortOptions(String sort, SortOrder order) {
    return new SortOptions.Builder().field(f -> f.field(sort).order(order)).build();
}
```


오늘도 미션 클리어! 👍
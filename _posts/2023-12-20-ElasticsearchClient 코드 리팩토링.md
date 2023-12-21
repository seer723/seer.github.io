---
title: "ElasticsearchClient 코드 리팩토링"
last_modified_at: 2023-12-20
categories:
  - ElasticSearch
tags:
  - ElasticSearch
  - ElasticsearchClient
  - SpringBoot
  - Refactoring
---

> 스프링부트에서 엘라스틱 서치 클라이언트를 사용하는 코드를 리팩토링해보려 해요.  
> 리팩토링할 부분은 요청 파라미터에 따라 분기처리를 적용하는 부분이예요.

##### 1. 리팩토링 전 코드
```java
public List<SchoolDTO> searchSchool(String query, Set<String> district, String sort, int page, int size) throws IOException {
    List<String> fields = Arrays.asList("name", "city");
    SearchResponse<SchoolDTO> response = null;
    if (district == null || district.isEmpty()) {
        response =  elasticsearchClient.search(s -> s
                .index("schoolIndex")
                .from(page)
                .size(size)
                .query(q -> q
                    .bool(b -> b
                        .must(m -> m
                            .multiMatch(v -> v
                                .type(TextQueryType.BoolPrefix)
                                .fields(fields)
                                .query(query)
                            )
                        )
                    )
                )
                .sort(st -> st.field(f -> f.field(SortType.valueOf(sort).fieldValue()).order(SortType.valueOf(sort).orderValue())))
            , SchoolDTO.class
        );
    } else {
        response = elasticsearchClient.search(s -> s
                .index("schoolIndex")
                .from(page)
                .size(size)
                .query(q -> q
                    .bool(b -> b
                        .must(m-> m
                            .match(mm -> mm
                                .field("district")
                                .query(district.toString())
                            )
                        )
                        .must(m -> m
                            .multiMatch(v -> v
                                .type(TextQueryType.BoolPrefix)
                                .fields(fields)
                                .query(query)
                            )
                        )
                    )
                )
                .sort(st -> st.field(f -> f.field(SortType.valueOf(sort).fieldValue()).order(SortType.valueOf(sort).orderValue())))
            , SchoolDTO.class
        );
    }

    List<Hit<SchoolDTO>> hits = response.hits().hits();

    return hits.stream().map(Hit::source).filter(Objects::nonNull).collect(Collectors.toList());
}
```

##### 2. 리팩토링 후 코드
```java
public List<SchoolDTO> searchSchool(String query, Set<String> district, String sort, int page, int size) throws IOException {
    List<Query> queries = new ArrayList<>();
    if(isExist(district)) {
        queries.add(matchQuery("district", district.toString()));
    }
    queries.add(multiMatchQuery(query, "name", "city"));

    SearchRequest searchRequest = SearchRequest.of(s -> s
        .index("schoolIndex")
        .from(page)
        .size(size)
        .query(boolQuery(queries))
        .sort(st -> st.field(f -> f.field(SortType.getValue(sort)).order(SortType.getOrder(sort))))
    );

    return elasticsearchClient.search(searchRequest, SchoolDTO.class).hits().hits()
        .stream().map(Hit::source).filter(Objects::nonNull).collect(Collectors.toList());
}

private Query term(String field, String value) {
    return QueryBuilders.term(m -> m
        .field(field)
        .value(value));
}

private Aggregation terms(String field, int size) {
    return new Aggregation.Builder().terms(new TermsAggregation.Builder()
        .field(field)
        .size(size).build()).build();
}

private boolean isExist(Object object) {
    return !ObjectUtils.isEmpty(object);
}

private Query matchQuery(String field, String query) {
    return QueryBuilders.match(m -> m
        .field(field)
        .query(query));
}

private Query multiMatchQuery(String query, String ...fields) {
    return QueryBuilders.multiMatch(m -> m
        .type(TextQueryType.BoolPrefix)
        .fields(Arrays.stream(fields).toList())
        .query(query));
}

private Query boolQuery(List<Query> queries) {
    BoolQuery.Builder boolQueryBuilder = QueryBuilders.bool();
    for (Query searchQuery : queries) {
        boolQueryBuilder.must(searchQuery);
    }
    return boolQueryBuilder.build()._toQuery();
}
```
    
##### 3. 리팩토링을 통해 얻은 효과

> 코드 가독성 향상  

district 변수를 체크하는 if 구문을 제거하고나니까 우선 코드가 간결해졌어요.  
if 구문으로 로직이 나뉘어 있다보니까 소스를 파악하기 위해서는 if 절과 else 절을 모두 확인해야하는 불편함이 있었는데요.  
if 구문이 꼭 필요한 district 변수에 대해서만 적용하고 나니까 소스 읽기가 더 편해졌어요.  
그리고 잦은 빌더 패턴으로 오히려 가독성이 떨어지는 부분이 있었는데요.  
빌더 패턴이 꼭 필요한 부분만 남기고 정리하니까 가독성이 더 좋아진 것을 체감할 수 있었어요.

> 코드 재사용 향상

위에서 진행했던 리팩토링 코드에서는 효과를 느끼기 어렵지만 만약 검색 함수가 100개가 되었다고 했을 때 리팩토링 이전의 방법대로 구현한다면 중복된 코드가 매우 많아질거예요.    
그래서 자주 쓰이는 코드를 따로 함수로 분리해서 재사용성을 높이는 작업을 했어요.  
나중에 검색 함수가 새로 추가되어도 기존에 작업하면서 만들어둔 term 이나 query 관련함수들로 구현하면 최대한 코드 중복을 피하면서 함수를 작성할 수 있어요.
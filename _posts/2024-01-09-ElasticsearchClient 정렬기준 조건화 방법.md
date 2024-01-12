---
title: "ElasticsearchClient 정렬기준 QueryDSL 적용 예제"
last_modified_at: 2024-01-09
categories:
  - Java
tags:
  - Java
  - Elasticsearch
  - ElasticsearchClient
  - QueryDSL
---

> 기존에 구현한 학교 검색 메소드에 추가 요구사항이 생겼어요.
> 정렬 파라미터에 따라 다른 정렬 기준을 적용하는 것이 내용이었는데요.
> ElasticsearchClient Java API 에서 제공하는 QueryDSL 을 이용해서 요구사항을 적용해봤어요.

##### 1. 기존 코드

아래는 기존에 구현한 학교 검색 메소드 소스예요.

맨 처음 코드를 작성할 때는 elasticsearchClient.search() 메소드안에 모든 파라미터를 전달하도록 구현했었는데요.

ElasticsearchClient Java API 의 QueryDSL 함수의 존재를 알고난 후로는 QueryDSL 을 사용하도록 코드를 리팩토링했어요.

[ElasticsearchClient 코드 리팩토링](https://seerlog.github.io/elasticsearch/ElasticsearchClient-%EC%BD%94%EB%93%9C-%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81/){:target="_blank"}에서 확인해보실 수 있답니다.

이렇게 한 번 리팩토링을 해둔 덕분에 이번 요구사항을 적용하는데 큰 어려움은 없었던 것 같아요.

```java
public List<SchoolDTO> searchSchool(String query, Set<String> district, String sort, int page, int size) throws IOException {
        List<Query> queries = new ArrayList<>();
        if (isExist(district)) {
            queries.add(matchQuery(ElasticSearch.FIELDS_DISTRICT, district.toString()));
        }
        queries.add(multiMatchQuery(query, "name", "city"));

        SearchRequest searchRequest = SearchRequest.of(s -> s
            .index("schoolIndex")
            .from(page)
            .size(size)
            .query(boolQuery(queries))
            .sort(st -> st.field(f -> f.field("_score").order(SortOrder.Desc)))
            .sort(st -> st.field(f -> f.field(SortType.getValue(sort)).order(SortType.getOrder(sort))))
            .sort(st -> st.field(f -> f.field("rank").order(SortOrder.Asc)))
        );

    return elasticsearchClient.search(searchRequest, SchoolDTO.class).hits().hits()
            .stream().map(Hit::source).filter(Objects::nonNull).collect(Collectors.toList());
}
```

##### 2. 정렬 기준 변경 요구사항

추가 변경 요구사항으로 들어온 것은 정렬 파라미터(sort) 값에 따라서 정렬 조건을 다르게 적용해야 하는 내용이 있었어요.

sort 값이 latest 인 경우에는 rank 필드를 우선 적용해야하고, 나머지 경우에는 rank 필드가 나중에 적용되야하는 요건이예요.

구글링을하면서 SortOptions 를 이용해서 정렬 조건을 QueryDSL 방식으로 만들어줄 수 있다는 것을 알게되었어요.

그래서 정렬 부분만 리팩토링을 진행했어요.

##### 3. 수정 후 코드

아래는 수정 후의 코드인데요.

sort 파라미터 값에 따라서 정렬 조건이 다르게 적용된 sortOptions 변수가 추가된 것을 확인할 수 있어요.

이 값은 SearchRequest 객체를 만들때 sort() 빌더 메소드에 전달해주면 된답니다.

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
```

테스트를 해보니까 의도했던 대로 잘 동작하네요 😁

리팩토링을 평소에 잘 해두면 추가 요구사항에 대응하기가 훨씬 수월해진다는 것을 피부로 느낄 수 있었던 작업이었던 것 같아요.

앞으로도 시간날 때 틈틈히 공부하고 코드 리팩토링을 게을리하지 않아야겠다는 생각을 해보게 되네요.

그럼, 오늘도 미션 클리어! 👍
> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 3편] 페이징 개선(querydsl, 커버링인덱스, count 쿼리)



# 커버링 인덱스
- 찾고자 하는 데이터가 전부 인덱스 안에 들어있는 경우 데이터 블록을 조회하지 않고 인덱스에서만 조회합니다. (=성능이 빠릅니다.)

### 사용 예시
- order by, limit offset 사용할 때, 데이터 블록에 접근해서 데이터를 접근합니다.
    - 커버링 인덱스를 이용해서 인덱스에만 접근해서 데이터를 가져와서 성능 개선을 할 수 있습니다.


### source
- Querydsl 의 경우 서브 쿼리를 지원하지 않기 때문에 쿼리를 두번 사용해야 합니다.

```java
public xxx testPagaConveringIndex(xxx) {
	List<Long> seqs = queryFactory
					.select(test.seq)
					.from(test)
					.where(test.title.eq("test")
					.orderBy(test.seq.desc())
					.limit(pageSize)
					.offset(pageNo * pageSize)
					.fetch();
	
	retrun queryFactory.select(test)
								.from(test)
								.where(test.seq.in(seqs)
								.orderBy(test.seq.desc())
								.fetch()
	
}

```

- jdbcTemplate 의 경우 서브쿼리에 위 쿼리를 다 넣어주면 됩니다.

### 단점
- 인덱스를 이용해서 데이터를 가져오기에 가져오는 컬럼 수가 많다면 인덱스 크기가 커집니다.
    - 인덱스 크기가 커지면 CUD 쿼리에 대해서 성능이 나빠질 수 있습니다.

# count 쿼리를 요청할 때만 수행시킨다.
- 페이지 번호를 보여주기 위해 count 쿼리를 수행해야합니다.
- 하지만 이 페이지 번호를 고정으로 보여주고 (1~5번) 해당 페이지 번호를 눌렀을 때만, count 쿼리가 동작되게 한다면 성능 개선이 있을 수 있습니다.
- 이 방법은 페이지 번호가 반드시 보여야하는 경우에는 쓸 수 없습니다.

# 페이지 번호를 호출하는 횟수가 빈번할 경우

### 개념

- 첫번째 count 쿼리를 수행 후, totalCount 값을 화면으로 던집니다.
- 화면에서는 이를 받아 캐시에 저장해놓습니다.
- 화면에서 페이지 번호를 누를 때, totalCount 값을 같이 넘겨줍니다.
- 서버에서는 totalCount 값이 존재하면 count 쿼리를 수행할 필요 없고, 값이 없으면 count 쿼리를 실행하면 됩니다.

### 제약사항

- 데이터가 정적 (마감 완료) 일 때만 사용 가능합니다.

# 동적으로 데이터가 변경될 때, 페이징 성능 개선 방안

- 페이지 번호를 고정으로 해서 보여줍니다. (1 ~ 5, 1 ~ 10) 페이지 번호 맨 끝에는 '다음' 버튼을 만들어줍니다.
- 검색 버튼을 누를 때마다 결과물을 보여줍니다. (No OffSet, 커버링 인덱스 사용)
- 1~5 를 고정적으로 보여준다고 가정하겠습니다. 2번을 눌렀을 때는 2번에 해당하는 결과를 보여줍니다.
- 다음 버튼을 누를 경우 데이터가 존재할 경우 6 ~ 10 을 보여줍니다. 위에 과정을 반복합니다.


## reference
- https://jojoldu.tistory.com/529?category=637935
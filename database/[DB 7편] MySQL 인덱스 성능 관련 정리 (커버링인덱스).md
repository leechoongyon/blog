> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 7편] MySQL 인덱스 성능 관련 정리 (커버링인덱스)



# 커버링 인덱스란?

- 쿼리의 결과 값이 Index 에 전부 존재하는 경우입니다.
- 데이터를 찾기 위해 추가로 디스크 접근을 할 필요가 없기에 성능이 좋습니다.



### 무엇이 좋은가?

- 인덱스의 경우 보통 1 row 보다 데이터가 적기에 디스크 접근 횟수가 전체 행을 조회하는 것보다 빠릅니다.
- 또한, 인덱스의 경우 같은 페이지내에서 보통 정렬되서 저장되므로 범위 조회를 할 때, 빠른 성능을 보여줍니다.



### 예시

- 아래와 같은 예시가 있을 경우 MySQL InnoDB 를 기준으로 실행계획을 수행하면 커버링 인덱스를 타게 됩니다.
- 실행계획 중 Extra 항목에 Using Index 라고 나온 것이 커버링 인덱스를 탄 항목입니다.

```sql
create table test (
	a int unsigned NOT NULL AUTO_INCREMENT,
  b varchar2(20),
  PRIMARY_KEY(a),
  KEY index_test(a,b)
) ENGINE=InnoDB

EXPLAIN select a, b from test where a = 50;

id: 1
select_type:SIMPLE
table: test
type: index (행에 어떻게 접근하고 조인했는지 등에 대한 설명)
...
...
...
Extra: Using index
```



### 조회하는 모든 행의 결과 컬럼이 인덱스에 포함이 되지 않았는데도 커버링 인덱스가 타는 경우

- 예를 들면 아래와 같습니다.
- 위 예제와는 다르게 아래 예제에는 sub_index(b) 가 보조 인덱스로 걸려있습니다.
- MySQL 은 보조 인덱스에 기본키 값이 있고, 인덱스에 기본키 ID 를 검색하는 쿼리가 있습니다. 그렇기에 a,b 가 복합인덱스로 안걸려있어도 b 는 기본키에 대한 접근이 가능하므로, 최종적으로는 인덱스로만 조회 결과를 가져올 수 있습니다. 
- 즉, 커버링 인덱스를 타게 됩니다. (=인덱스로만 데이터를 가져오는)



```sql
create table test (
	a int unsigned NOT NULL AUTO_INCREMENT,
  b varchar2(20),
  PRIMARY_KEY(a),
  KEY sub_index(b)
) ENGINE=InnoDB

EXPLAIN select a, b from test where a = 50;

id: 1
select_type:SIMPLE
table: test
type: index (행에 어떻게 접근하고 조인했는지 등에 대한 설명)
...
...
...
Extra: Using index
```



# 커버링 인덱스를 활용하는 방안

- 아래 예시와 같이 test 라는 테이블이 있고, b 에 인덱스가 걸려있습니다.
- 조회 조건에 b 가 있으니 인덱스를 탈 것입니다. 그런데 데이터 양이 너무 많아 인덱스를 걸어도 성능이 안나올 수 있습니다.

- 이럴 때, 인덱스를 확장해서 커버링 인덱스를 타도록해 성능을 개선할 수 있습니다.
- KEY test_index(b) ---> KEY test_index(b,c,d)

```sql
create table test (
	a int unsigned NOT NULL AUTO_INCREMENT,
  b varchar2(20),
  PRIMARY_KEY(a),
  KEY test_index(b),
) ENGINE=InnoDB

select b, c, d from test where b = 50;
```




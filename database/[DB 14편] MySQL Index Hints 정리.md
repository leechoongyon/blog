> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 14편] MySQL Index Hints 정리





# 예시에 사용된 DDL, DML

<script src="https://gist.github.com/leechoongyon/e11dd6c06211edd809040a454a11c807.js"></script>





# MySQL Index Hints

- 인덱스 힌트는 옵티마이저에게 인덱스를 어떻게 선택해야하는지에 대한 정보를 주는 옵션입니다.





# IGNORE 

- 해당 인덱스를 사용하지 않도록 옵티마이저에게 정보를 전달합니다.



### IGNORE INDEX 예시 쿼리

```sql
# 힌트 사용 전
explain format = tree
select tm.*
from team t, team_member tm
where tm.team_id = t.team_id
and tm.name = 'member-1'
and t.team_id = '1'
;

# 힌트 사용
explain format = tree
select tm.*
from team t IGNORE INDEX(PRIMARY), team_member tm
where tm.team_id = t.team_id
and tm.name = 'member-1'
and t.team_id = '1'
;

```



### IGNORE INDEX 예시 실행 계획

- 힌트 사용 전에는 TEAM 테이블의 기본키를 통해 조회를 했지만, 힌트 적용 후에는 테이블을 full scan 했습니다.

```text
[힌트 사용 전]
2. -> Filter: (tm.team_id = 1)  (cost=0.29 rows=0)
1.    -> Index lookup on tm using index3 (name='member-1')  (cost=0.29 rows=1)


[힌트 사용]
5. -> Inner hash join (no condition)  (cost=0.68 rows=0)
    4. -> Filter: (t.team_id = 1)  (cost=1.13 rows=1)
       3. -> Table scan on t  (cost=1.13 rows=2)
    -> Hash
       2.  -> Filter: (tm.team_id = 1)  (cost=0.29 rows=0)
          1.  -> Index lookup on tm using index3 (name='member-1')  (cost=0.29 rows=1)
```







# FORCE

- USE INDEX Hints 와 유사하게 동작하지만 옵티마이저에게 추가 정보를 전달합니다. (테이블 스캔은 매우 비용이 크다는 것을)



### 사용 예시

```sql
explain format = tree
select tm.*
from team t, team_member tm FORCE INDEX(index3)
where tm.team_id = t.team_id
and tm.name = 'member-1'
and t.team_name = 'team-name'
;
```





# USE INDEX

- 인덱스를 사용하도록 MySQL 에게 정보를 전달합니다.



### 사용 예시

```sql
explain format = tree
select tm.*
from team t, team_member tm USE INDEX(index3)
where tm.team_id = t.team_id
and tm.name = 'member-1'
and t.team_name = 'team-name'
;
```






# Reference

- https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html
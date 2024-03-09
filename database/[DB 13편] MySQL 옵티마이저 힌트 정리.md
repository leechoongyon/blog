> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 13편] MySQL 옵티마이저 힌트 정리





# 아래 예시에 사용된 DDL, DML

<script src="https://gist.github.com/leechoongyon/e11dd6c06211edd809040a454a11c807.js"></script>







# 옵티마이저 힌트

- 옵티마이저를 컨트롤 할 수 있는 방법 중의 하나이며, 개별 구문내에서 사용 가능합니다.
- /*+ ... */ 로 사용가능합니다.
  - ex) select /*+ ... HINT */
- 사용할 때, 주의사항으로는 테이블에 alias 가 있을 경우 alias 명을 써줘야 합니다.





# JOIN_FIXED_ORDER

- 조인순서를 강제합니다.
- 사용법은 select /*+ JOIN_FIXED_ORDER () */  이며, 쿼리에 입력된 순으로 조인순서를 강제합니다.



### HINT 적용 전 쿼리 및 실행계획

- 조인 순서가 team, team_member, member_addr 순으로 조인된 것을 알 수 있습니다.

```sql
# 사용 쿼리
explain format = json
select * from team t, team_member tm, member_addr ma 
where t.team_id = tm.team_id
and tm.team_member_id = ma.member_id
and tm.team_id = 1
;
```



```json
# 실행계획
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.95"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "t",
          "access_type": "const",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "team_id"
          ],
          "key_length": "8",
          "ref": [
            "const"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 1,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "0.00",
            "eval_cost": "0.10",
            "prefix_cost": "0.00",
            "data_read_per_join": "784"
          },
          "used_columns": [
            "team_id",
            "team_name"
          ]
        }
      },
      {
        "table": {
          "table_name": "tm",
          "access_type": "ref",
          "possible_keys": [
            "PRIMARY",
            "FK9ubp79ei4tv4crd0r9n7u5i6e"
          ],
          "key": "FK9ubp79ei4tv4crd0r9n7u5i6e",
          "used_key_parts": [
            "team_id"
          ],
          "key_length": "8",
          "ref": [
            "const"
          ],
          "rows_examined_per_scan": 2,
          "rows_produced_per_join": 2,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "0.50",
            "eval_cost": "0.20",
            "prefix_cost": "0.70",
            "data_read_per_join": "1K"
          },
          "used_columns": [
            "team_member_id",
            "name",
            "team_id"
          ]
        }
      },
      {
        "table": {
          "table_name": "ma",
          "access_type": "ALL",
          "rows_examined_per_scan": 5,
          "rows_produced_per_join": 2,
          "filtered": "20.00",
          "using_join_buffer": "hash join",
          "cost_info": {
            "read_cost": "0.25",
            "eval_cost": "0.20",
            "prefix_cost": "1.95",
            "data_read_per_join": "624"
          },
          "used_columns": [
            "addr_id",
            "zip_code",
            "addr_detail",
            "member_id"
          ],
          "attached_condition": "(`training`.`ma`.`member_id` = `training`.`tm`.`team_member_id`)"
        }
      }
    ]
  }
}
```





### HINT 적용 후 쿼리 및 실행계획

- 힌트를 적용 후, 조인순서가 team_member, member_addr, team 순으로 변경된 것을 확인할 수 있습니다.

```sql
# 쿼리
explain format = json
select /*+ JOIN_FIXED_ORDER () */ 
* 
from team_member tm, member_addr ma , team t
where t.team_id = tm.team_id
and tm.team_member_id = ma.member_id
and tm.team_id = 1
;
```



```json
# 실행계획
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "2.40"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "tm",
          "access_type": "ref",
          "possible_keys": [
            "PRIMARY",
            "FK9ubp79ei4tv4crd0r9n7u5i6e"
          ],
          "key": "FK9ubp79ei4tv4crd0r9n7u5i6e",
          "used_key_parts": [
            "team_id"
          ],
          "key_length": "8",
          "ref": [
            "const"
          ],
          "rows_examined_per_scan": 2,
          "rows_produced_per_join": 2,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "0.50",
            "eval_cost": "0.20",
            "prefix_cost": "0.70",
            "data_read_per_join": "1K"
          },
          "used_columns": [
            "team_member_id",
            "name",
            "team_id"
          ]
        }
      },
      {
        "table": {
          "table_name": "ma",
          "access_type": "ALL",
          "rows_examined_per_scan": 5,
          "rows_produced_per_join": 2,
          "filtered": "20.00",
          "using_join_buffer": "hash join",
          "cost_info": {
            "read_cost": "0.25",
            "eval_cost": "0.20",
            "prefix_cost": "1.95",
            "data_read_per_join": "624"
          },
          "used_columns": [
            "addr_id",
            "zip_code",
            "addr_detail",
            "member_id"
          ],
          "attached_condition": "(`training`.`ma`.`member_id` = `training`.`tm`.`team_member_id`)"
        }
      },
      {
        "table": {
          "table_name": "t",
          "access_type": "const",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "team_id"
          ],
          "key_length": "8",
          "ref": [
            "const"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 2,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "0.25",
            "eval_cost": "0.20",
            "prefix_cost": "2.40",
            "data_read_per_join": "1K"
          },
          "used_columns": [
            "team_id",
            "team_name"
          ]
        }
      }
    ]
  }
}
```





# JOIN_PREFIX

- 처음 조인순서를 명시합니다.



### 사용 예시

- team_member, member_addr 을 먼저 조인하라고 hist 를 준 예시입니다.

```sql
# 쿼리
explain format = json
select /*+ JOIN_PREFIX (tm, ma) */
* 
from team_member tm, member_addr ma , team t
where t.team_id = tm.team_id
and tm.team_member_id = ma.member_id
and tm.team_id = 1
;


```





# JOIN_ORDER

- JOIN_FIXED_ORDER 와는 다르게 옵티마이저에게 join 순서를 권유하는 명령어입니다. 옵티마이저가 판단했을 때, 효율이 안좋다면 사용되지 않습니다.



### 사용 예시

- 조인 순서를 team_member, member_addr, team 순으로 힌트를 준 것 입니다.

```sql
# 쿼리
explain format = json
select /*+ JOIN_ORDER (tm, ma, t) */
* 
from team_member tm, member_addr ma , team t
where t.team_id = tm.team_id
and tm.team_member_id = ma.member_id
and tm.team_id = 1
;

```





# Hash Join, No Hash Join

- Hash Join 을 사용할지 안할지를 명시하는 힌트 입니다.



### 사용 예시

```sql
select /*+ NO_HASH_JOIN (t,tm) */ * from team t, team_member tm
where t.team_id = tm.team_id
;


```





# Reference

- https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html

- http://minsql.com/mysql8/B-2-D-optimizerHint/

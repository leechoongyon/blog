> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 11편] MySQL 실행계획 1탄 (실행계획 순서)



# 예시에 사용된 테이블 DDL, DML

- TEAM : TEAM_MEMBER = 1 : n



```text
# DDL
CREATE TABLE `team` (
  `team_id` bigint NOT NULL AUTO_INCREMENT,
  `team_name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`team_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci


CREATE TABLE `team_member` (
  `team_member_id` bigint NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `team_id` bigint NOT NULL,
  PRIMARY KEY (`team_member_id`),
  KEY `FK9ubp79ei4tv4crd0r9n7u5i6e` (`team_id`),
  KEY `index3` (`name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci



# DML
INSERT INTO team (`team_id`,`team_name`) VALUES (1,'test-team');
INSERT INTO team (`team_id`,`team_name`) VALUES (2,'test-team-2');

INSERT INTO team_member (`team_member_id`,`name`,`team_id`) VALUES (1,'member-1',1);
INSERT INTO team_member (`team_member_id`,`name`,`team_id`) VALUES (2,'member-2',2);
INSERT INTO team_member (`team_member_id`,`name`,`team_id`) VALUES (3,'member-3',1);
INSERT INTO team_member (`team_member_id`,`name`,`team_id`) VALUES (4,'member-4',2);
INSERT INTO team_member (`team_member_id`,`name`,`team_id`) VALUES (5,'member-5',2);

```



# MySQL 실행계획 보는 방법



### 명령어를 통해 보는 방법

- 쿼리에 EXPLAIN 명령어를 붙이면 실행계획을 볼 수 있습니다.
- 사용한 쿼리

```text
explain
select * from team t1
where t1.team_id in (
	select tm.team_id from team_member tm, team t2
	where tm.team_id  = t2.team_id 
	and tm.name = 'member-1'
)
;
```



- 실행결과
  - 보기 힘든 것 같습니다...

![실행계획 1](./images/explain_1.png)



### 명령어를 통해 보는 방법 (explain format = tree, json, TRADITIONAL)

- 사용한 쿼리

```text
explain format = tree
select * from team t1
where t1.team_id in (
	select tm.team_id from team_member tm, team t2
	where tm.team_id  = t2.team_id 
	and tm.name = 'member-1'
)
;
```

- 실행계획 결과 및 분석
  - 보는 방법은 위에서 아래로 향하며, 안에 있는 것을 먼저 읽습니다.
  - 1~6 순서대로 읽습니다.

```text
-> 6. Remove duplicate (t1, t2) rows using temporary table (weedout)  (cost=1.05 rows=1)
    -> 5. Nested loop inner join  (cost=1.05 rows=1)
        -> 3. Nested loop inner join  (cost=0.70 rows=1)
            -> 1. Index lookup on tm using index3 (name='member-1')  (cost=0.35 rows=1)
            -> 2. Single-row index lookup on t1 using PRIMARY (team_id=tm.team_id)  (cost=0.35 rows=1)
        -> 4. Single-row index lookup on t2 using PRIMARY (team_id=tm.team_id)  (cost=0.35 rows=1)
```




### 이미지를 통해 보는 방법

-   위와 같이 EXPLAIN 명령어를 통해 확인하면 직관적으로 확인하기 힘듭니다.
-   mysql workbench 에서 제공해주는 visual explain 을 사용하면 보기 쉽습니다.

### 보는 순서

1.  team\_member.name 에 대해 먼저 조회를 해서 1건을 가져옵니다. (member-1 인 name 을)
2.  1건에 대한 것을 team t1 에서 nested loop 로 join 을 합니다.
  -   순서상으로 보면 team t2 를 조인할 것이라 생각했는데 옵티마이저는 해당 방법이 더 좋다고 판단을 한 것 같습니다.
3.  join 한 결과를 가지고 team t2 와 nested loop join 을 합니다.
4.  가져온 값을 가지고 임시 테이블을 만든 뒤, 결과를 출력합니다.

```
select * from team t1 where t1.team_id in (     select tm.team_id from team_member tm , team t2     where tm.team_id  = t2.team_id      and tm.name = 'member-1' ) ;
```


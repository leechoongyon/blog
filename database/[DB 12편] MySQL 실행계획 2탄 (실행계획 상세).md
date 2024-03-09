> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 12편] MySQL 실행계획 2탄 (실행계획 상세)



> [MySQL 실행계획 1탄](https://insanelysimple.tistory.com/424) 과 연결됩니다.





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






# ID

- select 를 구분하기 위한 용도입니다.
- 아래와 같이 조인으로 묶였을 때는 같은 ID 로 표현되지만 subquery 나 union 을 사용할 경우 다른 ID 로 부여됩니다.



##### 실행쿼리

```text
explain select * from team t, team_member m
where 1=1
and t.team_id  = m.team_id
;
```



##### 실행계획 결과

![ID EXPLAIN](./images/explain_2.png)





# select_type

- select 쿼리가 어떤 타입인지에 대한 컬럼입니다.





### SIMPLE 

- 단순 SELECT 쿼리입니다. (JOIN 포함)





### PRIMARY

- UNION ALL 등을 사용할 경우 가장 바깥쪽에 있는 SELECT 쿼리에 PRIMARY 가 표시됩니다. 



##### 예시 쿼리

```text
explain
select * from team t1
where t1.team_id in (
	select tm.team_id from team_member tm, team t2
	where tm.team_id  = t2.team_id 
	and tm.name = 'member-1'
)
union all
select * from team t1
where t1.team_id in (
	select tm.team_id from team_member tm, team t2
	where tm.team_id  = t2.team_id 
	and tm.name = 'member-2'
)
;
```



##### 실행계획 결과
![PRIMARY_DESC](./images/explain_4.png)


### UNION RESULT

-   union 관련 쿼리를 사용할 경우 UNION RESULT 가 생기는 경우가 있는데, 이는 임시테이블을 만드는 것을 의미합니다.
-   임시 테이블을 만드는 것은 성능상 문제가 있을 수 있으므로 될 수 있으면 고쳐야 합니다.

##### 예시 쿼리

```
explain select * from team_member tm, (     select * from team  ) as t where tm.team_id = t.team_id union select * from team_member tm, (     select * from team  ) as t where tm.team_id = t.team_id ;
```

##### 실행계획 결과

![UNION RESULT](https://blog.kakaocdn.net/dn/Ozula/btrSlYhxhtp/h9fOKQzNJDBD73iJoLckEK/img.png)

# type

-   테이블에 접근을 해서 어떻게 row 를 가져왔는지에 대한 필드입니다.

| 접근 방식 | 설명 |
| --- | --- |
| ALL | 전체 row 를 스캔합니다. full scan 입니다. 즉, 효율이 안좋습니다. |
| index | 인덱스 스캔입니다. 인덱스의 전체에 접근합니다. 즉, 효율이 안좋습니다. |
| const | 기본 키, 고유키 (unique key) eq 비교입니다. 1개의 행만 조회하기에 성능이 좋습니다. |
| range | 인덱스 특정 범위의 행에 접근합니다. 성능이 비교적 잘 나오는 편입니다. |
| eq\_ref | 조인 테이블에서 발생합니다. drive table (첫번째 읽은 테이블) 에서 읽은 값을 가지고 다음 테이블을 조인해서 읽을 때, 해당 테이블의 PK, Unique key 일 경우 (1건만 읽을 때) eq\_ref 가 표시됩니다. 즉, 효율이 좋습니다. |
| ref | eq\_ref 와 비슷하지만, PK, Unique key 가 아닌 인덱스에 대한 equals 비교일 때 나옵니다. 여러 개 행에 접근할 가능성이 있으며, 인덱스를 타는 것이기에 효율이 좋습니다. |
| index\_merge | 2개 이상의 인덱스를 활용해서 결과를 만든 후, merge 하는 방식입니다. |
| unique\_subquery | IN 사용 시, PK, UK 를 사용 또는 고유 키를 사용 시 접근하는 방식입니다. PK, UK 기반이므로 효율이 좋습니다. |
| index\_subquery | unique\_sunquery 와 비슷하지만 PK, UK 외 인덱스를 사용합니다. 인덱스를 사용하기에 효율이 좋습니다. |

# key

-   옵티마이저가 선택한 인덱스입니다. 없으면 null 이 표시됩니다.

# key_len

-   인덱스 중에서 몇 바이트까지 사용했는지를 알려줍니다.
-   단일, 다중 인덱스를 사용할 때 key\_len 이 높다는건 조회 조건을 정확하게 입력했을 확률이 높다는 것입니다.
    -   즉, key\_len 이 높다는건 인덱스를 잘 설계한 기준 중의 1가지로 생각해볼 수 있습니다. 물론 해당 쿼리에만 적용될 수 있는 기준입니다. 다른 쿼리를 사용했을 때, 해당 인덱스가 제대로 동작하지 않을수도 있습니다.

# ref

-   type (테이블 접근 방식) 이 ref 라면, 어떤 조회조건이 사용됐는지를 알려줍니다.

##### 사용 쿼리

```
explain select * from team_member tm, team t where tm.team_id = t.team_id and name = 'member-1' ;
```

##### 실행계획 결과

-   아래 예시를 보면 1번째 줄의 ref const ('member-1'), 2번째 줄의 tm.team\_id 가 조회 조건인 것을 알 수 있습니다.

![ref desc](https://blog.kakaocdn.net/dn/cWySdJ/btrSlYV9cAs/PvGHyBn7j3dZbqQ13dZGX1/img.png)

# rows

-   rows 는 비용 산정을 위해 테이블의 row 를 몇 개를 읽어야하는지를 나타내주는 값입니다. 실제 row 값이 아니라 통계에 의한 값이므로 정확하지는 않습니다.
-   처음 읽은 테이블에서 rows 는 전체 row count 를 나타내며, 2번째 테이블부터는 앞에서 읽은 테이블의 1개 row 당 몇 개의 row 에 접근했는지를 알려줍니다.
-   실제 데이터가 아니라 통계이겠지만 처음 테이블을 읽을 때부터 데이터 record 를 적게 읽어들이면 성능이 더 좋을 것 입니다.

##### 실행 쿼리

```
select tm.* from team_member tm, team t where tm.team_id = t.team_id and tm.team_id = 1
```

##### 실행계획 결과

-   아래 예시를 보면 2번째 줄에서 TEAM\_MEMBER rows 2 라는 것은 2줄을 읽었다는 것이고, 첫번째 줄의 rows 1은 TEAM\_MEMBER 에서 읽은 1줄당 1줄을 읽었다는 의미입니다.

![rows-desc](https://blog.kakaocdn.net/dn/GOObn/btrSehDawm5/EJgKQ9sNlHwURl0Tj8KNk0/img.png)

# filtered

-   rows 를 통해 대략적으로 어느정도의 데이터를 가져왔는지 파악했다면, 가져온 데이터 중 조회 조건 후 몇개의 row 가 남았는지를 표시해주는 필드입니다.
-   rows 와 filtered 를 활용해서 데이터 record 횟수가 적은 것을 찾는 것이 성능 향상에 도움이 되는 방법중 1개라고 생각됩니다.
-   예를 들면, 다음과 같습니다.

    1.  rows 100 이고, filtered 가 50% 이라면 100 \* 0.5 = 50 남았다는 의미입니다.
    2.  rows 10 이고, filtered 가 100% 라면 10 \* 1 = 10 건이 남았다는 의미입니다.

    -   동일한 쿼리에서 2번이 데이터를 더 적게 가져오므로 2번을 택했을 때, 성능에 더 유리할 수 있다 생각됩니다. 물론 뒤에 테이블을 조인할 때, 상황에 따라 아닐수도 있습니다.

# Extra

-   옵티마이저가 쿼리를 어떻게 실행했는지에 대한 정보를 표시해주는 필드입니다. 중요합니다.

| Extra | 설명 |
| --- | --- |
| Using index | 인덱스에서만 접근해서 데이터를 가져오는 것을 의미합니다. 커버링 인덱스라 하며, 성능이 좋습니다. [(커버링 인덱스)](https://insanelysimple.tistory.com/411) |
| Using filesort | using filesort 는 정렬한 것을 의미합니다. 추가적으로 MySQL 의 경우 GROUP BY 를 수행한 후, 정렬이 이루어집니다. 정렬이 필요할 수도 있지만 필요 없을 수도 있습니다. 정렬을 할 때, 인덱스 순서대로 정렬이 되면 빠르게 정렬이 되며, filesort 를 사용하지 않지만 만약 그렇지 않을 경우 성능이 안좋을 수 있습니다. |
| Using where | 추가적인 필터링 작업을 수행했을 때 나타납니다. ex) using index condition; using where |
| Using temporary | 쿼리가 수행되는 동안 결과를 담아 두기 위해 임시테이블을 사용한 것을 의미합니다. |
| Using index for group-by | Using index와 유사하지만 GROUP BY가 포함되어 있는 쿼리를 커버링 인덱스로 해결할 수 있음을 나타낸다 |
| Using join buffer | 조인을 할 때, 조인조건에 인덱스가 없으면 조인버퍼를 사용합니다. ex) A, B 테이블이 있고, A 테이블을 먼저 읽은 후, B 테이블을 조회 조건을 통해 읽을려고 하는데 조회 조건이 인덱스가 없다면 조인버퍼를 사용합니다. |
| Using index condition | ICP 라고도 하며, MySQL 이 인덱스를 사용하여 행을 검색할 때 최적화 기법입니다. 해당 기법이 적용되기 전에 스토리지 엔진은 테이블에서 rows 를 찾아 MySQL 서버로 전달을 합니다. MySQL 서버에서는 where 조건을 스토리지 엔진으로 전달합니다. 스토리지 엔진은 해당 조건을 통해 데이터 records 를 읽습니다. ICP 가 적용된다면 where 조건에 대해서 서로 데이터를 주고 받지 않고 스토리지 엔진에서 조건을 적용해서 데이터를 다 읽어들인 후 리턴합니다. 즉, 불필요한 과정이 사라지기에 성능이 좋습니다. |

# Reference

-   [https://jeong-pro.tistory.com/243](https://jeong-pro.tistory.com/243)
-   [https://cheese10yun.github.io/mysql-explian/](https://cheese10yun.github.io/mysql-explian/)
-   [https://myinfrabox.tistory.com/84](https://myinfrabox.tistory.com/84)



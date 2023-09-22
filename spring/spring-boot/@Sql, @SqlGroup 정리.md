> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.

## @Sql annotation 이란?
- SQL 스크립트 혹은 쿼리를 실행시킵니다. 주로 테스트 클래스, 메소드에 사용됩니다.
- 쉽게 말해 테스트 환경에서 데이터를 CRUD 할 수 있는 방법입니다.
- 아래 예제는 JPA DDL-AUTO: update 로 켜져있기에 따로 DDL 문은 작성하지 않았습니다.

## @SqlGroup 이란?
- @Sql 을 여러개 그룹화 해서 사용할 수 있습니다. 자세한건 아래 예제 소스를 참고하시면 됩니다.

## source
```java

    @Sql(
            scripts = {"/sql/MEMBER_INSERT_DML.sql"},
            config = @SqlConfig(
                    dataSource = "dataSource",  // 데이터 소스를 설정합니다.
                    transactionManager = "transactionManager"   // 트랜잭션 매니저를 설정합니다.
            ),
            executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD  // 테스트 메소드 시작 전에 동작합니다.
    )
    @Test
    public void sqlAnnotationTest() throws Exception {
        Member member = memberRepository.findById(1L).get();
        Assert.assertThat(member.getName(), is("test"));
    }

    @SqlGroup({
            @Sql(
                    scripts = {"/sql/MEMBER_INSERT_DML.sql"},
                    config = @SqlConfig(
                            dataSource = "dataSource",  // 데이터 소스를 설정합니다.
                            transactionManager = "transactionManager"   // 트랜잭션 매니저를 설정합니다.
                    ),
                    executionPhase = Sql.ExecutionPhase.BEFORE_TEST_METHOD  // 테스트 메소드 시작 전에 동작합니다.
            ),

            @Sql(
                    scripts = {"/sql/MEMBER_DELETE_DML.sql"},
                    config = @SqlConfig(
                            dataSource = "dataSource",  // 데이터 소스를 설정합니다.
                            transactionManager = "transactionManager"   // 트랜잭션 매니저를 설정합니다.
                    ),
                    executionPhase = Sql.ExecutionPhase.AFTER_TEST_METHOD  // 테스트 메소드 끝나고 동작합니다.
            )
    })
    @Test
    public void sqlGroupAnnotationTest() throws Exception {
        Member member = memberRepository.findById(1L).get();
        Assert.assertThat(member.getName(), is("test"));
    }
}
```

```sql
// /sql/MEMBER_INSERT_DML.sql
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES (1, 'test');
INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES (2, 'test2');

// /sql/MEMBER_DELETE_DML.sql
delete from MEMBER;
```

## 활용방안 및 결론
- reference 에서도 나왔지만 spring-batch 테스트에서 데이터가 롤백이 안되서 데이터를 지우기 위해 사용할 수 있습니다.
  - 예를 들면, Test 1 에서는 Member 를 auto increment 로 생성하고, Test 2 에서는 INSERT INTO MEMBER 구문으로 미리 MEMBER_ID 를 1로 생성해놓으면 Test 1 에서 PK 제약 사항 에러가 발생할 수 있습니다.
- dataSource 나 TransactionManager 를 설정하여 서로 다른 DataSource 가 묶이는지도 테스트가 가능합니다. 예를 들면, ChainedTranactionManager 가 있습니다.

```java
@Sql(
            scripts = {xxx},
            config = @SqlConfig(transactionManager = "CHAINED_TRANSACTION_MANAGER")
)
```

- 기본적으로는 EntityManager 나 Repository 를 이용해서 데이터를 CRUD 하는게 좋다 생각되며, 위와 같은 상황일 때, @Sql , @SqlGroup 을 사용하면 좋을 것 같습니다.

## Reference
- [https://cheese10yun.github.io/sql-test/](https://cheese10yun.github.io/sql-test/)
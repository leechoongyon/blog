> source 는 [Github](https://github.com/leechoongyon/spring-boot-jpa-example) 에 있습니다.

## spring data jpa ID 생성전략에 따른 동작 방식 정리
- Entity ID 생성전략에 따른 동작 방식을 정리했습니다.


## ID 생성 전략이 없을 때, 즉 애플리케이션에서 채번할 때
- 아래 테스트를 수행할 경우 애플리케이션에서 ID 를 생성해줬기에 DB 에 값이 있나 확인하기 위해 select 를 날려본 후에, INSERT 가 이루어집니다.
- 즉, merge 방식으로 동작합니다.

```text
Hibernate: 
    select
        nogenerati0_.id as id1_5_0_,
        nogenerati0_.name as name2_5_0_ 
    from
        no_generative_strategy nogenerati0_ 
    where
        nogenerati0_.id=?
```

```java

    @Test
    public void ID_생성전략이_없을떄_동작방식_테스트() throws Exception {
        // given
        NoGenerativeStrategy noGenerativeStrategy =
                NoGenerativeStrategy.builder()
                        .id(2L)
                        .name("test")
                        .build();

        // when
        noGenerativeStrategyRepository.save(noGenerativeStrategy);
        Optional<NoGenerativeStrategy> result = noGenerativeStrategyRepository.findById(2L);

        // then
        Assert.assertThat(result.get().getName(), is("test"));
    }


```

```java

@Entity
@Getter
@NoArgsConstructor
public class NoGenerativeStrategy {
    @Id
    private Long id;

    @Column
    private String name;

    @Builder
    public NoGenerativeStrategy(Long id, String name) {
        this.id = id;
        this.name = name;
    }
}

```


## ID 생성 전략이 존재할 때

#### GenerationType.IDENTITY
- IDENTITY 는 id 값을 세팅하지 않고, insert 쿼리를 날리면 database 시스템에서 자동으로 채번을 합니다.
- AUTO_INCREMENT 라고도 하며, INSERT 쿼리가 실행된 이후에야 ID 값을 알 수 있습니다.
- id 값 생성에 대해서 database 에서 관여하기에 save 메소드를 수행시 persist 로 동작하게 됩니다
    - select 하지 않고 바로 insert 하는 동작  
- database vendor 에 의존적이며, MySQL, PostgreSQL 등에서 지원합니다.


```text

Hibernate: 
    insert 
    into
        identity_strategy
        (id, name) 
    values
        (null, ?)

```

```java

@RunWith(SpringRunner.class)
@DataJpaTest
public class IdentityStrategyRepositoryTest {

    @Autowired
    private IdentityStrategyRepository identityStrategyRepository;

    @Test
    public void IDENTITY_생성전략일_경우_동작방식_테스트() throws Exception {
        // given
        IdentityStrategy identityStrategy =
                IdentityStrategy.builder()
                        .name("test")
                        .build();

        // when
        IdentityStrategy result = identityStrategyRepository.save(identityStrategy);

        // then
        Assert.assertThat(result.getId(), is(1L));
    }
}


```

```java

@Entity
@Getter
@NoArgsConstructor
public class IdentityStrategy {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String name;

    @Builder
    public IdentityStrategy(String name) {
        this.name = name;
    }
}

```

#### GenerationType.SEQUENCE
- database 의 sequence 기능을 사용해서 채번하는 기능입니다.
- 동작방식은 다음과 같습니다.
    - persist 를 호출하면 (spring data jpa 의 경우 save) sequence 를 가져옵니다.
    - 가져온 Sequence 를 id 에 할당하고 (영속성 상태), transaction 이 commit 될 때, insert 쿼리를 날립니다.
- @SequenceGenerator : DB Sequence 를 맵핑합니다. Sequence 관련된 설정도 가능합니다.


```java

@RunWith(SpringRunner.class)
@DataJpaTest
public class SequenceStrategyRepositoryTest {

    @Autowired
    private SequenceStrategyRepository sequenceStrategyRepository;

    @Test
    public void 시퀀스_전략생성_방식_테스트() throws Exception {
        // given
        SequenceStrategy sequenceStrategy = SequenceStrategy.builder()
                .name("test")
                .build();

        SequenceStrategy sequenceStrategy2 = SequenceStrategy.builder()
                .name("test22")
                .build();

        // when
        SequenceStrategy result1 = sequenceStrategyRepository.save(sequenceStrategy);
        SequenceStrategy result2 = sequenceStrategyRepository.save(sequenceStrategy2);

        // then
        Assert.assertThat(result1.getId(), is(1L));
        Assert.assertThat(result2.getId(), is(2L));
    }
}

@Entity
@Getter
@NoArgsConstructor
@SequenceGenerator(
        name="TEST_SEQUENCE_GENERATOR", // 제너레이터명
        sequenceName="TEST_SEQUENCE", // 시퀀스명
        initialValue= 1, // 시작 값
        allocationSize= 1 // 할당할 범위 사이즈
)
public class SequenceStrategy {

    @Id
    @GeneratedValue(strategy=GenerationType.SEQUENCE, generator="TEST_SEQUENCE_GENERATOR")
    private Long id;

    @Column
    private String name;

    @Builder
    public SequenceStrategy(String name) {
        this.name = name;
    }
}

```

###### allocationSize 관련 정리
- @SequenceGenerator 를 통해 allocationSize 를 설정할 수 있는데, allocationSize 의 용도는 성능 최적화 용도입니다.
- allocationSize 는 기본 값이 50이며, 다음과 같이 동작하게 됩니다.
- 아래와 같이 1부터 시작해서 50만큼 증가하는 시퀀스를 만든다고 가정하겠습니다.
- allocationSize 를 50으로 설정하면 persist 를 호출할 때, 시퀀스를 두번 호출합니다.
    - 한 번은 1을 가져오고 또 한 번은 51을 가져오게 됩니다.
- 이렇게 가져온 값을 가지고 내부에서 id 에 자동으로 맵핑해줍니다. (DB 쿼리 호출이 일어나지 않기에 네트워크 리소스가 감소하지 않습니다.)  
- 이러한 방식으로 성능 최적화를 할 수 있고, 만약 Sequence 의 증가 값이 1이라면 allocationSize 는 1이여야 합니다.
- 또한, 주의해야할게 1, 51 을 database 로부터 가져왔는데 애플리케이션에서 30번까지 채번할 때 에러가 나버리면 공백이 생깁니다.

```sql

CREATE SEQUENCE TEST_SEQ START WITH 1 INCREMENT BY 50;   

```
   

#### GenerationType.Table 
- 키 생성 전용 테이블을 만들어서 시퀀스처럼 동작하게 하는 방식입니다.
- 모든 데이터베이스에서 사용할 수 있지만 문제점이 있습니다.
- Sequence 나 Identity 방식은 하나의 Request 로 처리가 가능하지만 테이블 생성전략은 3개의 스텝이 필요합니다.
- lock 을 잡고, seq 를 증가시키고 데이터베이스에 저장합니다. (리소스 소모)
- 그렇기에 운영에서 잘 안씁니다.


## 결론
- 생성전략이 존재하지 않을 경우 merge 로 동작합니다.
    - merge 는 입력받은 entity 를 복사한 후 진행하기에 리소스 소모가 있습니다.
- 생성전략이 존재하는 경우 persist 로 동작합니다.
    - IDENTITY, SEQUENCE 등을 database vendor 에 맞게 사용하면 됩니다. 

## reference
- https://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.htmlhttps://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.html
- https://vladmihalcea.com/why-you-should-never-use-the-table-identifier-generator-with-jpa-and-hibernate/
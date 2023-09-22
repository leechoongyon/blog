> source 는 [Github](https://github.com/leechoongyon/spring-boot-jpa-example) 에 있습니다.

## spring data jpa 에서 데이터 생성일, 수정일 자동 생성
- spring data jpa 에서 데이터 생성일, 수정일 자동 생성에 대해서 정리했습니다.
- 데이터를 생성하거나 변경할 떄, 매번 컬럼을 생성하고 세팅할 필요 없이 자동으로 하는 방법입니다.


## 사용방법

#### @EnableJpaAuditing
- Application 에 @EnableJpaAuditing 를 붙여줘야 합니다.
  - Jpa Auditing 를 활성화시키는 annotation 입니다.
  - spring data jpa 에서 audit 은 시간을 자동으로 넣어주는 기능입니다.

```java
@EnableJpaAuditing
@SpringBootApplication
public class JpaExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(JpaExampleApplication.class, args);
    }
}
```

#### 공통 클래스 만들기
- 아래와 같이 공통 클래스를 만들어줘서 Entity 에서 상속을 받기만 하면 자동으로 생성시간, 수정시간을 세팅하도록 할 수 있습니다.

```java

@MappedSuperclass   // Entity 클래스가 BaseTime 을 상속받을 때, createdDate, modifiedDate 를 인식할 수 있도록 하는 설정입니다.
@EntityListeners(AuditingEntityListener.class)  // 자동으로 값을 넣어주도록 하는 annotation 입니다. 
@Getter
public abstract class BaseTime {

    @CreatedDate    // 데이터 생성할 때 시간 자동 생성 
    private LocalDateTime createdDate;

    @LastModifiedDate   // 데이터 수정할 때 시간 자동 수정
    private LocalDateTime modifiedDate;
}

```

## 사용 예시
- 아래와 같이 BaseTime 을 상속받기만 해도 자동으로 생성, 수정 시간이 적용됩니다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString
public class AutoCreateTime extends BaseTime {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column
    private String test;

    @Builder
    public AutoCreateTime(String test) {
        this.test = test;
    }
}
```

## 테스트
- 테스트 시 주의사항으로는 SpringBootTest 로 테스트 해야하는 것입니다.
- SpringBootTest annotation 을 붙인 이유는 Application 단의 @EnableJpaAuditing 옵션이 동작하기 떄문입니다.
- @DataJpaTest 로 테스트를 했을 때, 에러는 안나지만 자동으로 생성, 수정 시간이 적용되지 않습니다. 

```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class AutoCreateTimeRepositoryTest {

    @Autowired
    private AutoCreateTimeRepository autoCreateTimeRepository;

    @Test
    public void 생성시간_수정시간_자동생성_확인_테스트() throws Exception {
        AutoCreateTime autoCreateTime = AutoCreateTime.builder()
                .test("test")
                .build();
        AutoCreateTime result = autoCreateTimeRepository.save(autoCreateTime);

        Assert.assertNotNull(result.getCreatedDate());
        Assert.assertNotNull(result.getModifiedDate());
    }
}

```

## ZonedDateTime 설정 하는 방법
- 아래와 같이 annotation 을 달리 주면 ZonedDateTime 으로 생성이 됩니다.
- ZonedDateTime 은 는 타임존, 시차 개념이 필요한 날짜와 시간 정보를 나타내기 위해 사용됩니다.
  - [참고](https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html)

```java
@MappedSuperclass   // Entity 클래스가 BaseTime 을 상속받을 때, createdDate, modifiedDate 를 인식할 수 있도록 하는 설정입니다.
@EntityListeners(AuditingEntityListener.class)  // 자동으로 값을 넣어주도록 하는 annotation 입니다. 
@Getter
public abstract class BaseTime {

    @CreationTimestamp
    private ZonedDateTime createdAt;
    
    @UpdateTimestamp
    private ZonedDateTime updatedAt;
}



```

## 결론
- 자동으로 생성, 수정 시간을 적용함으로써 신경써야할 것에 대해 덜 신경쓰게 되고 코드도 깔끔해집니다.

## reference
- https://velog.io/@conatuseus/2019-12-06-2212-%EC%9E%91%EC%84%B1%EB%90%A8-1sk3u75zo9
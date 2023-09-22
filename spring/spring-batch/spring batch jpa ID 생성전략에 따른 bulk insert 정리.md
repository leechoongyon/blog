> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.

## spring batch jpa 환경에서 ID 생성전략에 따른 bulk insert 하는법
- spring batch jpa 환경에서 bulk insert 하는법을 정리했습니다.
- jpa ID 생성전략에 따라 bulk insert 하는법을 정리해봤습니다.


## 애플리케이션에서 ID 를 채번하는 방법
- 애플리케이션에서 ID 를 채번해서 repository.save(xxx) 를 호출하면 merge 방식으로 동작합니다.
- merge 의 상세 동작은 1차 캐시에서 엔티티를 찾게되고, 데이터가 없으면 DB 를 조회하게 됩니다.
- 그런 후 데이터를 반환할 때 복사본을 만들게 되는데 약간의 리소스를 소모하게 됩니다.
    - 복사본을 만드는 이유는 동시성 때문입니다.
    - 만약 하나의 Entity 에 여러 thread 가 접근하게된다면 동시성 접근에 대해 고려해야하기에 복잡해지고 문제가 생길 수 있기 때문입니다.
- 다시 정리하면 애플리케이션에서 ID 를 채번해서 저장하면 select, insert 쿼리가 발생되기에 효율적이지 못합니다.


## sequence ID 생성전략 방식을 이용해서 bulk insert
- sequence 를 지원하는 database 에서 해당 방법을 사용할 수 있습니다.
- sequence 를 이용한 ID 생성방식은 다음과 같은 동작을 합니다. (spring data jpa 를 기준으로 설명합니다)
    - save(), savelAll() 를 호출하면 persist 를 호출하게 되고, sequence 를 가져오게 됩니다. 
    - 가져온 sequence 를 id 에 할당하고 (영속성 상태), transaction 이 commit 될 때, insert 쿼리를 날립니다.
- 한 transaction 내에서 100번의 repository.save(xxx) 를 호출했다고 가정하면 다음과 같이 동작합니다.
    - 100번의 sequence 를 호출합니다. (allocationSize = 1, sequence increment 1 이라 가정)
    - 가져온 sequence 를 id 에 세팅합니다.
    - transaction 이 커밋될 때, insert 쿼리가 100번 날아갑니다.
- source 는 아래와 같습니다.    
    - source 를 설명하면 총 10000개의 데이터를 읽게 되고 chunk size  가 100이기에 100번씩 읽습니다.
    - writer 부분에서 총 100번의 sequence 를 호출하게 되고 이를 ID 에 할당합니다.
    - writer 가 종료되고 update 를 호출 후, transaction manager 가 transaction commit 을 하면서 100번의 insert 쿼리가 호출됩니다.
- 이 때, jdbc.batch_size 옵션과 아래 동작은 관련이 없습니다. 하지만 sequence 생성 방식에서 jdbc.batch_size 옵션을 사용하면 성능 향상을 기대해 볼 수 있습니다. 
- 채번을 배치 처리한 후, insert 를 배치 처리하니 성능 향상을 기대해 볼 수 있습니다.    


```text

// 1 cycle 
// sequence 100번 호출
Hibernate: 
    call next value for test_sequence

// insert 쿼리 100번 호출
Hibernate: 
    insert 
    into
        sequence_strategy
        (name, id) 
    values
        (?, ?)

// 위에 사이클을 100번 반복함.


```

```java

@RunWith(SpringRunner.class)
@SpringBatchTest
@SpringBootTest(classes={SequenceStrategyBulkInsertJob.class, SequenceStrategyBulkInsertBean.class, TestConfig.class})
public class SequenceStrategyBulkInsertJobTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    /**
     * 채번을 먼저 배치 처리해서 불러오고, 그 뒤에 insert 를 배치 처리.
     * @throws Exception
     */
    @Test
    public void SEQUENCE_생성전략_BULK_INSERT_테스트() throws Exception {
        long start = System.currentTimeMillis();
        JobExecution jobExecution = jobLauncherTestUtils.launchJob();
        Assert.assertThat(jobExecution.getStatus(),  is(BatchStatus.COMPLETED));
        Assert.assertThat(jobExecution.getExitStatus(),  is(ExitStatus.COMPLETED));
        System.out.println("elapsed time : " + (System.currentTimeMillis() - start) );
    }
}

```

```java

@Configuration
public class SequenceStrategyBulkInsertJob {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    private final int CHUNK_SIZE = 100;

    private SequenceStrategyBulkInsertBean sequenceStrategyBulkInsertBean;

    public SequenceStrategyBulkInsertJob(SequenceStrategyBulkInsertBean sequenceStrategyBulkInsertBean) {
        this.sequenceStrategyBulkInsertBean = sequenceStrategyBulkInsertBean;
    }

    @Bean
    public Job sequenceStrategyBulkInsertJob01() {
        return jobBuilderFactory.get("sequenceStrategyBulkInsertJob01")
                .start(sequenceStrategyBulkInsertStep01())
                .build()
                ;
    }

    @Bean
    public Step sequenceStrategyBulkInsertStep01() {
        return stepBuilderFactory.get("sequenceStrategyBulkInsertStep01")
                .<SequenceStrategy, SequenceStrategy>chunk(CHUNK_SIZE)
                .reader(sequenceStrategyBulkInsertBean)
                .writer(sequenceStrategyBulkInsertBean)
                .build();
    }
}

```

```java

@Slf4j
@Component
@StepScope
@RequiredArgsConstructor
public class SequenceStrategyBulkInsertBean implements ItemReader<SequenceStrategy>, ItemWriter<SequenceStrategy> {

    private final SequenceStrategyRepository sequenceStrategyRepository;

    // Thread not safe
    private int readCount = 10000;

    private int currentCount = 0;

    @Override
    public SequenceStrategy read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
        if (currentCount == readCount) {
            return null;
        } else {
            currentCount++;
            return new SequenceStrategy("test");
        }
    }

    @Override
    public void write(List<? extends SequenceStrategy> items) throws Exception {
        sequenceStrategyRepository.saveAll(items);
    }
}


```


#### 위 Sequence ID 생성방식을 가지고, jdbc.batch_size 옵션을 적용하고 안했을 때의 성능 테스트를 간단히 진행해봤습니다. 
- 주의사항으로는 batch 의 chunk size 와 jdbc.batch_size 는 일치하는게 효율이 좋습니다.
- 왜냐하면 보통 chunk size 만큼 데이터를 읽어서 쓰기를 진행할텐데 이 2개의 수치가 일치한다면 한 번에 보낼 수 있기 때문입니다.


```text
// 1만건의 데이터 sequence 생성방식, jdbc.batch_size : 100, chunk size : 100, h2 in memory db 
elapsed time : 6696

// 1만건의 데이터 sequence 생성방식, jdbc.batch_size : X, chunk size : 100, h2 in memory db
elapsed time : 7872

```


## IDENTITY 방식으로 bulk 처리
- IDENTITY 방식은 AUTO_INCREMENT 기능이며, 이를 지원해주는 database 가 있습니다. (MYSQL, PostgreSQL 등)
- AUTO_INCREMENT 의 동작 방식은 database 에서 채번해주기에 결국 database 에 접근해야 채번이 가능합니다.
- 즉, insert 쿼리가 동작해야만 seq 를 채번할 수 있는데 그렇게 동작하지 않으니 identity 방식은 jdbc batch 처리가 불가능합니다.
- jpa 는 지연쓰기 방식으로 동작하게 돼있는데 flush 한 후 배치 insert 가 되기 위해선 ID 가 세팅이 되어야 합니다. 왜냐하면 영속성 상태는 식별자 값을 반드시 가지고 있어야 하기 때문입니다.
- IDENTITY 방식은 insert 가 실행이 되어야 ID 값을 알 수 있기에 결국 batch 처리가 안됩니다.



## 위 ID 생성전략에 따른 생성방식과 별개로 jdbc 사용해서 bulk insert
- jdbc 를 이용해서 addBatch 해서 bulk 처리하는 방식입니다.
- jdbc 를 이용하는 것이기에 사용할 때마다 쿼리를 작성해줘야 합니다. 또한, 채번을 할 때도 개발자 본인이 어떻게 해야할지 고민해야 합니다.
- spring data jdbc 를 사용하면 조금 편리할 수도 있으나 sequence 를 통해 채번하는 것보다 성능이 그렇게 크게 차이가 나지 않습니다.
- 성능이 월등히 나아진다면 불편함을 감수하고 jdbc 를 활용해서 bulk 처리를 하겠으나 그런게 아니기에 그냥 sequence 지원하는 database 에서는 spring data jpa sequence 방식을 쓰는게 좋을 것 같습니다.
- 만약, sequence 를 지원안하는 database 라면 jdbc 방식도 고려해보면 좋을 것 같습니다.    


## reference
- https://homoefficio.github.io/2020/01/25/Spring-Data%EC%97%90%EC%84%9C-Batch-Insert-%EC%B5%9C%EC%A0%81%ED%99%94/
- https://stackoverflow.com/questions/27697810/why-does-hibernate-disable-insert-batching-when-using-an-identity-identifier-gen/27732138#27732138
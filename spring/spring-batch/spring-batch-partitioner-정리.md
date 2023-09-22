> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.

## spring batch partitioner 정리

## partitioner 개념
- partitioner 는 하나의 jvm 내에서 멀티 스레드로 처리하는 방법입니다.
- partitioner 에서 데이터를 나누고 각 스텝에 나눈 데이터를 분배하여, 스텝에서는 분배받은 데이터를 가지고 프로그램을 수행합니다.
- 스레드 측면에서 보면 partitioner 라는 스레드가 데이터를 나눠서 각 스텝에 나눠주는데, 이 스텝들은 별도의 스레드에서 동작하게 됩니다.
  - 즉, ItemReader, ItemWriter 가 thread-safe 하지 않아도 됩니다.
  - Multi thread step 의 경우 ItemReader, ItemWriter 는 thread-safe 해야 합니다.
  - Multi thread step 의 경우 step 의 chunks 를 여러 스레드가 번갈아가며 처리하는 구조이기 때문입니다.

## 장단점
- partitioner 의 장점은 데이터를 분배하고 멀티 스레드로 처리하기에 속도가 보통 잘나옵니다.
    - 하지만 속도가 반드시 잘나오는 경우는 아니기에 partitioner 전후 성능을 따져서 확인해봐야합니다.   
- partitioner 의 단점은 로그가 뒤섞여서 나오기에 로그 찾기가 힘듭니다.
    - 예를 들면, 스레드 3개가 돌고 있을 때, 스레드 2개에서 장애가 났을 경우 로그를 보며 에러를 찾기 힘듭니다.
- partitioner 를 사용한다는 것은 멀티스레드를 사용한다는 의미입니다. 즉, partitioner 로부터 데이터를 분배받는 bean 은 멀티 스레드에 안전하도록 개발을 해야 합니다. 
- 극히 드물지만 partitioner 는 멀티 스레드 환경에서 수행되기에 1개의 스레드가 장애 (Error) 발생 시 다른 스레드에 영향을 미칩니다. 이 점은 알고 있어야 합니다.

## source

```java

@RunWith(SpringRunner.class)
@SpringBatchTest
@SpringBootTest(classes={PartitionerExampleJob.class, PartitionerExampleBean.class, TestConfig.class})
public class PartitionerExampleJobIntegrationTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    public void 파티셔너_통합_테스트() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                .addString("test", "testData123")
                .toJobParameters();

        JobExecution jobExecution = jobLauncherTestUtils.launchJob(jobParameters);
        Assert.assertThat(jobExecution.getStatus(),  is(BatchStatus.COMPLETED));
        Assert.assertThat(jobExecution.getExitStatus(),  is(ExitStatus.COMPLETED));
    }
}

@Slf4j
public class SimplePartitioner implements Partitioner {
    @Override
    public Map<String, ExecutionContext> partition(int gridSize) {

        Map<String, ExecutionContext> result = new HashMap<>();

        /** 데이터 분배. 0~100 의 데이터를 gridSize 만큼 나눠서 분배.
         *  gridSize 가 3이라면 3번의 step 을 호출.
         *
         * */
        int start = 0;
        int end = 100;
        int dataSize = (end - start) / gridSize + 1;

        int startPos = start;
        int endPos = startPos + dataSize - 1;
        int num = 1;
        while (startPos <= end) {
            log.info("before data distribution,  startPos : {} , endPos : {}", startPos, endPos);
            ExecutionContext executionContext = new ExecutionContext();
            result.put("part" + num, executionContext);

            /** endPos 이 end 를 넘어설 경우 처리 */
            if(endPos >= end) {
                endPos = end;
            }

            executionContext.putInt("start", startPos);
            executionContext.putInt("end", endPos);


            startPos += dataSize;
            endPos += dataSize;

            num++;

            log.info("after data distribution,  startPos : {} , endPos : {}", startPos, endPos);
        }
        return result;
    }
}


@Slf4j
@Component
@StepScope
public class PartitionerExampleBean implements ItemStream, ItemReader<String>, ItemWriter<String> {

    private int start;
    private int end;

    @Override
    public String read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
        return null;
    }

    @Override
    public void open(ExecutionContext executionContext) throws ItemStreamException {
        start = executionContext.getInt("start");
        end = executionContext.getInt("end");

//        if (start == 34) {
//            log.error("partitioner thread exception test");
//            throw new RuntimeException("partitioner thread exception test");
//        }

        log.info("start : {} , end : {}", start, end);

    }
    
    ...
    ...
}


@Configuration
public class PartitionerExampleJob {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Autowired
    private PartitionerExampleBean partitionerExampleBean;

    @Bean
    public SimplePartitioner simplePartitioner() {
        return new SimplePartitioner();
    }

    @Bean
    public Job partitionerExampleJob01() {
        return jobBuilderFactory.get("partitionerExampleJob01")
                .start(partitionerExampleMasterStep())
                .build();
    }

    @Bean
    public Step partitionerExampleMasterStep() {
        return stepBuilderFactory.get("partitionerExampleMasterStep")
                .partitioner(partitionerExampleSlaveStep().getName(), simplePartitioner())
                .step(partitionerExampleSlaveStep())
                .gridSize(3)
                .taskExecutor(new SimpleAsyncTaskExecutor())
                .build();
    }

    @Bean
    public Step partitionerExampleSlaveStep() {
        return stepBuilderFactory.get("partitionerExampleSlaveStep")
                .<String, String>chunk(100)
                .reader(partitionerExampleBean)
                .writer(partitionerExampleBean)
                .build();
    }

}

```


## 결론
- partitoner 는 하나의 서버에서 하나의 프로세스로 병렬 처리를 할 수 있는 방법입니다.
- 중요하다 생각되는 배치에서는 partitioner 를 써서 개발하기보단 Job 을 여러 프로세스로 수행하여 처리하는 것이 좋을 것이라 생각됩니다.
- 여러 잡을 수행할 때, 데이터를 분배해주고 이를 통해 실행한다면 여러 장점이 있습니다.
    - 멀티 스레드 환경을 고려하지 않아도 됩니다.
    - 하나의 프로세스가 장애가 발생해도 다른 프로세스에는 영향을 미치지 않습니다. 

## reference
- https://howtodoinjava.com/spring-batch/spring-batch-step-partitioning/
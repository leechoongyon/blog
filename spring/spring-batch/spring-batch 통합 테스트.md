> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.

## spring-batch 통합 테스트 
- spring-batch 통합테스트 하는 방법 입니다.
- spring-batch 4.1 이상 기준으로 동작합니다.

## source
- tasklet 을 호출하는 간단한 예제입니다.


```java

@Slf4j
@Configuration
public class SimpleTaskletJob {
    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    private SimpleTaskletBean simpleTaskletBean;

    public SimpleTaskletJob(SimpleTaskletBean simpleTaskletBean) {
        this.simpleTaskletBean = simpleTaskletBean;
    }

    @Bean
    public Job simpleTaskletJob01() {
        return jobBuilderFactory.get("simpleTaskletJob01")
                .incrementer(new RunIdIncrementer())
                .flow(simpleTaskletStep01())
                .end()
                .build();
    }

    @Bean
    public Step simpleTaskletStep01() {
        return stepBuilderFactory.get("simpleTaskletStep01")
                .tasklet(simpleTaskletBean)
                .build();
    }
}


```


```java

/**
 * 통합 테스트 환경 구성 (Tasklet)
 */

@RunWith(SpringRunner.class)
@SpringBatchTest    // a
@SpringBootTest(classes={SimpleTaskletJob.class, SimpleTaskletBean.class, TestConfig.class}) // b
public class SimpleTaskletJobTest {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    @Test
    public void simpleTaskletJobTest() throws Exception {
        JobParameters jobParameters = new JobParametersBuilder()
                        .addString("test", "testData123")
                        .toJobParameters();
        
        JobExecution jobExecution = jobLauncherTestUtils.launchJob(jobParameters);

        Assert.assertThat(jobExecution.getStatus(),  is(BatchStatus.COMPLETED));
        Assert.assertThat(jobExecution.getExitStatus(),  is(ExitStatus.COMPLETED));
    }
}

```

a. Spring Batch 에서 test 관련 bean 을 자동으로 등록해줍니다.  
- 아래 내용을 보면 JobLaucherTestUtils, JobRepositoryTestUtils 등 테스트에 필요한 빈을 등록해줍니다. (편의성)
 
```text
Registers a JobLauncherTestUtils bean with the BatchTestContextCustomizer.JOB_LAUNCHER_TEST_UTILS_BEAN_NAME which can be used in tests for launching jobs and steps.
Registers a JobRepositoryTestUtils bean with the BatchTestContextCustomizer.JOB_REPOSITORY_TEST_UTILS_BEAN_NAME which can be used in tests setup to create or remove job executions.
Registers the StepScopeTestExecutionListener and JobScopeTestExecutionListener as test execution listeners which are required to test step/job scoped beans.
```

b. @SpringBootTest 를 등록해서 필요한 자동설정 등을 자동으로 등록해주면 테스트 할 때 편합니다. (권장)


## 주의사항
- 배치 통합 테스트 환경에서 jpa repository bean 을 못찾는 에러가 발생한 적이 있습니다.
- 아무리 원인을 찾아도 몰랐는데 entityManagerFactory 만들 때, packages 를 설정해주면 repository bean 을 찾아서 해결했습니다.
- TestConfig 클래스에 @ComponentScan("com.batch.config") 를 걸어주면 com.batch.config 패키지 아래에 JpaConfiguration 통해 repository bean 을 등록해줍니다.

```java

public class JpaConfiguration {
        
    private final EntityManagerFactoryBuilder entityManagerFactoryBuilder;

    @Primary
    @Bean(name = "entityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory (DataSource dataSource) {
            return entityManagerFactoryBuilder
                      .dataSource(dataSource)
                      .packages(packages)
                      .properties(properties)
                      .persistenceUnit("testUnit")
                      .build();
    }
}
   
```



## spring-batch-test 에서 rollback 이 안되는 이유
- 결론부터 얘기하면 spring-batch 는 실행되는 각각의 상태나 변수 등을 jobRepository 를 통해 DB 에 저장을 해둡니다.
- 이를 비즈니스 로직을 담당하는 transactionManager 와 묶어버리면 실행되는 상태나 변수 등이 한 번에 rollback 되거나 commit 되기에 rollback 이 안됩니다. 


#### spring-batch jobRepository 
- spring-batch 는 실행하는 종종 배치의 상태와 변수들을 관리하고 있습니다.
- 아래 소스는 AbstractStep 의 일부소스이며, 소스 부분부분을 보면 getJobRepository() 를 가져와서 update 를 하고 있습니다.
- 별도의 트랜잭션 매니저없이 처리를 하기에 autoCommit 으로 동작을 하게 됩니다.
    - 그 이유는 spring-batch 의 순서대로 진행하면서 어떻게 진행이 되는지 추적하기 위함이라 생각합니다. 

```java
@Override
public final void execute(StepExecution stepExecution) throws JobInterruptedException,
UnexpectedJobExecutionException {

	Assert.notNull(stepExecution, "stepExecution must not be null");

	if (logger.isDebugEnabled()) {
		logger.debug("Executing: id=" + stepExecution.getId());
	}
	stepExecution.setStartTime(new Date());
	stepExecution.setStatus(BatchStatus.STARTED);
	Timer.Sample sample = BatchMetrics.createTimerSample();
	getJobRepository().update(stepExecution);

	// Start with a default value that will be trumped by anything
	ExitStatus exitStatus = ExitStatus.EXECUTING;

	doExecutionRegistration(stepExecution);

    ...
    ...
    ...
    
    try {
    getJobRepository().updateExecutionContext(stepExecution);
	} catch (Exception e) {
	    stepExecution.setStatus(BatchStatus.UNKNOWN);
		exitStatus = exitStatus.and(ExitStatus.UNKNOWN);
		stepExecution.addFailureException(e);
		logger.error(String.format("Encountered an error saving batch meta data for step %s in job %s. "
				+ "This job is now in an unknown state and should not be restarted.", name, stepExecution.getJobExecution().getJobInstance().getJobName()), e);
    }        
```




#### spring-batch 비즈니스 로직의 트랜잭션
- 아래 소스를 보면 실제 비즈니스 로직이 시작하는 부분입니다.
- TaskletStep.doInChunkContext 은 비즈니스 로직이 시작하는 부분입니다.
- 비즈니스 로직이 시작하기 전 TransactionTemplate 을 통해 트랜잭션이 시작됩니다.
- 트랜잭션이 시작되고 ChunkedOrientedTasklet 이나 tasklet implements 한 로직이 실행 됩니다. 


```java

@Override
public RepeatStatus doInChunkContext(RepeatContext repeatContext, ChunkContext chunkContext)
		throws Exception {

	StepExecution stepExecution = chunkContext.getStepContext().getStepExecution();

	// Before starting a new transaction, check for
	// interruption.
	interruptionPolicy.checkInterrupted(stepExecution);

	RepeatStatus result;
	try {
		result = new TransactionTemplate(transactionManager, transactionAttribute)
		.execute(new ChunkTransactionCallback(chunkContext, semaphore));
	}
	catch (UncheckedTransactionException e) {
		// Allow checked exceptions to be thrown inside callback
		throw (Exception) e.getCause();
	}

	chunkListener.afterChunk(chunkContext);

	// Check for interruption after transaction as well, so that
	// the interrupted exception is correctly propagated up to
	// caller
	interruptionPolicy.checkInterrupted(stepExecution);

	return result == null ? RepeatStatus.FINISHED : result;
}

```

## 결론
- spring-batch-test 통합 테스트 에서 데이터를 rollback 할려면 데이터를 지우는 쿼리를 수행해줘야 합니다.

## reference
- https://docs.spring.io/spring-batch/docs/current/reference/html/testing.html
- https://jojoldu.tistory.com/455
- https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/test/context/SpringBatchTest.html

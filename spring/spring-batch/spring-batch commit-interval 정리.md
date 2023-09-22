## spring-batch 의 ChunkedOrientedTasklet 방식이란?
- commit-interval 을 알아 보기 전에 ChunkedOrientedTasklet 방식을 알고 넘어 가야합니다. 
- 이 구조는 reader - processor 가 commit-interval 만큼 반복된 후, 가공된 데이터를 writer 에서 bulk 처리하는 구조입니다.
- 이렇게 구조가 나온 이유는 배치 프로그램의 대부분의 형태가 입력 --> 데이터 가공 --> 쓰기 로 되있기에 이런 구조가 나온 것이라 생각됩니다.
- source 를 보면 아래와 같은데 reader - process 가 반복되고 bulk 데이터를 writer 하는 구조입니다.


```java

@Nullable
@Override
public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {

	@SuppressWarnings("unchecked")
	Chunk<I> inputs = (Chunk<I>) chunkContext.getAttribute(INPUTS_KEY);
	if (inputs == null) {
		inputs = chunkProvider.provide(contribution);
		if (buffering) {
			chunkContext.setAttribute(INPUTS_KEY, inputs);
		}
	}

	chunkProcessor.process(contribution, inputs);
	chunkProvider.postProcess(contribution, inputs);

	// Allow a message coming back from the processor to say that we
	// are not done yet
	if (inputs.isBusy()) {
		logger.debug("Inputs still busy");
		return RepeatStatus.CONTINUABLE;
	}

	chunkContext.removeAttribute(INPUTS_KEY);
	chunkContext.setComplete();

	if (logger.isDebugEnabled()) {
		logger.debug("Inputs not busy, ended: " + inputs.isEnd());
	}
	return RepeatStatus.continueIf(!inputs.isEnd());

}

```


## spring-batch commit interval 이란?
- 위에 ChunkedOrientedTasklet 방식을 사용하면 reader-processor 반복하는 것을 commit-interval 변수로 제어할 수 있습니다.
- 좀 더 정확하게 얘기하면 reader 에서 commit-interval 만큼 읽고 그 list 데이터를 processor 에서 처리하고, processor 에서 처리한 데이터를 writer 로 넘겨서 bulk 처리합니다.  
- commit-interval = 100 이라면, reader 를 통해 100번 읽어들이고 그 읽은 데이터 list 를 processor 에서 처리하고, processor 에서 처리한 데이터를 writer 에서 bulk 처리합니다.

## commit-interval 에 따른 영향도
- commit-interval 이 작다는 것은 데이터를 적게 읽어들여 트랜잭션을 여러 번 수행한다는 의미입니다.
- 이는 여러 번 수행되는 트랜잭션의 비용을 고려 해야 합니다.
- commit-interval 이 크다는 것은 한 트랜잭션 안에서 큰 데이터를 처리한다는 것이고, 이는 database 쪽에 rollback operation 에 대한 부하를 주게 됩니다.
    - 예를 들면, oracle 의 경우 rollback 기능을 위해 undo 라는 공간에 데이터를 저장해놓습니다.
    - oracle 에서는 rollback 이 일어날 경우 undo 의 공간을 이용해서 트랜잭션 처리 이전의 시점으로 되돌립니다.

## 결론
- 아래 책을 읽어보면 commit-interval 사이즈는 20~200 이 적당하다고 얘기하고 있습니다.
- 제가 spring-batch 를 써보면서 느낀 생각은 commit-interval 을 통해 최적의 성능구조를 찾고 싶다면 commit-interval 을 변경하면서 테스트 하는 것이 좋다고 생각됩니다. 

## reference
- spring-batch-in-action 책

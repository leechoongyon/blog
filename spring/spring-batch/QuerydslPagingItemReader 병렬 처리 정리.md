> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.

## QuerydslPagingItemReader 병렬 처리 가능한가?
- QuerydslPagingItemReader 는 querydsl 을 이용하여 db 데이터를 읽어올 때, 페이지 단위로 읽어오는 컴포넌트입니다.
  - 상세 설명 [QuerydslPagingItemReader](https://insanelysimple.tistory.com/296)
- 결론부터 얘기하면 QuerydslPagingItemReader 병렬 처리 시, thread-safe 합니다.    
- 아래 소스를 보면 QuerydslPagingItemReader 는 AbstractPagingItemReader 상속받고 있습니다.
- spring-bagch 내부 엔진에서 AbstractPagingItemReader 의 doRead() 를 호출하고, doRead() 내에서 QuerydslPagingItemReader.doReadPage 를 호출합니다.
- doRead 에는 synchronized(this.lock) 을 선언했고, 이 소스의 의미는 해당 메소드에 접근하는 스레드에 대해서 인스턴스 락을 걸겠다는 의미입니다.
- spring-batch 는 QuerydslPagingItemReader 를 Bean 으로 선언해서 싱글톤 빈으로 관리할 것입니다.
- 그렇기에 QuerydslPagingItemReader 인스턴스는 jvm 내에 하나가 존재하며, 멀티 스레드가 접근할 때, 인스턴스 락이 걸려있으므로 thread-safe 할 수 있습니다.

```java
public class QuerydslPagingItemReader<T> extends AbstractPagingItemReader<T> {
          ...
          ...
          ...
}

```

```java

public abstract class AbstractPagingItemReader<T> extends AbstractItemCountingItemStreamItemReader<T> implements InitializingBean {
  @Nullable
  protected T doRead() throws Exception {
    synchronized(this.lock) {
      if (this.results == null || this.current >= this.pageSize) {
        if (this.logger.isDebugEnabled()) {
          this.logger.debug("Reading page " + this.getPage());
        }

        this.doReadPage();
        ++this.page;
        if (this.current >= this.pageSize) {
          this.current = 0;
        }
      }

      int next = this.current++;
      return next < this.results.size() ? this.results.get(next) : null;
    }
  }
}


```


## QuerydslPagingItemReader 병렬 처리 테스트
- 다음 소스는 데이터 10건을 스레드 3개를 이용해서 QuerydslPagingItemReader 로 읽는 것입니다.
- 마지막에 reader 가 읽어들인 데이터 총 count 가 10건인지 검증하는 것이 있습니다. 
- 아래 테스트를 수행하면 정상적으로 10건의 데이터를 읽어들였다는 결과를 볼 수 있습니다. 

```java
private void asyncTest() throws InterruptedException, ExecutionException {
        QuerydslPagingItemReader<Member> reader =
                new QuerydslPagingItemReader<>(entityManagerFactory, pageSize,
                        queryFactory -> queryFactory
                                .selectFrom(QMember.member))
                ;
        reader.open(new ExecutionContext());

        CompletionService<List<Member>> completionService =
                new ExecutorCompletionService<>(Executors.newFixedThreadPool(THREAD_COUNT));
        for (int i = 0 ; i < THREAD_COUNT; i++) {
            completionService.submit(new Callable<List<Member>>() {
                @Override
                public List<Member> call() throws Exception {
                    List<Member> list = new ArrayList<>();
                    Member nextMember = null;
                    do {
                        nextMember = reader.read();
                        logger.debug("item: " + nextMember);
                        if (nextMember != null) {
                            list.add(nextMember);
                        }
                    } while (nextMember != null);
                    return list;
                }
            });
        }
        int count = 0;
        Set<Member> results = new HashSet<>();
        for (int i = 0; i < THREAD_COUNT; i++) {
            List<Member> items = completionService.take().get();
            count += items.size();
            logger.debug("total items count: " + items.size());
            logger.debug("items: " + items);
            assertNotNull(items);
            results.addAll(items);
        }
        assertEquals(TOTAL_COUNT, count);
        assertEquals(TOTAL_COUNT, results.size());
        reader.close();
}


```


## QuerydslNoOffsetPagingItemReader 도 병렬 처리가 가능한가?
- QuerydslNoOffsetPagingItemReader 는 QuerydslPagingItemReader 를 개선한 reader 입니다.
  - 상세 설명 [QuerydslNoOffsetPagingItemReader](https://insanelysimple.tistory.com/296)
- 결론부터 얘기하면 QuerydslNoOffsetPagingItemReader 또한 thread-safe 합니다.
- 왜냐하면 QuerydslNoOffsetPagingItemReader 는 QuerydslPagingItemReader 상속받고 있고, QuerydslPagingItemReader 는 AbstractPagingItemReader 상속받고 있기 때문입니다.
- QuerydslNoOffsetPagingItemReader 테스트 소스는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 을 참고하시면 됩니다.

## reference
- https://woowabros.github.io/experience/2020/02/05/springbatch-querydsl.html
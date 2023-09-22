## [spring boot, spring batch 정리 - 5편 대용량, low latency 를 위해 어떤 ItemReader 가 좋은가? (spring-batch 병렬 처리) ](https://leechoongyon.github.io/posts/springboot-batch-5)

## JdbcCursorItemReader vs JdbcPagingItemReader 
- reference 에 나온 답변을 보면 이용하는 데이터베이스와 처리하는 모델에 따라 달라진다고 하는데 이 말이 이해가 안간다.
    - 추측하기로는 각 Reader 들은 fetchSize 를 통해 데이터를 받아오고 이는 네트워크를 거치는데 특정 DBMS 에서는 이 옵션이 안먹히기 때문이다.
    - pagingItemReader 는 paging 만큼 데이터를 가져와야하는데 fetchSize 옵션이 제대로 설정돼지 않으면 어떻게 동작할지를 모를테고.
    - 마찬가지로 cursorItemReader 도 fetchSize 만큼 가져와야하는데 특정 DBMS 에서 옵션이 작동안할 수 있으니 어떤 것이 성능이 좋은지 판단할려면 테스트 해보는 수밖에 없다.
- 결국 위 내용의 결론은 테스트 환경을 만들어서 테스트 해보고 파악하는 것이 최선이라 생각된다.

## spring-batch parallel option
- 아래 글에 보면 spring-batch 병렬 처리 옵션에 대표적인 것이 2개가 있다고 얘기함.
- Multi Thread Step, Partitioning
- Multi Thread Step 은 구성하기 쉽다고 설명. (TaskExecutor 만 설정해주면 되니)
    - 단, restart 기능을 구현하기에 복잡하다. (chunked 단위가 어떻게 실행될지 감이 안잡히니)
- Partitioning 은 구성하기에 복잡하나 재시작을 보장한다 라고 설명하고 있다.
- 내 생각은 다음과 같다. Multi Process 를 써서 Job 을 실행하는게 가장 best 이지 않을까.
    - Multi Thread 는 restart 기능 이 복잡.
    - Partitioning 기능도 로그 보기가 힘들고, 하나의 스텝이 장애가 나면 다른 스텝에도 영향을 미친다. (하나의 Process 이니) 

## reference
- https://stackoverflow.com/questions/20386642/spring-batch-which-itemreader-implementation-to-use-for-high-volume-low-laten
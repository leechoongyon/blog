> 목차는 [개발자유주제](https://insanelysimple.tistory.com/category/%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%9C%A0%EC%A3%BC%EC%A0%9C) 에 있습니다.



# [개발자유주제 1편] 데이터 동시성 접근 문제가 발생했을 때 해결 방안 (DB lock, Redis)



# 데이터 동시성 접근 문제란?

- 쇼핑몰에서 '맥북' 상품을 3개 팔고 있다고 가정하겠습니다. '맥북' 을 살려고 동시간대에 수십, 수백개의 요청이 들어왔다고 가정을 해보면 동시성 접근에 대한 대처가 안돼있다면 '맥북' 재고는 3개이지만 3개 이상의 주문 처리가 될 수도 있고, 1개만 주문 처리가 될 수도 있습니다.
- 이는 DB 트랜잭션에 대한 문제이며, Oracle 의 경우 READ Commited 로 트랜잭션 격리수준이 기본 값으로 설정돼있습니다. 트랜잭션 격리수준에 관한 것은 상세히 설명한 글들이 많으니 스킵하겠습니다. 
- 이러한 트랜잭션 격리 수준 때문에 데이터를 변경할 때, 독립적으로 수행되는게 아니라 여러 트랜잭션이 같이 실행이 되면서 데이터 정합성이 안맞을 수 있습니다.



# 해결방안 1 (DB Lock)

- DB Lock 을 거는 방법에는 2가지가 있습니다. 낙관적 lock, 비관적 lock





### 비관적 lock

###### 개념

- 트랜잭션이 충돌할 것이라 가정하고 lock 을 설정하는 것입니다.
  - Oracle 의 select for update 와 같은 것이 있습니다. oracle 의 경우 기본 트랜잭션 격리 수준이 READ COMMITED 이지만 for update 를 사용할 경우 REPEATABLE Read 가 가능해집니다.
  - 참고로 접근한 row 에 대해서 lock 을 거는 방식입니다. Table 전체에 걸지 않습니다.



###### 예시

- 예를 들면, 트랜잭션 1, 2 가 있다고 가정하고 MEMBER 테이블의 MEMBER_ID = 5 데이터를 동시에 바꾼다고 가정하겠습니다.
- 트랜잭션 1은 MEMBER_ID = 5 에 대해 NAME 을 TEST1 로 바꾸고, 트랜잭션 2는 MEMBER_ID = 5 에 대해 NAME 을 TEST2 로 바꾼다고 가정하겠습니다.
- 트랜잭션 1 이 먼저 TEST1 로 바꾼다고 가정하면, 트랜잭션 2는 트랜잭션1이 끝날 때까지 MEMBER_ID = 5 에 대해 접근할 수 없습니다.



###### 특징

- 격리 수준을 높여서 데이터 무결성을 보장하는 수준이 높지만, 다른 트랜잭션에 대한 접근을 막아버리기에 성능상 안좋습니다.
- 트래픽이 적은 환경에서는 문제가 안되겠지만 트래픽이 많아질수록 성능상 문제가 발생할 수 있습니다.





### 낙관적 lock

###### 개념

- 비관적 락과는 다르게, 동시성 문제가 발생하면 해결을 하는 방법입니다.

- 즉, 커밋이 이루어지고 동시성 문제가 발생하면 예외가 발생하고 커밋된 것에 대한 롤백 처리가 수행되야 합니다.

   

 ###### 예시

- 예를 들면, 트랜잭션 1, 2 가 있다고 가정하고 MEMBER 테이블의 MEMBER_ID = 5 데이터를 동시에 바꾼다고 가정하겠습니다.
- 트랜잭션 1은 MEMBER_ID = 5 에 대해 NAME 을 TEST1 로 바꾸고, 트랜잭션 2는 MEMBER_ID = 5 에 대해 NAME 을 TEST2 로 바꾼다고 가정하겠습니다.
- 트랜잭션 1 이 먼저 TEST1 로 바꿔서 커밋을 하고, 트랜잭션 2가 뒤를 이어 TEST2 로 바꿔서 커밋을 했습니다. 트랜잭션 2는 동시성 문제가 발생한 것을 감지해서 Exception 이 발생하고, 커밋이 이루어진 것에 대해 롤백 처리를 해줘야 합니다.



###### 특징

- 데이터 동시성 문제가 빈번하게 발생하지 않는 업무에서는 낙관적 lock 이 성능이 좋습니다.
- 데이터 동시성 문제가 빈번하게 발생하는 업무에서는 롤백 처리를 추가로 해야하기에 성능이 안좋습니다.







# 해결방안 2 (Redis)

### 분산 락 (tryLock)

###### 개념

- Redis 에 접근해 lock 을 획득하는 방법입니다.
- 정확하게는 Redis 에 접근해 값이 존재하지 않으면 값을 세팅하고 그에 대한 응답을 받고, 값이 존재하면 마찬가지로 그에 대한 응답을 받습니다.
- 이 때, Lock 을 설정하는 단위는 다양한 기준이 있겠지만 여기서 간단히 예시를 소개하면 다음과 같습니다.
  - 상품과 재고라는 도메인이 있으며, 특정 상품에 대한 재고 관리를 분산락으로 관리하고자 한다면 상품ID 를 Key 로 재고를 Value 로 잡을 수 있습니다.



###### 분산 락 구현 시, 알아두어야 할 점 (Lettuce)

- Redis Client 중 **Lettuce** 이 있습니다.
- Lettuce 를 통해 분산 락을 구현할 경우 락을 획득하는 타임아웃 기능이 없고, 스핀 락 방식입니다.
- 타임아웃 기능이 없으면 예상치 못한 장애 (락을 획득하기 위한 무한루프) 등이 발생할 수 있습니다.
- 스핀 락 방식은 lock 을 획득하기 위해 Redis 에 계속해서 요청을 보내는 것입니다. 이 방식을 사용한다면 락을 획득하기 위해 많은 트래픽이 발생되고 이는 성능 저하를 일으킬 수 있습니다.



분산 락 구현 시, 알아두어야 할 점 (Redisson)

- Redis Client 중 **Redisson** 이 있습니다.
- Redisson 를 통해 분산 락을 구현할 경우 락을 획득하는 타임아웃 기능이 있으며, 락을 획득하는 방식이 pub/sub 입니다. 
- pub / sub 방식은 Client 들이 Redis 를 구독하고 있다가 Redis 에서 Lock release 가 발생하면 구독하고 있는 Client 에게 알림을 줍니다. 알림을 받은 Client 들은 다시 lock 획득을 시도하는 방식이기에 스핀 락 방식에 비해 낫다고 생각됩니다.



### Increment, Decrement 방식

###### 개념

- Redis 의 명령어 중 값을 +-1 하고 값을 반환하는 명령어가 있습니다. 해당 명령어들은 atomic 하게 동작합니다.
- 이런 특성을 이용해 특정 데이터에 대한 수량 (동시 접근 데이터) 등을 관리할 수 있습니다.



###### 예시

- 예를 들면, 특정 상품에 대한 수량이 5개 라면 해당 데이터에 접근해서 수량 관리를 진행할 수 있습니다.
- Redis 는 Single Thread 로 동작하며, atomic command 를 지원하기에 동시성 문제를 해결할 수 있습니다.





# Redis 와 DB Lock 비교

### 성능

- In memory cache 가 DB 보다 성능이 좋습니다. [(참고)](https://www.cybertec-postgresql.com/en/postgresql-vs-redis-vs-memcached-performance/) 
  - 결정적인 차이는 DB 는 Disk 에 접근해서 데이터를 가져오는 것이고, Cache 는 메모리에 접근해서 가져오는 것이기 때문입니다.



### 트래픽 부하에 따른 대안

- DB 의 경우 보통 ROW LOCK 을 걸기에 트래픽이 많아질수록 이에 대한 대안이 당장 떠오르지 않습니다. 공부해서 생각나면 적어놓겠습니다.
- Redis 의 경우 분산락 또는 increment, decrement 방식을 사용할 때, 트래픽이 늘어난다면 Key 를 여러개 만들어서 분리시키는 것도 고려해볼 수 있습니다.
  - 예를 들면, PRODUCT_001 이라는 ID가 있다고 가정하면 PRODUCT_001_A, PRODUCT_001_B, PRODUCT_001_C 이런식으로 키를 여러개 만들어서 사용하는 것입니다. 이렇게 키를 여러개 만들어서 트래픽을 분산시킨다면 트래픽 부하에 어느 정도 대처가 될 것이라 생각됩니다.
  - 물론 이 방법의 전제는 키를 분산시킬 수 있는 조건을 만족해야하는 것입니다.



### 확장성 및 고가용성

- Redis 의 경우 클러스터 기능을 지원하기에 확장성 및 고가용성 기능을 지원합니다.
- RDB 의 경우 DB 확장이 각 DBMS 마다 다르기에 하나하나 열거하지는 않겠지만 DB 자원을 쓰는 것보다 Application 자원을 쓰는게 더 좋다고 생각됩니다.





# 결론

- Redis 를 이용해 동시성 처리를 해야한다면 많은 resource 가 들기에, 트래픽이 적다면 DB 락 방식을 고려해보는 것이 좋을 것 같습니다. 
- 트래픽이 점점 늘어나거나 많아서 DB Lock 방식으로 해결이 힘들다면 Redis 를 이용한 분산락 또는 Increment, Decrement 방식을 사용해보는 것도 좋을 것 같습니다.
- 추후 관련 예제 소스도 작성할 예정입니다.









# Reference

- https://hyperconnect.github.io/2019/11/15/redis-distributed-lock-1.html
- https://pkgonan.github.io/2020/04/stock
- https://redis.io/commands/incr/

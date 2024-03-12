> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



> 공부한 내용을 정리할 목적으로 작성했습니다.



# [DB 16편] MySQL Slave 데이터 분배하는 방법



# Slave Scale out 시, 애플리케이션에서 데이터 분배

- 아래 참고해서 Spring Boot 에서 Slave (Read 전용) 분배하는법 정리
- RoutingDataSource 를 통해 Request 를 분배
  - Transactional(ReadOnly=true) 인 것들을 분배
  - 안 붙은 것은 Write DataSource 로 분배



# 데이터가 많을 경우 Write, Read 분배 방법

- 앞에 Apigateway 나, LB 를 두고 쓰기 요청은 쓰기 Application 에, Read 요청은 Read Application 으로 호출
- Write Application -> Write DataSource -> Read DataSource 복제 sync 진행
- Read Application 에서는 Read DataSource 접근 



# Database 부하 줄이는 방법

- 캐시를 쓸지 고민 -> 읽기 많음. 빠른 응답. 
- Read DataSource (복제본) 에 있는 데이터를 Redis Cluster 와 주기적으로 동기화를 맞춰줌.
- 이렇게 하면 Redis 에 먼저 접근을 해서 데이터를 가져오고, 없으면 Database 에 접근하는 식으로 처리가 가능
- Database 부하 줄일 수 있음



# 참고

- https://velog.io/@ch4570/MySQL-Master-Slave-%EA%B5%AC%EC%A1%B0%EC%97%90%EC%84%9C-Slave%EB%A5%BC-Scale-Out-%ED%95%B4%EB%B3%B4%EA%B8%B0

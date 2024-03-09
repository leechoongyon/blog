> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 8편] MySQL 인덱스 성능 관련 정리 (정렬 및 IO 접근)



- 정렬에는 2가지 방법이 있음
  - 인덱스 스캔
  - file sort

- 인덱스 스캔이 훨씬 빠르겠지. 인덱스는 이미 정렬돼있으니

- 단, 조건이 있음. order by 절과 인덱스 순서가 동일
  - a,b,c key 이면, order by 도 a,b,c 순이여야됨. (데이터 정렬도 같아야 함. asc, desc)
  - 이건 인덱스가 데이터를 저장할 때, 이렇게 저장하기 때문에 그럼.
    - a,b,c 순으로 저장

랜덤 I/O vs 순차 I/O 정리하고 넘어가기


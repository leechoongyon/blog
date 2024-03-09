> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 9편] MySQL 인덱스 성능 관련 정리 (Auto increment, UUID)



# PK Auto increment, UUID 관련 정리

- PK 에 Auto increment 와 UUID 를 사용했을 때 차이는 다음과 같습니다.
  - Auto Increment 를 사용했을 때, 인덱스 데이터 사이즈가 UUID 보다 작습니다.
  - Auto Increment 를 사용했을 때, INSERT 속도가 더 빠릅니다.



### 왜 Auto increment 가 INSERT 속도와 인덱스 데이터 사이즈가 작은가?

- PK 로 Auto increment 를 사용할 경우 클러스터드 인덱스는 1,2,3,4,5 순으로 차례대로 저장이 될 것 입니다. 클러스터드 인덱스 특성상 데이터는 인접하게 저장이 될 것이고, 인접하게 데이터가 저장이 된다는 것은 인접한 페이지에 데이터가 저장된다는 것입니다.
- 예를 들어, 1,2,3,4,5 순으로 데이터를 페이지에 저장하다가 페이지가 꽉 차면 6,7,8,9,10 은 바로 옆의 페이지에 저장이 될 것 입니다. 그러면 디스크 탐색에 사용되는 resource 가 그리 크지 않으며, 데이터 공간도 UUID 보다 적게 사용이 가능합니다.





### UUID 데이터 사이즈나 INSERT 성능을 개선하는 방법?

- 다음 링크를 [참고](https://insanelysimple.tistory.com/361) 하면 UUID 를 기본키로 설정했을 때, 성능을 개선하는 방법이 있습니다.
- 나중에 글로벌 Key 를 PK 로 잡을 때, 위 내용을 응용하면 성능과 데이터 사이즈를 개선할 수 있을 것이라 생각됩니다.





# 결론

- 데이터를 insert 할 때는 기본키 순서로 데이터를 넣는 것이 좋습니다.
  - 데이터가 인접하게 쌓이고 이는 데이터 사이즈가 줄고, INSERT 성능이 좋기 때문입니다.

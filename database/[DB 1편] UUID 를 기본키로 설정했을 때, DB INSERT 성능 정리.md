> 목차는 [DB 목차](https://insanelysimple.tistory.com/category/database) 에 있습니다.



# [DB 1편] UUID 를 기본키로 설정했을 때, DB INSERT 성능 정리



## UUID 를 기본키로 설정했을 때, DB INSERT 성능 정리

- InnoDB 를 기준으로 정리했습니다.
- UUID 는 36 무작위 문자로 이루어졌습니다.
- InnoDB 의 특성상 데이터를 저장할 때, 기본키로 정렬해서 저장을 합니다. (기본키가 유사하다면 물리적인 위치도 유사합니다.)
  - 이것을 클러스터링이라 하며, 범위 검색을 할 경우 빠른 시간내에 찾을 수 있습니다.
- 그렇기에 UUID 를 기본키로 설정하는 것은 INSERT 할 떄, 물리적으로 데이터가 흩어져서 저장이 될 수 밖에 없습니다.
- 이는 다음과 같은 영향을 미칩니다.
  - 무작위로 데이터를 저장하기에 Index 나 파일 사이즈가 커질 수 밖에 없습니다.
  - 왜냐하면 Index 나 데이터 또한 물리적인 공간에 저장이 되야하는데 데이터가 흩어져서 저장이 되면 파일 사이즈가 커질 수 밖에 없는 것 같습니다.
  - 운영 체제 시간에 배운거 같은데 데이터 블록이 꽉차서 옆에 데이터 블록에 이어서 써야하는데 물리적인 위치로 바뀌다 보니 기존 블록에 빈 공간들이 많이 남는 걸로 이해했습니다. (틀릴 수 있습니다)
  - 이렇게 무작위로 저장이 되면 PK 를 통해 조회를 할 때도, 시간이 많이 소요될 수 밖에 없습니다. 


## Ordered UUID, BigInt 를 이용한 데이터 쓰기
- UUID 를 36 글자를 다 쓸 필요 없으니 압축을 시킵니다. 대쉬는 크게 의미없으니 삭제해도 괜찮습니다.
- 이렇게 압축을 하고 sorting 했을 때, Ordered UUID 는 저장될 떄, 비슷한 키끼리 근처에 저장되게 됩니다.
- BigInt 또한, 마찬가지로 크기도 작고 저장될 떄, 비슷한 키끼리 근처에 저장되게 됩니다.
- 아래 reference 의 benchmarking 을 보면 BigInt, Ordered UUID 는 파일 크기나 검색 속도가 비슷하며, 기존 UUID 가 검색속도나 파일 크기가 제일 안좋습니다. 


## Reference
- [https://runebook.dev/ko/docs/mariadb/guiduuid-performance/index](https://runebook.dev/ko/docs/mariadb/guiduuid-performance/index)
- [https://www.percona.com/blog/2014/12/19/store-uuid-optimized-way/](https://www.percona.com/blog/2014/12/19/store-uuid-optimized-way/)
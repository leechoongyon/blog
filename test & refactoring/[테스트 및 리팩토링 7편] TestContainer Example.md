> source 는 [Github](https://github.com/leechoongyon/TestContainerExample) 에 있습니다.



> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.



# [테스트&리팩토링 7편] TestContainer Example



# TestContainer 란?

- DB, 큐, 메시지 브로커 등을 제공해주는 가상 컨테이너입니다. Java, Kotlin 과 같은 JVM 기반에서 동작하며, Docker 를 필요로 합니다. 
- docker-compose 는 언어와 관계 없이 사용할 수 있으며, TestContainer 는 코드 레벨에서 제어할 수 있습니다.
- 코드에서 제어할 수 있기에 docker 만 떠있다면 테스트 코드에 container 기동, 중단 로직을 넣어서 테스트를 수행할 때, 한 번에 수행할 수 있는 특징이 있습니다.



# 소스 설명

- Spring boot, spring data jpa, MySQL 환경에서 DB 컨테이너를 띄워서 DB CRUD 를 진행하는 예제를 작성했습니다.
- DB 만 테스트 하기 위해 `@DataJpaTest` 를 사용했습니다.



 # 소스

- Test Source 와 Container, DataSource Config 를 작성했습니다. DataSource 를 빈초기화 할 때는, TestMySQLContainer 작업이 끝나야 MySQL DB Container 가 생성되기에 DependsOn 을 설정했습니다.
- 나머지 소스는 [github](https://github.com/leechoongyon/TestContainerExample) 에 있습니다.

<script src="https://gist.github.com/leechoongyon/9153b03d950ff49f372d03d826f21d4a.js"></script>



# SharedContainer

- Container annotation 을 사용하면 테스트 클래스가 수행될 때마다 새로운 인스턴스가 생성되므로, 테스트 간에 영향을 주지 않습니다. Shared Container 는 생성한 인스턴스를 재사용하는 것이기에 각각의 테스트에서는 인스턴스를 공유해서 사용할 수 있으며, 새로 생성할 필요가 없기에 부하가 줄게 됩니다.
- 상황에 맞게 Container 를 쓸지 SharedContainer 를 쓸지 판단해서 쓰면 좋을 것 같습니다.

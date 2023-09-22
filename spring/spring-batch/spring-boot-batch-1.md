
## spring boot, spring batch 정리 - 1편 환경 구성 (spring boot, spring batch, gradle, multi project) 

## 환경 구성
- spring boot
- spring batch
- jpa
- querydsl
- gradle (multi project)


## github source
- [spring-boot-batch-example source](https://github.com/leechoongyon/spring-boot-example/tree/master/spring-boot-batch-example) 참고

## 프로젝트 전체적인 구성
- RootProject : spring-boot-example
- SubProject : spring-boot-batch-example


## 프로젝트 구성 방법
- [Intellij multi project 만드는 법](https://leechoongyon.github.io/posts/intellij-gradle-multi-project) 참고

- root build.gradle
{% gist leechoongyon/5354a9575492739a98cc048378d2f0b4 %}

- spring-boot-batch-example build.gradle
<script src="https://gist.github.com/leechoongyon/d8295d1623f2c29fe484ce16d410336a.js"></script>

## 구성 시 주의사항
- SpringBoot 2.3.1 과 JPA 를 같이 사용하면 배치 프로그램 수행 시, ShutdownHook 가 걸리는거 같음.
- 프로그램 종료가 바로 안되고, 1분 정도 대기하다 프로그램이 종료 됨.
- 해당 기능을 끌려고 찾아봤는데 안보여서 boot version 2.2.1 로 수정



> source 는 [Github](https://github.com/leechoongyon/docker-flyway-example) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 17편] spring-boot, docker, flyway 활용해서 로컬 mysql 테스트 환경 구축





# spring-boot, docker, flyway 활용해서 로컬 mysql 테스트 환경 구축

- 로컬에 docker + flyway 를 통해 로컬에 mysql 테스트 환경 구축하는 방법을 정리했습니다.
- docker 는 가상화 컨테이너이고, flyway 는 database 마이그레이션 tool 입니다.
  - flyway 는 ddl 이력 관리도 가능합니다. (형상 관리)
  - 또한, flyway 를 spring-boot 에서 사용하는 것은 테스트 환경을 구축할 때, 편합니다.
- 이 예제에서는 docker 를 통해 가상의 mysql 을 생성하고, flyway 를 통해 테이블을 하나 생성해 볼 예정입니다.
- docker 는 설치돼있다고 가정하고 진행하겠습니다.



## source

<script src="https://gist.github.com/leechoongyon/af5303d0cc60abf7b756a92edd76956e.js"></script>






## 실행방법
1. 위에 docker-compose 명령어를 실행해서 docker 를 생성해줍니다.
2. spring-boot application 을 실행시켜줍니다.
3. classpath:/db/migration 위치에 있는 DDL 파일이 실행되며, 테이블이 생성됩니다.



## 마무리
- 로컬에서 mysql 테스트 환경을 구축할 때, spring-boot, docker, flyway 를 활용하는 방식에 대해 간략히 정리했습니다.

- docker 사용 시, M1 mac 의 경우 `no matching manifest for linux/arm64/v8 in the manifest list entries` 에러가 발생했습니다.
- 아래와 같이 플랫폼을 명시해서 처리했습니다.
```yaml
local-db:
    platform: linux/x86_64    # 추가된 라인
```

- 추가로 flyway 를 통해 테이블에 대한 변경 등을 관리할 수 있습니다.
  - 예를 들면, test 라는 테이블에 컬럼이 추가가 되거나 삭제가 된다면 v1, v2 이런식으로 flyway 를 통해 버전관리를 할 수 있습니다.
  - 이렇게 버전관리된 것을 git 을 이용해 remote 환경에서 관리할 수 있고, 개발팀이 DB 운영 배포를 할 수 있다면 flyway 를 통해 운영 까지 배포도 가능합니다. 
  
  


## Reference
- https://sabarada.tistory.com/193
- https://unluckyjung.github.io/develop-setting/2021/03/27/M1-Docker-Mysql-Error/
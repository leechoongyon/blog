> 목차는 [Infra 목차](https://insanelysimple.tistory.com/category/infra) 에 있습니다.



# [Infra 4편] Container, Docker, k8s 정리

> 정리 용도로 작성했습니다. 계속해서 내용 보강할 예정입니다.

# Container

- Application Code 와 모든 Depedencies 를 패키징하는 소프트웨어 표준 단위입니다.
- 예를 들면, Tomcat, Mysql 과 같은 애플리케이션을 독립적으로 실행할 수 있는 패키징 단위입니다.
- 컨테이너는 하나의 os 위에서 독립적으로 실행되며, 영향을 주지 않습니다. 즉, 하나의 os 위에 여러개의 container 가 실행됩니다.



### Container vs 가상화

- 착각하기 쉬워 정리했습니다. Container 의 경우 하나의 OS 에서 여러 Conatiner 가 독립적으로 실행된다면, VM 은 os, app 까지 포함해서 실행됩니다.



# 이미지

- 이미지는 컨테이너를 만들기 위한 파일을 의미합니다. 이미지로 컨테이너를 생성할 수 있습니다.



# Docker

- Docker 애플리케이션을 신속하게 구축, 테스트, 배포할 수 있게 해주는 소프트웨어입니다.
- Docker 는 Container 를 빌드, 배포 등 관리할 수 있습니다.



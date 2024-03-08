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



# 컨테이너 오케스트레이션

- 서비스가 계속 성장해서 Docker 서버 1~2대로 관리가 어려울 경우, Docker 서버를 여러대 구성하여 클러스터를 구성해야할 필요가 있습니다.
- 이러한 필요 때문에 컨테이너 오케스트레이션이 나왔으며, 컨테이너화된 서비스를 관리하는 역할을 담당합니다.



# Kubernetes (k8s)

- 오픈 소스 컨테이너 오케스트레이션 플랫폼입니다.
- 컨테이너화된 애플리케이션의 배포, 관리, 확장을 예약하고 자동화하는 플랫폼입니다.





# 참고

- https://www.docker.com/resources/what-container

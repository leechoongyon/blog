> 목차는 [Infra 목차](https://insanelysimple.tistory.com/category/infra) 에 있습니다.



# [Infra 1편] 포워드 프록시, 리버스 프록시, 게이트웨이 개념 정리



## 프록시 서버
- 중계 서버입니다.
  - 클라이언트 ← 프록시 서버 → 서버



## 포워드 프록시
- 클라이언트에서 인터넷망으로 접근할 때, 중계 서버 역할을 포워드 프록시라 합니다.

- 클라이언트 → 포워드 프록시 서버 → 인터넷 → 결과

- 예를 들면, 내부망에서 외부 API 를 연동한다고 가정하겠습니다.
    - 내부망 Client → 포워드 프록시 서버 → API 서버
    
    

## 리버스 프록시
- Client 에서 내부 서비스로 접근할 때, 중계 서버 역할을 리버스 프록시라고 합니다.
    - nginx, httpd 가 있습니다.

- Client → Proxy Server → 내부 서비스
- 예를 들면, 외부망에서 내부 서비스를 호출 했다고 가정하겠습니다.
    - 외부망 Client → 리버스 프록시 서버 → 내부 서비스 (WAS)



## 포워드 프록시, 리버스 프록시 특징

#### 보안
- 포워드 프록시의 경우 클라이언트의 IP 를 감출 수 있습니다.
- 리버스 프록시는 서버 정보를 감출 수 있습니다.





# 게이트웨이

- 네트워크에서 서로 다른 통신망, 프로토콜을 사용하는 네트워크 간의 통신을 가능하게 하는 컴퓨터, 소프트웨어를 일컫는 용어입니다.





# Api Gateway 패턴

- 혹시나 게이트웨이랑 헷갈릴수도 있을 것 같아 같이 정리했습니다. 



### 개념

- Client 와 마이크로 서비스 서버 사이에 위치하는 역방향 프록시입니다.



### 장점

- Api Gateway 가 위치함으로써 클라이언트에서는 Api Gateway 만 바라보면 되므로 시스템간 결합이 낮아집니다.
- 인증이나 공통 기능이 필요하다면 Api Gateway 를 통해 제공이 가능합니다. 클라이언트에서 여러번 호출할 필요가 없는 것입니다.



### 단점

- Api Gateway 에 트래픽이 몰릴 수 있으니 그에 따른 방안이 필요합니다. Scale out 이나 장애 발생 시, 어떻게 복구할지 등 이런 부분에 있어서 고려가 필요합니다.



## Reference

- [https://www.lesstif.com/system-admin/forward-proxy-reverse-proxy-21430345.html](https://www.lesstif.com/system-admin/forward-proxy-reverse-proxy-21430345.html)
- https://docs.microsoft.com/ko-kr/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern
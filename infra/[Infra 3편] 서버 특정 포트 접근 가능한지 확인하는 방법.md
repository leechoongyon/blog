> 목차는 [Infra 목차](https://insanelysimple.tistory.com/category/infra) 에 있습니다.



# [Infra 3편] 서버 특정 포트 접근 가능한지 확인하는 방법



>  접근하고자 하는 서버의 포트가 listen 상태여야 telnet, nc 명령어가 동작이 됩니다.





# NC (netcat)

- 넷캣은 네트워크 연결에서 데이터를 읽고 쓰는 유틸리티입니다.

```shell
// 3000 포트를 listen 모드로 오픈합니다
nc -l 3000


// localhost 3000 번 포트로 접속합니다. 접속이 성공해서 text 를 입력하고 엔터를 치면 3000 포트를 listen 하는 프로세스에서 동일한 text 가 보입니다.
nc localhost 3000



// hello world 엔터 치면 아래와 같이 나옵니다.

1번 프로세스 (nc -l 3000 명령어 수행)

hello world

2번 프로세스 (nc localhost 3000)

hello world
```

 






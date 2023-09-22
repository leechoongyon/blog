> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 16편] Rest Template, WebClient 정리



> RestTemplate 동작방식에 대해 공부할려고 검색하다가 WebClient 내용이 많이 나와 같이 정리했습니다.





> RestTemplate 은 spring 5.0 부터 버그나 마이너한 이슈만 유지보수한다고 안내해주고 있습니다.


```text
NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.
```



# RestTemplate 동작방식

- RestTemplate 의 경우 request 할 때마다 스레드를 사용하는 구조입니다.
- request 마다 스레드를 할당하는 구조이기에 응답을 받기 전까지 thread 는 block 이 됩니다. (동기, Blocking)
  - blocking, non-blocking 에 대한 설명은 다음 내용을 참조하시면 됩니다. [(참고)](https://insanelysimple.tistory.com/401)
- 만약 너무 많은 요청이 들어온다면 많은 스레드를 사용할 것이고, 많은 스레드를 사용한다면 스레드 풀이 고갈되며, 스레드 컨텍스트 스위칭이 빈번하게 일어납니다. 즉, 성능 저하가 일어날 것입니다.



### 그림 설명

- 아래 그림 예시를 보면 여러 Request 를 호출하고 있으며, 여러 Request 는 싱글톤 RestTemplate 을 호출하고 있습니다.
- 각각의 RestTemplate 은 thread 를 할당 받아 요청을 처리합니다. (blocking, sync)



![rest-template-desc](/Users/leejungyun/Intellij/etc/leechoongyon.github.io/study/spring/spring-series/images/rest-template-desc.png)





# WebClient 란?

- Reactor 를 기반으로 하는 비동기가 가능한 client 모듈입니다.



# WebClient 동작방식

- WebClient 는 비동기, non-blocking 방식을 지원합니다. (동기 방식도 지원합니다.)
- RestTemplate 의 경우 request 가 올 때마다 스레드를 할당하는 방식이였다면, WebClient 는 event driven 아키텍처 방식을 사용합니다.
  - event driven 아키텍처는 적은 리소스 (스레드, 시스템 리소스) 를 사용하여 더욱 많은 요청을 처리하는 아키텍처입니다.



### 그림 설명

- 아래 예시를 보면 여러 Request 에 대해 RestTemplate 처럼 thread 를 할당받아 처리하는게 아니라 request 를 event 로 만들어서 Queue (Event-loop) 와 같은 곳에 집어넣습니다. 
- event 를 하나씩 꺼내서 Handler 가 처리하게 되고, 결과에 대해 callback 을 보내게 됩니다.
- 아래와 같은 event-driven 아키텍처를 이용해 적은 리소스를 사용해서 많은 요청을 처리할 수 있게 됩니다.

![web-client-desc](/Users/leejungyun/Intellij/etc/leechoongyon.github.io/study/spring/spring-series/images/web-client-desc.png)









# Reference

- [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)
- [https://www.baeldung.com/spring-webclient-resttemplate](https://www.baeldung.com/spring-webclient-resttemplate)
- [https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)
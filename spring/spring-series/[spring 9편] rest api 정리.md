> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 9편] rest api 정리



## rest 란?

- Representational state transfer
- rest 는 스펙이 아니라 규칙의 집합입니다.
- HTTP 기반으로 자원에 접근하는 방법을 명시한 아키텍처 입니다.
- 자원은 DB 데이터, 이미지, 파일 등을 의미 합니다.
- URI 을 통해 자원을 명시하고, HTTP METHOD 를 통해 해당 자원에 대한 행위를 표현합니다.



## rest 특징

#### 1. 서버에 있는 resource 는 고유 URI 를 가지고 있습니다.
- Member 테이블에 접근하고자 할 때, '/member/1 (GET)' URI 로 클라이언트에서 요청합니다.
- Address 테이블에 접근하고자 할 때, '/addr/1' (GET)' URI 로 클라이언트에서 요청합니다.

#### 2. 세션 정보를 별도로 관리할 필요가 없음.
- rest api 는 http 프로토콜을 사용하며, http 프로토콜은 stateless 입니다. 즉, 서버에 상태를 저장할 수 없으며, 이는 분산 환경에 사용하기 적합합니다.

#### 3. HTTP METHOD 사용
- GET,POST,PUT,DELETE 4개 메소드를 통해 자원에 대한 행위를 정의할 수 있습니다.
- HTTP Method 로만 행위를 표현할 수 있기에 복잡한 행위를 표현하기에 부족할 수 있습니다.
- 멱등성이란 여러 번 수행을 해도 같은 경우를 의미합니다.
- HTTP Method
  - GET : 조회 메소드입니다. 멱등성을 보장합니다.
  - POST : 데이터 생성 메소드입니다. 멱등성 보장하지 않습니다.
  - PUT : 데이터 갱신을 의미하며, 멱등성 보장합니다.
  - PATCH : 해당 자원의 일부 교체합니다. PUT 은 자원 전체를 갱신하는 것을 의미합니다. PATCH 는 그렇기에 멱등성을 보장하지 않습니다.
  - DELETE : 데이터를 삭제합니다.
- 응답 상태 코드
  - 1xx : 리퀘스트를 받고 처리중에 있다는 상태입니다.
  - 2xx : request 성공적으로 처리한 상태입니다.
  - 3xx : 리다이렉션을 의미하는 상태입니다.
  - 4xx : 클라이언트 오류 입니다.
  - 5xx : 서버 오류입니다.



## rest api (구성요소, 멱등성 등)

- rest 한 것을 따른 API 를 restful api 라고 합니다.

#### REST (Representational state transfer) 구성요소

- 리소스, 메서드, 메세지 3가지 요소로 구성됩니다.
- 리소스를 통해 자원을 명시하고, 메서드를 통해 자원에 대한 행위를 결정하며, 메세지를 통해 행위에 대한 상세 또는 응답에 대한 상세를 표현할 수 있습니다.

```html
HTTP POST https://localhost/members/
{
    "members": {
         "name" : "test",
         "age" : 20
     }
}
```

#### Resource Naming

- 리소스를 지칭하는 것 이기에 명사를 사용하는 것이 좋으며, 복수를 쓰는 것이 좋습니다. 세부, 상세 내용에 접근할 때는, ID, seq 등을 사용합니다.
  - /members/{memberId}

- 하나의 REST API 에 복잡한 관계를 표현해야할 때도 있습니다. '/' 를 이용해서 계층관계를 표현할 수 있습니다.
  - 예를 들면, /users/{userId}/articles/{articlesId}
  - 특정 user 의 article 상세 정보를 접근하는 리소스명
- URI 길이가 긴 경우 하이픈을 사용합니다.
  - /users/{userId}/payment-method


#### 캐시 기능
- REST 는 HTTP 프로토콜을 사용하기에 HTTP 프로토콜 캐시를 사용할 수 있습니다.
- 동작 방식은 다음과 같습니다.
  - Client —> Server 로 Last-Modified 태그를 보내고, 서버에서 변한게 없다면 304 Not Modified 라고 응답을 줍니다.

```html
Client --> Server
HTTP GET /api/members/1
Last-Modified: Mon, 20 Feb 2022 11:12:13 GMT

Server --> Client (304 Not Modified)
If-Modified-Since: Mon, 20 Feb 2022 11:12:13 GMT

```



## spring rest api source

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {
    private final MemberService memberService;
    @GetMapping("/api/v1/members/{memberId}")
    public CommonResponse getMember(@PathVariable("memberId") String memberId) {
        MemberDto.Main response = memberService.getMember(memberId);
        return CommonResponse.success(response);
  }
}
```



## reference

- [https://bcho.tistory.com/953](https://bcho.tistory.com/953)
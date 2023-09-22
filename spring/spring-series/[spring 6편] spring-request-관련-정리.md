> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.

# [spring 6편] spring request 관련 정리

## @RequestParam
- Http 요청 파라미터를 받기 위해서 spring 에서 사용하는 annotation 입니다. 
- @RequestParam 은 값이 반드시 있어야 하며, 값이 없으면 400 Error 가 발생합니다.
  - 값을 반드시 받지 않아도 되는 옵션이 있습니다 (required = false)
- 예를 들면, http://localhost:8080?page=2&size=10  get 방식으로 요청한다고 가정을 합니다.
- 위와 같은 url 이 있을 때 RequestParam 을 사용하면 값을 가져올 수 있습니다.

```java

// http://localhost:8080?page=2&size=10

@GetMapping(value ="/member", params = {"page", "size"})
public ResponseEntity<?> getMembers(@RequestParam("page") int page, 
                                    @RequestParam("size") int size) {
    
}

```

## @PathVariable
- PathVariable 의 정의는 요청 URI 매핑에서 변수를 매핑하는 annotation 입니다.

```java

@GetMapping("/member/{id}")
@ResponseBody
public String getMember(@PathVariable("id") String memberId) {
    
}

```

## RequestParam 과 PathVariable 차이점
- RequestParam 과 유사한 기능이여서 헷갈릴 수 있습니다.
- 둘을 구별하는 방법은 다음과 같습니다.
- http://localhost:8080/?page=2&size=10
  - 이렇게 key, value 형식으로 들어오면 @RequestParam 을 사용합니다.
- http://localhost:8080/member/3
  - 이럴 경우 @PathVariable 을 사용하면 됩니다.


## RequestParam 과 PathVariable 내부 동작
- HandlerMethodInvoker.resolveHandlerArguments 에서 Parsing 이 이루어지며, 값이 없을 경우 empty string 을 return 합니다.

## @RequestBody
- RequestBody 는 Http Request Body 를 자바 객체로 비직렬화하는 annotation 입니다.
- 관련 자세한 내부 동작은 [MessageConveter](https://insanelysimple.tistory.com/308) 정리했습니다.


# Reference
- https://stackoverflow.com/questions/6891539/internal-working-of-springss-requestparam-annotation
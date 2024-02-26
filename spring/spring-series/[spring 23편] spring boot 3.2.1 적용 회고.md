> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 23편] spring boot 3.2.1 적용 회고



> 나중에 볼려고 정리했습니다. 





# Intro

- spring boot 3.2.1 적용해보며 경험했던 점을 정리했습니다. 기억이 나는데로 적었으며, 틀린 내용이 있을 수 있습니다.
- 지금 글을 쓰는 시점에 spring boot 3.2.2 가 나왔습니다.



# Parameter Name Discovery

- spring boot 3.2.1 적용하면서 bean 을 가져올 때(생성자 주입 또는 Autowired 등), 빈이 2개 이상 발견됐다는 에러가 발생했습니다. 
- 해결방안은 Qualified 를 사용해서 bean 을 명시적으로 주입합니다.
- 아래 패치 내용으로 인해 그런 것이라 추정됩니다. 확실하지 않습니다.

```tex
The version of Spring Framework used by Spring Boot 3.2 no longer attempts to deduce parameter names by parsing bytecode. If you experience issues with dependency injection or property binding, you should double check that you are compiling with the -parameters option. See this section of the "Upgrading to Spring Framework 6.x" wiki for more details.
```



# Method Validation

- Controller Method Parameter 에 `@Constraint` 가 선언돼있으면 Method Validation 동작합니다. (신규 기능)
- Method Validation 동작하면서 기존 MethodArgumentNotValidException → HandlerMethodValidationException 발생합니다.
- GlobalExceptionHandler 에서 HandlerMethodValidationException 추가해서 처리합니다.

- 참고 : https://github.com/spring-projects/spring-framework/issues/31775



### example source

```java
@RestController
public class TestController {
  	public ResponseEntity test(@Valid @RequestBody @ValidAccount ReqeustDto requestDto) {
      ...
      ...
    }
}


// 예시로 작성한 것입니다. 문법이 틀릴수 있습니다.
@Documented
@Constraint
@Target({ ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidAccount {
    ...
    ...
    ...
}



```





# MethodValidation index cannot find local variable

- MethodValidation 이 동작하면서 custom validation 이 동작할 때, Property 를 접근 못하는 예외가 발생했습니다. 자세한 내용은 https://github.com/spring-projects/spring-framework/issues/31746 참고하면 됩니다.

- 해결방안은 spring 6.1.3 을 적용하는 것인데, 글을 쓰는 지금 이 순간 spring boot 3.2.2 가 나와서 3.2.2 를 적용하면 해결될 것이라 생각됩니다. 

- 참고 : https://github.com/spring-projects/spring-framework/issues/29043





# Pageable.unpaged not serializable

- 기존에 Pageable.unpaged 는 json 으로 serialize 가능했습니다. spring boot 3.2.1 버전이 나오면서 unpaged 가 json serialize 하지 않아서 에러가 발생합니다. PageRequest.of(0, 10) 방식으로 처리 가능합니다.
- https://github.com/spring-projects/spring-data-commons/issues/2987



# Reference

- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.2-Release-Notes

> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.

# [spring 8편] spring response 관련 정리

## Spring ResponseEntity 란?
- HttpBody, HttpHeader, HttpStatus 을 담고 있음.
- 해당 데이터들을 response 로 넘겨줄 수 있음.
- 아래 소스를 보면, getMember 를 호출하면 member 데이터 (JSON) 와 HttpStatus.OK (200) 을 클라이언트에서 받게 됨.

```java

@RestController
public class MemberController {
    
    @GetMapping("/member/{id}")
    public ResponseEntity getMember(@PathVariable("id") String id) {
            
        ...
        ...

        return new ResponseEntity(member, HttpStatus.OK);
    }
}
```

## ResponseBody annotation 란?
- 자바 객체를 HTTP body 로 매핑해서 클라이언트로 전송합니다.
- 관련 자세한 내부 동작은 [MessageConveter](https://insanelysimple.tistory.com/308) 정리했습니다.
- 추가로 RestController 에는 ResponseBody 가 있기 때문에 RestController annotation 을 사용하면 ResponseBody annotation 을 사용 안해도 됩니다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any (or empty String otherwise)
	 * @since 4.0.1
	 */
	@AliasFor(annotation = Controller.class)
	String value() default "";

}

```


## Reference
- https://devlog-wjdrbs96.tistory.com/182 
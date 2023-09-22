> source 는 [Github](https://github.com/leechoongyon/spring-boot-example-3.0.1) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 18편] spring validation (유효성 검사)



# spring validation

- spring 에서 입력 값에 대해 validation 하는 것에 대해 정리했습니다.
- 아래 예제는 controller 입력 값에 대해 Validation 하는 로직입니다.
- Controller 의 경우 '@Valid' 옵션을 붙여주면 동작합니다.
- Service 의 경우 '@Validated' 옵션을 붙여주면 됩니다.



### `@Valid`

- `@Valid` 는 자바 표준 스펙에 위해 구현된 객체의 제약 조건을 검증하는 어노테이션 입니다.
- Controller method argument 를 만들어주는 ArgumentResolver 에서 `@Valid` 선언된 것을 validation 합니다.
- 참고로 Service layer 에서도 Validated 가 선언돼있으면 Valid (Jarkara validation) 사용 가능합니다.
- 아래 예시와 같이 MemberCommand.Create 값이 없으면 registerMember 호출 시, validation 에러가 발생합니다.

```java
@Service
@Validated
@RequiredArgsConstructor
public class MemberServiceImpl implements MemberService{

    @Override
    public String registerMember(@Valid MemberCommand.Create memberCommandCreate) {
        return memberCommandCreate.getId();
    }
}

public class MemberCommand {

    @Data
    @Builder
    @AllArgsConstructor
    public static class Create {
        @NotBlank
        private String id;
        @NotBlank
        private String name;
    }
}

```





### `@Validated`

- `@Valid` 와는 다르게 Spring 프레임워크에서 제공하는 validation 기능입니다.
- AOP 기반으로 동작하며, 아래와 같이 사용 가능합니다.

```java
@Service
@Validated
public MemberServiceImpl implements MemberService {
  
  public void addMember(@Valid AddMemberRequest request) {
    // do something
  }

}
```

- AOP 기반으로 동작하기에 controller, service 등 어떤 계층에 상관없습니다.





# Controller Example

<script src="https://gist.github.com/leechoongyon/bfe05c355a1f88fcda5556ace8435737.js"></script>





# Reference

- https://meetup.nhncloud.com/posts/223
- https://mangkyu.tistory.com/174


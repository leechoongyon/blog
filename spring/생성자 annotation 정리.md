## 생성자 annotation 정리

#### @NoArgsConstructor
- 파라미터가 없는 기본생성자를 만들어줍니다.

```java

// before
@NoArgsConstructor
public class NoArgsConstructorTestClass {
}

// after
public class NoArgsConstructorTestClass {
    public NoArgsConstructorTestClass() {}
}


```


#### @AllArgsConstructor
- 클래스에 선언된 모든 필드 값에 대한 생성자를 만들어줍니다.

```java

// before
@AllArgsConstructor
public class AllArgsConstructorTestClass {
    private String name;
    private int age;
}

// after
public class AllArgsConstructorTestClass {
    private String name;
    private int age;
    public AllArgsConstructorTestClass(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

```

#### @RequiredArgsConstructor
- final 또는 @NotNull 인 필드 값을 파라미터로 받아 생성자를 만들어 줍니다.

```java

// before
@RequiredArgsConstructor
public class RequiredArgsConstructorTestClass {
    @NotNull
    private String name;
    private final TestA testA;
}

// after
public class RequiredArgsConstructorTestClass {
    @NotNull
    private String name;
    private final TestA testA;
    
    public RequiredArgsConstructorTestClass(String name, TestA testA) {
        if (name == null) {
            throw new IllegalArgumentException(xxx);
        }
        this.name = name;
        this.testA = testA;
    }
}
```



## 추천 방식 1 - @NoargsConstructor(AccessLevel.PROTECTED) + @Builder

- @NoargsConstructor(AccessLevel.PROTECTED) 를 사용함으로써 기본생성자는 패키지 내에 있는 것만 생성할 수 있게 캡슐화 했습니다.
    - 이렇게 할 경우 내가 빠트릴수 있거나 의도치 않게 추가한 항목에 대해 한 번 검증이 가능합니다.
```java
NoArgsConstructorTestClass test = new NoArgsConstructorTestClass();
test.setName("test");
test.setTelNo("02-123-1234");   // 이 항목은 내가 원치 않았던 항목임.
```

- @Builder 를 사용함으로써 내가 사용하고 싶었던 항목들만 세팅이 가능하니 실수할 일이 적어집니다.  
  
```java

@NoArgsConstructor(AccessLevel.PROTECTED)
public class NoArgsConstructorTestClass {
    private String name;
    private String telNo;
    
    @Builder
    public NoArgsConstructorTestClass (String name) {
        this.name = name;
    }
}

```


## 추천방식 2 - @RequiredArgsConstructor
- 아래 두 소스는 ApiController 에서 서비스를 호출할 떄, 2가지 방식을 사용했습니다.
- Spring team 에서는 @RequiredArgsConstructor 방식을 추천합니다.
- 생성자 주입을 추천하는 이유는 다음과 같습니다.

```java
@RestController
@RequiredArgsConstructor
public class MemberApiController {
    private final MemberService memberService;
}

```

#### 순환 참조 방지
- 순환 참조 사전 체크 가능
  - Autowired 를 통해 빈 순환 참조가 일어나면 해당 코드가 실행되어야지만 순환참조로 인해 에러가 난 것을 알 수 있습니다.
  - 생성자 주입 방식 (RequiredArgsConstructor) 을 이용하면 application 이 구동될 때, 순환 참조를 알 수 있습니다.
 
#### 테스트 편리
- 생성자 주입을 이용할 경우 테스트 할 때, 구현체에 대한 교체가 쉽습니다.
- 아래 처럼 memberService 에 내가 원하는 구현체를 넣어서 테스트 할 수 있습니다.
```java

public MemberApiController(MemberService memberService) {
}

``` 

#### 코드 품질 및 불변성
- RequiredArgsConstructor 선언할 경우 객체는 불변이 되며, 깔끔해집니다.
- Autowired 를 사용한다면 객체가 추가될 때마다 Autowired 를 해줘야 하니 코드수가 늘어날테고, Autowired 로 선언한 것은 코드 내에서 객체가 변경이 가능합니다.

## reference
- https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection
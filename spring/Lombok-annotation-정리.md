# Lombok Annotation 정리

## 생성자 관련

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
        this.name = name;
        this.testA = testA;
    }
}


```

#### (access = AccessLevel.PROTECTED)
- lombok 생성자 annotation 을 사용할 때, 위 옵션을 주게 되면 외부에서 생성자 생성이 제한됩니다. (캡슐화가 protected 이므로. 같은 패키지만 접근 가능)
- 이렇게 하는 이유는 외부에서 생성자를 만들 때, setxxx 을 통해 변수를 세팅할려고 하면 값을 뺴먹을 수 있기 때문입니다.
- 아래와 같은 예제에서는 name 이 실수로 set 을 안해서 빠지게 됩니다.
```java

// [Before]

// Test
@Setter
public class Test {
    private String id;
    private String name;
}

/// Main
public static void main(String[] args) {    
    Test test = new Test();
    test.setId("123");
    
}

-----------------------------------------------------------

// [After]

// Test
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Test {
    private String id;
    private String name;


    @Builder
    public Test(String name) {
        this.id = "defaultId";
        this.name = name;
    }
    
    @Builder
    public Test(String id, String name) {
        this.id = id;
        this.name = name;
    }
}

/// Main
public static void main(String[] args) {
    Test test = new Test().builder()
            .name("test name")
            .id("test id")
            .build();
}

```
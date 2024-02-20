> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.

# [테스트 및 리팩토링 9편] Junit5 정리



> 추후 찾아보기 위해 정리했습니다. 내용을 보강할 예정입니다.



# ExtendWith

- Junit5 의 라이프사이클 중 Test 에서 사용할 기능을 확장하는 것입니다. 



### ExtendWith (SpringExtension.class)

- spring TestContext + Junit5 통합하여 사용합니다.
- 인터페이스 : BeforeAllCallback, AfterAllCallback, TestInstancePostProcessor, BeforeEachCallback, AfterEachCallback, BeforeTestExecutionCallback, AfterTestExecutionCallback, ParameterResolver



 ### ExtendWith (MockitoExtension.class)

- Mockito 관련된 클래스 사용합니다.
- 인터페이스 : BeforeEachCallback, AfterEachCallback, ParameterResolver





# InjectMocks

- `@Mock` 이 붙은 클래스를 주입 해줍니다.

- 아래 소스를 예를 들어 설명하면, TestService 에 InjectMocks 가 설정돼있으며, TestService 내의 Mock 대상에 객체를 주입해주는 역할을 합니다.



```java
public class TestServiceTest {
  @InjectMocks
  private TestService testService;

  @Mock
  private TestRepository testRepository;

}

@Service
@RequiredArgsConstructor
public class TestService {
	private final TestRepository testRepository;
}
```



# Mock

- 가짜 객체를 뜻하며, Mock 으로 선언된 객체는 스터빙을 (when, return 등) 해줘야 합니다.



# 단위 테스트 예시

- 아래 소스 예시를 보면 TestService 에서 testRepository 를 사용하며, testRepository 에 mocking 을 해서 testService 를 단위테스트 하는 예제입니다.

```java
@ExtendWith (MockitoExtension.class)
public class TestServiceTest {
  @InjectMocks
  private TestService testService;

  @Mock
  private TestRepository testRepository;

  @Test 
  void testFindRepository() {
 		testRepository.save(TestEntity.builder.name("test").build());
    
    List<TestDto> list = testService.getTestList();
    
    Assertions.assertEquals(1, list.size());
  }
}
```


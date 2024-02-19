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






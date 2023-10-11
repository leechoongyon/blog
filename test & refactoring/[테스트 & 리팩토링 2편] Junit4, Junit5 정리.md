> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.

# [테스트 & 리팩토링 2편] Junit4, Junit5 정리


## JUnit4, JUnit5 개념
- JUnit 은 테스트 컴포넌트입니다.
- JUnit4 의 업그레이드 버전이 JUnit5 입니다.
- JUnit4 가 단일 Jar 를 사용한 것에 반해 JUnit5 는 3가지 모듈로 구성돼있습니다.
    - JUnit 플랫폼 + 주피터 + Vintage
- 여기서 알아둘만한 것은 Vintage 모듈을 통해 JUnit5 이전 버전 소스도 JUnit5 에서 동작이 가능합니다.

## JUnit4, JUnit5 혼용 시 문제점
- junit4 의 경우 vintage 가 발견해서 실행을 시킵니다.
- junit5 의 경우 jupiter 가 발견해서 실행을 시킵니다.
- 이 2개 동작은 독립적으로 수행되며, 아래 예시를 보면 Before annotation 은 Junit4 의 annotation 입니다.
- junit5_테스트() 메소드에서는 setUp 이 동작되지 않습니다. junit4_테스트() 에서만 setUp 이 동작합니다.
- 그렇기에 2개를 혼용해서 사용하면 예측하지 못한 동작이 발생할 수 있습니다. 

```java

public class 혼용테스트 {
    @Before
    public void setUp() {
        ...
        ...
        ...
    }
    
    @org.junit.jupiter.api.Test
    public void junit5_테스트() {

    }

    @org.junit.Test
    public void junit4_테스트 {

    }
}


```


## 결론
- Junit4, Junit5 를 한 클래스 내에서 섞어서 쓰지말아야 합니다.
- Junit4, Junit5 중 하나를 선택해서 사용하는게 좋으며, 혼용해서 사용하고 싶다면 Junit4, Junit5 각각 별개 클래스에 구현해야 합니다.  

## refernce
- https://igorski.co/mixing-junit-4-and-junit-5-tests/
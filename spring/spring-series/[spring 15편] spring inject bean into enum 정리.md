> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 15편] spring inject bean into enum 정리





# inject bean into enum 무엇이며, 어떨 때 사용하는가?

- enum 에 bean 을 주입하는 방법에 대해 정리했습니다. 
- 예를 들면, enum 에서 공통된 동작인데 구현체는 다르다면 사용하면 좋을 것 같습니다.
  - 아래 예제와 같이 MemberType 별로 별도 할인 정책을 가져가야하는 부분이 있다면 사용하면 좋을 것 같습니다.





# source

- 아래 예시를 설명하자면 MemberType 의 GOLD, SILVER member 의 할인율을 가져오는 예시입니다.
- enum 의 Inner class 를 만들어서 bean 을 가져와서 enum 에 동적으로 값을 변경하도록 했습니다.
- Test 는 Bean 이 미리 만들어져야하는 상황이기에 통합테스트로 (SpringBootTest) 돌렸습니다.

<script src="https://gist.github.com/leechoongyon/a96991a9d28142329ef2dc09bcd2fe04.js"></script>





# enum, bean, static inner class life cycle 정리

### Enum

- enum 은 싱글톤으로 생성되며, class 가 classLoader 에 로드될 때 생성됩니다.

- jvm 이 최초 기동되면서 아래와 같이 byte code 를 만들어서 싱글톤으로 만들게 됩니다.
  - 최초 기동되면서 BCI 로 코드를 만들거라 추측되는데 확실치는 않습니다. (찾아봤는데 안나오네요)
  - 아래와 같이 static 으로 만들며, static 로 만들었다는 것은 jvm 로드 할 때, 전역 변수에 설정이 된 것이며, JVM 이 내려갈 때까지 GC 가 일어나지 않습니다.

```java
Compiled from "MemberType.java"
public final class MemberType extends java.lang.Enum<MemberType> {
  public static final MemberType CONST;

  static {};
    Code:
       0: new           #1                  // class MemberType
       3: dup
       4: ldc           #12                 // String CONST
       6: iconst_0
       7: invokespecial #13                 // Method "<init>":(Ljava/lang/String;I)V
      10: putstatic     #17                 // Field CONST:LMmberType;
      13: iconst_1
      14: anewarray     #1                  // class MemberType
      17: dup
      18: iconst_0
      19: getstatic     #17                 // Field CONST:LMemberType;
      22: aastore
      23: putstatic     #19                 // Field ENUM$VALUES:[LMemberType;
      26: return

}
```





### static inner class life cycle

- 클래스에 static 이 붙었다고 전역 변수처럼 동작하는 것이라 오해할 수 있습니다.
- 아래 예시를 보면 이해가 쉽습니다. 즉, 인스턴스를 생성하는 순간 메모리에 할당됩니다.

```java
public class Test {

  class InnerTest {
    
  }
  
  static class InnerStaticTest {
    
  }
}


Test.InnerTest t = new Test.InnerTest();
Test.InnerTest t2 = new Test.InnerTest();

// t != t2 (다른 인스턴스이므로 다릅니다.)

Test.InnerStaticTest t = new Test.InnerStaticTest();
Test.InnerStaticTest t2 = new Test.InnerStaticTest();

// t != t2 (다른 인스턴스이므로 다릅니다.)



```





### bean life cycle

- Jvm 이 기동되면서 ioc container 가 올라오면서 bean life cycle 이 동작하게 됩니다.
- BeanFactory 에는 classLoader 가 있으며, ClassLoader 에 Bean class 들이 등록됩니다.









# Inject bean into Enum 동작 방식 정리



> 여기서부터는 제 생각을 쓴 것이며, 내용이 틀릴 수 있습니다. 틀린점 comment 남겨주시면 감사하겠습니다.



1. jvm 이 최초 기동되면서 MemberType 에 대한 static 을 만듭니다.
2. spring boot 이 최초 기동되면서 @SpringBootApplication 의 @ComponentScan 에 의해 MemberTypeInjector 와 GoldMemberDiscountPolicy, SilverMemberDiscountPolicy Bean 이 생성됩니다.

3. MemberTypeInjector 의 Bean 이 만들어지면서 PostConstruct (생성자 생성 이후 동작) 에 위해 MemberType.GOLD, MemberType.SILVER 에 MemberDiscountPolicy instance 가 설정됩니다.
4. MemberType Enum 에는 Bean 이 각각 inject 됐습니다.



```java
@Getter
public enum MemberType {
    GOLD("골드"),
    SILVER("실버")
    ;

    private String desc;

    private MemberDiscountPolicy memberDiscountPolicy;

    MemberType(String desc) {
        this.desc = desc;
    }
    
    @Component
    @RequiredArgsConstructor
    public static class MemberTypeInjector {
        private final GoldMemberDiscountPolicy goldMemberDiscountPolicy;
        private final SilverMemberDiscountPolicy silverMemberDiscountPolicy;
        @PostConstruct
        public void postConstruct() {
            MemberType.GOLD.injectMemberDiscountPolicy(goldMemberDiscountPolicy);
            MemberType.SILVER.injectMemberDiscountPolicy(silverMemberDiscountPolicy);
        }
    }

    private void injectMemberDiscountPolicy(MemberDiscountPolicy memberDiscountPolicy) {
        this.memberDiscountPolicy = memberDiscountPolicy;
    }
}
```









# Reference

- https://johngrib.github.io/wiki/java-inner-class-may-be-static/
- https://stackoverflow.com/questions/31356845/life-span-of-a-java-enum-instance
- https://stackoverflow.com/questions/19971982/enum-class-initialization-in-java

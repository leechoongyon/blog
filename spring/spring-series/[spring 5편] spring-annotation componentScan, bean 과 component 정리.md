> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.

# [spring 5편] spring componetScan, bean, component 정리

## @ComponentScan
- 쉽게 말해 Bean 이 될 대상들을 찾아 application context 에 등록을 해주는 역할을 합니다.
- @Component 어노테이션을 빈 등록 대상으로 인식하고 스캔합니다.
- @Component, @Controller, @Servicie, @Repository 선언된 클래스가 대상입니다.
  - controller, serivce 등 annotation 에 선언된 것을 추적하면 @Component 로 선언된 것을 볼 수 있습니다.
- @Configuration 과 함꼐 사용하며, 아규먼트가 없다면 선언된 위치가 basePackages 입니다.
  - @Configuration 은 bean 등록을 위한 설정파일 역할을 합니다.
  - 옛날 spring 에서 xml 로 빈 등록을 하는 역할을 Configuration 이 합니다.

## spring boot 에서의 componentScan
- 얫날 버전에서는 xml 을 통해 bean 들을 등록 해서 applicationContext 를 구성하였습니다.
- spring boot 에서는 @SpringBootApplication annotation 을 이용하여 componentScan 을 제공하고 있습니다.
- 보통 spring boot 에서는 해당 @SpringBootApplication 이 속한 패키지가 componentScan 의 basePackages 가 됩니다.
  - basePackages 란 scan 이 일어나는 시작 위치입니다.
  - SpringBootApplication annotation 에는 Configuration 과 ComponentScan 이 같이 있기 때문입니다.
- ComponentScan 이 SpringBootApplication 에 있으며, filter 를 통해 bean 을 include 또는 exclude 할 수 있습니다.
  - 해당 기능은 왠만하면 사용하지 않는 것을 추천드립니다.
  - 클래스 수가 몇천 단위가 아닌 이상 로딩하는데 그렇게 오래 걸리지 않기 때문이며, 보통 기본 값으로 사용하는 경우가 많을 거라 생각됩니다.

```java
@Target(ElementType.TYPE)   // compiler 가 annotation 이 어디에 적용될지 결정하기 위해 사용합니다. ElementType.TYPE 은 Type 선언 시를 의미합니다.
@Retention(RetentionPolicy.RUNTIME) // 적용되고 유지되는 범위. 컴파일 이후에도 JVM 에 의해서 계속 참조가 가능합니다
@Documented // javadoc 으로 api 문서를 만들 때, 어노테이션에 대한 설명도 포함하도록 지정해주는 것입니다.
@Inherited  // 어노테이션을 선언한 슈퍼클래스를 상속한 서브클래스에서도 적용
@SpringBootConfiguration // Spring boot 의 configuration = bean 관련 xml 설정을 여기서 하겠다입니다. 
@EnableAutoConfiguration    // spring boot AutoConfiguration 관련 annotation, 쉽게 말하면 여러 컴포넌트에 대한 설정을 자동으로 import 시킵니다.
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ...
    ...
    ...
}


```


## @Bean, @Component 차이점은?
- Component 는 클래스에 선언하며, component-scan 이 이 annotation 을 찾아 bean 을 등록해줍니다.
  - 또한, 직접 제어가 가능한 클래스에 선언이 가능합니다.
- Bean 의 경우 메소드에 선언되며, 보통 Configuration 과 같이 쓰입니다.
  - Configuration 에는 Component annotation 이 선언돼있습니다.
  - Bean 의 경우 제어할 수 없는 클래스 등을 빈으로 등록하고 싶을 때 사용합니다.
  
```
@Configuration  
public class TestConfiguration {

  @Bean
  public List<String> getA() {
      return new ArrayList<>();
  }   

}
```

```

@Component
public class Test {

}
```


## reference
- https://www.baeldung.com/spring-component-scanning
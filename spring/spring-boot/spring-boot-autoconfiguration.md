## 1. SpringBoot 의 autoConfiguration 이란?
- 자동으로 환경설정을 해주는 것이다.
- 어떤 API, Component 를 작성하기 위해서 설정 클래스를 만들고, properties 에 설정하고. 이러한 과정들을 생략하여 spring boot 에서 처리해주는 것이다.
- 예를 들면, 다음과 같은 예가 있다.
    - Mybatis 설정
    - connection pool 설정 (hikari, apache...)
- 위와 같이 properties 로만 설정하여 원하는 기능을 쓸 수 있게 해준다.
    

## 2. AutoConfiguration 장점
- 버전 관리
- 초기 빈 호출을 등록안해도 된다는 점. 개발 편리성 증가

## 3. 언제 동작하는가? 누가 실행시키는가?
- @EnableAutoConfiguration, @SpringBootApplication 을 사용하면 자동 환경설정 기능 사용할 수 있음.

## 4. 내부 상세 동작
- 저 위에 2개 Annotation 이 사용된다면, @Import 가 호출된다.

```java
// @EnableAutoConfiguration
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
   ...
}  
```

- 자동설정을 담당해주는 AutoConfigurationImportSelector 을 호출

```java
// AutoConfigurationImportSelector.class
public String[] selectImports(AnnotationMetadata annotationMetadata) {
     if (!this.isEnabled(annotationMetadata)) {
         return NO_IMPORTS;
     } else {
         AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
         AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
         return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
     }
}
```

- 호출할 빈들을 위 클래스에서 뽑아옴. (중복 처리 등도 알아서 처리)

## 5. 빈 또는 프로퍼티 호출할 목록
- META-INF/spring.factories
    - 초기 빈 호출 목록
    - 각 jar 별로 META-INF/spring.factories 가 있음.
    - 예를 들면, Mybatis, AutoConfigurer, spring-boot 등 각 jar 별로 있음.
- META-INF/spring-configuration-metadata.json
    - 자동 설정에 사용할 프로퍼티 정의 파일
    - 얘도 jar 별로 META-INF/spring-configuration-metadata.json 파일이 있음.
- org/springframework/boot/autoconfigure
    - 미리 구현돼있는 자동 설정 리스트
    - 마찬가지로 얘도 jar 별로 폴더가 있음.

## 6. Reference
- [https://velog.io/@adam2/SpringBoot-자동-환경-설정AutoConfiguration](https://velog.io/@adam2/SpringBoot-%EC%9E%90%EB%8F%99-%ED%99%98%EA%B2%BD-%EC%84%A4%EC%A0%95AutoConfiguration)
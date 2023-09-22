> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 12편] spring boot 통합 test 실행 중 mybatis mapper not found 문제 정리



# 에러 로그
- spring boot 통합 test 환경 에서 mybatis 를 사용하는 도중 아래와 같은 에러가 발생했습니다.



### 로그

```textmate
No qualifying bean of type 'TestMapper' available
```



### 원인 및 해결방안

- Bean 을 못찾는 문제였으며, 다음과 같이 해결했습니다.

- 아래와 같이 TestConfig 라는 테스트 환경 전용을 담당하는 클래스를 만들어줘서 MapperScan 을 통해 해결했습니다.

<script src="https://gist.github.com/leechoongyon/31e50c5c45e1a5889e1bd2c84d1982be.js"></script>

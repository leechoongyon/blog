> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 23편] spring boot 3.2.1 적용 회고



> 나중에 볼려고 정리했습니다. 





# Intro

- spring boot 3.2.1 적용해보며 경험했던 점을 정리했습니다. 기억이 나는데로 적었으며, 틀린 내용이 있을 수 있습니다.
- 지금 글을 쓰는 시점에 spring boot 3.2.2 가 나왔습니다.



# Parameter Name Discovery

- spring boot 3.2.1 적용하면서 bean 을 가져올 때(생성자 주입 또는 Autowired 등), 빈이 2개 이상 발견됐다는 에러가 발생했습니다. 
- 해결방안은 Qualified 를 사용해서 bean 을 명시적으로 주입합니다.
- 아래 패치 내용으로 인해 그런 것이라 추정됩니다. 확실하지 않습니다.

```tex
The version of Spring Framework used by Spring Boot 3.2 no longer attempts to deduce parameter names by parsing bytecode. If you experience issues with dependency injection or property binding, you should double check that you are compiling with the -parameters option. See this section of the "Upgrading to Spring Framework 6.x" wiki for more details.
```


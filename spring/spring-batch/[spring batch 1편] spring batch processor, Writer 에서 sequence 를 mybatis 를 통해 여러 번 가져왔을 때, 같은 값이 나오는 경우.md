> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.



> 목차는 [spring batch 목차](https://insanelysimple.tistory.com/category/Spring/batch) 에 있습니다.




# [spring batch 1편] spring batch processor, Writer 에서 sequence 를 mybatis 를 통해 여러 번 가져왔을 때, 같은 값이 나오는 경우



# 현상

-   spring-batch chunked tasklet 의 processor 또는 Writer 에서 sequence 를 mybatis 를 통해 nextval 로 여러 번 가져왔을 때, 같은 값이 나오는 경우가 발생했습니다.



# 원인

-   같은 트랜잭션일 경우에만 이런 현상이 나옵니다.

- sequence.nextval 을 가져올 때, 서버 쪽에 commit 명령어를 날려줘야 다음 값을 가져오도록 세팅되는 것이라 추정하고 있습니다.



# 해결방안

-   mapper 에 @Transactional(propagation = Propagation.REQUIRES\_NEW) 설정하면 됩니다. (임시 방편)

```java
@Mapper
public interface TestMapper {
	@Transactional(propagation = Propagation.REQUIRES_NEW) 
	int findSequence();
}
```

-   이렇게 하면 성능 이슈가 있으니 새로운 방법을 찾아봐야할 것 같습니다.
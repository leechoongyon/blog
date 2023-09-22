## spring boot, spring batch 정리 - 3편 repeat step 예제 (파라미터만 변경해서 실행 )

## repeat step 예제 (파라미터만 변경해서 실행)
- 동일한 step 을 파라미터만 변경해서 반복해서 실행
- 반복되는 횟수는 동적으로 제어.

## 반복 횟수 job configuration
- 아래 내용 중 중요한 것은 COMPLETED 와 FAILED 에 따라 step 이 계속실행될지 종료될지를 결정하는 것.
{% gist leechoongyon/abe76918615f534b0d03999e480b955e %}

## source
- [spring-boot-batch-example source](https://github.com/leechoongyon/spring-boot-example/tree/master/spring-boot-batch-example) 참고

## reference
- https://jojoldu.tistory.com/328 참고
> source 는 [Github](https://github.com/leechoongyon/spring-boot-example-3.0.1) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 20편] spring validate custom (spel 이용해서 validate 수행할지 결정)



# spring validation custom

- spel (Spring Expression Language) 을 이용해서 validate 을 수행할지 안할지를 결정하는 예제입니다.
- 소스를 간략히 설명하면 ItemConstraint 를 선언하면 validation 이 수행됩니다. 이 때, condition 조건에 따라 validate 를 수행할지 안할지를 결정할 수 있습니다.
- condition 조건은 spel (Spring Expression Language) 을 이용해서 제한적으로 체크합니다. (추후 보완하겠습니다.) 






# source

- 테스트 예시를 하나 들어보면 testItemConstraintValidatorTest01 테스트는 조건을(condition) 먼저 체크합니다.

- condition = "id =='ItemId'"  --> parameter id 값이 "ItemId" 일 경우 validate 를 진행하겠다는 것입니다.
- 비교할 필드는 "desc" 라는 필드이며, 해당 필드의 값이 compareValue 와 동일할 경우 validate 통과합니다.



<script src="https://gist.github.com/leechoongyon/ed3dda3aa8bac84b893cace5d067af78.js"></script>


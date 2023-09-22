> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.



> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 11편] spring boot test h2 sequence, function 테스트



# Spring boot h2 환경에서 function 사용 방법

- spring boot test (h2 inmemory 환경) 를 작성하는 도중 function 을 테스트해야할 때가 있습니다.
- 그럴 때 어떻게 function 을 테스트 하는지 정리했습니다.
- 아래 예제는 nativeQuery (oracle, mysql) 을 사용했을 때, H2 에서는 function 을 어떻게 사용할 수 있는지에 대해 정리했습니다.



# Source

<script src="https://gist.github.com/leechoongyon/09a412d48d8bf0d50b023cd38a16ae80.js"></script>



## reference

- http://www.h2database.com/html/features.html#user_defined_functions

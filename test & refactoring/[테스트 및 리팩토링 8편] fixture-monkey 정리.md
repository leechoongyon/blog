> source 는 [Github](https://github.com/leechoongyon/spring-boot-example-3.0.1) 에 있습니다.



> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.




# [테스트 및 리팩토링 8편] fixture-monkey 정리



# fixture-monkey

- 테스트 데이터를 만들어주는 오픈소스입니다.
- 나중에 찾아볼 용도로 정리했습니다. 



# source

- 자바, gradle 환경 에서 실행했습니다. (Java17)
- gradle 설정하는데 시간이 오래걸렸는데 처음에는 fixture-monkey-starter 만 설치하면 관련 plugin 은 전부 import 되는줄 알았는데, 그게 아니였습니다. 그래서 그냥 서드 파티 관련 gradle 설정을 전부 넣어버렸습니다.
- 아래 중에 필요한 서드 파티 설정만 추가하시면 될 것 같습니다.  

`주의사항으로는 FixtureMonkey 를 통해 데이터를 만들 때, 사용한 annotation 이 Jakarta 이면 JakartaValidationPlugin 을 써줘야 하고, Javax annotation 을 썼으면 JavaxPlugin 을 사용해야합니다.`



<script src="https://gist.github.com/leechoongyon/0f0b41be7dac18994b0783b6a2dd52f8.js"></script>



# Reference

- https://naver.github.io/fixture-monkey/kr/docs/v0.5/getting-started/


















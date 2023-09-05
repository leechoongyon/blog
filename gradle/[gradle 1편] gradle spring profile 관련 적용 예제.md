> 목차는 [gradle 목차](https://insanelysimple.tistory.com/category/gradle) 에 있습니다.



# [gradle 1편] gradle spring profile 관련 적용 예제



> 나중에 볼려고 정리했습니다.



# 상황

- gradle 에서 spring profile 관련 내용을 적용하고 싶은 순간이 있습니다.
- 예를 들면, prod, dev, local profile 이 있다고 가정했을 때, application.yml 설정을 달리 가져가고 싶은 필요가 생길 수 있습니다.



# 예시

- SPRING_PROFILES_ACTIVE=local  또는 -Dspring.profiles.active=local 을 통해 spring profile 을 적용합니다. 



<script src="https://gist.github.com/leechoongyon/852d5cfca3c30563a5a2a3807e93a15a.js"></script>

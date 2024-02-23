>  source 는 [Github](https://github.com/leechoongyon/Java16Examples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 11편] Extend enum (EnumMap, interface, function)



# Extend enum (EnumMap, interface, function)

- EnumMap, interface, function 을 통해 enum 을 확장하는 법에 대해 정리했습니다.



# EnumMap 이란?

- EnumMap은 Map 에서 키를 enum 타입만을 사용하도록 하는 구현체입니다.





# source

- enum 이 있고, enum map 을 EnumMap 으로 초기화하고 있습니다.
- EnumMap 은 MemberTypeEnum 을 Key, Supplier<String> 을 value 로 설정했습니다.
- 아래 소스가 의미하는건 Enum 타입에 따라 구현내용을 달리 줘서 유연성을 가질 수 있습니다.



<script src="https://gist.github.com/leechoongyon/1cd0365df34d890eea0840155c3482ec.js"></script>



# Reference

- https://www.baeldung.com/java-extending-enums

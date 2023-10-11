> source 는 [Github](https://github.com/leechoongyon/JavaExamples) 에 있습니다.



> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.



# [테스트 & 리팩토링 6편] 팩토리 패턴 if else 줄이기 (map, functional interface 사용)



# 팩토리 패턴이란?

- 객체를 생성해주는 패턴 중 하나 입니다.
- 객체를 생성하는 인터페이스를 미리 정의하고, 인스턴스를 만들 클래스의 결정을 서브 클래스에서 결정합니다.

- 아래 소스는 팩토리 패턴으로 구현한 소스이며, 리팩토링 전입니다.

<script src="https://gist.github.com/leechoongyon/af363fb4c3463366b93ed1f992f28775.js"></script>



# If else 를 줄이는 방법

- 아래와 같이 Map 에 집어넣고 데이터를 가져온다면 if else 를 줄일 수 있습니다.

<script src="https://gist.github.com/leechoongyon/4b3b64da26a803aa77b222012d535c1a.js"></script>



# interface 구현 없이 Functional interface 사용

- Supplier<T> functional interface 는 매개변수 없이 T 를 리턴합니다.
  - T get()

<script src="https://gist.github.com/leechoongyon/0fc340145f34b79f605cb00c498cd5e9.js"></script>

>  source 는 [Github](https://github.com/leechoongyon/Java16Examples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 15편] 자바 컴퍼지트 패턴



> 예전에 공부해둔 내용을 remind 할려고 다시 작성했습니다.





# 자바 컴퍼지트 패턴

- 전체-부분의 관계를 갖는 객체들 사이의 프로그래밍을 할 때 유용합니다.

- 아래와 같이 전체-부분 관계를 갖는 것을 컴포지트 패턴을 사용했을 시, 제일 큰 장점은 변화에 유연한 구조가 됩니다.
- 재료가 추가돼도 HamBurger 재료 List 에 객체를 넣어주면 됩니다. 즉, 영향도가 적습니다.



### Example

- 햄버거가 있고 햄버거에는 여러 재료가 들어갑니다. (햄버거 - 전체 , 재료 - 부분) 
- 아래 소스 예제는 햄버거의 칼로리를 구하고 싶습니다. 







# Source

<script src="https://gist.github.com/leechoongyon/bb9c85a5d5589eae119f41bbf94767d8.js"></script>

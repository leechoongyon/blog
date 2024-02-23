>  source 는 [Github](https://github.com/leechoongyon/Java16Examples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 14편] 자바 스트레지트 패턴



> 예전에 공부해둔 내용을 remind 할려고 다시 작성했습니다.



## 자바 스트레지트 패턴

-   알고리즘 전략을 유연하게 변경할 수 있는 패턴입니다.
-   아래 소스를 간략히 설명하면 Player 의 play 행동을 전략에 맞게 선택할 수 있습니다.
-   main 에서 player 는 play 를 할 때, BaseBall 을 할지, soccer 를 할지 전략적으로 선택할 수 있습니다.
-   변경에 유연화된 구조입니다.



# Source

- <script src="https://gist.github.com/leechoongyon/c6f1dd1b5b1080a1655c59df3530a6da.js"></script>

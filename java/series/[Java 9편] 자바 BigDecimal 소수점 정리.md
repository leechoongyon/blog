>  source 는 [Github](https://github.com/leechoongyon/JavaExamples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 9편] 자바 BigDecimal 소수점 계산 주의사항



#  BigDecimal 소수점 계산 주의사항

- 자바에서 소수점 계산을 할 때, 정확한 계산을 위해 BigDecimal 을 사용합니다.
- 이 때, 소수점 계산을 할 때, BigDecimal 의 construct 을 사용할 수 있습니다.
- construct 에 들어가는 값이 어떤 값이냐에 따라 원치 않는 결과가 나올 수 있습니다.





### double construct BigDecimal

- 아래 예시를 보면 construct 에 double 이 들어가면 원치않는 값이 나옵니다.
  - double construct bigdecimal 은 정확하게 값을 표현할려고 합니다. 그렇기에 아래와 같은 결과가 나옵니다.
  - 0.1 을 double 로 정확하게 표현할 수 없습니다.



### string construct BigDecimal

- 아래 예시를 보면 construct 에 string 이 들어가면 원하는 값이 나옵니다.



# 예시

<script src="https://gist.github.com/leechoongyon/814c37e343c0a08d8116996d2687a399.js"></script>



# 결론

- BigDecimal 을 통해 소수점 계산을 할 때, string construct 을 사용해야 원하는 값을 얻을 수 있습니다. 



# Reference

- https://stackoverflow.com/questions/7186204/bigdecimal-to-use-new-or-valueof/7186298#7186298
> source 는 [Github](https://github.com/leechoongyon/spring-boot-batch-example) 에 있습니다.



> 목차는 [spring batch 목차](https://insanelysimple.tistory.com/category/Spring/batch) 에 있습니다.




# [spring batch 2편] FlatFileItemWriter Fixed length format 정리 (Header, Footer 사용)

- 파일을 쓸 때, Fixed length 와 Formatter 를 조합하는 예제에 대해서 정리했습니다.
- DTO to FixedLength format 으로 변환해서 파일을 쓰는 예제입니다. 파일을 쓸 때, Java 의 Formatter 를 사용할 수 있습니다.




# Source

- 아래 예제에서 formatted() 옵션을 주면 Java 의 Formatter 를 사용할 수 있습니다.
- 자바의 Formatter 는 [링크](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html) 를 참고하시면 됩니다.
- Formatter 를 이용한다면 왼쪽 정렬, 오른쪽 정렬, 값이 비어있다면 space, '0' 으로 채우는 등의 작업이 가능합니다.

<script src="https://gist.github.com/leechoongyon/ecb549485c0a4b2e65ac42801d021bc2.js"></script>



# Reference

- [https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#flatFileItemWriter](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#flatFileItemWriter)
- [https://godekdls.github.io/Spring Batch/itemreadersanditemwriters/](https://godekdls.github.io/Spring%20Batch/itemreadersanditemwriters/)
- [https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html)

> source 는 [Github](https://github.com/leechoongyon/JavaExamples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 2편] stream 내에서 CheckedException, RuntimeException 처리


## source code
- stream 내에서 CheckedException 과 RuntimeException 처리 관련 코드 예시에 대해서 정리했습니다.


```java
public class StreamWithExceptionTest {

    @Test
    public void stream_내에서_RuntimeException_테스트() {
        String[] strings = new String[] {"hello", "world", "hi"};
        List<String> list = Arrays.asList(new String[]{"abcde", "test", "abc"});

        Assertions.assertThrows(IllegalStateException.class, () -> {
            Arrays.asList(strings).stream().filter(str -> !list.contains(str))
                    .findFirst().ifPresent(str -> {
                        throw new IllegalStateException();
                    });
        } );
    }

    @Test
    public void stream_내에서_CheckedException_발생_테스트() {
        List<String> result =
                Arrays.stream(new String[]{"테스트1", "테스트2"})
                        .map(wrap(s -> URLEncoder.encode(s, Charset.defaultCharset().name())))
                        .collect(Collectors.toList());
        Assertions.assertEquals(2, result.size());
    }

    @FunctionalInterface
    private interface FunctionWithException<T, R, E extends Exception> {
        R apply(T t) throws E;
    }

    private <T, R, E extends Exception> Function<T, R> wrap(FunctionWithException<T, R, E> f) {
        return arg -> {
            try {
                return f.apply(arg);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        };
    }
}
```


## reference
- https://dzone.com/articles/exception-handling-in-java-streams
- https://www.oreilly.com/content/handling-checked-exceptions-in-java-streams
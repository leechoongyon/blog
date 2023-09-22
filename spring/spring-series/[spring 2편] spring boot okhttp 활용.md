> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.

> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.

# [spring 2편] spring boot okhttp 활용


## okhttp 란 무엇인가?
- http 통신을 편리하게 사용할 수 있도록 도와주는 라이브러리입니다.


## build.gradle
- gson 은 json 을 변환해주는 라이브러리인데, okhttp 와 조합해서 사용하면 편리합니다. (json + http 와 같은 통신 방법)

```groovy
// okhttp
implementation("com.squareup.okhttp3:okhttp:4.9.1")

// GSON
implementation group: 'com.google.code.gson', name: 'gson', version: '2.8.5'
```


## okhttp example source

```java
public class OkhttpUtils {

    /**
     * post 호출
     * @param url
     * @param body
     * @param mediaType
     * @return
     */
    public static String post(String url, Map<String, String> body, MediaType mediaType) {
        OkHttpClient client = new OkHttpClient();
        RequestBody requestBody = RequestBody.create(new Gson().toJson(body), mediaType);
        Request request = new Request.Builder()
                    .url(url)
                    .post(requestBody)
                    .build();
        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        } catch (IOException e) {
            log.error(e.getMessage(), e);
            throw new RuntimeException(e);
        }
    }

    /**
     * post 호출
     * @param url
     * @param body
     * @param mediaType
     * @return
     */
    public static String post(String url, Map<String, String> paramHeader, Map<String, String> body, MediaType mediaType) {
        if (ObjectUtils.isEmpty(paramHeader)) {
            throw new IllegalArgumentException("paramHeader is null");
        }
        OkHttpClient client = new OkHttpClient();
        RequestBody requestBody = RequestBody.create(new Gson().toJson(body), mediaType);
        Request request = new Request.Builder()
                .url(url)
                .post(requestBody)
                .headers(Headers.of(paramHeader))
                .build();
        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        } catch (IOException e) {
            log.error(e.getMessage(), e);
            throw new RuntimeException(e);
        }
    }

    /**
     * GET METHOD
     * @param url
     * @return
     */
    public static String get(String url) {
        OkHttpClient client = new OkHttpClient();
        Request request = new Request.Builder()
                .url(url)
                .get()
                .build();
        try (Response response = client.newCall(request).execute()) {
            return response.body().string();
        } catch (IOException e) {
            log.error(e.getMessage(), e);
            throw new RuntimeException(e);
        }
    }
}

```


## test source
```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class OkhttpExampleApiControllerLiveTest {

    /**
     * 내장 톰캣 띄우고 테스트 해야 함. ExampleApplication 실행해야 함.
     * @throws Exception
     */
    @Test
    public void okhttp_post_메소드_테스트() throws Exception {
        String url = "http://localhost:8080/api/v1/okhttp";

        // body
        Map<String, String> body = new HashMap<>();
        body.put("name", "test");

        String result = OkhttpUtils.post(url, body, MediaType.parse("application/json; charset=utf-8"));
        OkhttpExampleDto.PostResponse response =
                new Gson().fromJson(result, OkhttpExampleDto.PostResponse.class);
        Assert.assertEquals("test", response.getName());
    }

    /**
     * 내장 톰캣 띄우고 테스트 해야 함. ExampleApplication 실행해야 함.
     * @throws Exception
     */
    @Test
    public void okhttp_get_메소드_테스트() throws Exception {
        String url = "http://localhost:8080/api/v1/okhttp/1";
        String result = OkhttpUtils.get(url);
        Assert.assertEquals("1", result);
    }
}
```

## reference
- https://www.baeldung.com/guide-to-okhttp
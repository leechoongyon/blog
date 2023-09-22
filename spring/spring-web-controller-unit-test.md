## spring web controller 단위 테스트 하는 방법
- controller 만 단위 테스트 하는 방법을 정리했습니다.

## source

```java

package com.example.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    @GetMapping("/test")
    public String test() {
        return "test";
    }
}



package com.example.controller;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import static org.junit.Assert.*;

/**
 * @RunWith는 jUnit 테스트 실행방법을 확장할 때 사용하는 어노테이션이다. SpringRunner 을 선언함으로써
 * 테스트에만 필요한 빈들만 application context 에 로딩하므로 빠른 테스트가 가능합니다.
 */
@RunWith(SpringRunner.class)

/**
 * Web Test 전용 어노테이션입니다.
 * 주로 controller 테스트에 사용됩니다.
 */
@WebMvcTest(controllers = TestController.class)
public class TestControllerTest {

    @Autowired
    private MockMvc mockMvc;    // 스프링 MVC의 동작을 재현할 수 있는 클래스입니다.

    @Test
    public void controller_test메소드_테스트() throws Exception {
        String s = "test";
        mockMvc.perform(get("/test"))   // get 요청을 호출합니다.
                .andExpect(status().isOk())         // HTTP status 가 200 인지 검증합니다.
                .andExpect(content().string(s));    // response body 가 "test" 로 return 했는지 검증합니다.
    }
}


```
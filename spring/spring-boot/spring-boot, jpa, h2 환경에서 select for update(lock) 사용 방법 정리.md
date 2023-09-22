> source 는 [Github](https://github.com/leechoongyon/spring-boot-jpa-example) 에 있습니다.

## spring boot h2 환경에서 jpa 명시적 락 사용하는 예제
- spring boot h2 환경에서 select for update 를 사용하는 방법에 대해 정리했습니다.
- 결론부터 얘기하면 repository interface 에 메소드에  @Lock(LockModeType.PESSIMISTIC_WRITE) 선언하면 select for update 가 설정 됩니다.
- 예제에 대해 간략하게 설명하면 deposit(입금), withdraw(인출) 기능이 있으며, 여러 요청이 들어왔을 때, 동시성이 보장되는지 확인합니다.


```java

package com.example.jpa.api;

import com.example.jpa.dto.BankAccountRequest;
import com.example.jpa.service.BankAccountService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequiredArgsConstructor
public class BankAccountApiController {

    private final BankAccountService bankAccountService;

    @PutMapping("/account/withdrawal/noLock")
    public ResponseEntity withdraw(@RequestBody BankAccountRequest request) {
        bankAccountService.withdraw(request);
        return ResponseEntity.ok().build();
    }

    @PutMapping("/account/withdrawal/useLock")
    public ResponseEntity withdrawalForUpdate(@RequestBody BankAccountRequest request) {
        bankAccountService.withdrawForUpdate(request);
        return ResponseEntity.ok().build();
    }
}

```


```java

package com.example.jpa.service;

import com.example.jpa.domain.BankAccount;
import com.example.jpa.dto.BankAccountRequest;
import com.example.jpa.repository.BankAccountRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Slf4j
@Service
@RequiredArgsConstructor
public class BankAccountService {

    private final BankAccountRepository bankAccountRepository;

    public void withdraw(BankAccountRequest request) {
        BankAccount bankAccount =
                bankAccountRepository.findByAccountNo(request.getAccountNo());
        bankAccount.withdraw(request.getAmount());
    }

    @Transactional
    public void withdrawForUpdate(BankAccountRequest request) {
        BankAccount bankAccount =
                bankAccountRepository.findByAccountNoForUpdate(request.getAccountNo());
        bankAccount.withdraw(request.getAmount());
    }
}

```

```java

package com.example.jpa.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter @Setter
@NoArgsConstructor
public class BankAccountRequest {
    private String accountNo;
    private Long amount;

    @Builder
    public BankAccountRequest(String accountNo, Long amount) {
        this.accountNo = accountNo;
        this.amount = amount;
    }
}


```

```java

package com.example.jpa.repository;

import com.example.jpa.domain.BankAccount;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import javax.persistence.LockModeType;


public interface BankAccountRepository extends JpaRepository<BankAccount, Long> {

    BankAccount findByAccountNo(String accountNo);

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select b from BankAccount b where b.accountNo = :accountNo")
    BankAccount findByAccountNoForUpdate(@Param("accountNo") String accountNo);
}

```

```java

package com.example.jpa.domain;

import com.example.jpa.dto.BankAccountRequest;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
@Getter
@ToString
@NoArgsConstructor
public class BankAccount {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String accountNo;
    private Long amount;

    @Builder
    public BankAccount(String accountNo, Long amount) {
        this.accountNo = accountNo;
        this.amount = amount;
    }

    public static BankAccount of(BankAccountRequest request) {
        return BankAccount.builder()
                .accountNo(request.getAccountNo())
                .amount(request.getAmount())
                .build();
    }

    public void withdraw(Long amount) {
        this.amount = this.amount - amount;
    }

    public void deposit(BankAccountRequest request) {
        this.amount = this.amount + request.getAmount();
    }
}

```


```java

package com.example.jpa.api;

import com.example.jpa.domain.BankAccount;
import com.example.jpa.dto.BankAccountRequest;
import com.example.jpa.repository.BankAccountRepository;
import com.example.jpa.service.BankAccountService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

import static org.hamcrest.CoreMatchers.is;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class BankAccountApiControllerTest {

    @Autowired
    private MockMvc mockMvc;

    private String ACCOUNT_NO = "123-123-123";

    @Autowired
    private ObjectMapper objectMapper;

    @Autowired
    private BankAccountService bankAccountService;

    @Autowired
    private BankAccountRepository bankAccountRepository;

    @Before
    public void setUp() {
        BankAccount bankAccount = BankAccount.builder()
                .accountNo(ACCOUNT_NO)
                .amount(1000000L)
                .build();
        bankAccountRepository.save(bankAccount);
    }

    @After
    public void clean() {
        bankAccountRepository.deleteAll();
    }

    @Test
    public void 인출_락없이_테스트() throws Exception {

        AtomicInteger successCount = new AtomicInteger();
        int numOfExecute = 100;
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        CountDownLatch countDownLatch = new CountDownLatch(numOfExecute);

        // given
        for (int i = 0 ; i < numOfExecute ; i++) {
            executorService.execute(() -> {
                try {
                        BankAccountRequest request = BankAccountRequest.builder()
                                .accountNo(ACCOUNT_NO)
                                .amount(10000L)
                                .build();
                        bankAccountService.withdraw(request);

                        successCount.getAndIncrement();
                } catch (Throwable th) {
                    System.err.println(th);
                }

                countDownLatch.countDown();
            });
        }

        countDownLatch.await();

        // when
        BankAccount bankAccount = bankAccountRepository.findByAccountNo(ACCOUNT_NO);

        // then
        Assert.assertThat(successCount.get(),is(numOfExecute));
        Assert.assertNotEquals(bankAccount.getAmount(),is(0L));
    }

    @Test
    public void 인출_락걸고_테스트() throws Exception {
        int numOfExecute = 100;
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        CountDownLatch countDownLatch = new CountDownLatch(numOfExecute);

        for (int i = 0 ; i < numOfExecute ; i++) {
            executorService.execute(() -> {
                try {
                    BankAccountRequest request = BankAccountRequest.builder()
                            .accountNo(ACCOUNT_NO)
                            .amount(10000L)
                            .build();
                    String content = objectMapper.writeValueAsString(request);

                    mockMvc.perform(MockMvcRequestBuilders.put("/account/withdrawal/useLock")
                            .content(content)
                            .contentType(MediaType.APPLICATION_JSON_VALUE)
                            .accept(MediaType.APPLICATION_JSON))
                            .andExpect(status().isOk())
                            .andDo(MockMvcResultHandlers.print());
                } catch (Throwable th) {
                    System.err.println(th.getMessage());
                }
                countDownLatch.countDown();
            });
        }

        countDownLatch.await();

        // then
        BankAccount bankAccount = bankAccountRepository.findById(1L).get();
        Assert.assertThat(bankAccount.getAmount(),is(0L));
    }
}


```



## 결론
- 위 테스트를 통해 계좌에서 금액을 인출할 때, select for update 를 통해 동시성을 보장하는 방안을 테스트 해봤습니다.
- 이 외에도 redis 를 이용해서 lock 을 잡는 방법도 있습니다.
- '인출_락걸고_테스트' 테스트케이스의 경우 controller 를 호출해서 동시성 테스트를 수행했습니다. 그 이유는 @Lock 해당 어노테이션을 사용할 때, 트랜잭션이 존재해야하는데 Service 단에서 호출할 경우 Transaction 이 생성되지 않아 no transaction 에러가 발생합니다. 
  - 위 에러를 해결하기 위해 EntityManager 를 직접 선언해서 트랜잭션을 제어하는 방법과 controller 를 호출해서 처리하는 방법이 있는데 controller 를 호출해서 처리하는 방안으로 해결했습니다.
- 추가적으로 여러 request 요청이 들어와서 select for update 를 잡을 떄, 스레드가 대기하기 떄문에 지연이 발생할 수 있습니다. 너무 오래 지연이 발생하면 성능에 영향을 줄 수 있으니 for update wait 3 과 같이 몇 초 동안 기다린 후, Exception 을 발생시키는 방안을 고려해보면 좋을 것 같습니다.
  - 오라클의 경우 select for udpate wait 3 명령어가 먹히며, 다른 DBMS 는 확인해봐야 합니다.



## Reference
- [https://junhyunny.github.io/spring-boot/jpa/junit/jpa-pessimitic-lock/](https://junhyunny.github.io/spring-boot/jpa/junit/jpa-pessimitic-lock/)
> source 는 [Github](https://github.com/leechoongyon/spring-boot-example) 에 있습니다.

## spring boot h2 환경에서 function 사용 방법
- test 하는 도중 function 을 테스트해야할 때가 있습니다.
- 그럴 때 어떻게 function 을 테스트 하는지 정리했습니다.
- 아래 예제는 nativeQuery (oracle, mysql) 을 사용했을 때, H2 에서는 function 을 어떻게 사용할 수 있는지에 대해 정리했습니다.

## source

- FunctionTestDomainRepositoryTest, Domain, Repository

```java

@RunWith(SpringRunner.class)
@DataJpaTest
public class FunctionTestDomainRepositoryTest {
    @Autowired
    private FunctionTestDomainRepository functionTestDomainRepository;

    @Test
    public void H2_FUNCTION_TEST() throws Exception {
        String result = functionTestDomainRepository.functionTest("test");
        Assert.assertThat(result, is("test"));
    }
}

// Repository
public interface FunctionTestDomainRepository extends JpaRepository<FunctionTestDomain, Long> {
    @Query(value = "SELECT FUNCTION_TEST(:str) from dual", nativeQuery = true)
    String functionTest(String str);
}


// Domain
@Entity
public class FunctionTestDomain {
    @Id
    @GeneratedValue
    private Long id;
}


```

---

- H2FunctionUtils

```java

public class H2FunctionUtils {
    public static String functionTest(String str) {
        return str;
    }
}

```

---

- functionTest.sql

```sql

CREATE ALIAS IF NOT EXISTS FUNCTION_TEST FOR "com.example.utils.H2FunctionUtils.functionTest"

```


## reference
- http://www.h2database.com/html/features.html#user_defined_functions
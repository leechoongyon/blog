## 1. @Transactional, RuntimeException, rollback marking 설명

## 2. @Transactional 일 때, RuntimeException 이 났는데 try catch 를 이용해서 Exception 을 안냈을 때 어떻게 되는가?
- 결론부터 얘기하면 Transactional annotation 안에서 RuntimeException 이 일어나면 (예외를 일으키지 않아도), rollback marking 이 찍힌다.


## 3. source
- 소스를 설명하면, ServiceA.call() --> ServiceB.call() 를 호출
- ServiceA.call 과 ServiceB.call() 은 Transactional 로서 같은 트랜잭션으로 묶일 것이다.
- serviceB.call() 은 save 한 후, 갑자기 RuntimeException 이 발생했다.
- serviceA.call() 에서는 RuntimeException 을 catch 해서 로그만 찍고 넘어가기에, save 는 commit 이 되리라 생각하지만 rollback 이 되버렸음.
- Spring 에서는 @Transactional 안에서 RuntimeException (예상치 못한 Exception) 이 발생하면 Rollback marking 을 찍는다고 함.
- 그렇기에 아래와 같이 RuntimeException 을 catch 해서 예외를 먹었어도, rollback 이 이루어짐.
  
```java
@Service
public class ServiceA {
        
    @Autowired
    private ServiceB serviceB;

    @Transactional
    public void call() {
    
        try {
            serviceB.call();
        } catch (RuntimeException e) {
            log.error(e.getMessage(), e);
        }
    }
}

@Service
public class ServiceB {
    
    @Transactional
    public void call() {
        xxxRepository.save(...);
        
        // 이 부분에서 Exception 이 발생됐다고 가정.    
    }
}
```

## 4. 정리
- 다시 정리하면, @Transactional 안에서 RuntimeException (예상치 못한 Exception) 이 발생하면 rollback marking 이 되고 rollback 이 된다.
- 위에 내용이 기본 정책이고, rollback 을 안하고 싶으면 옵션을 사용해서 처리 가능.
- 또한, CheckedException 은 위 처럼 하면 rollback 이 안된다. (예상한 Exception 이니)

## 5. Reference
- [https://woowabros.github.io/experience/2019/01/29/exception-in-transaction.html](https://woowabros.github.io/experience/2019/01/29/exception-in-transaction.html)
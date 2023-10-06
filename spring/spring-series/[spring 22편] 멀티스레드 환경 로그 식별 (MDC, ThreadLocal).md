> 목차는 [spring series 목차](https://insanelysimple.tistory.com/category/Spring/series) 에 있습니다.



# [spring 22편] 멀티스레드 환경 로그 식별 (MDC, ThreadLocal)



> 나중에 볼려고 정리했습니다.





# 상황

- 멀티 스레드 환경에서 로그를 찍다보면 식별이 안되는 경우가 있습니다. 멀티스레드 환경에서는 컨텍스트 스위칭이 일어나 실행이 되기 때문에 로그가 섞이기 때문입니다.

- 이를 해결하는 방법중 Correlation ID 라는 것이 있습니다.



# Correlation ID, ThreadLocal

- 위와 같은 상황을 해결하기 위해 같은 스레드 요청에 대해 특정 ID 를 부여해 grouping 할 수 있습니다. 이를 Correlation ID 라고 합니다.

- 자바에서는 이를 구현하기 위해 ThreadLocal 을 제공하며, ThreadLocal 은 동일한 스레드내에서 자원을 공유할 수 있는 변수입니다.



# MDC (Mapping Diagnostic Context)

- 위의 개념을 구현한 로그 프레임워크가 MDC 입니다.



### 사용 예시

- 아래와 같이 request_id 를 선언하고 로그를 찍게 되면 request_id (=java uuid) 로 거래를 식별할 수 있습니다.

```java
 public static void main( String[] args ) {
    	MDC.put("request_id", "java uuid");
   
      log.info("Hello World...");
   
    	MDC.clear();
 }
```





# 추가 내용 



### 분산 환경 로그

- 위 내용은 단일 서버에서 사용 가능한 방식입니다. 만약 분산 환경이라면 위 방식을 응용해서 request_id 와 같은 global uuid 를 key 로 해서 로그를 찍고, 이를 분산 로깅 프레임워크로 수집을 한다면 분산 환경에서도 로그를 찾을 수 있습니다.



### spring 환경 MDC 활용법

- filter 에서 MDC 를 put 한 후, 서비스를 실행한다면 서비스 요청에 대한 trace 를 추적할 수 있습니다.
- Filter 의 제일 처음 시작하는 것이 좋으며, filter -> 서비스 -> filter 마지막에 MDC.clear 를 해줘야합니다. 안그러면 데이터가 계속해서 메모리에 남아있기에 위험합니다. 



### Reactive 환경에서 ThreadLocal

- 위 내용은 일반적인 Spring 환경에서 스레드 동작 방식에서 적용 가능한 방법입니다. (요청이 들어올 때마다 Thread 할당)
- 만약 Reactive 환경이라면 worker thread 로 동작할텐데, 이에 대한 처리를 해줘야 정상적으로 동작할 것 입니다. 



### 비동기 환경에서 (TaskExecutor) ThreadLocal

- 일반적인 Spring 환경은 request 당 Thread 를 할당하는 방식입니다.
- request 요청이 들어오고 Thread 를 할당받았는데, TaskExecutor 를 사용하여 별도 스레드를 호출한다고 가정하겠습니다.
- Thread 1 (main) -> Thread 10 (sub) 이렇게 되며, 로깅을 할 경우 데이터가 꼬이거나 사라질 수 있습니다.
- 이를 위해 Spring 4.3 이상부터 TaskDecorator 라는 것이 나왔는데 이것은 Thread 를 생성해서 실행할 때, customizing 할 수 있는 것입니다.



##### source

- 아래와 같이 스레드를 실행하기 전 MDC 를 copy 해서 새로운 스레드에 할당시키는 것입니다.
- 메인 스레드의 내용을 자식 스레드에 할당하는 것이기에 side-effect 도 없을 것이라 생각됩니다.

```java
public class CustomTaskDecorator implements TaskDecorator {
		@Override
		public Runnable decorate(Runnable task) {
			Map<String, String> copiedContextMap = MDC.getCopyOfContextMap();
			return () -> {
				MDC.setContextMap(copiedContextMap);
				task.run();
			};
		}
}
```









# Reference

- https://mangkyu.tistory.com/266
- https://blog.gangnamunni.com/post/mdc-context-task-decorator/

> 목차는 [Infra 목차](https://insanelysimple.tistory.com/category/infra) 에 있습니다.



# [Infra 2편] JIT, HotSpot 정리



# JIT 란?

- just-in-time compilation 의 약자이며, 프로그램을 실행하는 시점에 기계어로 번역하는 컴파일 기법입니다.



### 정적 , 동적 컴파일 방식

- 기계어로 번역하는 방법에는 2가지가 있습니다. 정적, 동적
- 정적 컴파일은 실행하기 전에 프로그램 코드를 기계어로 번역하는 것입니다.
- 동적 컴파일은 프로그램 코드를 읽으면서 실행하는 것입니다.



### JIT 는 어떤 컴파일 방식인가?

- JIT 는 정적 + 동적 컴파일 방식을 합친 방식이며, 다음과 같은 FLOW 를 통해 거칩니다.
- Java Code --> 컴파일러 --> Java byte code (JVM 이 해석할수 있는) --> JVM(JIT 가 관여) --> 기계어 --> OS 상에서 수행



### JIT 특징?

- 반복적으로 사용되는 코드는 캐시를 통해 빠른 성능을 보입니다. (정적 컴파일러 방식 이용)
- 처음에는 분석하고 변환해야하니 시간이 조금 걸립니다.







# HotSpot

- 빈번하게 발생하는 지점을 찾아서 해당 부분에서는 JIT 컴파일러를 사용하는 방법입니다. 프로파일링을 통해 핫스팟을 찾아내고, 해당 부분에 대한 네이티브 코드를 생성합니다. 네이티브 코드를 생성하는 방법에서 Client와 Server라는 두가지가 있습니다.
- HotSpot Client
  - 프로그램의 시작 시간을 줄이는데 focusing 하고 있습니다.

- HotSpot Server
  - 전체적인 성능 최적화에 관점을 둡니다.



# Reference

- https://applefarm.tistory.com/52?category=930605
- https://velog.io/@aki/HotSpot-JVM


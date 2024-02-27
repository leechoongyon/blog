>  source 는 [Github](https://github.com/leechoongyon/Java16Examples) 에 있습니다.



> 목차는 [Java series](https://insanelysimple.tistory.com/category/Java/series) 에 있습니다.



# [Java 16편] Native Memory Tracking

> 나중에 찾아보기 위해 작성했습니다. 



# Native Memory Tracking

1.JVM OPTION 에 아래 내용을 추가해줍니다.

- -XX:NativeMemoryTracking=summary or -XX:NativeMemoryTracking=detail
- summary 는 요약해서 보여주고, detail 자세하게 보여줍니다.

2.baseline 을 설정합니다. baseline 용도는 나중에 메모리 사용량을 비교하기 위한 기준점입니다.

- 예를 들면, 현재 heap 사용량이 100MB 인데, 나중에 heap 사용량이 어느정도 늘어났거나 줄었는지 baseline 을 기준으로 확인할 수 있습니다. 

```bash
jcmd <pid> VM.native_memory baseline
```

3.jcmd 명령어 수행

```bash
jcmd <pid> VM.native_memory detail.summary
```



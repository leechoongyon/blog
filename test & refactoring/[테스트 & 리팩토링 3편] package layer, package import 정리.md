> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.



# [테스트 & 리팩토링 3편] package layer, package import 정리


## package layer
- 상위 레이어는 하위 레이어를 참조하도록 하며, 하위레이어는 상위레이어를 참조하지 않도록 하는게 좋습니다.
  - 소스 코드의 응집성과 추후 유지보수 확장성을 위해 이 규칙을 지키는게 좋습니다.
- 예를 들면 아래와 같습니다. controller 에 있는 HelloDto.Request 라는 것을 그대로 service 쪽에 넘기면 추후 HelloDto.Request 가 변경될 경우 HelloService 에 영향을 줄 수 있습니다.
- 그렇기에 아래 처럼 HelloService 의 parameter 와 맞는 아규먼트로 변환해서 넘겨주는 것이 좋습니다. 즉, 하위레이어에서 상위 레이어를 참조하는건 피하는 것이 좋습니다. 

```java

@RestController
@RequiredArgsConstructor
public class HelloApiController {
    
    private final HelloService helloService;
    
    public CommonApiResponse helloWorld(@RequestBody HelloApiDto.Request request) {
        
        // 추후 확장성이나 유지보수에 좋지 않은 코드
        helloService.helloWorld(request);        
                
        // HelloServie 에 맞는 dto 로 변환
        HelloServiceDto.Request dto = helloApiDtoMapper.of(request);
        helloService.helloWorld(dto);
    }
}

```



## Package Import

- Package 내에서 다른 패키지를 참조하는 것은 의존이며, Package 간에 양방향 참조가 이루어진다면 객체 설계를 잘못한거라 할 수 있습니다. 왜냐하면 양방향 참조가 이루어진다는건 하나가 바뀌면 양쪽이 다 바뀐다는 것이기 때문입니다.
- 이를 해결하기 위해 같은 패키지에 두는 것이 좋습니다.



## 외부 연동, data access

- business layer 에서 외부 연동 (http, ftp 등), data access 에 대한 코드가 직접적으로 연결되는건 추후 확장성이나 유지 보수 측면에서 좋지 않다고 생각합니다.

- 예를 들면 아래와 같습니다.

- HelloService Business Layer 에서 ftp 소스를 직접 작성해서 하기보다는 interface 와 구현체로 분리시켜 개발을 하는 것이 추후 유지보수나 확장성에 좋다고 생각합니다.
  - 만약 추후에 FileUploader 가 ftp 에서 sftp 로 변경이 된다면 구현체만 변경해주면 손쉽게 코드 수정이 가능합니다.

  
  
- before source code
```java

// Before
@Service
public class HelloService {
    
    public HelloDto.Response helloWorld(HelloDto.Request request) {
        
        // ftp 에 업로드 하는 소스
        ftp.connect();
        ftp.upload(xxx);
        
    }
    
} 

```



- after source code

```java
// After
@Service
public class HelloService {

    private final FileUploader fileUploader;
    
  public HelloDto.Response helloWorld(HelloDto.Request request) {
    
      fileUploader.upload(xxx);
    
  }

}

public interface FileUploader {
    
    public void upload(xxx);
    
}

public class FileUploaderImpl implements FileUploader {
    
    public void upload(xxx) {
      xxx  
    };
}



```







## refernce
- https://www.youtube.com/watch?v=RVO02Z1dLF8
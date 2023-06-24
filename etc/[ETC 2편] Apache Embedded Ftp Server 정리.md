> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 2편] Apache Embedded Ftp Server 정리



```text
테스트 용도로 FTP Server 를 사용할 일이 있어 Embedded FTP Server 에 대해 알아봤습니다. 
```




## Embedded Ftp Server 준비해야할 것
- [apache embedded ftp server](https://mina.apache.org/ftpserver-project/embedding_ftpserver.html) 를 참고해서 작성했습니다.



#### Depedency

- mina-core, 2.0-M3 or later
   
  - Apache Mina는 네트워크 프레임워크로 이벤트 중심 비동기 api 및 사용자가 쉽게 네트워크 서버를 구현할 수 있게 도와주는 프레임 워크 입니다.

- slf4j-api

  - 로깅 관련 open source 입니다.

- A SLF4J implementation of your choice, for example slf4j-simple-1.5.3.jar

  - 로깅 구현체입니다.

- ftplet-api

  - FtpServer의 생명주기와 관련된 오픈소스입니다. Ftplet은 세션 연결 및 연결 해제 시뿐만 아니라 사용자 세션 내의 각 명령 전후에 호출됩니다.

- ftpserver-core

  - ftpserver core 입니다.



#### source 

#### server source

- 서버 소스 예제는 아래 링크로 들어가서 참고하시면 됩니다. 

  - [서버 소스 참고](https://mina.apache.org/ftpserver-project/embedding_ftpserver.html)

  - [서버 소스 참고 2](https://github.com/apache/mina-ftpserver/blob/master/core/src/test/resources/dbusermanagertest-hsql.sql)



#### build.gradle 

- spring boot 를 활용해서 만들었기에, Slf4j-api, SLF4J Implementation 은 따로 선언안했습니다.

```groovy
// build.gradle
implementation group: 'org.apache.mina', name: 'mina-core', version: '3.0.0-M2'
implementation group: 'org.apache.mina', name: 'mina-codec', version: '3.0.0-M2'
implementation 'org.apache.ftpserver:ftpserver-core:1.2.0'
implementation group: 'org.apache.ftpserver', name: 'ftplet-api', version: '1.2.0'
```




#### user.properties

```properties
ftpserver.user.test1.homedirectory=./ftp    # ftp server home directory 입니다. 
ftpserver.user.test1.maxloginnumber=3
ftpserver.user.test1.writepermission=true
# password : pw1
ftpserver.user.test1.userpassword=6E6FDF956D04289354DCF1619E28FE77 # test1 라는 유저의 비밀번호는 pw1 입니다. (암호화 방식은 Base64 입니다.)
```





## [기타] Apache FTP Client

- [Apache FTP Client](https://commons.apache.org/proper/commons-net/apidocs/org/apache/commons/net/ftp/FTPClient.html) 참고 

- apache ftp client 사용할 때, 한글파일명이 업로드 안되는 경우가 있습니다. 이럴 때, FTPClient controlEncoding 의 encoding 을 변경해주면 됩니다. (UTF-8, EUC-KR 등)





## Reference

- [https://mina.apache.org/ftpserver-project/embedding_ftpserver.html](https://mina.apache.org/ftpserver-project/embedding_ftpserver.html)
- [https://github.com/apache/mina-ftpserver/blob/master/core/src/test/resources/dbusermanagertest-hsql.sql](https://github.com/apache/mina-ftpserver/blob/master/core/src/test/resources/dbusermanagertest-hsql.sql)
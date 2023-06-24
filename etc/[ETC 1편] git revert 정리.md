> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 1편] git reset, revert 정리






# git revert 란?

- remote repository 에 이미 반영한 내용을 되돌리고 싶을 때가 있습니다.
- 그럴 때, revert 를 사용하면 좋으며, revert 란 특정 commit 의 내용을 되돌리는 commit 을 새로 만듭니다.



### 예제 1 (특정 commit 을 revert)
- Develop1.java 를 생성해서 commit and push 를 했습니다. (hash 1)
- Develop2.java, Develop3.java 를 생성해서 commit and push 를 했습니다. (hash 2)
- Develop4.java, Develop5.java 를 생성해서 commit and push 를 했습니다. (Hash 3)
- 위와 같이 수행했을 때, 3개의 commit 이 생기며, 3개의 hash 값이 생깁니다.
- git revert hash1 을 입력하면 Develop1.java 가 삭제되는 커밋이 생깁니다. (push 를 해줘야 원격에 반영됩니다.)



### 예제 2 (범위 revert)

- Develop1.java 를 생성해서 commit and push 를 했습니다. (hash 1)
- Develop2.java, Develop3.java 를 생성해서 commit and push 를 했습니다. (hash 2)
- Develop4.java, Develop5.java 를 생성해서 commit and push 를 했습니다. (Hash 3)
- 위와 같이 수행했을 때, 3개의 commit 이 생기며, 3개의 hash 값이 생깁니다.
- 현재 branch 의 HEAD 는 hash3 입니다. 
- git revert HEAD~3.. 
  - 위 명령어 입력 시, 위 3개의 commit 을 되돌리는 commit 이 생깁니다. (push 해줘야 원격 repository 반영)
- git revert --no-commit HEAD~3.. 
  - no commit 옵션을 주면 되돌리는 3개의 commit 이 생기지 않습니다. 





# RESET 

- commit 자체를 삭제해버리는 방식입니다. (혼자 branch 를 사용할 때 추천하는 방법입니다.)





### 예제

- Develop1.java 를 생성해서 commit and push 를 했습니다. (hash 1)
- Develop2.java, Develop3.java 를 생성해서 commit and push 를 했습니다. (hash 2)
- Develop4.java, Develop5.java 를 생성해서 commit and push 를 했습니다. (Hash 3)
- Develop6.java, Develop7.java 를 생성해서 commit and push 를 했습니다. (Hash 4)
- 위와 같이 수행했을 때, 4개의 commit 이 생기며, 4개의 hash 값이 생깁니다.
- 현재 branch 의 HEAD 는 hash4 입니다. 
- git reset --hard HEAD~3 
  - 이렇게 하면 HEAD 가 hash1을 가리킵니다.
- Git push 0f origin [현재 branch]
  - 명령어를 수행한 이후 local, remote 에 있는 commit history hash2, hash3, hash4 가 삭제됩니다.

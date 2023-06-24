> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 3편] git pull conflict 해결방안





# git pull conflict 상황

- main, develop, feature branch 가 있다고 가정하겠습니다.
- develop branch 는 main(=master) branch 를 기준으로 만들었습니다.
- feature branch 는 develop branch 를 기준으로 만들었습니다.

- feature branch 에서 작업하고, develop branch pull request 를 날릴려고 하는데 develop 에 이미 작업한 것이 있어 pull request 안됩니다.
- 그럴 때는 develop branch 를 pull 해와야 합니다.

- pull 할 때, 아래와 같은 에러가 발생할 수 있습니다. 
- pull 전략이 fast-forward 전략으로 돼있기 때문입니다. (=develop 에 새로 반영된 소스 들이 전부 ahead 에 위치해있어야 함)
  - git config pull.ff only   (fast-forward 전략 설정)

```
 * branch            master     -> FETCH_HEAD
fatal: Not possible to fast-forward, aborting.
Completed with errors, see above
```



# 해결방안

- pull 을 해서 충돌이 나도록합니다. 충돌난 부분을 해결합니다.
- git config --unset --global pull.ff   (fast-forward 전략 설정)
- git pull --no-ff origin master (pull 할 때, fast-forward 옵션 X
- 위와 같이 옵션을 줘서 fast-forward 옵션을 제거한 후, git pull origin develop 명령어를 통해 develop branch 를 pull 해오면 충돌이 나고, 충돌을 해결한 후, push 합니다.
- 그런 후, pull request 를 feature --> develop 수행하면 내가 작업한 변경된 소스만 pull request 대상이 됩니다. 








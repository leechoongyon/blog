> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 5편] git feature branch 내용을 develop release 둘다 반영해야하는 경우



> 나중에 볼려고 정리했습니다.





# 상황

- git 에서 feature branch 에서 작업하고 있는데 develop, release 에 둘다 반영해야하는 경우가 생길 수 있습니다.
- feature branch 는 develop branch 를 기준으로 만들어진거라 feature branch -> develop branch 로 pr 을 날려서 merge 하면 문제 없습니다. (feature branch 변경사항만 반영됨)

- 문제는 feature branch 를 release 에 반영할 때인데, feature branch 가 develop 을 기준으로 만들어진거라 release 에 pr 을 올릴 때, feature branch 의 변경사항만 반영되는게 어려울 수 있습니다.



```
예를 들면,
develop branch 는 a,b,c 소스 작업이 돼있고,
release branch 는 a,b 만 소스 작업이 돼있다고 가정을 하면
develop branch 를 기준으로 feature branch 에서 d 라는 작업을 하고 
release pr 을 올리면 c,d 가 pr 대상이 될 것 입니다.
```



# 해결방안

- release 를 checkout 받아서 branch 를 새로 만들어서 작업하는게 간단할 수 있습니다.

```
1. git checkout release/1.0
2. git checkout -b feature/modifyXxx
3. modifyXxx 에서 수정하고 remote push
4. modifyXxx -> release/1.0 pull request
5. pr review -> merge
```

> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 4편] git stash, cherry-pick 정리



> 나중에 볼려고 정리했습니다.



# git stash

- 현재 stash_test 라는 branch 에서 작업하고 있습니다. 작업을 하다가 develop branch 에서 작업을 해야 합니다.
- 이럴 때, stash 를 사용하면 좋습니다.
- stash 를 사용하면 현재 작업중인 것을 스택에 저장해놨다가 나중에 꺼내쓸 수 있습니다.



# git stash example

```shell
git check stash_test

// do something

git stash // 명령어 수행

git stash list // stash 한 스택 목록 가져오기.

git checkout develop

// do something

git checkout stash_test

git stash apply <stash@{index}>	// stash 스택에 있는 index 를 입력해서 이전 작업을 불러오기

git stash drop <stash@{index}> // stash 스택에 있는 작업을 삭제


```







# Cherry-pick

- 체리픽은 필요한 commit 만 반영할 수 있는 기능입니다.



### 예시

- source branch 와 target branch 가 있고, source branch 에서 커밋을 했습니다. 해당 커밋의 hash 값이 100 이라고 가정하겠습니다.
- target branch 로 이동해서 git cherry-pick 100 이라고 입력하면 해당 커밋을 가져옵니다.

- 충돌이 났으면 처리하고, 안났으면 remote push 하면 됩니다.



```shell
git checkout source-branch

// do something commit (hash - 100)

git checkout target-branch

git cherry-pick 100 // commit hash 100 에 있는 commit 을 가져와서 반영

```


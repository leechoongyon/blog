> 목차는 [ETC 목차](https://insanelysimple.tistory.com/category/ETC) 에 있습니다.



# [ETC 6편] git tag란? tag 를 이용해서 branch 생성



> 나중에 볼려고 정리했습니다.





# tag 란?

- 특정 커밋을 표시하기 위한 기능입니다.
- 즉, 특정 커밋 번호를 통해 작업을 하던 것을 태그를 이용해서 할 수 있습니다.
  - ex) git revert qwasd123 -> git revert release/1.0.0



# tag, 커밋번호 차이점

- tag 경우 수정이 불가능하며, 읽기 전용입니다. 커밋번호는 checkout 하여 수정이 가능합니다.

- 그래서 tag 의 경우 release 를 관리할 때 사용합니다.

ex) release/1.0.0



# tag 생성 방법



### lightweight

- lightweight 의 경우 특정 커밋을 가리키는 기능입니다. 설명 등을 입력할 수 없습니다.

```bash
git tag release/1.0.0    # 현재 최신 버전의 커밋번호로 태그를 만듭니다. remote push 까지 해야 remote 저장소에 생성됩니다.

git tag release/1.0.0 abd123  # abd123 특정 커밋번호에 대해 커밋번호를 지정합니다.
```



### annotated

- 태그를 생성할 때, 정보를 같이 저장하고 싶을 경우 사용합니다.

```bash
git tag -a release/1.0.0 -m "release/1.0.0 입니다."	# tag -a 옵션을 붙이면 annotated 태그를 생성합니다.
```





# tag 를 기반으로 branch 를 생성하는 방법



### 1. 원하는 태그를 로컬로 다운로드 받습니다.

```bash
git tag   # tag 목록을 조회합니다.

git fetch origin --tags # origin remote 에 있는 모든 태그를 가져옵니다.

git fetch origin tags/release1.0.0   # tags/release1.0.0 태그를 로컬로 가져옵니다.
```



### 2. 태그를 fetch 한 이후, checkout 합니다.

```bash
git checkout tags/release1.0.0    # tags/release1.0.0 를 checkout 합니다.
```



### 3. 태그를 기반으로 branch 생성합니다.

```bash
git checkout tags/release1.0.0 -b release/1.0.0		# tags/release1.0.0 태그를 기반으로 release/1.0.0 branch 를 생성합니다.
```



### 4. branch 를 remote 에 push 합니다.


> 목차는 [테스트 & 리팩토링 목차](https://insanelysimple.tistory.com/category/test%20%26%20refactoring) 에 있습니다.



# [테스트 & 리팩토링 4편] github pull request template 및 CodeReview 정리



## Code Review & Pull Request 생각
- Pull Request 은 reviewer 들이 이해하기 쉽도록 작성해야 한다고 생각합니다.
  - 그렇기에 코드는 될 수 있으면 짧게 기능 단위로 요청하는 것이 좋으며, 서로 연관된 기능들을 한 번에 올려야하는 경우 reviewer 들에게 양해를 구하고 올리는 comment 를 달아주는 것이 좋다고 생각합니다.
- Code Review 를 하는 입장에서는 공격적으로 하지않고, 친절하게 suggest 하는 식으로 review 를 달며, 괜찮다면 code example 을 같이 제안해주면 좋을 것이라 생각합니다. (물론 code example 을 너무 과도하게 하면 그것 또한 상대방에게 실례가 될 수 있을 것 같습니다.)
- 코드가 많으면 리뷰어가 고통스럽니다. 변경 기능에 대해 200줄 이하로 Pull Request 를 올리는게 좋다고 생각합니다. 그 이상의 코드는 상세하게 주석과 설명을 달아줘야 리뷰어가 이해하기 쉽습니다.  



## Pull Request template 만들기

1. root 프로젝트에 .github 폴더를 생성 합니다.
2. .github 폴더 밑에 PULL_REQUEST_TEMPLATE.md 파일을 만들어줍니다.
3. 아래와 같이 markdown 을 작성해줍니다.
```markdown
## 소스 변경 이유

## 설명

## 테스트 방법

```



## Pull Request example

#### 소스 변경 이유

- 추가 요구사항에 따른 소스 변경

#### 설명

- Email 연동 개발

#### 테스트 방법

- BRANCH : feature/test_pull_request 를 받아서 checkout
  - git fetch origin feature/test_pull_request
  - git checkout feature/test_pull_request
- project root 로 이동 --> ./gradlew test 수행



# Reference
- https://trustyoo86.github.io/github/2019/02/15/github-template.html
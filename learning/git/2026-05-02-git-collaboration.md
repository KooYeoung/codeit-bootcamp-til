# GitHub를 활용한 협업 정리

## 학습 날짜

2026-05-02

## 학습 주제

- Git 협업의 필요성
- GitHub 협업 기능
- Pull Request
- Pull Request 머지 방식
- Fork 기반 협업
- Merge Conflict 해결
- 좋은 Pull Request
- Code Review
- 좋은 커밋
- Branch Protection
- 브랜치 관리 전략
- GitHub Actions와 협업 자동화

---

## 1. Git 협업의 필요성

소프트웨어 프로젝트는 규모가 커질수록 여러 사람이 함께 작업하게 된다.

이때 Git과 GitHub를 제대로 사용하지 못하면 다음과 같은 문제가 발생할 수 있다.

- 같은 작업을 중복해서 진행
- 코드 충돌 증가
- 브랜치 관리 실패
- 버그 발생
- 팀원 간 소통 부족
- 작업 의도 파악 어려움
- 프로젝트 일정 지연

협업을 잘하기 위해서는 Git 명령어만 아는 것이 아니라, GitHub의 Pull Request, 코드 리뷰, 브랜치 관리 전략, 자동화 기능을 함께 이해해야 한다.

---

## 2. Git 협업 기본 흐름

일반적인 Git 협업 흐름은 다음과 같다.

```text
git pull
→ 브랜치 생성
→ 작업
→ git add
→ git commit
→ git push
→ Pull Request 생성
→ 코드 리뷰
→ merge
```

각 단계의 의미는 다음과 같다.

### git pull

작업을 시작하기 전에 원격 저장소의 최신 내용을 로컬 저장소로 가져온다.

```bash
git pull
```

다른 사람이 먼저 반영한 코드가 있을 수 있으므로, 작업 전에 최신 상태를 유지하는 것이 중요하다.

### 브랜치 생성

새로운 기능이나 수정 작업은 main 브랜치에서 직접 진행하지 않고 별도 브랜치를 만들어 작업한다.

```bash
git checkout -b 브랜치명
```

또는 다음 명령어를 사용할 수 있다.

```bash
git switch -c 브랜치명
```

### 작업 후 커밋

변경사항을 스테이징하고 커밋한다.

```bash
git add -A
git commit -m "feat: add user profile page"
```

### git push

로컬 브랜치의 커밋을 원격 저장소로 올린다.

```bash
git push origin 브랜치명
```

### Pull Request 생성

GitHub에서 Pull Request를 생성하여 작업 내용을 리뷰받고 병합을 요청한다.

---

## 3. GitHub 협업 기능

GitHub는 단순한 원격 저장소 서비스가 아니라 협업 플랫폼이다.

대표적인 협업 기능은 다음과 같다.

### Organization

Organization은 팀이나 회사 단위로 저장소와 권한을 관리할 수 있는 기능이다.

Organization 안에서 여러 저장소를 관리하고, 팀원 권한을 설정할 수 있다.

### Teams

Teams는 Organization 안에서 사용자들을 목적에 따라 그룹화하는 기능이다.

예를 들어 다음과 같이 팀을 나눌 수 있다.

```text
frontend team
backend team
infra team
reviewer team
```

Teams는 Pull Request 리뷰 요청, CODEOWNERS, 권한 관리 등에 활용될 수 있다.

### Issues

Issues는 버그, 기능 요청, 작업 항목 등을 관리하는 기능이다.

프로젝트에서 해야 할 일이나 논의할 내용을 이슈로 남기면 작업 추적과 소통이 쉬워진다.

### Pull Requests

Pull Request는 작업한 코드 변경사항을 검토하고 병합해 달라고 요청하는 기능이다.

### Code Reviews

Code Review는 Pull Request에서 변경된 코드를 검토하고 피드백을 주고받는 과정이다.

코드 품질을 높이고, 팀원 간 지식을 공유하는 데 도움이 된다.

---

## 4. Pull Request

Pull Request는 내가 작업한 변경사항을 다른 사람에게 검토받고, 대상 브랜치에 병합해 달라고 요청하는 GitHub 기능이다.

줄여서 PR이라고 부른다.

PR은 단순히 merge 요청만을 의미하지 않는다.

PR은 다음 역할을 한다.

- 코드 리뷰 요청
- 변경사항 설명
- 작업 의도 공유
- 기능 단위 기록
- 문서화
- 복구 가능한 작업 단위 생성

개인 프로젝트에서도 PR을 사용하면 내가 작성한 코드를 다시 검토할 수 있고, 팀 프로젝트에서는 다른 사람의 리뷰를 통해 코드 품질을 높일 수 있다.

---

## 5. Pull Request 생성 흐름

Pull Request를 생성하는 기본 흐름은 다음과 같다.

```text
repository clone
→ 브랜치 생성
→ 코드 수정
→ git add
→ git commit
→ git push
→ GitHub에서 Pull Request 생성
```

예시 명령어는 다음과 같다.

```bash
git clone https://github.com/username/repository-name.git
cd repository-name

git checkout -b my-first-branch

git add -A
git commit -m "feat: update README"

git push origin my-first-branch
```

push 후 GitHub에서 Pull Request를 생성할 수 있다.

---

## 6. Pull Request 상태

Pull Request는 대표적으로 다음 상태를 가진다.

### Open

아직 검토 중이거나 추가 작업이 필요한 상태이다.

Open 상태에서는 추가 커밋을 올리거나, 리뷰를 주고받을 수 있다.

### Merged

Pull Request의 변경사항이 대상 브랜치에 병합된 상태이다.

병합된 이후에는 해당 PR에 더 이상 커밋을 추가할 수 없다.

### Closed

병합하지 않고 닫힌 상태이다.

작업이 더 이상 필요하지 않거나, 잘못 생성된 PR일 때 닫을 수 있다.

Merged도 넓게 보면 Closed 상태에 포함된다.

---

## 7. Pull Request 머지 방식

GitHub에서는 Pull Request를 병합할 때 주로 세 가지 방식을 제공한다.

### Merge Commit

Merge Commit 방식은 브랜치의 커밋 이력을 모두 유지하면서 병합 커밋을 생성하는 방식이다.

장점은 다음과 같다.

- 브랜치 작업 이력이 보존된다.
- 어떤 브랜치가 언제 병합되었는지 확인하기 쉽다.
- 커밋 해시가 바뀌지 않는다.

단점은 다음과 같다.

- 커밋 히스토리가 복잡해질 수 있다.
- 브랜치가 많아지면 로그가 복잡하게 보일 수 있다.

### Squash and Merge

Squash and Merge는 PR 안의 여러 커밋을 하나의 커밋으로 합쳐서 병합하는 방식이다.

장점은 다음과 같다.

- 커밋 히스토리가 깔끔해진다.
- 하나의 PR이 하나의 커밋으로 남는다.
- 자잘한 수정 커밋을 정리할 수 있다.

단점은 다음과 같다.

- 개별 커밋의 세부 이력이 Git 히스토리에서는 사라질 수 있다.
- 기존 커밋 해시가 유지되지 않는다.
- 여러 사람이 같은 브랜치를 기반으로 작업 중이라면 혼란이 생길 수 있다.

base branch에는 하나의 새 커밋만 남지만, GitHub의 PR 기록에는 기존 커밋 정보가 남아 있으므로 필요하면 PR에서 확인할 수 있다.

### Rebase and Merge

Rebase and Merge는 작업 브랜치의 커밋들을 대상 브랜치 위로 재배치한 뒤 병합하는 방식이다.

장점은 다음과 같다.

- 커밋 히스토리가 선형적으로 유지된다.
- 로그가 깔끔하게 보인다.
- 코드 변화 흐름을 한 줄로 따라가기 쉽다.

단점은 다음과 같다.

- 커밋 해시가 변경된다.
- 협업 중인 브랜치에서 사용하면 충돌이나 혼란이 생길 수 있다.
- PR 단위의 작업 흐름이 히스토리에서 명확히 보이지 않을 수 있다.

---

## 8. Fork 기반 협업

Fork는 GitHub에서 제공하는 기능으로, 다른 사람의 원격 저장소를 내 계정으로 복사하는 기능이다.

Fork는 Git 자체의 기능이 아니라 GitHub, GitLab 같은 Git 호스팅 서비스에서 제공하는 기능이다.

Fork한 저장소는 원본 저장소와 분리되어 있으므로, 내 계정의 저장소에서 자유롭게 작업할 수 있다.

작업이 끝나면 원본 저장소에 Pull Request를 보내 변경사항 반영을 요청할 수 있다.

### 브랜치 기반 PR과 Fork 기반 PR

일반적인 팀 프로젝트에서는 원본 저장소에 접근 권한이 있는 팀원이 브랜치를 만들어 PR을 생성한다.

오픈소스 프로젝트에서는 모든 사람에게 원본 저장소 쓰기 권한을 줄 수 없기 때문에 Fork 기반 PR을 많이 사용한다.

정리하면 다음과 같다.

```text
브랜치 기반 PR
→ 원본 저장소 안에서 브랜치를 만들어 작업
→ 팀 내부 협업에 많이 사용

Fork 기반 PR
→ 원본 저장소를 내 계정으로 복사한 뒤 작업
→ 오픈소스 기여에 많이 사용
```

---

## 9. Merge Conflict

Merge Conflict는 Git이 두 브랜치의 변경사항을 자동으로 병합하지 못할 때 발생한다.

주로 같은 파일의 같은 부분을 서로 다르게 수정했을 때 충돌이 발생한다.

충돌이 발생하면 파일에는 다음과 같은 표시가 생긴다.

```text
<<<<<<< HEAD
현재 브랜치 내용
=======
병합하려는 브랜치 내용
>>>>>>> branch-name
```

충돌을 해결하려면 충돌 표시를 직접 확인하고, 최종적으로 남길 코드를 직접 수정해야 한다.

---

## 10. Pull Request 충돌 해결 방법

Pull Request에서 충돌이 발생하면 바로 병합할 수 없다.

충돌 해결 방법은 크게 두 가지이다.

```text
1. GitHub의 Resolve conflicts 버튼으로 해결
2. 로컬에서 직접 충돌 해결
```

### GitHub UI에서 해결

GitHub의 Resolve conflicts 버튼을 사용하면 웹 화면에서 충돌을 수정할 수 있다.

간단한 충돌이라면 빠르게 해결할 수 있다.

다만 복잡한 코드의 흐름을 파악하기 어렵고, IDE의 도움을 받기 어렵다는 단점이 있다.

### 로컬에서 해결

복잡한 충돌은 로컬에서 해결하는 것이 더 안전하다.

기본 흐름은 다음과 같다.

```bash
git checkout 작업브랜치
git merge main
```

충돌 파일을 직접 수정한 뒤 다음 명령어를 실행한다.

```bash
git add .
git commit
git push
```

로컬에서는 IDE나 VS Code의 Merge Editor 같은 도구를 사용할 수 있기 때문에 복잡한 충돌을 해결하기 더 좋다.

---

## 11. 충돌을 줄이는 방법

충돌은 해결할 수 있지만, 가장 좋은 방법은 충돌이 발생하지 않도록 줄이는 것이다.

충돌을 줄이는 방법은 다음과 같다.

### 최신 코드 유지

작업 전 또는 작업 중간에 main 브랜치의 최신 내용을 자주 반영한다.

```bash
git pull
```

작업 브랜치가 오래될수록 main 브랜치와 차이가 커지고 충돌 가능성이 높아진다.

### 작은 Pull Request 만들기

PR이 작을수록 변경되는 코드 양이 적고, 충돌 가능성도 줄어든다.

작은 PR은 리뷰하기도 쉽다.

### 파일을 작게 나누기

큰 파일은 여러 사람이 동시에 수정할 가능성이 높다.

파일을 역할별로 작게 나누면 충돌 가능성을 줄일 수 있고, 코드 가독성도 높아진다.

### 커뮤니케이션

팀원들과 어떤 파일이나 기능을 작업하는지 미리 공유하면 중복 작업을 줄일 수 있다.

협업에서는 기술적인 능력뿐만 아니라 커뮤니케이션도 중요하다.

---

## 12. 좋은 Pull Request

좋은 Pull Request는 리뷰어가 목적과 변경사항을 쉽게 이해할 수 있는 PR이다.

좋은 PR을 만들기 위해서는 다음을 고려한다.

- 하나의 PR은 하나의 문제나 기능을 다룬다.
- PR 제목은 변경 목적이 드러나도록 작성한다.
- PR 설명에는 무엇을, 왜 변경했는지 작성한다.
- 필요한 경우 스크린샷, 테스트 결과, 관련 문서 링크를 첨부한다.
- PR 크기를 너무 크게 만들지 않는다.
- 리뷰어가 어떤 부분을 중점적으로 보면 좋을지 안내한다.
- PR을 올리기 전에 직접 테스트한다.

PR은 단순 코드 변경이 아니라 협업을 위한 문서 역할도 한다.

---

## 13. Code Review

Code Review는 Pull Request에서 다른 사람이 작성한 코드를 검토하고 피드백을 주고받는 과정이다.

코드 리뷰의 목적은 다음과 같다.

- 코드 품질 향상
- 버그 예방
- 팀원 간 지식 공유
- 프로젝트 이해도 향상
- 유지보수성 향상

코드 리뷰는 사람을 평가하는 것이 아니라 코드를 함께 개선하는 과정이어야 한다.

피드백은 작성자가 아니라 코드와 문제 상황을 향해야 한다.

---

## 14. GitHub 코드 리뷰 기능

GitHub는 Pull Request에서 다양한 코드 리뷰 기능을 제공한다.

### 라인별 코멘트

변경된 코드의 특정 라인에 의견을 남길 수 있다.

### 멀티라인 코멘트

여러 줄을 선택해서 한 번에 코멘트를 남길 수 있다.

### Suggested Change

리뷰어가 직접 수정 제안을 남길 수 있다.

작성자는 제안된 변경사항을 바로 반영할 수 있다.

### Viewed

리뷰한 파일을 Viewed로 체크하면 이미 확인한 파일을 접을 수 있다.

변경 파일이 많을 때 리뷰 진행 상황을 관리하기 좋다.

### 커밋별 리뷰

PR 안에 커밋이 여러 개 있을 경우, 특정 커밋의 변경사항만 따로 볼 수 있다.

커밋이 논리 단위로 잘 나뉘어 있다면 커밋별 리뷰가 쉬워진다.

---

## 15. 리뷰 제출 방식

GitHub에서 리뷰를 제출할 때는 다음 세 가지 방식을 사용할 수 있다.

### Comment

의견이나 질문을 남길 때 사용한다.

단순한 제안, 궁금한 점, 참고 의견 등을 남길 수 있다.

### Approve

변경사항이 문제없다고 승인할 때 사용한다.

Branch Protection 설정에 따라 Approve가 있어야만 병합할 수 있도록 제한할 수 있다.

### Request Changes

수정이 필요하다고 요청할 때 사용한다.

리뷰어가 Request Changes를 남기면 작성자는 해당 피드백을 반영한 뒤 다시 리뷰를 요청할 수 있다.

---

## 16. 좋은 커밋

좋은 커밋은 의미 있는 작업 단위로 나누어진 커밋이다.

Pull Request가 하나의 큰 작업 단위라면, 커밋은 그 PR을 구성하는 작은 논리 단위이다.

좋은 커밋을 만들기 위해서는 다음을 고려한다.

- 하나의 커밋에는 하나의 의미 있는 변경사항을 담는다.
- 커밋은 가능한 정상 동작 가능한 상태여야 한다.
- 커밋 메시지는 변경 내용을 이해할 수 있게 작성한다.
- 너무 작은 의미 없는 커밋을 남발하지 않는다.
- 너무 많은 변경사항을 하나의 커밋에 몰아넣지 않는다.

예시:

```bash
git commit -m "feat: add login validation"
git commit -m "fix: handle empty username"
git commit -m "docs: update API usage guide"
```

---

## 17. Branch Protection

Branch Protection은 중요한 브랜치를 보호하기 위한 GitHub 설정이다.

주로 `main`, `develop` 같은 공유 브랜치에 설정한다.

Branch Protection을 사용하면 다음과 같은 규칙을 적용할 수 있다.

- main 브랜치에 직접 push 금지
- Pull Request를 통해서만 병합 허용
- 특정 수 이상의 Approve 필요
- Code Owner 리뷰 필수
- 테스트나 Linting 통과 후 병합 허용
- 선형 히스토리 유지
- 설정 우회 방지

이 설정을 사용하면 실수로 main 브랜치에 문제가 있는 코드가 반영되는 것을 줄일 수 있다.

---

## 18. 브랜치 관리 전략

브랜치 관리 전략은 팀이 브랜치를 어떤 기준으로 만들고, 병합하고, 배포할지 정하는 규칙이다.

프로젝트 규모, 배포 주기, 팀의 협업 방식에 따라 적절한 전략을 선택해야 한다.

---

## 19. GitHub Flow

GitHub Flow는 `main` 브랜치를 중심으로 하는 단순한 브랜치 전략이다.

특징은 다음과 같다.

- `main` 브랜치는 항상 배포 가능한 상태를 유지한다.
- 새로운 작업은 main에서 별도 브랜치를 만들어 진행한다.
- 작업이 끝나면 Pull Request를 생성한다.
- 코드 리뷰 후 main에 병합한다.
- 구조가 단순하다.

기본 흐름은 다음과 같다.

```text
main
→ feature branch 생성
→ 작업 및 commit
→ push
→ Pull Request 생성
→ review
→ main에 merge
```

GitHub Flow는 빠르게 개발하고 자주 배포하는 프로젝트에 적합하다.

---

## 20. Git Flow

Git Flow는 여러 종류의 브랜치를 사용하는 전략이다.

대표 브랜치는 다음과 같다.

```text
main
develop
feature
release
hotfix
```

각 브랜치의 역할은 다음과 같다.

### main

실제 배포 버전을 관리하는 브랜치이다.

### develop

다음 배포를 위한 개발 내용을 모으는 브랜치이다.

### feature

새로운 기능을 개발하는 브랜치이다.

개발이 끝나면 develop 브랜치로 병합한다.

### release

배포 준비를 위한 브랜치이다.

버그 수정이나 배포 전 점검을 진행한다.

### hotfix

운영 중 발생한 긴급 문제를 수정하기 위한 브랜치이다.

Git Flow는 브랜치 역할이 명확하지만, 관리해야 할 브랜치가 많아 복잡해질 수 있다.

배포 주기가 명확하고 버전 관리가 중요한 프로젝트에 적합하다.

---

## 21. GitLab Flow

GitLab Flow는 GitHub Flow와 Git Flow의 장점을 섞고, CI/CD 환경에 맞게 확장한 브랜치 전략이다.

GitLab Flow에서는 보통 다음 브랜치를 사용한다.

```text
main
feature
environment
```

### main

항상 배포 가능한 상태를 유지하는 중심 브랜치이다.

### feature

기능 개발을 위한 브랜치이다.

### environment

배포 환경을 구분하기 위한 브랜치이다.

예시는 다음과 같다.

```text
production
develop
qa
staging
```

GitLab Flow는 여러 배포 환경을 운영하는 프로젝트에서 유용하다.

---

## 22. Trunk Based Development

Trunk Based Development는 중심 브랜치에 작은 변경사항을 자주 병합하는 전략이다.

여기서 trunk는 보통 `main` 또는 `master` 브랜치를 의미한다.

특징은 다음과 같다.

- 긴 수명의 브랜치를 오래 유지하지 않는다.
- 작은 변경사항을 자주 main에 병합한다.
- PR 크기가 작고 주기가 짧다.
- 빠른 피드백을 받을 수 있다.
- CI/CD 환경과 테스트 자동화가 중요하다.
- 완성되지 않은 기능은 Feature Toggle로 비활성화한 채 병합할 수 있다.

Trunk Based Development는 빠른 배포와 자동화 환경이 잘 갖춰진 팀에서 효과적이다.

---

## 23. GitHub를 사용한 협업 자동화

GitHub는 Pull Request, 코드 리뷰, 브랜치 보호뿐만 아니라 협업 과정의 일부를 자동화할 수 있는 기능도 제공한다.

대표적인 기능은 GitHub Actions이다.

GitHub Actions는 특정 이벤트가 발생했을 때 미리 정의한 작업을 자동으로 실행하는 기능이다.

예를 들어 다음 이벤트에서 자동화 작업을 실행할 수 있다.

```text
push
pull_request
release
workflow_dispatch
```

병합 후 작업을 실행하고 싶다면 보통 `main` 브랜치에 대한 `push` 이벤트를 사용한다.

또는 `pull_request` 이벤트에서 PR이 `closed` 상태이고 실제로 `merged`된 경우인지 조건으로 확인해 처리할 수 있다.

자동화할 수 있는 작업 예시는 다음과 같다.

- 테스트 실행
- 빌드 실행
- Linting 실행
- 코드 스타일 검사
- 배포
- PR 검증

---

## 24. Workflow와 YAML

GitHub Actions에서 자동화 작업은 Workflow라고 부른다.

Workflow는 프로젝트의 다음 경로에 작성한다.

```text
.github/workflows/
```

Workflow 파일은 YAML 형식으로 작성한다.

예시:

```yaml
name: Basic Workflow Example

on: [push]

jobs:
  example_job:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Print Hello
        run: echo "Hello, GitHub!"
```

위 예시는 push 이벤트가 발생했을 때 GitHub Actions가 실행되는 간단한 예시이다.

---

## 25. Linting과 테스트 자동화

Linting은 코드 스타일이나 규칙 위반을 자동으로 검사하는 과정이다.

테스트 자동화는 PR이나 push 시점에 테스트를 자동으로 실행해 코드가 정상 동작하는지 확인하는 과정이다.

Linting과 테스트 자동화를 적용하면 다음 장점이 있다.

- 코드 스타일을 일정하게 유지할 수 있다.
- 사람이 직접 확인하지 않아도 기본 품질 검사가 가능하다.
- 리뷰어가 코드 스타일보다 핵심 로직 검토에 집중할 수 있다.
- 문제가 있는 코드가 main 브랜치에 병합되는 것을 줄일 수 있다.

Java 프로젝트에서 사용할 수 있는 도구 예시는 다음과 같다.

```text
IntelliJ Formatter
Checkstyle
Spotless
google-java-format
SpotBugs
```

처음에는 IDE Formatter로 코드 스타일을 정리하고, 프로젝트가 커지면 Checkstyle이나 Spotless 같은 도구를 적용해볼 수 있다.

---

## 26. Branch Protection과 자동화 연결

Branch Protection은 GitHub Actions와 함께 사용하면 더 효과적이다.

예를 들어 main 브랜치에 병합하기 전에 다음 조건을 강제할 수 있다.

- Pull Request 생성 필수
- Approve 1개 이상 필수
- 테스트 통과 필수
- Linting 통과 필수
- 직접 push 금지

이렇게 하면 main 브랜치에 문제가 있는 코드가 반영되는 것을 줄일 수 있다.

협업 자동화는 팀원들이 매번 수동으로 확인해야 하는 일을 줄이고, 일정한 품질 기준을 유지하는 데 도움을 준다.

---

## 27. 오늘 정리

오늘은 GitHub를 활용한 협업 흐름을 학습했다.

가장 중요한 흐름은 다음과 같다.

```text
git pull
→ 브랜치 생성
→ 작업
→ commit
→ push
→ Pull Request
→ Code Review
→ merge
```

Pull Request는 단순히 코드를 병합하는 요청이 아니라, 작업 의도와 변경사항을 공유하고 리뷰받는 협업 단위이다.

좋은 협업을 위해서는 다음을 신경 써야 한다.

- PR을 작게 만들기
- 커밋을 의미 있는 단위로 나누기
- 코드 리뷰를 통해 품질 높이기
- 충돌이 나지 않도록 최신 코드 유지하기
- 브랜치 전략을 팀에 맞게 선택하기
- Branch Protection과 GitHub Actions로 실수 줄이기

브랜치 전략은 프로젝트 상황에 따라 다르게 선택할 수 있다.

```text
GitHub Flow
→ 단순하고 빠른 PR 중심 협업

Git Flow
→ 배포 버전 관리가 중요한 프로젝트

GitLab Flow
→ 여러 배포 환경과 CI/CD를 고려한 프로젝트

Trunk Based Development
→ 작은 변경을 자주 병합하고 자동화가 잘 갖춰진 팀
```

GitHub Actions와 자동화는 PR 병합 전에 테스트, Linting, 빌드 등을 자동으로 확인하게 해주는 안전장치이다.

협업에서는 Git 명령어뿐만 아니라 PR 작성, 코드 리뷰, 커뮤니케이션, 자동화까지 함께 이해하는 것이 중요하다.

# GitHub와 커밋 관리 심화

## 학습 날짜

2026-05-01

## 학습 주제

- Git 생성 배경
- Git 작업 영역과 파일 상태
- GitHub와 원격 저장소
- push, pull, clone
- README와 Markdown
- 커밋 메시지 작성 가이드
- Git alias
- reset 옵션
- tag
- merge conflict와 merge abort
- remote, origin, upstream

---

## 1. Git 생성 배경

Git은 리누스 토발즈가 만든 분산 버전 관리 시스템이다.

리누스 토발즈는 Linux 운영 체제를 만든 사람이며, Linux 프로젝트의 버전 관리를 위해 기존에 사용하던 도구 대신 직접 Git을 만들었다.

Git은 다음과 같은 목표를 가지고 만들어졌다.

- 빠른 속도
- 단순한 구조
- 비선형적 개발 지원
- 완전 분산형 시스템
- 대규모 프로젝트도 안정적으로 관리할 수 있는 구조

Git은 단순히 파일의 변경 이력을 저장하는 도구가 아니라, 여러 개발자가 함께 협업할 수 있도록 도와주는 중요한 개발 도구이다.

---

## 2. Git의 3가지 작업 영역

Git은 크게 3가지 작업 영역을 기준으로 동작한다.

### Working Directory

실제로 파일을 생성, 수정, 삭제하는 작업 공간이다.

예를 들어 프로젝트 폴더에서 코드를 작성하거나 파일을 수정하는 공간이 Working Directory이다.

### Staging Area

커밋에 포함할 변경사항을 임시로 올려두는 공간이다.

`git add` 명령어를 사용하면 변경사항이 Staging Area에 올라간다.

### Repository

커밋 이력이 저장되는 공간이다.

`git commit`을 실행하면 Staging Area에 올라간 변경사항이 Repository에 저장된다.

Git의 기본 흐름은 다음과 같다.

```text
Working Directory
→ git add
→ Staging Area
→ git commit
→ Repository
```

---

## 3. Git 파일 상태

Git에서 파일은 크게 `Untracked`와 `Tracked` 상태로 나눌 수 있다.

### Untracked

Git이 아직 추적하지 않는 파일이다.

새로 만든 파일을 아직 한 번도 `git add` 하지 않았다면 Untracked 상태이다.

### Tracked

Git이 변경사항을 추적하고 있는 파일이다.

Tracked 상태는 다시 다음과 같이 나뉜다.

### Staged

파일의 변경사항이 Staging Area에 올라간 상태이다.

```bash
git add 파일명
```

### Unmodified

최신 커밋과 비교했을 때 변경사항이 없는 상태이다.

커밋 직후의 파일들은 보통 Unmodified 상태가 된다.

### Modified

최신 커밋과 비교했을 때 파일 내용이 변경된 상태이다.

파일 상태 흐름은 다음과 같이 정리할 수 있다.

```text
Untracked
→ git add
→ Staged
→ git commit
→ Unmodified
→ 파일 수정
→ Modified
→ git add
→ Staged
```

---

## 4. GitHub와 원격 저장소

GitHub는 Git 저장소를 온라인에서 관리하고 공유할 수 있게 해주는 서비스이다.

Git은 로컬에서 버전 관리를 수행하는 도구이고, GitHub는 Git 저장소를 원격에서 관리하고 협업할 수 있게 해주는 서비스이다.

### 로컬 저장소

내 컴퓨터에 있는 Git 저장소이다.

```text
Local Repository
```

### 원격 저장소

GitHub 같은 외부 서비스에 있는 저장소이다.

```text
Remote Repository
```

로컬 저장소에서 작업한 내용을 원격 저장소에 반영하려면 `git push`를 사용한다.

원격 저장소의 내용을 로컬 저장소로 가져오려면 `git pull`을 사용한다.

---

## 5. push, pull, clone

### git push

로컬 저장소의 커밋을 원격 저장소에 반영할 때 사용한다.

```bash
git push
```

처음 원격 저장소에 push할 때는 다음과 같이 사용할 수 있다.

```bash
git push -u origin main
```

여기서 `-u` 옵션은 로컬 브랜치와 원격 브랜치를 연결하는 역할을 한다.

한 번 연결해두면 이후에는 `git push`만 입력해도 연결된 원격 브랜치로 push할 수 있다.

### git pull

원격 저장소의 최신 내용을 로컬 저장소로 가져올 때 사용한다.

```bash
git pull
```

협업 중에는 다른 사람이 원격 저장소에 올린 변경사항을 내 로컬에도 반영해야 하므로 `git pull`을 사용한다.

### git clone

GitHub에 있는 프로젝트를 내 컴퓨터로 복사할 때 사용한다.

```bash
git clone 원격저장소주소
```

`clone`을 하면 원격 저장소의 파일뿐만 아니라 Git 이력도 함께 가져온다.

---

## 6. README와 Markdown

GitHub에서는 `README.md` 파일이 저장소 메인 화면에 표시된다.

README에는 보통 다음 내용을 작성한다.

- 프로젝트 소개
- 사용 방법
- 설치 방법
- 실행 방법
- 폴더 구조
- 학습 기록 목적

`.md`는 Markdown 파일을 의미한다.

Markdown은 간단한 문법으로 문서를 보기 좋게 작성할 수 있는 형식이다.

예시:

```md
# 제목 1

## 제목 2

- 목록
- 목록

~~~bash
git status
~~~
```

README를 잘 작성하면 저장소의 목적과 내용을 다른 사람이 쉽게 이해할 수 있다.

---

## 7. 커밋 메시지 작성 가이드

커밋 메시지는 프로젝트의 변경 이력을 이해하는 데 중요한 역할을 한다.

커밋에는 보통 다음 정보가 포함된다.

- 커밋한 사용자
- 커밋한 날짜와 시간
- 커밋 메시지

사용자와 날짜는 Git이 자동으로 기록하지만, 커밋 메시지는 개발자가 직접 작성해야 한다.

좋은 커밋 메시지를 작성하기 위해서는 다음을 고려한다.

- 커밋 메시지는 변경 내용을 명확하게 작성한다.
- 제목과 본문을 나눌 수 있다.
- 제목은 너무 길지 않게 작성한다.
- 제목 끝에 마침표를 붙이지 않는다.
- 하나의 커밋에는 하나의 작업 단위만 포함한다.
- 커밋하기 전에 코드가 정상적으로 동작하는지 확인한다.

예시:

```bash
git commit -m "docs: add GitHub learning notes"
git commit -m "fix: correct README typo"
git commit -m "chore: update folder structure"
```

커밋은 나중에 다시 봤을 때 어떤 작업을 했는지 이해할 수 있도록 남기는 것이 중요하다.

---

## 8. Git Alias

Git alias는 긴 Git 명령어에 별명을 붙여 짧게 사용할 수 있게 해주는 기능이다.

예를 들어 다음 명령어는 커밋 히스토리를 한 줄로 출력한다.

```bash
git log --pretty=oneline
```

이 명령어가 길기 때문에 alias를 설정할 수 있다.

```bash
git config alias.history "log --pretty=oneline"
```

이후에는 다음 명령어로 같은 결과를 볼 수 있다.

```bash
git history
```

주의할 점은 `history`는 원래 Git 명령어가 아니라 직접 만든 alias라는 것이다.

---

## 9. Git Reset 옵션

`git reset`은 HEAD가 특정 커밋을 가리키도록 이동시키는 명령어이다.

기본 형식은 다음과 같다.

```bash
git reset 옵션 커밋해시
```

`reset`은 옵션에 따라 영향을 주는 범위가 다르다.

### --soft

```bash
git reset --soft 커밋해시
```

HEAD만 특정 커밋으로 이동한다.

변경사항은 Staging Area에 남아 있다.

### --mixed

```bash
git reset --mixed 커밋해시
```

HEAD와 Staging Area를 특정 커밋 상태로 되돌린다.

변경사항은 Working Directory에 남아 있다.

옵션을 생략하면 기본적으로 `--mixed`가 적용된다.

```bash
git reset 커밋해시
```

### --hard

```bash
git reset --hard 커밋해시
```

HEAD, Staging Area, Working Directory를 모두 특정 커밋 상태로 되돌린다.

작업 중인 변경사항까지 사라질 수 있으므로 주의해서 사용해야 한다.

---

## 10. HEAD 기준 상대 표현

`git reset`을 사용할 때 커밋 해시 대신 HEAD 기준 상대 표현을 사용할 수 있다.

### HEAD^

현재 HEAD가 가리키는 커밋의 바로 이전 커밋을 의미한다.

```bash
git reset --hard HEAD^
```

### HEAD~숫자

현재 HEAD가 가리키는 커밋에서 숫자만큼 이전 커밋을 의미한다.

```bash
git reset --hard HEAD~2
```

위 명령어는 현재 HEAD 기준으로 2단계 이전 커밋으로 이동한다.

정리하면 다음과 같다.

```text
HEAD     현재 커밋
HEAD^    바로 이전 커밋
HEAD~1   바로 이전 커밋
HEAD~2   두 단계 이전 커밋
HEAD~3   세 단계 이전 커밋
```

커밋 해시를 매번 찾기 어려울 때 HEAD 기준 상대 표현을 사용하면 편리하다.

---

## 11. Git Tag

Tag는 특정 커밋에 이름표를 붙이는 기능이다.

보통 프로젝트의 버전 출시 지점처럼 중요한 커밋에 태그를 붙인다.

태그를 생성하려면 다음 명령어를 사용한다.

```bash
git tag 태그명 커밋해시
```

현재 저장소의 태그 목록을 보려면 다음 명령어를 사용한다.

```bash
git tag
```

특정 태그가 가리키는 커밋 정보를 확인하려면 다음 명령어를 사용한다.

```bash
git show 태그명
```

예시:

```bash
git tag v1.0.0 abc1234
git show v1.0.0
```

태그를 사용하면 중요한 커밋을 나중에 쉽게 찾을 수 있다.

---

## 12. Merge Conflict와 Abort

Merge Conflict는 Git이 두 브랜치의 변경사항을 자동으로 병합하지 못할 때 발생한다.

보통 같은 파일의 같은 부분을 서로 다른 브랜치에서 수정한 경우 conflict가 발생한다.

Conflict가 발생하면 파일 안에 다음과 같은 표시가 생긴다.

```text
<<<<<<< HEAD
현재 브랜치 내용
=======
병합하려는 브랜치 내용
>>>>>>> branch-name
```

Conflict를 해결하려면 다음 순서로 진행한다.

```text
충돌 파일 확인
→ 원하는 내용으로 직접 수정
→ git add
→ git commit
```

파일이 여러 개 충돌난 경우에도 원리는 같다.

각 파일의 충돌을 해결한 뒤 `git add`로 Staging Area에 올리고, 마지막에 커밋하면 된다.

### merge 취소

충돌이 너무 많거나 지금 병합을 진행하지 않으려면 merge 자체를 취소할 수 있다.

```bash
git merge --abort
```

이 명령어를 사용하면 merge를 시도하기 전 상태로 돌아간다.

다만 꼭 병합해야 하는 상황이라면 conflict를 직접 해결하고 커밋하는 것이 일반적인 흐름이다.

---

## 13. Remote, Origin, Upstream

### remote

`remote`는 원격 저장소와 관련된 작업을 할 때 사용하는 개념이다.

로컬 저장소에 원격 저장소를 등록할 때 다음 명령어를 사용한다.

```bash
git remote add origin 원격저장소주소
```

### origin

`origin`은 원격 저장소에 붙이는 기본 이름으로 많이 사용된다.

예를 들어 다음 명령어는 GitHub 저장소 주소를 `origin`이라는 이름으로 등록한다.

```bash
git remote add origin https://github.com/username/repository.git
```

`origin`이라는 이름은 관례적으로 많이 사용된다.

다른 이름을 사용할 수도 있지만, 일반적으로는 `origin`을 사용한다.

### upstream

`upstream`은 로컬 브랜치가 연결해서 바라보는 원격 브랜치를 의미한다.

다음 명령어를 실행하면 로컬 `main` 브랜치와 원격 `origin/main` 브랜치가 연결된다.

```bash
git push -u origin main
```

여기서 `-u`는 `--set-upstream`의 줄임말이다.

한 번 upstream 연결을 해두면 이후에는 다음 명령어만으로 push와 pull을 할 수 있다.

```bash
git push
git pull
```

정리하면 다음과 같다.

```text
origin
→ 원격 저장소의 기본 이름

origin/main
→ origin 원격 저장소의 main 브랜치

upstream
→ 현재 로컬 브랜치가 연결된 원격 브랜치

tracking connection
→ 로컬 브랜치와 원격 브랜치가 연결된 상태
```

---

## 14. 오늘 정리

오늘은 Git 기본 개념에서 GitHub와 원격 저장소 개념으로 학습 범위를 확장했다.

특히 중요한 흐름은 다음과 같다.

```text
로컬에서 작업
→ git add
→ git commit
→ git push
→ GitHub 원격 저장소에 반영
```

원격 저장소와 관련된 핵심 명령어는 다음과 같다.

```bash
git remote add origin 원격저장소주소
git push -u origin main
git push
git pull
git clone 원격저장소주소
```

커밋 관리에서는 다음 내용을 중요하게 정리했다.

- 커밋은 하나의 작업 단위로 나누는 것이 좋다.
- 커밋 메시지는 나중에 봐도 이해할 수 있게 작성해야 한다.
- `git reset`은 옵션에 따라 HEAD, Staging Area, Working Directory에 미치는 영향이 다르다.
- `--hard` 옵션은 작업 내용이 사라질 수 있으므로 주의해야 한다.
- 중요한 커밋에는 tag를 붙여 나중에 쉽게 찾을 수 있다.
- merge conflict는 직접 해결할 수도 있고, 필요하면 `git merge --abort`로 병합 자체를 취소할 수도 있다.

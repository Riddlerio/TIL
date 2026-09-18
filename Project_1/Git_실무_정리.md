# Git 실무 완전 정리 — 전체 흐름과 GitHub Actions

> 이번 주 내가 GPT한테 물어봤던 Git/CI-CD 질문들(PR이 뭐야, merge는 폴더를 합치는 거야, fetch랑 pull 차이, .env는 왜 gitignore에 넣어, GitHub Actions가 Docker까지 실행하는 거야 등)을 하나의 흐름으로 정리한 참고용 문서.

## 1. Git이 뭐고 왜 쓰는가

Git은 **코드의 변경 이력을 추적하고, 여러 사람이 같은 코드베이스를 안전하게 같이 고칠 수 있게 해주는 버전 관리 시스템**이다.

혼자 개발할 때도 필요한 이유:
- 예전 버전으로 되돌릴 수 있음 (실수해도 복구 가능)
- 기능별로 브랜치를 나눠서 안전하게 실험 가능

여러 명이 개발할 때 필요한 이유:
- 같은 파일을 동시에 고쳐도 **변경 이력을 비교해서** 합칠 수 있음 (덮어쓰기 아님)
- 누가 언제 무엇을 왜 고쳤는지 기록이 남음(commit message, blame)

## 2. 기본 흐름: clone → branch → commit → push

```text
[GitHub 원격 저장소]
        │ clone (최초 1회, 전체 복사)
        ▼
[내 컴퓨터의 로컬 저장소]
        │ 1. 브랜치 생성 (git checkout -b feature/login)
        ▼
   feature/login 브랜치에서 작업
        │ 2. git add .        → 변경사항을 스테이징
        │ 3. git commit -m "" → 스냅샷 하나 확정
        │ 4. git push         → 원격 브랜치로 업로드
        ▼
[GitHub의 feature/login 브랜치]
```

- **clone**: 원격 저장소 전체(커밋 이력 포함)를 내 컴퓨터로 복사. 처음 한 번만.
- **branch**: `main`을 건드리지 않고 새 기능을 독립된 줄기에서 작업하기 위한 것. 작업 단위마다 새로 만드는 게 일반적.
- **add → commit → push**: "저장 → 스냅샷 확정 → 서버 업로드" 3단계. commit은 로컬에만 쌓이고, push를 해야 원격에 반영된다.

## 3. 협업 흐름: PR → 리뷰 → merge

### PR(Pull Request)이 뭔가
> "제 브랜치의 변경사항을 메인 브랜치에 합쳐도 될지 검토해주세요" 라는 **요청**.

PR을 만든다고 메인이 바로 바뀌지 않는다. 팀원이 코드를 리뷰하고, 승인해야 merge된다. 그래서 **PR 자체는 메인에 아무 영향이 없다** — 이게 이번 주에 헷갈렸던 부분.

```text
[회사 GitHub]
     │ clone
     ▼
[내 컴퓨터]
     │ git checkout -b feature/login
     ▼
[feature/login에서 작업 + commit]
     │ git push origin feature/login
     ▼
[GitHub에 feature/login 브랜치 생성됨]
     │ Pull Request 생성 (feature/login → main)
     ▼
[팀원이 코드 리뷰]
     │ 승인 (Approve)
     ▼
[Merge] → main에 반영됨
     │
     ▼
[CI/CD가 자동으로 테스트·배포] (4번 섹션에서 설명)
```

### merge는 폴더를 합치는 게 아니다
Git은 파일을 통째로 복사하지 않는다. **커밋 이력(누가 어떤 줄을 어떻게 바꿨는지)을 비교해서** 코드를 합친다.

```text
main:            A ─ B ─ C
feature/login:   A ─ B ─ C ─ D  (D = 로그인 기능 추가)

merge 후 main:   A ─ B ─ C ─ D
```

충돌(conflict)은 **같은 파일의 같은 줄을 서로 다르게 고쳤을 때만** 발생한다. 그 경우는 사람이 직접 어느 걸 남길지 선택해야 한다. 대부분의 merge는 충돌 없이 자동으로 합쳐진다.

## 4. fetch vs pull, Behind/Ahead가 뭔지

```text
git fetch  → 원격의 최신 정보를 "가져오기만" 함. 내 작업 파일은 그대로.
git pull   → fetch + 내 브랜치에 즉시 merge까지 함.
```

GitHub 화면에 보이는 **Behind / Ahead**:

```text
내 브랜치:        A ─ B ─ C ─ D
origin/main:      A ─ B ─ C

→ Ahead 1   (내가 원격보다 커밋 1개 앞서 있음, 아직 push 안 했거나 push된 상태)
→ Behind 0  (원격이 나보다 뒤처진 건 없음)
```
반대로 팀원이 원격에 새 커밋을 올렸는데 내가 안 받아왔으면 **Behind**로 표시된다. 이때는 fetch/pull로 받아와야 최신 상태가 된다.

## 5. 오픈소스 기여 흐름: Fork → PR

"공개된 GitHub repo를 가져와서 내가 고친 다음 PR해도 되나? 전혀 모르는 사람이 만든 PR을 원작자가 가져다 쓸 수도 있는 건가?" → 둘 다 **가능**하다. 이게 오픈소스의 핵심 구조.

```text
[원본 오픈소스 저장소] (내 소유 아님)
        │ Fork (내 계정으로 통째로 복사)
        ▼
[내 GitHub의 복제 저장소]
        │ clone → 수정 → commit → push
        ▼
[내 저장소에 반영됨]
        │ Pull Request 생성 (내 저장소 → 원본 저장소)
        ▼
[원본 관리자가 코드 리뷰]
     ┌────────────┴────────────┐
     ▼                         ▼
   승인 → Merge            수정 요청 → 내가 다시 고쳐서 push
```

라이선스를 확인해야 하는 것은 별개 문제지만, **모르는 사람의 PR도 관리자가 검토해서 merge할 수 있다** — 이게 오픈소스가 커지는 방식이다.

## 6. 비밀 정보 관리: .gitignore + .env

```text
.env 파일 안에:
  OPENAI_API_KEY=sk-xxxxxxxx
  LUNA_API_KEY=xxxxxxxx

.gitignore 파일 안에:
  .env
```

- `.env`에 API 키 같은 비밀값을 넣는 것 자체는 맞다.
- 하지만 **그 `.env`를 Git에 올리면 안 되기 때문에** `.gitignore`에 반드시 등록해야 한다.
- `.gitignore`는 "이 파일/폴더는 Git이 추적하지 말라"는 목록. `.env`, `node_modules/`, `__pycache__/` 같은 게 대표적으로 들어간다.
- 서버에 배포할 때는 `.env` 파일을 직접 올리는 대신, **GitHub Actions Secrets**나 서버의 환경변수로 주입한다 (7번 섹션).

## 7. GitHub Actions — CI/CD 상세

### CI/CD가 뭔지 먼저
```text
CI (Continuous Integration, 지속적 통합)
  → 코드를 GitHub에 push하면 자동으로 테스트·검사·빌드

CD (Continuous Delivery/Deployment, 지속적 배포)
  → 테스트를 통과한 코드를 자동으로 서버에 배포
```
즉 **CI/CD = 코드 변경부터 배포까지를 사람이 손으로 하지 않고 자동화하는 시스템**.

### GitHub Actions의 구조
GitHub Actions는 GitHub가 제공하는 **CI/CD 자동화 도구**다. `.github/workflows/` 폴더에 YAML 파일을 넣으면 그게 하나의 "워크플로우"가 된다.

```text
Workflow (하나의 자동화 파이프라인)
  └─ Trigger (언제 실행할지: push, PR, 매일 정해진 시간 등)
      └─ Job (실행 단위, 여러 개 병렬 가능)
           └─ Step (Job 안의 개별 명령어들)
```

### 실제 예시 (테스트 → Docker 빌드 → 배포)

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]        # main에 push되면 실행

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4      # 코드 가져오기
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest                     # ① 테스트 자동 실행

  build-and-deploy:
    needs: test                         # 테스트를 통과해야만 실행
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t my-app:latest .     # ② Docker 이미지 빌드
      - name: Push to registry
        run: docker push my-registry/my-app:latest   # ③ 이미지 저장소에 push
      - name: Deploy to server
        env:
          API_KEY: ${{ secrets.LUNA_API_KEY }}    # ④ 비밀값은 Secrets에서 주입
        run: ssh user@server "docker pull my-registry/my-app:latest && docker restart my-app"
```

**여기서 중요한 포인트 (질문했던 부분들)**
- GitHub Actions가 **항상 Docker까지 실행하는 건 아니다.** 테스트만 자동화할 수도 있고, 빌드+배포까지 확장할 수도 있다. 프로젝트 필요에 따라 워크플로우를 얼마나 채우느냐가 다르다.
- `.env` 파일을 그대로 올리지 않고, **GitHub 저장소 Settings → Secrets and variables → Actions**에 키를 등록해서 `${{ secrets.KEY_NAME }}`으로 불러온다. 이게 `.gitignore`로 숨긴 비밀값을 배포 파이프라인에서 안전하게 쓰는 방법이다.
- Job은 여러 개를 만들 수 있고 `needs:` 로 순서를 강제할 수 있다 (테스트 실패하면 배포 안 되게).

### GitHub Actions ↔ Docker 관계 정리

```text
GitHub에 코드 push
        ↓
GitHub Actions 실행 (workflow 트리거)
        ↓
① 코드 테스트 (pytest, jest 등)
        ↓
② Docker 이미지 빌드 (Dockerfile 기준으로 실행 환경 통째로 패키징)
        ↓
③ 이미지를 저장소(Docker Hub, ECR 등)에 push
        ↓
④ 서버(EC2 등)가 최신 이미지를 pull → 재시작
```
Docker는 "이 코드가 어떤 환경(파이썬 버전, 라이브러리 등)에서 돌아가는지"를 통째로 포장하는 도구이고, GitHub Actions는 "이 전체 과정을 언제·어떻게 자동으로 실행할지"를 관리하는 도구. 둘은 역할이 다르고, CI/CD 파이프라인 안에서 함께 쓰인다.

## 8. 실무 전체 그림 (한 장으로)

```text
[로컬 개발]
   git checkout -b feature/xxx
   코드 작성 → git add → git commit
   git push origin feature/xxx
        │
        ▼
[GitHub]
   Pull Request 생성
        │
        ▼
[코드 리뷰 & CI 자동 실행]  ← GitHub Actions가 push/PR 시점에 자동으로 테스트
   테스트 통과 + 승인
        │
        ▼
[Merge to main]
        │
        ▼
[CD 자동 실행]  ← GitHub Actions가 main 브랜치 push를 감지
   Docker 이미지 빌드 → 레지스트리 push → 서버 배포
        │
        ▼
[실제 서비스에 반영]
```

## 결론
- Git 명령어 자체(add, commit, push)는 도구일 뿐이고, 실무에서 진짜 중요한 건 **"내가 만든 코드를 어떻게 팀의 메인 코드에 안전하게, 자동으로 합치고 배포하는가"** 라는 흐름이다.
- 이 흐름은 `브랜치 분리 → PR/리뷰 → merge → CI(테스트) → CD(배포)` 5단계로 요약되고, GitHub Actions는 이 중 CI/CD 두 단계를 자동화하는 도구다.

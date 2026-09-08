---
title: Git Fast-Forward vs Merge vs Rebase 완벽 정리
draft: false
created: 2025-07-07
tags:
  - git
  - version-control
  - Main
  - Feat
  - Release
  - HotFix
AI-generated: true
read: false
AI:
  - claude
---

## 초기 상황 설정

```bash
# main 브랜치에서 시작
main: A ─ B ─ C

# feature 브랜치 생성
git checkout -b feature
feature: A ─ B ─ C ─ D ─ E

# hotfix 브랜치 생성 (main에서)
git checkout main
git checkout -b hotfix
hotfix: A ─ B ─ C ─ F ─ G
```

현재 상태:

```
main:    A ─ B ─ C
feature:         └─ D ─ E
hotfix:          └─ F ─ G
```

## 1. Fast-Forward Merge

### 조건

- 대상 브랜치에 새로운 커밋이 없을 때
- 현재 브랜치가 대상 브랜치의 직접적인 후손일 때

### 예시

```bash
# main으로 이동해서 feature를 merge
git checkout main
git merge feature
```

### 결과

```
main/feature: A ─ B ─ C ─ D ─ E
hotfix:               └─ F ─ G
```

### 특징

- ✅ **장점**: 포인터만 앞으로 이동, 깔끔한 히스토리, merge 커밋 없음
- ❌ **단점**: 브랜치 존재 흔적이 사라짐

## 2. 3-way Merge (Non-fast-forward)

### 조건

- 두 브랜치 모두 새로운 커밋이 있을 때
- 브랜치들이 diverged 상태일 때

### 예시

```bash
# main에 hotfix를 merge (main에는 이미 feature가 merge됨)
git checkout main
git merge hotfix
```

### 결과

```
main: A ─ B ─ C ─ D ─ E ─ M
               └─ F ─ G ─┘
```

### 특징

- ✅ **장점**: 새로운 merge 커밋(M) 생성, 브랜치 히스토리 보존, 안전한 병합
- ❌ **단점**: 복잡한 히스토리, merge 커밋 증가

## 3. Rebase

### 목적

- Linear한 히스토리를 만들고 싶을 때
- 커밋을 깔끔하게 정리하고 싶을 때

### 시나리오: auth 브랜치를 main 위에 재배치

```bash
# 초기 상태
main: A ─ B ─ C ─ H ─ I
auth:         └─ D ─ E ─ F

# auth 브랜치에서 rebase 실행
git checkout auth
git rebase main
```

### 결과

```
main: A ─ B ─ C ─ H ─ I
auth:                 └─ D' ─ E' ─ F'
```

### 특징

- ✅ **장점**: 커밋들이 새로운 base 위에 재적용, 선형적이고 깔끔한 히스토리
- ❌ **단점**: 커밋 해시 변경, 충돌 가능성, 공유된 브랜치에서는 위험

## 실제 프로젝트 예시

### 현재 브랜치 상황

```bash
main:        ... ─ M1 ─ M2
Feat/last-time:         └─ L1 ─ L2 ─ L3
Feat/auth:              └─ A1 ─ A2
```

### 1. Fast-forward (Feat/last-time → main)

```bash
git checkout main
git merge Feat/last-time  # Fast-forward 발생
```

**결과**: `main: ... ─ M1 ─ M2 ─ L1 ─ L2 ─ L3`

### 2. Fast-forward (main → Feat/auth)

```bash
git checkout Feat/auth
git merge main  # Fast-forward 발생
```

**결과**: `Feat/auth: ... ─ M1 ─ M2 ─ L1 ─ L2 ─ L3 ─ A1 ─ A2`

## 언제 어떤 방법을 사용할까?

### Fast-Forward

```bash
# 개인 feature 브랜치, 선형 히스토리 원할 때
git merge feature-branch
```

**사용 시기**:

- 개인 작업 브랜치
- 빠른 hotfix
- 선형 히스토리를 유지하고 싶을 때

### 3-way Merge

```bash
# 브랜치 히스토리 보존하고 싶을 때
git merge --no-ff feature-branch
```

**사용 시기**:

- 팀 작업에서 브랜치 히스토리 보존
- Feature 완성 시점을 명확히 하고 싶을 때
- 롤백이 쉬워야 하는 경우

### Rebase

```bash
# 깔끔한 히스토리, 커밋 정리하고 싶을 때
git rebase main
git rebase -i HEAD~3  # 대화형 rebase로 커밋 정리
```

**사용 시기**:

- 개인 브랜치 정리
- 커밋 메시지 수정
- 불필요한 커밋 제거
- PR 전 히스토리 정리

## 충돌 해결 차이

### Merge 충돌

```bash
git merge feature
# 충돌 해결 후
git add .
git commit  # merge 커밋 생성
```

### Rebase 충돌

```bash
git rebase main
# 충돌 해결 후
git add .
git rebase --continue  # 다음 커밋으로 계속
# 또는 중단: git rebase --abort
```

## 팀 작업 권장사항

### 1. Feature 브랜치 워크플로우

#### 이유
![[Feature 브랜치 워크플로우 권장사항]]

<br>
<br>

### 2. Release 브랜치

#### 이유
![[Release 브랜치 전략]]
<br>
<br>

### 3. Hotfix 브랜치

#### 이유
![[Hotfix 브랜치 전략]]
<br>
<br>

## 주의사항

### ⚠️ Rebase 사용 시 주의점

- **절대로 공유된 브랜치를 rebase하지 마세요**
- 이미 push된 커밋을 rebase하면 다른 팀원에게 문제 발생
- `git push --force`는 매우 위험함

### 🔍 히스토리 확인 명령어

```bash
# 그래프로 히스토리 보기
git log --oneline --graph --all

# 특정 브랜치 간 차이 보기
git log main..feature

# 머지 커밋만 보기
git log --merges
```

## 정리

|방법|사용 시기|장점|단점|
|---|---|---|---|
|**Fast-forward**|개인 작업, 간단한 변경|깔끔한 히스토리|브랜치 흔적 없음|
|**3-way Merge**|팀 작업, 중요한 기능|히스토리 보존, 안전|복잡한 그래프|
|**Rebase**|커밋 정리, 선형 히스토리|깔끔한 라인|위험성, 충돌 가능|

상황에 맞게 선택하여 깔끔하고 추적 가능한 Git 히스토리를 유지하세요!

---
title: Release 브랜치 전략
draft: false
created: 2025-07-07
tags:
  - git
  - release
  - deployment
  - version-control
AI-generated: true
read: true
AI:
  - claude
---

# Release 브랜치 전략

## 권장 명령어

```bash
# Release 브랜치는 3-way merge로 히스토리 보존
git checkout main
git merge --no-ff release/v1.2.0
```

## 왜 3-way Merge를 사용하는가?

### 1. 릴리즈 히스토리 보존

```bash
# ✅ 3-way merge: 릴리즈 지점 명확
main: A ─ B ─ C ─ F ─ G ─ M
                └─ D ─ E ─┘
                  v1.2.0

# ❌ Fast-forward: 릴리즈 흔적 없음
main: A ─ B ─ C ─ D ─ E ─ F
                        ↑
                    v1.2.0?
```

### 2. 쉬운 롤백

- 🔄 **빠른 복구**: `git reset --hard HEAD~1`로 릴리즈 전 상태로 즉시 복구
- 📍 **명확한 지점**: 어디까지가 릴리즈인지 한눈에 파악
- 🛡️ **안전성**: 문제 발생 시 신속한 대응 가능

### 3. 릴리즈 추적성

- 📅 **시점 기록**: 정확한 릴리즈 시간과 내용 추적
- 🏷️ **버전 연결**: 태그와 merge commit의 명확한 연결
- 📝 **변경 내역**: 해당 릴리즈에 포함된 기능들 명확히 구분

## Release 워크플로우

### 1. Release 브랜치 생성

```bash
git checkout -b release/v1.2.0 main
```

### 2. 릴리즈 준비 작업

```bash
# 버전 업데이트
echo "v1.2.0" > VERSION
git commit -m "Bump version to v1.2.0"

# 릴리즈 노트 작성
git commit -m "Update CHANGELOG for v1.2.0"
```

### 3. 테스트 및 버그 수정

```bash
# 릴리즈 관련 버그만 수정
git commit -m "Fix: Critical payment validation bug"
```

### 4. Main에 Merge

```bash
git checkout main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
```

## --no-ff 플래그의 중요성

### Fast-forward의 문제점

- 릴리즈 작업이 일반 개발과 구분되지 않음
- 롤백 시 개별 커밋을 하나씩 되돌려야 함
- 릴리즈에 포함된 변경사항을 한눈에 파악하기 어려움

### --no-ff의 장점

- 🎯 **명확한 구분**: 릴리즈 작업과 일반 개발 구분
- 📊 **통계 가능**: 릴리즈별 작업량 측정 용이
- 🔍 **검토 용이**: 릴리즈 범위를 한눈에 파악

## Feature vs Release 비교

|구분|Feature 브랜치|Release 브랜치|
|---|---|---|
|**목적**|새 기능 개발|릴리즈 준비|
|**Merge 방식**|Fast-forward|3-way merge (--no-ff)|
|**히스토리**|선형 유지|히스토리 보존|
|**롤백**|개별 커밋|전체 릴리즈 단위|
|**추적성**|기능별|버전별|

## 장점 요약

1. **명확한 릴리즈 포인트**: 언제 무엇이 릴리즈되었는지 명확
2. **쉬운 롤백**: 문제 발생 시 빠른 복구 가능
3. **버전 관리**: 체계적인 버전 히스토리 유지
4. **팀 커뮤니케이션**: 릴리즈 범위와 내용을 쉽게 공유

---
title: Git Flow 방식
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

**[[Git Fast-Forward vs Merge vs Rebase 완벽 정리]]**를 읽고 오면 이해가 더 쉽습니다.

### 1. Main에서 Feature 개발

```bash
main ← feature/login
     ← feature/payment
     ← feature/dashboard
```

### 2. Release 브랜치 생성

```bash
# v1.2.0에 포함될 기능들이 main에 모두 merge된 후
git checkout main
git checkout -b release/v1.2.0

# 이 시점의 main에는 이미 모든 feature가 포함됨
```

### 3. Release에서는 버그픽스만

```bash
release/v1.2.0: A ─ B ─ C ─ D ─ E
                            ↑   ↑
                        bugfix1 bugfix2
```

## 🎯 실제 워크플로우

### 단계별 진행

**1단계: Feature 개발 (병렬)**

```bash
# 각자 main에서 브랜치 생성
git checkout main
git checkout -b feature/login

# 개발 완료 후 main에 merge
git checkout main  
git merge feature/login
```

**2단계: Release 준비**

```bash
# 모든 feature가 main에 들어간 후
git checkout main
git checkout -b release/v1.2.0

# 버전 번호 업데이트, 문서 정리 등
```

**3단계: 테스트 및 버그픽스**

```bash
# Release 브랜치에서 발견된 버그만 수정
git checkout release/v1.2.0
git checkout -b bugfix/login-validation

# 버그 수정 후 release에 merge
git checkout release/v1.2.0
git merge bugfix/login-validation
```

**4단계: 배포 및 Backport**

```bash
# Production 배포
git checkout main
git merge release/v1.2.0

# 버그픽스를 main에도 반영
git checkout main
git merge release/v1.2.0
```

## 🏗️ 서버 개발 실제 예시

### 스프린트 계획

```
v1.2.0 목표:
- 사용자 로그인 시스템
- 결제 시스템 통합  
- 관리자 대시보드
```

### 개발 진행

```bash
# Week 1-2: Feature 개발
feature/user-auth    → main
feature/payment-api  → main  
feature/admin-dash   → main

# Week 3: Release 준비
main → release/v1.2.0

# Week 4: QA 및 버그픽스
bugfix/auth-timeout    → release/v1.2.0
bugfix/payment-retry   → release/v1.2.0

# Week 5: 배포
release/v1.2.0 → main → production
```

## ⚠️ 왜 Release에서 Feature를 만들면 안될까?

### 1. 불안정한 기반

```bash
# Feature A 개발 중
release/v1.2.0: X ─ Y

# Feature B가 먼저 merge됨  
release/v1.2.0: X ─ Y ─ Z

# Feature A가 이제 오래된 base에서 작업하고 있음!
```

### 2. 테스트 복잡성

- 🧪 **격리 어려움**: 어떤 feature가 버그를 일으켰는지 파악 힘듦
- 🔄 **회귀 테스트**: 매번 모든 feature 조합 테스트 필요

### 3. 롤백 어려움

```bash
# 만약 payment feature에 치명적 버그 발견
# 하지만 이미 다른 feature들이 그 위에 쌓여있다면?
# 전체를 되돌려야 함...
```

## 💡 핵심 원칙

**"Release 브랜치는 안정화를 위한 공간"**

- ✅ **Main**: 새로운 기능 개발과 통합
- ✅ **Release**: 출시 준비와 품질 보증
- ✅ **Production**: 안정적인 서비스 운영

이렇게 하면 각 브랜치의 목적이 명확하고, 위험도를 최소화할 수 있습니다!

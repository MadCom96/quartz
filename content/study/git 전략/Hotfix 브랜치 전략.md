---
title: Hotfix 브랜치 전략
draft: false
tags:
- git
- hotfix
- emergency
- production 
AI-generated: true 
read: false 
AI:
- claude
---

# Hotfix 브랜치 전략

## 권장 명령어

```bash
# Hotfix는 Fast-forward merge로 빠른 적용
git checkout main
git merge hotfix/critical-bug
```

## 왜 Fast-forward Merge를 사용하는가?

### 1. 신속한 배포

```bash
# ✅ Fast-forward: 즉시 적용
main: A ─ B ─ C ─ D ─ E ─ F
                        ↑
                   hotfix 즉시 적용

# ❌ 3-way merge: 불필요한 시간 소요
main: A ─ B ─ C ─ F ─ G ─ M
                └─ D ─ E ─┘
                  (시간 낭비)
```

### 2. 단순성 우선

- 🚨 **긴급 상황**: 복잡한 과정보다 빠른 해결이 우선
- 🎯 **명확한 목적**: 한 가지 문제만 해결 (보통 1-2개 커밋)
- ⚡ **즉시 반영**: merge commit 생성 시간도 절약

### 3. 실수 방지

- 🚫 **복잡성 제거**: 긴급 상황에서 단순한 과정으로 실수 최소화
- 🔧 **집중**: 문제 해결에만 집중, 히스토리 관리는 나중에
- ✅ **확실성**: 빠르고 확실한 적용

## Hotfix 워크플로우

### 1. 긴급 상황 발생

```bash
🚨 ALERT: Critical production bug detected!
```

### 2. Hotfix 브랜치 생성

```bash
git checkout main  # 현재 프로덕션 버전
git checkout -b hotfix/payment-crash
```

### 3. 빠른 수정

```bash
git commit -m "Fix: Null pointer exception in payment validation"
```

### 4. 즉시 배포

```bash
git checkout main
git merge hotfix/payment-crash  # Fast-forward
git tag v1.2.1  # 간단한 태그 (배포 트리거)
git push origin main v1.2.1
```
### 5. 다른 브랜치에도 반영

```bash
# 개발 브랜치에도 적용 (중복 버그 방지)
git checkout develop
git merge hotfix/payment-crash
```

## 시간 비교 예시

### Fast-forward 방식 (권장)

```
13:00 - 버그 발견
13:05 - hotfix 브랜치 생성  
13:15 - 수정 완료
13:16 - merge, 태그, 푸시
13:20 - 서비스 정상화
```
### 3-way merge 방식 (비권장)

```
13:00 - 버그 발견
13:05 - hotfix 브랜치 생성
13:15 - 수정 완료  
13:18 - merge commit 작성
13:22 - merge 완료 및 배포
13:27 - 서비스 정상화
```

**결과**: 7분의 다운타임 단축!

## 브랜치별 전략 비교

|구분|Feature|Release|Hotfix|
|---|---|---|---|
|**긴급도**|낮음|중간|🚨 높음|
|**Merge 방식**|Fast-forward|3-way merge|Fast-forward|
|**이유**|깔끔한 히스토리|히스토리 보존|빠른 배포|
|**우선순위**|품질|계획성|속도|

## 언제 Hotfix를 사용하는가?

### ✅ Hotfix 사용 상황

- 🚨 **프로덕션 장애**: 서비스 중단 상황
- 🔒 **보안 취약점**: 즉시 패치 필요
- 💰 **비즈니스 크리티컬**: 매출에 직접 영향
- 📊 **데이터 무결성**: 데이터 손실 위험

### ❌ Hotfix 남용 주의

- 🐛 일반적인 버그 → 다음 릴리즈 포함
- 🎨 UI 개선 → 계획된 개발 진행
- ⚡ 성능 최적화 → Feature 브랜치 사용
- 📱 새로운 기능 → Feature 브랜치 사용

## 주의사항

### 필수 체크리스트

- ✅ **테스트**: 빠르더라도 핵심 테스트는 필수
- ✅ **문서화**: 긴급 수정 내용 기록
- ✅ **전파**: 모든 관련 브랜치에 반영
- ✅ **모니터링**: 배포 후 즉시 확인

## 장점 요약

1. **최대 속도**: 긴급 상황에서 가장 빠른 대응
2. **단순성**: 복잡한 과정 없이 즉시 적용
3. **실수 방지**: 간단한 워크플로우로 실수 최소화
4. **집중**: 문제 해결에만 집중 가능
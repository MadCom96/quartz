---
title: Feature 브랜치 워크플로우 권장사항
draft: false
tags:
  - git
  - workflow
  - feature-branch
  - team-development
AI-generated: true
read: true
AI:
  - claude
---

## 워크플로우 단계

```bash
# 1. Feature 브랜치에서 작업
git checkout -b feature/new-function

# 2. 작업 완료 후 main 브랜치 최신화
git checkout main
git pull origin main

# 3. Feature 브랜치에 main 내용 rebase
git checkout feature/new-function
git rebase main

# 4. Main에 Fast-forward merge
git checkout main
git merge feature/new-function
```

## 왜 이 방식이 권장되는가?

### 1. 안전한 개발 환경

- 🛡️ **main 브랜치 보호**: 실험적 코드가 main에 영향 주지 않음
- 👥 **병렬 작업**: 여러 개발자가 동시에 다른 기능 개발 가능
- 🧪 **실험 가능**: 실패해도 main에 영향 없음

### 2. 최신 코드와의 호환성

- 📱 **최신 반영**: 다른 팀원의 변경사항 즉시 반영
- 🚫 **충돌 최소화**: 오래된 base에서 작업하지 않음
- 🔄 **지속적 통합**: 매번 최신 상태에서 개발

### 3. 깔끔한 히스토리

```bash
# ✅ 권장 방식: 선형 히스토리
main: A ─ B ─ C ─ D ─ E ─ F ─ G
              ↑feat1  ↑feat2

# ❌ 비권장: 복잡한 히스토리  
main: A ─ B ─ C ─ F ─ G ─ M1 ─ H ─ M2
           └─ D ─ E ─┘     └─ I ─┘
```

### 4. 안전한 충돌 해결

- ✅ **격리된 수정**: feature 브랜치에서만 충돌 해결
- 🚫 **main 보호**: main 브랜치는 항상 안정적
- 💯 **확신**: merge 시점에 충돌 없음 보장

>[!faq]- 왜 충돌 해결이 안전한가?
> 충돌은 #3. rebase시에만 발생한다.
> 이를 모두 해결하고 난 뒤 main에 fast forward merge를 하면 충돌 가능성이 0%
> ```bash
> git checkout feature/new-function
> git rebase main  # ← 여기서 conflict 발생!
> 
> # 1. Conflict 파일 수정
> # VS Code나 에디터에서 <<<< ==== >>>> 마커 해결
> 
> # 2. 수정된 파일 스테이징
> git add .
> 
> # 3. Rebase 계속 진행
> git rebase --continue
> 
> # 4. 추가 conflict가 있다면 1-3 반복

## 장점 요약

1. **개발 안전성**: main 브랜치가 항상 안정적
2. **팀 협업**: 병렬 개발과 코드 통합 용이
3. **히스토리 관리**: 깔끔하고 추적 가능한 커밋 히스토리
4. **충돌 최소화**: 체계적인 충돌 해결 과정
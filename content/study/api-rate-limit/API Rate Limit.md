---
title: API Rate Limit
draft: false
tags:
  - API
  - rate-limiting
  - backend
  - security
  - Main
AI:
  - claude
AI-generated: true
read: false
---

# API Rate Limit

## 📄 개요

API Rate Limiting은 특정 시간 프레임 내에서 클라이언트가 API에 요청할 수 있는 횟수를 제어하는 기술입니다. 서버 리소스를 보호하고, 공정한 사용을 보장하며, 악의적인 공격으로부터 시스템을 보호하는 핵심적인 방어 메커니즘입니다.

## 🎯 주요 목적

### 리소스 보호
- **서버 과부하 방지**: 동시 요청으로 인한 시스템 다운 예방
- **성능 유지**: 모든 사용자에게 안정적인 응답 시간 보장
- **비용 관리**: 클라우드 리소스 사용량 최적화

### 보안 강화
- **DDoS 공격 방어**: 대량의 요청으로부터 시스템 보호
- **브루트 포스 공격 차단**: 반복적인 로그인 시도 제한
- **악의적 트래픽 식별**: 비정상적인 사용 패턴 탐지

### 공정한 사용
- **사용자 간 형평성**: 특정 사용자의 독점적 리소스 사용 방지
- **서비스 품질 유지**: 모든 클라이언트에게 동등한 접근 기회 제공

## 📊 Rate Limiting 유형

### 1. 시간 기반 제한 (Time-based)
```
• 초당 요청 수 (RPS): 10 requests/second
• 분당 요청 수: 600 requests/minute  
• 시간당 요청 수: 36,000 requests/hour
• 일일 요청 수: 100,000 requests/day
```

### 2. 동시 요청 제한 (Concurrent)
```
• 사용자당 동시 연결: 5개
• IP당 동시 요청: 10개
• 전체 시스템 동시 처리: 1000개
```

### 3. 사용자별 제한 (User-based)
```
Free Tier: 100 requests/hour
Basic Plan: 1,000 requests/hour
Pro Plan: 10,000 requests/hour
Enterprise: 무제한 또는 매우 높은 제한
```

### 4. IP 기반 제한 (IP-based)
```
• 단일 IP: 1,000 requests/hour
• 의심스러운 IP: 즉시 차단
• 지역별 제한: 국가/지역에 따른 차등 적용
```

## ⚙️ 구현 알고리즘

### [[Token Bucket Algorithm]]
토큰이 담긴 버킷에서 요청마다 토큰을 소모하는 방식입니다. 토큰은 일정한 속도로 버킷에 채워지며, 짧은 시간 동안의 트래픽 버스트를 허용하면서도 장기적으로는 일정한 속도를 유지할 수 있는 유연한 알고리즘입니다.

### [[Fixed Window Counter Algorithm]]
고정된 시간 윈도우(예: 1분) 내에서 요청 횟수를 카운트하는 단순한 방식입니다. 구현이 간단하고 메모리 효율적이지만, 윈도우 경계에서 순간적으로 많은 요청이 집중될 수 있는 단점이 있습니다.

### [[Sliding Window Log Algorithm]]
각 요청의 타임스탬프를 기록하고, 현재 시점에서 윈도우 크기만큼 이전의 요청들을 확인하는 방식입니다. 정확한 제한이 가능하지만 모든 요청을 저장해야 하므로 메모리 사용량이 많습니다.

### [[Sliding Window Counter Algorithm]]
Fixed Window과 Sliding Window Log의 하이브리드 접근법입니다. 이전 윈도우의 카운트와 현재 윈도우의 카운트를 가중평균으로 계산하여 메모리 효율성과 정확성의 균형을 맞춘 알고리즘입니다.

### [[Leaky Bucket Algorithm]]
요청을 큐에 담고 일정한 속도로 처리하는 방식입니다. 네트워크 트래픽 제어에 주로 사용되며, 출력 속도가 일정하게 유지되는 특징이 있어 서버 부하를 예측 가능하게 만들어줍니다.


## 🏢 실제 서비스 사례

### 주요 플랫폼별 제한

#### Salesforce
- **동시 API 요청**: 25개 (프로덕션 환경)
- **장기 실행 요청**: 20초 이상 지속 시 제한 적용
- **일일 요청**: 15,000개 (기본), 추가 구매 가능

#### HubSpot
- **10초 제한**: 100-200개 요청 (플랜별 차등)
- **일일 제한**: 250,000-1,000,000개 요청
- **무료 플랜**: 가장 낮은 제한 적용

#### QuickBooks Online
- **분당 제한**: 사용자 ID당 500개 요청
- **엔드포인트별**: 모든 영역 합쳐서 500개/분

#### GitHub
- **인증된 요청**: 더 높은 제한 적용
- **미인증 요청**: 낮은 제한으로 제한적 사용

## 💻 구현 예시

### Express.js + Redis
```javascript
const express = require('express');
const redis = require('redis');
const client = redis.createClient();

const rateLimiter = (limit = 100, window = 3600) => {
    return async (req, res, next) => {
        const key = `rate_limit:${req.ip}`;
        
        try {
            const current = await client.incr(key);
            
            if (current === 1) {
                await client.expire(key, window);
            }
            
            if (current > limit) {
                return res.status(429).json({
                    error: 'Rate limit exceeded',
                    retryAfter: window
                });
            }
            
            res.set({
                'X-RateLimit-Limit': limit,
                'X-RateLimit-Remaining': Math.max(0, limit - current),
                'X-RateLimit-Reset': Date.now() + (window * 1000)
            });
            
            next();
        } catch (error) {
            next(error);
        }
    };
};

app.use(rateLimiter(1000, 3600)); // 시간당 1000개 요청
```

### Python Flask + Redis
```python
from flask import Flask, request, jsonify
import redis
import time
from functools import wraps

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def rate_limit(max_requests=100, window=3600):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            key = f"rate_limit:{request.remote_addr}"
            
            current = redis_client.incr(key)
            if current == 1:
                redis_client.expire(key, window)
            
            if current > max_requests:
                return jsonify({
                    'error': 'Rate limit exceeded',
                    'retry_after': window
                }), 429
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/api/data')
@rate_limit(max_requests=1000, window=3600)
def get_data():
    return jsonify({'data': 'response'})
```

## 🚨 HTTP 응답 처리

### 429 Too Many Requests
```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1691172000

{
    "error": "Rate limit exceeded",
    "message": "API rate limit exceeded for user",
    "retry_after": 60,
    "limit": 1000,
    "remaining": 0,
    "reset_time": "2024-08-04T15:00:00Z"
}
```

### 클라이언트 측 처리
```javascript
async function apiCall(url, options = {}) {
    try {
        const response = await fetch(url, options);
        
        if (response.status === 429) {
            const retryAfter = response.headers.get('Retry-After');
            console.log(`Rate limited. Retry after ${retryAfter} seconds`);
            
            // 지수 백오프로 재시도
            await new Promise(resolve => 
                setTimeout(resolve, retryAfter * 1000)
            );
            
            return apiCall(url, options); // 재시도
        }
        
        return response.json();
    } catch (error) {
        console.error('API call failed:', error);
        throw error;
    }
}
```

## 📏 제한 설정 가이드라인

### 일반적인 권장사항

#### 동시 요청 수
```
일반 API: 5-10개 동시 요청
무거운 작업: 1-3개 동시 요청
실시간 API: 10-50개 동시 연결
파일 전송: 2-5개 동시 전송
```

#### 시간당 요청 수
```
소셜미디어/뉴스: 100-300 req/min
이커머스: 60-180 req/min  
금융서비스: 30-100 req/min
AI/ML API: 10-60 req/min
```

#### 서버 리소스별
```
CPU 집약적: 동시 1-2개 (이미지 처리, AI)
메모리 집약적: 동시 2-3개 (대용량 데이터)
I/O 집약적: 동시 5-10개 (DB 쿼리)
네트워크 집약적: 동시 3-8개 (외부 API)
```

### 사용자 계층별 설정
```
Free Tier: 동시 1-2개, 100 req/hour
Basic: 동시 3-5개, 1,000 req/hour
Pro: 동시 5-10개, 10,000 req/hour
Enterprise: 동시 10-50개, 무제한
```

## 🛠️ 구현 도구

### API Gateway 솔루션
- **Kong**: 오픈소스 API 게이트웨이, 다양한 rate limiting 플러그인
- **Nginx Plus**: 상용 웹서버의 rate limiting 모듈
- **AWS API Gateway**: 클라우드 네이티브 솔루션
- **Azure API Management**: Microsoft 클라우드 플랫폼

### 라이브러리 및 미들웨어
- **Express Rate Limit**: Node.js Express 프레임워크용
- **Flask-Limiter**: Python Flask 프레임워크용  
- **Django Ratelimit**: Django 웹 프레임워크용
- **Go Rate**: Go 언어용 rate limiting 라이브러리

## 📊 모니터링 및 분석

### 핵심 지표
```
• 평균 응답 시간
• 429 에러 발생률
• 사용자별 요청 패턴
• 시간대별 트래픽 분포
• 동시 연결 수 추이
```

### 알람 설정
```python
# Prometheus + Grafana 예시
rate_limit_violations = Counter(
    'api_rate_limit_violations_total',
    'Total number of rate limit violations',
    ['user_id', 'endpoint']
)

# 임계값 초과 시 알람
if violation_rate > 5:  # 5% 초과
    send_alert("High rate limit violation detected")
```

## 🎛️ 동적 조정

### 서버 부하 기반 조정
```python
def dynamic_rate_limit():
    cpu_usage = get_cpu_usage()
    memory_usage = get_memory_usage()
    
    if cpu_usage > 80 or memory_usage > 85:
        return base_limit * 0.5  # 50% 감소
    elif cpu_usage < 50 and memory_usage < 60:
        return base_limit * 1.5  # 50% 증가
    
    return base_limit
```

### 시간대별 조정
```python
def time_based_limit():
    current_hour = datetime.now().hour
    
    # 피크 시간 (9-18시)
    if 9 <= current_hour <= 18:
        return base_limit * 0.8
    # 한가한 시간 (새벽 2-6시)
    elif 2 <= current_hour <= 6:
        return base_limit * 2.0
    
    return base_limit
```

## 🔗 참고 자료

### 공식 문서 및 가이드
- [Tyk API Management - Rate Limiting Guide](https://tyk.io/learning-center/api-rate-limiting/)
- [RESTful API - Rate Limit Guidelines](https://restfulapi.net/rest-api-rate-limit-guidelines/)
- [Stoplight - Rate Limiting vs Throttling](https://blog.stoplight.io/best-practices-api-rate-limiting-vs-throttling)

### 클라우드 플랫폼 문서
- [AWS API Gateway - Throttling](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)
- [Azure API Management - Rate Limiting](https://learn.microsoft.com/en-us/azure/api-management/limit-concurrency-policy)
- [Google Cloud Endpoints - Quotas](https://cloud.google.com/endpoints/docs/openapi/quotas-overview)

### 구현 도구 및 라이브러리
- [Kong API Gateway](https://konghq.com/blog/learning-center/what-is-api-rate-limiting)
- [Express Rate Limit](https://github.com/express-rate-limit/express-rate-limit)
- [Flask-Limiter](https://flask-limiter.readthedocs.io/)

### 모범 사례 가이드
- [Testfully - API Rate Limiting Strategies](https://testfully.io/blog/api-rate-limit/)
- [HubSpot - API Rate Limit Guide](https://blog.hubspot.com/website/api-rate-limit)
- [Merge - Rate Limit Best Practices](https://www.merge.dev/blog/api-rate-limit-best-practices)

### 고급 주제
- [Moesif - Rate Limits and Quotas Best Practices](https://www.moesif.com/blog/technical/rate-limiting/Best-Practices-for-API-Rate-Limits-and-Quotas-With-Moesif-to-Avoid-Angry-Customers/)
- [Knit - 10 Best Practices for Rate Limiting](https://www.getknit.dev/blog/10-best-practices-for-api-rate-limiting-and-throttling)

### 참고 블로그
- [https://etloveguitar.tistory.com/126](https://etloveguitar.tistory.com/126)
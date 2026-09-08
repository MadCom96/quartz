---
title: Sliding Window Log Algorithm
draft: false
created: 2025-07-08
tags:
  - API
  - rate-limiting
  - algorithm
  - sliding-window
---

# Sliding Window Log Algorithm

```python
def sliding_window_log(user_id, limit=100, window=3600):
    now = time.time()
    key = f"requests:{user_id}"
    
    # 이전 요청 기록 정리
    redis.zremrangebyscore(key, 0, now - window)
    
    # 현재 요청 수 확인
    request_count = redis.zcard(key)
    
    if request_count < limit:
        redis.zadd(key, {str(uuid.uuid4()): now})
        redis.expire(key, window)
        return True
    return False
```

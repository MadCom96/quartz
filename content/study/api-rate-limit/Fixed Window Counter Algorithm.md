---
title: Fixed Window Counter Algorithm
draft: false
tags:
  - API
  - rate-limiting
  - algorithm
  - fixed-window
---

# Fixed Window Counter Algorithm

```python
def fixed_window_limiter(user_id, limit=100, window=3600):
    key = f"rate_limit:{user_id}:{int(time.time() // window)}"
    current_count = redis.incr(key)
    if current_count == 1:
        redis.expire(key, window)
    return current_count <= limit
```
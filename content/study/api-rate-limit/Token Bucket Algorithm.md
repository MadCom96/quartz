---
title: Token Bucket Algorithm
draft: false
tags:
  - API
  - rate-limiting
  - algorithm
  - token-bucket
---

# Token Bucket Algorithm

```python
class TokenBucket:
    def __init__(self, capacity=10, refill_rate=1):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate
        self.last_refill = time.time()
    
    def consume(self, tokens=1):
        self._refill()
        if self.tokens >= tokens:
            self.tokens -= tokens
            return True
        return False
    
    def _refill(self):
        now = time.time()
        tokens_to_add = (now - self.last_refill) * self.refill_rate
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill = now
```
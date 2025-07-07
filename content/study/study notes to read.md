---
title: study notes to read
draft: false
---

```dataview
TABLE tags as "태그", ai as "AI", file.mtime as "수정일"
FROM "study"
WHERE read = false
SORT file.ctime DESC
```
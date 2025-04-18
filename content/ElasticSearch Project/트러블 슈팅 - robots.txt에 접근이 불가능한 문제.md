---
title: 트러블슈팅 - robots.txt에 접근이 불가능한 문제
draft: false
tags:
  - project
  - ElasticSearch
  - certificate
  - python
date: 2025-04-18
---
## 개요

크롤러에서 robots.txt에 접근하기 위해 보통 robotparser를 쓴다.

이를 사용해서 www.mk.co.kr의 robots.txt에 접근해보려 했다.

<img src=".ElasticSearch-Project/images/250418165303.png" />

그림처럼, 아예 robots.txt에 접근하지 못한다고 한다.

## Issue
### 1. 에러메시지
```log
<urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1000)>
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1000)

During handling of the above exception, another exception occurred:

  File "/Users/jinha/workspace/GitHub/web_crawler_study/url_filter.py", line 26, in is_allowed
    rp.read() # robots.txt 파일 읽기 시도
    ^^^^^^^^^
  File "/Users/jinha/workspace/GitHub/web_crawler_study/crawler.py", line 62, in run
    if not url_filter.is_allowed(url):
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/jinha/workspace/GitHub/web_crawler_study/crawler.py", line 85, in <module>
    run()
urllib.error.URLError: <urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1000)>
```

ssl 인증에 실패했다고나온다. 파이썬에서 요청을 보낼 때 이 인증서를 사용하는 듯 하다.

### 2. 파이썬의 루트 인증서를 업데이트해준다.
<img src=".ElasticSearch-Project/images/250418170127.png" />

위 certificates.command 파일을 실행해줌으로써 파이썬의 인증서를 갱신할 수 있다.

## 후기
<img src=".ElasticSearch-Project/images/250418170330.png" />
[[250418170330.png]]

허무함은 남지만 요청을 보낼 때 이런 인증서를 사용해서 보낸다는 사실을 새로 알게 됐다.

다른 버전의 파이썬을 사용할 때 비슷한 문제가 또 발생할 수도 있을 것 같다. 그때는 빠르게 해결할 수 있을 것이다.
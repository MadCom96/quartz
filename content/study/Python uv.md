---
title: Python uv
draft: false
tags:
  - python
  - uv
  - Main
---

## 개요
mcp 관련 깃허브를 돌아다니다 보면 최신 python 프로젝트들은 대부분 uv를 통해 실행이되는 것을 알 수 있다.

이게뭐지 해서 알아봤는데, 실행할 때 환경을 자동으로 설정해주기에 안전하고 편할 뿐 아니라 빠르기까지 하다고 한다.

본격적으로 알아보자.

## [UV git (link)](https://github.com/astral-sh/uv)

깃허브 페이지의 설명에 따르면

> 엄청나게 빠른 파이썬 ***패키지 & 프로젝트 매니저***
> 
> 러스트로 작성됨

<img alt="Shows a bar chart with benchmark results." src="https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d" style="visibility:visible;max-width:100%;background-color: white;">

## highlights
1. - [x] 🚀 pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv 등을 대체할 수 있는 단일 도구  
2. - [x] ⚡️ pip보다 10~100배 빠름  
3. - [x] 🗂️ 범용 락파일과 함께 포괄적인 프로젝트 관리 제공  
4. - [ ] ❇️ 인라인 의존성 메타데이터를 포함한 스크립트 실행 지원  
5. - [x] 🐍 Python 버전 설치 및 관리  
6. - [x] 🛠️ Python 패키지로 배포된 도구 실행 및 설치  
7. - [ ] 🔩 익숙한 CLI로 성능 향상된 pip 호환 인터페이스 제공  
8. - [x] 🏢 확장 가능한 프로젝트를 위한 Cargo 스타일의 워크스페이스 지원  
9. - [ ] 💾 의존성 중복 제거를 위한 글로벌 캐시로 디스크 공간 효율적 사용  
10. - [x] ⏬ Rust나 Python 없이도 curl 또는 pip로 설치 가능 
11. - [ ] 🖥️ macOS, Linux, Windows 지원

표시한 것들만 둘러보도록 하자.

## 설치

> 10 ⏬ Rust나 Python 없이도 curl 또는 pip로 설치 가능 

```bash
# On macOS and Linux.
curl -LsSf https://astral.sh/uv/install.sh | sh
```
```bash
# On Windows.
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

혹은 

```bash
pip install uv
```

터미널에서 바로 사용이 가능한 프로그램이 pip를 통해 설치가 가능하다.

새삼스럽지만 편리하다.

## 파이썬 버전 설치 및 관리

> 1 🚀 pip, pip-tools, pipx, poetry, pyenv, twine, virtualenv 등을 대체할 수 있는 단일 도구  
>
> 5 🐍 Python 버전 설치 및 관리 
> 
> 6 🛠️ Python 패키지로 배포된 도구 실행 및 설치  

```log
$ uv python install 3.10 3.11 3.12
Searching for Python versions matching: Python 3.10
Searching for Python versions matching: Python 3.11
Searching for Python versions matching: Python 3.12
Installed 3 versions in 3.42s
 + cpython-3.10.14-macos-aarch64-none
 + cpython-3.11.9-macos-aarch64-none
 + cpython-3.12.4-macos-aarch64-none
```

`파이썬 설치 [버전] [버전] ...` 과 같이 써서, 파이썬을 설치할 수 있다.

node에서 npm의 역할을 uv에서도 가지는 것 같다.

```log
$ uv venv --python 3.12.0
Using Python 3.12.0
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

$ uv run --python pypy@3.8 -- python --version
Python 3.8.16 (a9dbdca6fc3286b0addd2240f11d97d8e8de187a, Dec 29 2022, 11:45:30)
[PyPy 7.3.11 with GCC Apple LLVM 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>>
```

`가상환경을 만들어 [필요하다면 버전 지정]`으로 간단히 가상환경을 만들 수도 있다.

CPython, Pypy같은 다양한 런타임도 지정해줄 수 있다.

버전지정이 없다면 기본설정으로 만들어진다.

```log
$ uv python pin 3.11
Pinned `.python-version` to `3.11`
```

현재 디렉토리의 기본 설정 파이썬 버전을 지정해 줄 수도 있다.

## 속도
> 2 ⚡️ pip보다 10~100배 빠름  

실험하고 싶었는데 streamlit이라는 설치에 오래 걸리는 파일이 있는 것을 알았다.

1. 가상환경을 설정하고
2. 내부에 streamlit을 설치하고
3. 시간을 측정해주는

파일을 만들자. pip 버전과 uv버전으로.

```bash
# pip.sh
#!/bin/bash

echo "🐍 Python virtual environment 생성 시작..."
START_TIME=$(date +%s)

# 가상환경 생성
python3 -m venv .venv
source .venv/bin/activate

echo "📦 streamlit 설치 중..."
pip3 install --upgrade pip >/dev/null
pip3 install streamlit

END_TIME=$(date +%s)
ELAPSED_TIME=$((END_TIME - START_TIME))

echo "✅ streamlit 설치 완료!"
echo "⏱️ 소요 시간: ${ELAPSED_TIME}초"

# 가상환경 비활성화 및 삭제
deactivate
rm -rf .venv
echo "🧹 가상환경(.venv) 삭제 완료"
```

```bash
#uv.sh
#!/bin/bash

echo "🚀 uv를 이용한 streamlit 설치 시작!"
START_TIME=$(date +%s)

# uv로 venv 생성
uv venv --python 3.12.0

# 가상환경 활성화
source .venv/bin/activate

# streamlit 설치
echo "📦 streamlit 설치 중..."
uv pip install streamlit

END_TIME=$(date +%s)
ELAPSED_TIME=$((END_TIME - START_TIME))

echo "✅ streamlit 설치 완료!"
echo "⏱️ 소요 시간: ${ELAPSED_TIME}초"

# 가상환경 비활성화 및 삭제
deactivate
rm -rf .venv
echo "🧹 가상환경(.venv) 삭제 완료"
```

### pip.sh 결과
![[Pasted image 20250419025902.png]]

### uv.sh 결과
![[Pasted image 20250419030035.png]]

### 의견
내 환경(macbook m1 pro)에서 생각보다 큰차이가 안나나? 싶지만 그것보다는 준비하는데 시간이 조금 걸렸고, 다운로드 시간에는 더 큰 차이가 났다.

![[Pasted image 20250419030212.png]]

준비하는 데 2초나 걸렸다.



## 프로젝트 환경설정 & 실행
> 3 🗂️ 범용 락파일과 함께 포괄적인 프로젝트 관리 제공  
> 
> 8 🏢 확장 가능한 프로젝트를 위한 Cargo 스타일의 워크스페이스 지원  

프로젝트를 시작하기 위한 명령어와 실행시 생성되는 파일은 다음과 같다.

```log
❯ uv init
Initialized project `uv` # 현재 폴더명으로 기본 생성

❯ uv init mytest
Initialized project `mytest` at `경로` # 폴더를 만들고 내부에 프로젝트 생성

❯ cd mytest
❯ ls -a
.git            .python-version main.py         pyproject.toml  README.md
```

이 중 toml 파일은 처음보는 것 같다.

Rust는 이런 파일을 통해 프로젝트를 관리한다고 하는데, 파이썬에서도 비슷하게 할 수 있도록 만든 파일이다.

```toml
# pyproject.toml
[project]
name = "mytest"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
```

dependencies에는 내가 설치한 의존성들이 저장된다.

### 의존성

의존성들이 자동으로 추가가 되는 지 확인해보자.

```log
❯ uv add streamlit
Using CPython 3.12.0
Creating virtual environment at: .venv
Resolved 39 packages in 202ms
Prepared 36 packages in 2.92s
Installed 36 packages in 90ms

❯ ls -a
.               .python-version main.py         README.md
..              .venv           pyproject.toml  uv.lock
```

```toml
# pyproject.toml
[project]
... # 생략
requires-python = ">=3.12"
dependencies = [
    "streamlit>=1.44.1",
]
```

정리해보자. uv로 프로젝트에 streamlit을 더했을 뿐인데...

1. venv 설정이 되었다.
2. uv.lock 파일이 생겼다. (yarn.lock 등의 파일과 비슷하게, 강력한 의존성 기록 파일)
3. toml 파일에도 의존성이 저장되었다.

공유한 파일을 uv 표준대로 실행만 하면 정말 최대한 같은 환경에서 실행할 수 있다.

반대로 지우고싶은 것은 `uv remove [의존성]`을 통해 가능하다

### 프로젝트 파이썬 버전

이 또한 당연히 쉽게 설정할 수 있다.

1. `.python-version` 파일의 내용을 원하는 버전으로 바꾼다.
2. `pyproject.toml`파일의 `requires-python = ">=[버전]"` 버전 부분을 원하는 버전으로 바꾼다.

```log
❯ uv run main.py
Using CPython 3.11.12
Removed virtual environment at: .venv
Creating virtual environment at: .venv
Hello from mytest!
```

바뀐 버전에 맞는 venv를 다시 만들어주는 부분이 킥이다.

## 후기
- 너무 잘 쓸것 같은 패키지 관리자 & 파이썬 프로젝트 관리자가 만들어졌다.
	- 이미 많은 mcp를 비롯한 파이썬 프로젝트에 개발자들이 적극 활용하고 있고, 그 이유를 알 것 같다.
	- 다루지 않았지만, mcp에서 스크립트 등을 넣을 때, 어떤 파이썬을 쓸 지 등의 경로를 정확하게 작성해야하는 문제도 해결해준다고 한다.
	- 패키지 설치도 빠르다. 단점이 없다. 굳이굳이 꼽자면 새로운 사용 커맨드?
- pycharm같은 좋은 ide를 사용하면, .venv등을 자동으로 만들어주는 것을 쉽게 발견할 수 있다.
	- 이런 과정을 모두 자동으로 해주는게, 진짜 괜찮은 ide 정도의 가치를 가질수도 있지 않을까 생각한다.
- 당장 [[0.엘라스틱 프로젝트 개요]] 프로젝트에서 사용하자고 제안했다.
	- ~~다만 제대로 사용법을 알아보고 나니, 프로젝트 시작, 환경설정, 공유 등에 장점을 가지기에 장기간 프로젝트에는 딱히 필요가 없어진다고 생각이 들긴한다.~~
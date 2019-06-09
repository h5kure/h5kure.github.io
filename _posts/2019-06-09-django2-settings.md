---
title: "Django settings"
date: 2019-06-09 11:30:00 -0400
categories: Django
---

Django 로 개발을 하면서 설정을 개발용과 제품용으로 구분을 하고 싶을 때가 있다.

제품용을 위한 환경을 로컬에 일일이 설정하기가 귀찮고 개발할 때에는 굳이 필요 없는 경우가 많기 때문이다.

기본적으로 Django 프로젝트를 생성하면 settings.py 를 이용하게 되는데 이를 쪼개어 사용하는 법을 기록한다.

## base, development, product

settings 폴더를 만들어 하위에 settings.py 를 3개의 파일로 나눈다.

`base.py` 는 기본적으로 기존의 settings.py 를 파일명만 바꾼 것이다.

그리고 이 파일을 계승하여 필요한 환경별로 설정을 생성하여 해당 환경에 맞춘 설정을 수행한다.

```python
from dpon_jp_travel.settings.base import *
```

`development.py` 나 `product.py` 에서는 위 `base.py` 를 import 하여 모든 설정을 그대로 가져온 다음에,
필요한 설정들만 기존과 동일한 방식으로 설정해주면 되는 것이다.

### 주의
`settings.py` 를 `settings` 하위에 3개의 파일로 분리시키면서 file depth 가 한단계 증가해버린다.

이는 기존의 `BASE_DIR` 에 영향을 끼치기에 기존과 동일한 `BASE_DIR` 을 사용하고자 한다면 `os.path.dirname` 으로 한번 더 감싸주어야 한다.

```python
# BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__))) # 기존
BASE_DIR = os.path.dirname(os.path.dirname(os.path.dirname(os.path.abspath(__file__))))
```   

## run
그럼 이제 설정을 어떻게 사용하느냐.

이에는 3가지 방법이 있다.

### --settings 파라미터

* manage.py 명령어에 기본적으로 `--settings` 파라미터를 이용하여 이용할 설정을 지정해주는 것이다.

```bash
python manage.py --settings=myproject.settings.development
```

### 시스템 환경 변수

시스템 환경 변수에 설정하여 Django 에게 사용할 설정을 알려주는 방법이다.

`DJANGO_SETTINGS_MODULE` 이라는 환경 변수에 사용할 환경 변수를 지정하는 것.

```bash
export DJANGO_SETTINGS_MODULE=myproject.settings.development
```

wsgi 등을 이용할 경우에는 `wsgi.py` 에 해당 환경 변수에 지정된 설정을 사용하도록 알려줘야 한다.

```python
import os
os.environ['DJANGO_SETTINGS_MODULE'] = 'myproject.settings.development'
```


### wsgi.py 및 manage.py 에 직접 지정

`wsgi.py` 및 `manage.py` 에는
 
```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')
```

와 같이 기본 설정을 사용하도록 설정되어 있다.

이를 `myproject.settings.product` 와 같이 아예 바꿔주는 것이 간단한 방법일 수 있다.

기본 설정을 product 로 한 다음에 개발환경에서는 development 환경을 지정하여 사용하는 등의 방법이다.

콘솔에서 일일이 명령어를 실행하는 경우가 아니라면 IDE 등에 한번 development 환경 변수를 사용하도록 설정을 해 놓으면

Git 등 으로 관리되는 실제 소스변경 없이 개발을 하고 deploy 한 서버에서는 기본으로 product 설정을 사용하게 되는 것이다.

  
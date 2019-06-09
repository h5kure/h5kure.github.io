---
title: "PostgreSQL with Django Application"
date: 2019-06-09 11:00:00 -0400
categories: Django
---

`PostgreSQL` 을 Django 에서 사용하기 위한 설정을 기록

* Ubuntu 18.04
* Python3

## Install

* Python 2 의 경우

```bash
sudo apt update
sudo apt install python-pip python-dev libpq-dev postgresql postgresql-contrib
```

* Python 3 의  경우

```bash
sudo apt update
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib
```

## Create Database and User

PostgreSQL 에서 기본으로 제공하는 Administrative user 인 `postgres` 로 로그인한다.

그리고 `psql` 을 실행하여 PostgreSQL 에 진입한다.

```bash
sudo -u postgres psql
```

### Database

```sql
CREATE DATABASE myproject;
```

### User

```sql
CREATE USER myprojectuser WITH PASSWORD 'password';
```

### ROLE

```sql
ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed;
ALTER ROLE myprojectuser SET timezone TO 'UTC';
```

`default_transaction_isolation` 을 `read committed` 로 설정하면 commit 되지 않은 트랜잭션에서 읽기를 차단한다.


### Access Rights

```sql
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
```

## Django Settings

여기에서는 기본적인 Django 의 설정은 이미 되어 있다고 가정하고 생략한다.

### psycopg2

Django 에서 PostgreSQL 을 사용하기 위한 패키지를 설치한다.

```bash
pip install psycopg2
```

### settings.py

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject'
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
} 
```

### Migrate

이제 Project 의 Database 를 migration 하여 사용하면 된다.

```bash
python manage.py makemigrations
python manage.py migrate
```
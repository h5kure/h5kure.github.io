---
title: "Django 2 with Apache 2 and WSGI"
date: 2019-04-09 22:00:00 -0400
categories: django apache wsgi ubuntu debian
---
Django 앞에 Apache (with WSGI, Web Server Gateway Interface) 를 붙여서 서비스를 하기 위해 간단 요약 및 메모.

검색으로 나오는 유사 정보들은 대부분 예전 버전 기준이라던가 Python 2 기준으로 되어 있어 Python 3 을 사용하는 경우에 대한 정보를 찾기가 어려웠다.

## Environment
OS: Ubuntu
Apache: 2.4
Django: 2.1.7 (virtualenv 사용)

## Prepare Packages
Python 2 의 경우,
> libapache2-mod-wsgi : Apache WSGI 모듈. 대부분의 안내는 해당 모듈을 사용하도록 되어 있는데 이 모듈은 Python2 에서만 이용가능

Python 3 의 경우,
> libapache2-mod-wsgi-py3 : Apache WSGI 모듈. Python 3 과 사용하기 위해 필요.

위 모듈들만 제대로 설치하면 다른 설정들은 이미 나와 있는 많은 정보들과 별 차이가 없을 것 같다.

위 두 모듈은 동시에 사용하는 것이 불가능한 것으로 보이며 나중에 설치한 쪽으로 사용되는 것으로 보인다.

Apache WSGI 모듈을 설치하면 대부분 자동으로 활성화 되지만, 필요한 경우 활성화 명령을 실행 시켜 줘야 한다.

``` bash
# wsgi 모듈 활성화
a2enmod wsgi
```

실행 후에 apache 에 설정을 reload 시켜주거나 restart 시켜줘야 하지만 아직 설정할게 남았으니 굳이 여기에서 할 필요는 없겠다.

그 외는 Python 버전에 맞게 pip 를 설치하고 pip 를 통해 virtualenv 등을 설치한다.

## Python Virtual Environment
새로운 프로젝트를 생성하는 경우라면 virtualenv 를 구성하여 pip 를 통해 Django 를 설치하고 Django project 를 생성해야 하고
이미 있는 프로젝트를 서버에 deploy 해서 사용하는 경우에는 virtualenv 만 구성하면 될 것이다.

``` bash
$ mkdir ~/myproject
$ cd ~/myproject
$ virtualenv venv
$ source ~/myproject/venv/bin/activate
(venv) $ pip install django
(venv) $ (venv) django-admin startproject myproject ~/myproject
```

### Project Settings
이제 Django project 의 settings 설정한다.

설정이 필요한 항목은 딱 두가지가 있는데 접근 가능한 Host 설정과 Static File 접근을 위한 STATIC_ROOT 를 설정한다.

``` python
# ~/myproject/myproject/settings.py
...
ALLOWED_HOSTS = ["myproject.com", "127.0.0.1"]
...
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

### Initial Project Setup
Database 를 새롭게 생성할 필요가 있다면,

``` bash
(venv) $ python manage.py makemigrations
(venv) $ python manage.py migrate
```

Super User (admin) 이 필요하다면,

``` bash
(venv) $ python manage.py createsuperuser 
```

Static File 들을 settings.py 에 설정한 곳에 모으기 위해,

``` bash
(venv) $ python manage.py collectstatic
```

Django 동작 확인

``` bash
(venv) $ python manage.py runserver 0.0.0.0:8000
```

이제 python (Django 설정) 은 끝이니 venv 에서 빠진다.

``` bash
(venv) $ deactivate
```

## Apache Configuration
Django 프로젝트가 사용할 VirtualHost 를 설정한다.

`/etc/apache2/sites-available` 에 선호하는 에디터를 사용하여 적당한 설정파일을 만들어서 편집한다. 

``` bash
$ sudo emacs /etc/apache2/sites-available/myproject.com.conf
```

설정하는 항목은 크게 나눠서
* Django settings 에 설정한 static 파일들에 접근할 수 있는 위치를 알려주는 것
* Django wsgi.py 에 접근할 수 있는 위치를 알려 주는 것
* WSGI Daemon 이 사용할 virtualenv 환경 path

```
<VirtualHost *:80>
...
ServerName myproject.com
...
Alias /static /home/user/myproject/static
<Directory /home/user/myproject/static>
    Require all granted
</Directory>
...
<Directory /home/user/myproject/myproject>
    <Files wsgi.py>
        Require all granted
    </Files>
</Directory>
...
WSGIDaemonProcess myproject python-home=/home/user/myproject/venv python-path=/home/user/myproject
WSGIProcessGroup myproject
WSGIScriptAlias / /home/user/myproject/myproject/wsgi.py
...
```

ServerName 에는 Django 에서 사용할 Domain 등을 설정한다.
이를 이용해서 apache 에 해당 사이트를 활성화시킬 수도 있다.

``` bash
# enable myproject.com site
$ sudo a2ensite myproject.com
```

이로써 Apache 설정도 끝

## Permissions
Apache 가 Django 에 접근 권한이 없어서 문제가 발생할 수도 있다.
이 때를 위해서 주요 파일들에 Apache 가 접근할 수 있도록 권한을 설정한다.

``` bash
$ chmod 664 ~/myproject/db.sqlite3
$ chmod 775 ~/myproject
$ sudo chown :www-data ~/myproject/db.sqlite3
$ sudo chown :www-data ~/myproject
```

이제 Apache 재시작을 해서 설정들을 적용하고 Django 에 접근 가능하도록 한다.

``` bash
$ sudo apache2ctl configtest
...
Syntax OK
$ sudo systemctl restart apache2
```

그 외에 방화벽을 사용하고 있어 외부에서 Apache 에 접근할 수 없다면 ufw firewall 나 iptables 설정으로 접근 가능하도록 해줘야 할 것이다.

방화벽 관련 설정은 생략한다.

해당 설정들은 [How to Serve Django Applications with Apache and mod_wsgi on Debian 8](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-apache-and-mod_wsgi-on-debian-8) 을 대부분 참조했다.

영어로 되어 있어도 알기 쉽게 설명되어 있어 따라하는데 별 무리가 없었다.

위 문서에서는 Apache 기본 설정인 000-default.conf 에 위와 같은 값들을 설정하여 별도의 사이트 활성화가 필요없지만 Apache 로 여러 사이트를 운영하고 있다면 별도의 설정을 작성하는 것이 맞을 것이다.

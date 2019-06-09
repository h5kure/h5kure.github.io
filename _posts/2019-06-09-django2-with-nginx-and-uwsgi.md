---
title: "Django 2 with Nginx and uWSGI"
date: 2019-06-09 10:00:00 -0400
categories: django
---

* Ubuntu 18.04
* Python3

## Virtual Env & Wrapper
Django Project 를 위한 가상 환경 설정

### Install

Python 3 를 사용할 경우에는 pip3 를 설치하고 이용

```bash
sudo apt update
sudo apt install python3-pip
```

시스템 전체에서 사용하기 위해 sudo 를 사용
```bash
sudo pip3 install virtualenv virtualenvwrapper
```

현재 계정에서 Virtual Env 환경을 위한 설정을 .bashrc 에 추가
```bash
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=~/venv
source /usr/local/bin/virtualenvwrapper.sh
```

### Create Project's Virtual Env

Project Virutal Env

```bash
mkvirtualenv project
```

가상환경 생성과 함께 activate 가 되면 pip 로 Django 나 필요한 패키지들을 설치한다.

Django 프로젝트 생성은 여기에서는 생략

프로젝트 가상환경을 activate 하기 위해서는 `workon` 명령어를 사용

```bash
workon project
```

## uWSGI

uWSGI 를 이용하기 위해선 python3-dev 가 설치되어 있어야 함.

```bash
sudo apt-get install python3-dev
```

uWSGI 를 시스템 전체에서 사용할 수 있도록 sudo 를 사용.

```bash
sudo pip install uwsgi
```

### Configuration File

uWSGI 에서 사용할 ini 설정파일들을 모아둘 곳을 생성하고 해당 폴더에 각 프로젝트별 uWSGI 설정을 `.ini` 파일로 만든다.
```bash
sudo mkdir -p /etc/uwsgi/sites
```

`myproject.ini` 를 생성

```ini
[uwsgi]
project = myproject
base = /home/user

chdir = %(base)/%(project)
home = %(base)/venv/%(project)
module = %(project).wsgi:application

master = true
processes = 5

socket = %(base)/%(project)/%(project).sock
chmod-socket = 664
chown-socket = user:www-data
vacuum = true
```

Nginx 와 통신에는 unix socket 을 사용하고 이 때 소켓이 생성될 때에는 접근 권한과 `사용자:그룹` 을 설정하여 Nginx 에서 접근이 가능하도록 한다.

`vacuum` 설정으로 `uWSGI` 가 종료되면 자동으로 socket file 을 삭제하는 등의 정리도 할 수 있도록 한다.

`unix socket`을 사용하지 않고 `http` 를 사용할 경우에는 `http = 0.0.0.0:8000` 과 같이 설정가능하다.

### Service Configuration

`uWSGI` 를 `systemctl` 명령어로 간단히 사용할 수 있도록 한다. 이를 이용해서 서버가 재시작되어도 자동으로 실행되도록 할 수도 있다.

* /etc/systemd/system/uwsgi.service

```
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown user:www-data /run/uwsgi'
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

이제 `uwsgi` 를 실행하면 `/etc/uwsgi/sites` 에 있는 `ini` 파일들을 사용하여 실행하게 된다.

### Install Nginx

```bash
sudo apt install nginx
``` 

### Nginx Site Configuration

* /etc/nginx/sites-available/myproject.conf

```
# configuration of the server                                                                                                                                                                                            
server {
    # the port site will be served on                                                                                                                                                                                    
    listen      80;
    server_name myproject.com;
    charset     utf-8;

    # max upload size                                                                                                                                                                                                    
    client_max_body_size 75M; # adjust to taste                                                                                                                                                                          

    location = /favicon.ico { access_log off; log_not_found off; }
    location /media/ {
        root /home/user/myproject;
    }

    location /static/ {
        root /home/user/myproject;
    }

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/user/myproject/myproject.sock;
    }
}

```

* enable myproject site

```bash
sudo ln -s /etc/nginx/sites-available/myproject.conf /etc/nginx/sites-enabled
```

## setup start nginx & uwsgi on boot
```bash
sudo systemctl enable nginx
sudo systemctl enable uwsgi

```


## 실행
이제 nginx 와 uwsgi 를 실행하면 django 프로젝트를 이용할 수 있다.
```bash
sudo systemctl start nginx
sudo systemctl start uwsgi
```

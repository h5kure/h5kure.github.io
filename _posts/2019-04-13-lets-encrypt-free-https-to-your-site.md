---
title: "Free HTTPS using Let's Encrypt"
date: 2019-04-13 22:00:00 -0400
categories: certificate
---
Apache 나 Nginx 등의 웹서버 기반으로 암호화를 위한 HTTPS 웹사이트를 구축하기 위해서는 도메인에 기반한 인증서가 필요

보통 openssl 을 이용하여 직접 인증서를 발급하는 self-signed 인증서를 만드는 방법과 인증서를 제공하는 기관에서 구입하는 방법이 있다.

self-signed 인증서의 경우, openssl 명령어로 마음대로 만들 수 있어 돈이 들지 않지만 신뢰할 수 있는 인증 기관에서 발금된 것이 아니기에
해당 웹서버에 접속하는 단말에 설치되지 않으면 웹브라우저에서는 신뢰할 수 없는 사이트라며 경고 메세지를 표시한다.

한,두대나 직접 설치를 한다고 해도 불특정 다수를 위한 서비스를 제공하기에는 적합하지 않다.

인증서를 제공하는 기관들은 대부분의 OS 에 신뢰할 수 있는 인증기관으로 등록되어 있기에 위와 같은 문제는 없지만 비용의 문제가 발생

# Let's Encrypt

 [Let's Encrypt](https://letsencrypt.org/) 는
 
 > TLS ( Transport Layer Security ) 암호화를 위한 X.509 인증서 를 무료로 제공하는 ISRG ( Internet Security Research Group)에서 운영하는 비영리 인증 기관
 
 이고,
 
 > 목표는 월드 와이드 웹 서버를 암호된 연결을 사용하는 유비쿼터스로
 
 누구나 간단하게 자신의 웹 사이트를 HTTPS 로 할 수 있게 간단한 툴을 제공하고 자동화도 가능하다.
 
 Domain-validated 한 인증서이고 사설 기관 등에서 제공하는 조직 인증 등의 확장 기능은 제공하지 않는다.
 
 그도 그럴 것이 누구나 간단히 발급할 수 있기 때문에 어쩔 수 없을 듯.
 
 간단한 툴이라는 것은 [certbot](https://certbot.eff.org) 이라는 것으로 간단한 몇중의 명령만으로 자신이 운영하는 웹 서버 도메인에 해당하는 인증서를 발급할 수 있다.
 
 certbot 홈페이지에서는 각자 필요로 하는 웹 서버의 형태 (Apache, Nginx, etc)와 OS 에 맞는 가이드 라인을 제공한다.
 
 Debian, Ubuntu 에서는 이 certbot 을 APT 명령어로 간단히 설치하여 이용할 수 있다.
 
 ## Limit
 
 보통 사설 기관에서 발급하는 인증서는 최소 1년 단위가 기본이지만, Let's Encrypt 에서 발급하는 인증서는 최대 90 일의 유효 기간을 가진다.
 
 다만 유효 기간이 지나기 전에 인증서를 재발급하는 것을 자동화 하는 것이 가능하다.
 
 특히, 인증서를 재발급 시에 웹 서버가 일시 중단되어 서비스가 불가능한 공백이 발생할 수 있지만, Apache Plug-in 을 이용하는 경우에는 그러한 공백 없이 이용하는 것이 가능하다고 한다.
 
 ## Apache 2, Ubuntu 18.04 bionic 에서
 
설치해야 하는 필요한 패키지로

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot python-certbot-apache 
```

위와 같이 안내하고 있는데 최신 Ubuntu 의 경우에는 Repository 가 이미 기본으로 제공되고 있어 Repository 를 별도로 추가할 필요없이 바로 설치가 가능했다.
그리고 이용하는 Python 버전으로 3 을 선혼한다면, `python-certbot-apache` 가 아니라 `python3-certbot-apache` 를 설치해야 할 것.

```bash
$ sudo apt-get install certbot python3-certbot-apache
```

DNS plug-in 도 제공을 하고 있어서 `*` 등을 이용할 필요가 있다면 필요에 따라 이용하는 기관에 해당하는 DNS plug-in 도 설치를 해야 한다고 한다.
이 역시 python3 를 선호한다면 그에 해당하는 plug-in 을 설치, 이용해야 할 것. (생략)

### 인증서 생성

```bash
$ sudo certbot --apache
```

certbot 명령어에 옵션으로 `--apache` 를 주면 Apache 설정을 참고하여 적용할 도메인 리스트를 자동으로 가져와서 보여주고 인증서를 발행한 후에는 자동으로 수정해준다고 한다.

나의 경우, 자동으로 설정을 적용하는 부분에서 오류가 발생하여 정상적으로 완료되지는 못 했다.

수동으로 설정하고 인증서만 발행하려면

```bash
$ sudo certbot --apache certonly
```

명령어를 이용한다.

그 후에 Apache VirtualHost 설정에

```
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/avsigner.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/avsigner.com/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/avsigner.com/chain.pem
```

와 같이 인증서를 설정해주고 Apache 를 재시작 해주면 끝이다.

```bash
$ sudo apache2ctl configtest
$ sudo systemctl restart apache2
```

### 인증서 갱신

인증서 재발행 역시 간단하게 한줄의 명령으로 가능하다.

```bash
$ sudo certbot renew
```

이를 자동화하고 싶으면 위 명령을 crontab 에 등록하여 정해진 주기마다 실행되도록 하면 될 것.

```
$ sudo crontab -e

...
0 0 1 * * certbot renew
...
```

나는 매달 1일에 인증서를 새로 생성하도록 설정했다.

# Conclusion

간단하게 이용 가능하고 복잡한 내용도 없어서 위 정도의 정보면 충분할 것이다.

더 자세한 내용을 참고하기 위해서는 홈페이지를 참고하는 것이 좋겠다.

[Let's Encrypt](https://letsencrypt.org/)

[certbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-apache)

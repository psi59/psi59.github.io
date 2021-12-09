---
title: "Let's Encrypt로 https 서비스 하기"
tags: ["ssl", 'tls', 'https', 'nginx', 'letsencrypt']
date: 2017-03-30T20:08:07+09:00
aliases:
  - "/2017/03/letsencrypt-certification-issue"
cover:
  image: "images/covers/letsencrypt.jpg"
---

Let's Encrypt 공짜 SSL 인증서 발급하기!!
<!--more-->
  
## 시작하기 앞서
우선 이 포스트는 개인적으로 공부한 것을 정리하고 문서화 하기 위한 포스트이며 [아웃사이더](https://blog.outsider.ne.kr/1178?category=31)님의 글을 보고 참고한 것이 많습니다. 참고하면서 겪었던 시행착오들을 기억을 더듬어 정리한 것입니다. 부족한 부분이 있을 수 있으니 그 점을 감안하여 참고하시기 바랍니다.
  
## 설치 환경 
OS: CentOS7  
WEB SERVER: nginx  

## 인증서 발급  
우선, 인증서를 발급받기 위해 [certbot](https://certbot.eff.org/)을 설치 해줍니다.
```bash
yum install epel-release
yum install certbot -y
```

공식 문서를 잘 보면 자동으로 인증해주는 방법도 있지만 이렇게 직접 해보는 방법도 공부라 생각하기에 이번 포스트에서는 직접 인증서를 발급 받도록 하겠습니다.
_추후에 자동으로 발급받는 방법도 업데이트 하도록 하겠습니다._

인증서를 발급받기 전에 사전 작업부터 해놓겠습니다.
```bash
mkdir -p /home/<user계정>/www/.well-known/acem-challenge  
# 폴더 생성 후 nginx 설정 파일에 아래의 내용 입력 
location /.well-known {
  root /home/<user계정>/www;
}
# 저장 후 꼭 리로드
```

다시 인증서를 발급하는 과정으로 넘어가겠습니다.
```bash
certbot certonly --manual
```

명령어를 입력하면 인증서 갱신이나 보안에 관련된 정보를 받을 수 있게 이메일을 입력하라고 나옵니다. 이메일 입력 후 진행 
```
Enter email address (used for urgent renewal and security notices)  
(Enter 'c' to cancel) : 이메일 주소
```

다음은 ACME 서버에 등록할 것인지 동의를 구하는 문구가 나오지만 사실 동의하지 않고 진행할 수 없기에 동의하고 진행 
```
-----------------------------------------------------------------
Please read the Terms of Service at  
https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf.   
You must agree in order to register with the ACEM server at  
https://acme-v01.api.letsencrypt.org/directory  
-----------------------------------------------------------------  
(A)gree/(C)ancel: A
```
  
다음은 방금 입력한 이메일로 메일을 보내도 되는지 묻지만 저는 받지 않도록 하겠습니다.
```
---------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier  
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and  
our work to encrypt the web, protect its users and defend digital rights.  
---------------------------------------------------------------------------  
(Y)es/(N)o: N
```
  
등록할 도메인을 입력하고 진행(복수의 도메인을 입력할 경우 공백이나 콤마로 구분해주면 됩니다.)
```
Please enter in your domain name(s) (comma and/or space separated)  
(Enter 'c' to cancel): www.example.com
```

![](/images/2017-03-30-letsencrypt/1.png)

웹서버가 존재한다면 지정된 위치에 파일을 생성하면 되지만 웹서버가 없다면 사진 아래부분처럼 간단하게 웹서버를 만들 수 있도록 예시를 적어준 것입니다. 저는 웹서버가 있으니 새로운 쉘을 띄워 위에 그림의 밑줄 1번을 _파일명_으로 밑줄 2번을 _파일 내용_으로 입력하여 저장합니다.
```bash
$ cat > /home/<user계정>/www/.well-known/acem-challenge/밑줄1
밑줄2
# ctrl + D 입력하여 저장
```

파일을 작성했다면 다시 인증서를 발급받던 쉘로 돌아와 enter를 눌러 진행하시면 됩니다.
이때 만약 서비스하려는 서버가 외부 접속을 막아놨다면 잠시 풀어주셔야 합니다. 왜냐하면 
Let's Encrypt가 `http://<발급하려는 도메인>/.well-known/acem-challenge/<작성한 파일명>`로 요청을 보내 파일이 존재하는지 파일 내용이 맞는지 인증하는 과정을 거치기 때문입니다.

![](/images/2017-03-30-letsencrypt/2.png)

인증에 성공하면 위와 같이 인증서가 저장된 경로를 알려줍니다.
인증서를 발급받은 후 아래의 내용을 적절히 수정하여 nginx에 반영해주면 끝입니다.
아래의 내용은 [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator/)에서 생성할 수 있습니다.

```
server {
    listen 443 ssl;

    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /path/to/dhparam.pem;

    # intermediate configuration. tweak to your needs.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;

    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    resolver 8.8.8.8 8.8.4.4 valid=86400;
    resolver_timeout 10;

    ....
}
```

위의 코드에서 수정할 부분은

```
ssl_certificate /etc/letsencrypt/live/<인증서를 발급한 도메인>/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/<인증서를 발급한 도메인>/privkey.pem;
```

```
ssl_dhparam /etc/letsencrypt/live/<인증서를 발급한 도메인>/dhparam.pem;
```
_dhparam.pem는 **`openssl dhparam -out dhparam.pem 2048`**로 생성할 수 있습니다._


```
ssl_trusted_certificate /etc/letsencrypt/live/<인증서를 발급한 도메인>/chain.pem;
```
위와 같이 수정하게 되면 대략 아래와 같은 내용이 완성됩니다.

```
server {
    ssl_certificate /etc/letsencrypt/live/<인증서를 발급한 도메인>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<인증서를 발급한 도메인>/privkey.pem;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /etc/letsencrypt/live/<인증서를 발급한 도메인>/dhparam.pem;

    # intermediate configuration. tweak to your needs.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;

    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/letsencrypt/live/<인증서를 발급한 도메인>/chain.pem;

    resolver 8.8.8.8 8.8.4.4 valid=86400;
    resolver_timeout 10;

    ...
}
```
저장 후 nginx를 리로드 합니다.

제대로 확인하고 따라했다면 이제 https를 반영한 도메인으로 접속하게되면 반영이 된 것을 확인 할 수 있다.
참고로 저는 창피하지만 아직 경험이 부족하다라는 핑계로 443포트를 열지 않아 왜 안되지 하고 있었습니다. 아직도 얼굴이 화끈거립니다. 이 글을 보는 분들은 이런 실수를 하지 않으셨으면 좋겠습니다.

## 마치며
부족하지만 끝까지 읽어주셔서 감사하며 부족한 부분이나 궁금한 부분들은 댓글로 남겨주시거나 메일로 남겨주시면 서로 도움이 될거 같습니다. 감사합니다.

#### Reference
 - [Lets' Encrypt로 무료로 HTTPS 지원하기](https://blog.outsider.ne.kr/1178)
 - [Let's Encrypt 공식 사이트](https://letsencrypt.org/)
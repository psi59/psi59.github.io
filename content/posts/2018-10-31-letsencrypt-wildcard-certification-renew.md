---
title: "Let's Encrypt 와일드카드 인증서 갱신해보기"
tags: ['https', 'letsencrypt']
date: 2018-10-31T20:29:00+09:00
aliases:
  - "/2018/10/letsencrypt-wildcard-certification-renew"
cover:
  image: "images/covers/letsencrypt.jpg"
---

<!--more-->

안녕하세요 이번 포스트에서는 지난번 게시한 [Let’s Encrypt 와일드카드 인증서 발급해보기](https://realsangil.github.io/http/18-08-04-letsencrypt)에서 받은 wildcard 인증서를 갱신하는 방법에 대해서 이야기 해보려합니다.  

## 인증서 갱신

기본적으로 `certbot`을 이용해 발급받은 인증서를 갱신하는 것은 매우 쉽습니다.
```bash
$ certbot-auto renew
```
위의 커맨드를 입력하시면 일반적으로 `certbot`이 인증서를 갱신을 진행하며 이 단계에서 별 무리없이 갱신이 끝날 것입니다. 

```bash
$ certbot-auto renew
Requesting to rerun /usr/local/bin/certbot-auto with root privileges...
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/lalaworks.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert is due for renewal, auto-renewing...
Could not choose appropriate plugin: The manual plugin is not working; there may be problems with your existing configuration.
The error was: PluginError('An authentication script must be provided with --manual-auth-hook when using the manual plugin non-interactively.',)
Attempting to renew cert (lalaworks.com) from /etc/letsencrypt/renewal/lalaworks.com.conf produced an unexpected error: The manual plugin is not working; there may be problems with your existing configuration.
The error was: PluginError('An authentication script must be provided with --manual-auth-hook when using the manual plugin non-interactively.',). Skipping.
All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/lalaworks.com/fullchain.pem (failure)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/lalaworks.com/fullchain.pem (failure)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1 renew failure(s), 0 parse failure(s)
``` 

하지만 저는 커맨드를 입력해 실행해봤지만 에러가 발생했습니다. 에러가 발생하는 이유는 제가 발급받은 인증서는 route53을 이용해서 받았기 때문에 인증서를 갱신할때 도메인 유효성을 체크하기 위해 route53에 세팅된 값을 체크해야하는데 그 값을 확인하지 못해기 때문에 에러가 발생했습니다.  

여기서 잠깐 `Let's Encrypt`가 어떻게 도메인 유효성 검사를 하는지 간단하게 설명 드리겠습니다.  

![let's encrypt의 domain validation](https://letsencrypt.org/images/howitworks_authorization.png)
출처: https://letsencrypt.org/how-it-works/

사용자의 서버에서 인증서를 갱신하거나 발급할 도메인에 `Let's Encrypt`에서 지정한 값을 리턴할 수 있도록 세팅합니다. 그 다음 `Let's Encrypt`가 그 도메인에 세팅된 값을 체크하고 값이 맞다면 인증을 완료합니다.

## route53 플러그인??

`Let's Encrypt`에서 route53을 공식적으로 지원하지만 route53을 이용해서 인증서를 갱신하기 위해서는 `certbot-dns-route53` 플러그인이 필요합니다. 플러그인은 아래의 커맨드로 설치하실 수 있습니다.

```bash
pip3 install certbot-dns-route53
```
플러그인을 사용하기 위해서는 `--dns-route53` 옵션을 주면 됩니다.

## 재시도..

```bash
certbot-auto renew --dns-route53
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/lalaworks.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert is due for renewal, auto-renewing...
Could not choose appropriate plugin: The requested dns-route53 plugin does not appear to be installed
Attempting to renew cert (lalaworks.com) from /etc/letsencrypt/renewal/lalaworks.com.conf produced an unexpected error: The requested dns-route53 plugin does not appear to be installed. Skipping.
All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/lalaworks.com/fullchain.pem (failure)
```

그렇죠 역시나 기대를 저버리지 않죠? 쉬운일 하나 없네요 ㅎㅎ 에러 로그를 보니 플러그인이 없다네요.. 아니 설치했자나!!!!!!!!!!!!!!!!!!!!!!!  

```bash
sudo cat /var/log/letsencrypt/letsencrypt.log
```

이런 하찮은 에러따위에 줄 시간 따윈 없으니 에러로그를 확인해보겠습니다. `Let's Encrypt`의 로그를 보려면 위의 커맨드로 로그를 확인해봅시다.

```bash
...
2018-10-31 11:43:33,708:DEBUG:certbot.renewal:Traceback was:
Traceback (most recent call last):
  File "/opt/eff.org/certbot/venv/local/lib/python2.7/site-packages/certbot/renewal.py", line 430, in handle_renewal_request
    main.renew_cert(lineage_config, plugins, renewal_candidate)
  File "/opt/eff.org/certbot/venv/local/lib/python2.7/site-packages/certbot/main.py", line 1191, in renew_cert
    installer, auth = plug_sel.choose_configurator_plugins(config, plugins, "certonly")
  File "/opt/eff.org/certbot/venv/local/lib/python2.7/site-packages/certbot/plugins/selection.py", line 237, in choose_configurator_plugins
    diagnose_configurator_problem("authenticator", req_auth, plugins)
  File "/opt/eff.org/certbot/venv/local/lib/python2.7/site-packages/certbot/plugins/selection.py", line 341, in diagnose_configurator_problem
    raise errors.PluginSelectionError(msg)
PluginSelectionError: The requested dns-route53 plugin does not appear to be installed
...
```

로그를 대략적으로 보면 아까 메시지와 비슷하게 플러그인을 못찾아서 발생한거 같습니다. 근데 여기서 자세히 보면 certbot은 `virtualenv`를 사용하여 파이썬환경을 분리해서 사용하고 있다는걸 알 수 있습니다 `virtualenv`를 간략하게 설명드리자면 프로젝트별로 파이썬환경을 분리하고자 만들어진것입니다. 자세한 내용은 구글신에게 물어보시면 더 잘 이해하실 수 있습니다.  
 
다시 본론으로 돌아와 **요약해보자면,** 저는 시스템 파이썬에 플러그인을 설치했기 때문에 분리된 환경을 사용하는 certbot은 당연히 플러그인이 없다고 에러를 발생시키는겁니다. 그럼 이제 저 분리된 공간에 플러그인을 설치해보겠습니다.

 ```bash
 sudo /opt/eff.org/certbot/venv/bin/pip install certbot-dns-route53
 ```

 위의 커맨드를 실행하여 플러그인을 설치하고 다시 갱신 커맨드를 실행해보겠습니다.

```bash
$ certbot-auto renew --dns-route53
...
Attempting to renew cert (lalaworks.com) from /etc/letsencrypt/renewal/lalaworks.com.conf produced an unexpected error: An error occurred (AccessDenied) when calling the ListHostedZones operation: User: arn:aws:sts::00000000000:assumed-role/AmazonLightsailInstanceRole/i-000000xxxxxx is not authorized to perform: route53:ListHostedZones
To use certbot-dns-route53, configure credentials as described at https://boto3.readthedocs.io/en/latest/guide/configuration.html#best-practices-for-configuring-credentials and add the necessary permissions for Route53 access.. Skipping.
All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/lalaworks.com/fullchain.pem (failure)
...
```

또 다시 에러가 발생했네요 메시지를 보면 aws route53을 awscli를 통해 제어해야 하는데 awscli에 route53를 이용하기 위한 인증정보를 세팅해야 한다는 것을 알 수 있습니다.

AWS의 `IAM`를 이용해서 발급받으시면 됩니다.
액세스 키를 발급받으시고 아래의 커맨드를 실행하시고 발급받으신 키를 입력하시면 됩니다.

```bash
sudo aws configure
```

이 때 반드시 awscli를 관리자 권한으로 실행해주셔야 합니다 왜냐하면 awscli는 리눅스 계정별로 인증정보를 관리하게 되는데 관리자 권한 없이 커맨드를 실행하면 현재 사용중인 리눅스 계정에 aws 인증정보가 생성되어 관리자 권한으로 실행되는 certbot이 인증정보를 제대로 읽지 못해 같은 에러가 발생하게 됩니다.

이런것도 싫다 하시면 아래와 같이 config파일을 생성해주시면 됩니다.

```bash
sudo vi /root/.aws/config

[default]
aws_access_key_id=<access_key_id>
aws_secret_access_key=<access_secret_key>
```

이제 준비가 된거 같으니 다시 갱신을 시도해보겠습니다.

```bash
$ certbot-auto renew --dns-route53
...
Cert is due for renewal, auto-renewing...
Found credentials in shared credentials file: ~/.aws/credentials
Plugins selected: Authenticator dns-route53, Installer None
Renewing an existing certificate
Performing the following challenges:
dns-01 challenge for lalaworks.com
dns-01 challenge for lalaworks.com
Waiting for verification...
Cleaning up challenges
... 

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/lalaworks.com/fullchain.pem (success)
```

오 성공했습니다. 이 맛에 개발하는거 같내요 성취감 굳ㅎㅎ

## 마무리하며...
 친절히 메일로 만료가 얼마 남지 않았다는걸 알려줬지만 미루고 미루던 인증서를 갱신했고 포스트도 작성했습니다.  
 이번 포스팅을 통해서 let's encrypt가 어떻게 인증서를 갱신하는지 대략적인 플로우도 알 수 있었던거 같습니다. 다음 포스팅에서는 갱신 자동화와 인증서 배포에 대해서 한번 알아보도록 해보겠습니다.

 이 포스트에 잘못된 정보나 궁금하신점이 있으시다면 tkddlf59@gmail.com으로 메일을 주시거나 댓글로 알려주시기 바랍니다. 여러분의 따뜻한 관심이 절실히 필요한 때입니다.

 ## 참고자료
  - https://github.com/certbot/certbot/issues/4875
  - https://letsencrypt.org/how-it-works/
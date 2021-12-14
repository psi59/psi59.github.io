---
title: "Let's Encrypt 와일드카드 인증서 발급해보기"
tags: ['https', 'letsencrypt']
date: 2018-08-04T11:41:00+09:00
aliases:
  - "/2018/08/04/letsencrypt-wildcard-certification"
  - "/2018/08/letsencrypt-wildcard-certification"
cover:
  image: "images/covers/letsencrypt.jpg"
---

<!--more-->

이번 포스트에서는 Let's Encrypt에서 wildcard 인증서를 발급받는 방법을 포스팅해보려 합니다.  
들어가기 앞서 서브 도메인과 wildcard 인증서에 대해서 간단히 설명드리고 시작하겠습니다.

## Sub Domain이란?

일반적인 도메인들은 `www.lalaworks.com`의 포멧으로 이뤄져 있습니다.  
여기서 `com`은 *Top Level Domain (TLD)*, `lalaworks`는 *Second Level Domain (SLD)*, `www`는 *Subdomain* 이라 칭합니다.  
**Subdomain**은 보조 도메인으로서 기억하기 쉬운 도메인 주소를 만들어 줄 뿐만 아니라 성격이 다른 서비스를 구분할 수 있도록 해주는 역할을 합니다.


## Wildcard 인증서란?

일반적인 인증서는 인증서당 하나의 도메인만 적용할 수 있습니다. 예를 들면 `www.lalaworks.com`의 도메인으로 발급한 인증서는 `example.lalaworks.com`의 도메인에서는 `www.lalaworks.com`로 발급한 인증서를 사용할 수 없습니다. 일반적인 인증서는 각각의 도메인마다 발급받아야 한다는 단점이 있습니다. 이러한 단점을 해결하기 위해 만들어진 것이 Wildcard 인증서인데요. Wildcard 인증서는 `lalaworks.com`의 이름을 가진 모든 도메인에서 사용할 수 있는 인증서 입니다.  

## 발급받기

**주의: certbot에서 아직 aws는 실험적으로 지원하고 있으므로 기존에 사용하고 계시던 인증서는 반드시 백업해주시기 바랍니다.**  

이 포스트에서는 AWS Instance를 기준으로 설명드릴 것이며 redhat계열의 리눅스는 거의 동일할 것이며, debian계열에서는 `git`을 설치하는 과정만 다를것이며 나머지 방법은 동일합니다.

제가 사용하고 있는 인스턴스의 OS의 정보는 아래와 같습니다.

```bash
/etc/os-release:NAME="Amazon Linux AMI"
/etc/os-release:VERSION="2018.03"
/etc/os-release:ID="amzn"
/etc/os-release:ID_LIKE="rhel fedora"
/etc/os-release:VERSION_ID="2018.03"
/etc/os-release:PRETTY_NAME="Amazon Linux AMI 2018.03"
/etc/os-release:ANSI_COLOR="0;33"
/etc/os-release:CPE_NAME="cpe:/o:amazon:linux:2018.03:ga"
/etc/os-release:HOME_URL="http://aws.amazon.com/amazon-linux-ami/"
```

### certbot clone 받기
```bash
# clone 받기 위해 git 설치
yum install -y git
git clone https://github.com/certbot/certbot.git
cd /path/to/certbot
```

### 발급 Command
```bash
./certbot-auto certonly \
--manual \
--preferred-challenges=dns \
--email <you@email.com> \
--server https://acme-v02.api.letsencrypt.org/directory \
--agree-tos \
--debug \
--no-bootstrap \
-d yourdomain.com \
-d *.yourdomain.com
```
위의 command를 보시면 아시겠지만 `yourdomain.com`, `*.yourdomain.com` 두 개의 도메인을 입력해주셔야 합니다.

저는 `lalaworks.com`라는 도메인으로 한번 발급받아 보겠습니다.

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
```

명령어를 입력하면 이메일로 let's encrypt의 소식을 받을 것이냐고 묻는데 이건 각자의 기호에 따라 동의하시거나 거절하시면 됩니다.


```bash
NOTE: The IP of this machine will be publicly logged as having requested this
certificate. If you're running certbot in manual mode on a machine that is not
your server, please ensure you're okay with that.

Are you OK with your IP being logged?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
```

그 다음은 인증서를 발급할 머신의 IP를 기록해도 될 것이냐고 묻는데 거절하시면 반드시 동의해야 한다는 문구가 나오므로 동의하시고 넘어가시면 됩니다.  

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.lalaworks.com with the following value:

XXXXXXXXXXXXXXXXXXXXXXXX

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.lalaworks.com with the following value:

XXXXXXXXXXXXXXXXXXXXXXXX

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

_DNS에 레코드를 등록하지 않고 바로 Enter를 입력하시면 레코드가 없는 상태에서 인증을 시도하기 때문에 반드시 DNS에 레코드를 등록하신 후에 다음으로 진행해주세요._

위의 메시지를 보시면 DNS에 `_acme-challenge.lalaworks.com`라는 이름으로 TXT레코드를 등록해달라고 하네요
앞서 말씀드렸다 싶이 저는 AWS를 사용중이므로 `Route 53`에 이걸 등록해보도록 하겠습니다.

![Route_53_1](/images/2018-08-04-letsencrypt/create_record.png)

사진과 같이 위의 커맨드라인에서 `XXXXXXXXXXXXXXXXXXXXXXXX` 위치에 나온 값 2개를 등록해 주세요.  
_참고로 Route 53에서는 newline으로 값을 구분하며 ""으로 값을 묶어주시면 됩니다._

레코드를 등록하셨다면 레코드에 요청을 했을 때 정상적으로 입력한 값들이 내려오는지 한번 확인해 본 후에 진행하는것을 추천드립니다.
Route 53에서 레코드를 테스트할 때는 `Test Recode Set`이라는 버튼을 눌러주세요.

![Route_53_2](/images/2018-08-04-letsencrypt/check_record_in_route53.png)

위의 사진과 같이 Reccord name에  `_acme-challenge`를 입력해주시고 Type을 `TXT`로 바꿔주신 후에 Get response를 눌러주시고 값을 확인한 후 다음으로 진행하시면 됩니다.

```bash
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/lalaworks.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/lalaworks.com/privkey.pem
   Your cert will expire on 2018-11-02. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

다시 Terminal로 돌아와서 Enter를 눌러주신 후 성공적으로 인증을 마쳤다면 위의 메시지가 보이는데 간단히 말씀드리면 `/etc/letsencrypt/live/lalaworks.com/fullchain.pem`, `/etc/letsencrypt/live/lalaworks.com/privkey.pem`에 발급받은 키들이 저장되었다는걸 알 수 있으며 만료일은 세 달 뒤인 `2018-11-02`이며 `certbot-auto renew`의 커맨드로 갱신할 수 있다고 하네요 ㅎㅎ 친절한 CertBot씨


`./certbot-auto certificates` 커맨드로 제대로 발급되었는지 확인해볼 수 있습니다.

```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: lalaworks.com
    Domains: lalaworks.com *.lalaworks.com
    Expiry Date: 2018-11-02 02:08:26+00:00 (VALID: 89 days) 
    Certificate Path: /etc/letsencrypt/live/lalaworks.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/lalaworks.com/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

적용하는 부분은 [Let’s Encrypt로 https 서비스 하기](https://realsangil.github.io/http/17-03-31-letsencrypt)를 참고하시면 될거 같습니다.

## Conclusion
인증서를 발급받으면서 간단하게 도메인 구조에 대해서도 다시 공부하게 됐고 인증서도 발급받고 아주 유익한 시간이었습니다.
위의 과정은 어디까지나 참고하시는 사항이고 저와 환경이 같지 않다면 저 흐름에서 크게 벗어나지는 않겠지만 조금은 다를 것입니다. 

대부분의 let's encrypt 사용자가 사용하시겠지만은 팁을 드리자면 crontab에 주기적으로 갱신스크립트를 호출하는 스케쥴을 등록하시면
매번 직접 갱신하지 않으셔도 자동으로 갱신된다는 아주 꾸르팁!!

궁금하신 사항이나 잘못된 정보가 있다면 댓글이나 tkddlf59@gmail.com으로 메일 주시기 바랍니다.

## References
 - https://kr.godaddy.com/help/what-is-a-subdomain-296

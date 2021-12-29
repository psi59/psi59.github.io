---
title: 'Docker DNS 문제 해결하기'
date: 2018-04-15T06:15:00+09:00
tags: ['docker', 'dns']
aliases:
  - '/2018/04/docker-dns-problem/'
cover:
  image: "/images/covers/docker.png"
  caption: "출처: docker.com"
  relative: true
---

<!--more-->

이번 포스트는 제가 우분투 이미지를 사용해 개발 환경을 만들다 발생한 에러에 대해 얘기해보려 합니다.

## Error Log

```bash
Could not resolve 'archive.ubuntu.com'
```

dns에서 `archive.ubuntu.com`을 찾지 못했을 때 발생하는 에러이며 CentOS라던지 다른 리눅스의 패키지 매니저를 사용할 때 저 에러와 비슷한 에러를 보셨을 때 해결할 수 있는 방법을 얘기해보려 합니다.

## 왜 이런 문제가 발생하는가?

이 에러는 `/etc/resolve.conf`에 정의된 DNS 서버를 찾지 못했을 때 발생하는 에러입니다.

## 어떻게 해결하는가?

### 해결법 1. docker container를 실행할 때 파라메터 추가

이 방법은 해당 컨테이너에 한해서만 dns 설정을 변경하고 싶을 때 사용할 수 있는 방법입니다.

```bash
docker run ... --dns 8.8.8.8
```

### 해결법 2. docker 설정 변경

이 방법은 docker의 모든 컨테이너에 영향을 주는 설정이므로 신중히 생각해보시고 적용하셔야 합니다.

```bash
# /etc/default/docker 파일에 아래 내용 삽입
DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4"
```

수정 후에는 반드시 docker service를 재시작 해주셔야 적용 됩니다.

```bash
service docker restart
```

## 마치며..

대부분의 서비스에서는 `8.8.8.8`(google dns)로 설정해 에러를 해결합니다만 최근에 cloudflare가 `1.1.1.1`의 주소로 국내에 dns서버를 오픈했습니다. 때문에 싱가포르에 서버를 두고 있는 google dns보다 30ms정도 더 빠르다고 합니다. 참고하시어 설정하시면 좋을거 같습니다.
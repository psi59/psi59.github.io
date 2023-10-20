---
title: "AWS Lambda 로컬 디버깅 (provided.al2 Runtime)"
tags: ["golang", "docker", "aws", "lambda"]
date: 2023-10-20T23:00:07+09:00
---

이 포스트는 2023-12-31부터 `go1.x` 런타임이 deprecate 됨에 따라 `provided.al2` 런타임으로 마이그레이트한 람다 함수를 로컬에서 디버깅하는 방법에 대해 포스팅한 내용임.

<!--more-->


## Docker Image 생성

아래 스크립트는 람다 컨테이너 이미지다.

```dockerfile
FROM golang:1.21-alpine as builder
WORKDIR /usr/src/app
COPY . .
RUN go install github.com/go-delve/delve/cmd/dlv@latest
RUN go mod vendor
RUN	GOOS=linux GOARCH=arm64 go build -gcflags="all=-N -l" -tags lambda.norpc -o /usr/src/app/main .

FROM public.ecr.aws/lambda/provided:al2
RUN mkdir /www
COPY --from=builder /usr/src/app/main /usr/bin/main
COPY --from=builder /go/bin/dlv /usr/bin/dlv
EXPOSE 2345
ENTRYPOINT ["/usr/bin/"]
```

AWS에서 게재한 예시와 차이점은 dlv를 추가한 것과 Go 바이너리 빌드시 `-gcflags="all=-N -l"` 플래그를 추가했다는 점이다.

```sh
# 도커 이미지 빌드
docker build -t lambda-test .
```

## 로컬에서 디버깅

```sh
# 도커 컨테이너 실행
docker run --rm --name lambda-test \
	-v ~/.aws-lambda-rie:/aws-lambda \
	-p 9000:8080 -p 2345:2345 \
    --entrypoint /aws-lambda/aws-lambda-rie \
    lambda-test \
    /usr/bin/dlv --listen=:2345 --headless=true --api-version=2 --accept-multiclient exec /usr/bin/main --continue

# 바이너리에 추가로 플래그를 추가해야할 경우 {your-flag} 자리에 필요한 플래그를 추가하면 된다.
docker run --rm --name lambda-test \
	-v ~/.aws-lambda-rie:/aws-lambda \
	-p 9000:8080 -p 2345:2345 \
    --entrypoint /aws-lambda/aws-lambda-rie \
    lambda-test \
    /usr/bin/dlv --listen=:2345 --headless=true --api-version=2 --accept-multiclient exec /usr/bin/main --continue -- {your-flag}
```

위 명령어에서 볼륨으로 연결하는 `~/.aws-lambda-rie` 폴더는 [링크](https://github.com/aws/aws-lambda-runtime-interface-emulator#installing)를 통해서 환경에 맞게 설치하면 홈폴더 아래에 설치된다.

명령어를 실행하여 컨테이너를 띄우고 IDE에 remote 디버깅을 위한 환경을 세팅한다.
이 포스트는 jetbrains의 IDE를 기준으로 작성되었다.

`Run/Debug Configurations`에서 go remote를 세팅해준다.

![run configuration](/images/2023-10-20-lambda-local-debugging/6D928A39-C1D8-4778-B382-F43218630D68.png)

2345 포트를 세팅한 후 실행한다.

```sh
# 테스트 요청
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d '{}'

# 파일로 저장된 이벤트 요청
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" -d @your_file_name.json
```

## References

- https://aws.amazon.com/ko/blogs/compute/migrating-aws-lambda-functions-from-the-go1-x-runtime-to-the-custom-runtime-on-amazon-linux-2/
- https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/go-image.html#go-image-clients
- https://mingrammer.com/debugging-containerized-go-app/#%EB%94%94%EB%B2%84%EA%B9%85-%EC%9D%B4%EB%AF%B8%EC%A7%80
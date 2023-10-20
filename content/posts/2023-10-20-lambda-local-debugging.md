---
title: "AWS Lambda 로컬 디버깅 (Go1.x Runtime)"
tags: ["golang", "aws", "lambda"]
date: 2023-10-20T13:18:01
draft: true
---

Go1.x 런타임을 사용한 AWS Lambda function을 로컬에서 디버깅하는 방법에 대한 포스팅입니다.

<!--more-->

> 해당 내용은 Go1.x 런타임으로 배포된 람다 함수의 디버깅 방법이며 M1 프로세서에서는 동작하지 않습니다.

## 리눅스용 delve 빌드하기
```bash
git clone https://github.com/go-delve/delve.git
cd /path/to/delve
GOARCH=amd64 GOOS=linux go build -o dlv github.com/go-delve/delve/cmd/dlv
```

## SAM 커맨드 세팅
### Go binary 빌드
```bash
# go build
## 빌드 시 -gcflags "all=-N -l" 옵션을 줘서 빌드한다.
GOOS=linux GOARCH=amd64 go build -gcflags "all=-N -l" -o $(handler_name) .
```

### SAM 실행
```bash
# --debugger-path your_delve_bin_dir ==> 리눅스용 delve 바이너리가 있는 directory
# -d port ==> delve 디버깅
# --debug-args "-delveAPI=2" ==> delve parameter
sam local invoke FuncName -d 2345 --debugger-path your_delve_bin_dir --debug-args "-delveAPI=2" --event $(event) --profile your_profile --debug
```

## Goland 세팅
`Run/Debug Configurations`에서 go remote를 세팅해준다.

2345 포트를 세팅한 후 실행한다.

![run configuration](/images/2023-10-20-lambda-local-debugging/6D928A39-C1D8-4778-B382-F43218630D68.png)

## References
* [Step-Through Debugging - Golang Functions Locally - AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-debugging-golang.html)
* [GitHub - go-delve/delve: Delve is a debugger for the Go programming language.](https://github.com/go-delve/delve)
* [컨테이너 내부 Go 애플리케이션 디버깅하기 · mingrammer's note](https://mingrammer.com/debugging-containerized-go-app/)
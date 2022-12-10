---
title: "도커 컨테이너로 배포된 서버를 우아하게 종료하기✨"
tags: ["docker", "golang"]
date: 2020-11-14T09:30:00+09:00
aliases:
  - "/2020/11/docker-container-graceful-shutdown/"
cover:
  image: "/images/covers/docker.png"
  caption: "출처: docker.com"
  relative: true
---

프로덕션 환경에서 서버를 정상적으로 종료되는 것은 아주 중요합니다. 
예를 들어 이미 수신한 요청을 온전히 처리하지 않고 서버가 종료될 때 클라이언트는 `502`나 `504` 에러를 수신하게 됩니다. 
가장 기본적인 방법으로는 프로세스 종료 SIGNAL 수신 후 일정 시간을 기다린 후에 서버를 종료하는 방법이 있습니다.

<!--more-->

## 서버 종료 요구사항
서버를 정상적으로 종료하기 위해서는 기본적으로 2가지의 요구사항이 존재합니다. 
1. 서버가 종료될 때는 수신한 요청을 모두 응답한 후 종료.
2. 서버가 닫힌 후에는 요청을 수신하면 안됨.

## Go에서 Graceful Shutdown 구현하기
아래는 요청을 수신하고 5초 뒤 OK 메시지를 반환하는 간단한 예제 입니다.
```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	h := &HTTPHandler{}
	server := &http.Server{Addr: ":1202", Handler: h}
	if err := server.ListenAndServe(); err != nil {
		fmt.Printf("error: %v\n", err)
	}
}

type HTTPHandler struct{}

func (h *HTTPHandler) ServeHTTP(res http.ResponseWriter, req *http.Request) {
	time.Sleep(5 * time.Second)
	res.WriteHeader(http.StatusOK)
	res.Write([]byte("OK\n"))
}
```

서버 실행 후 `curl`를 통해 테스트 하면 5초 뒤 OK 메시지를 수신할 수 있습니다.

이 때, 응답을 수신하기 전에 서버를 종료하는 경우를 테스트 해보겠습니다.
```bash
❯ timeout 3 go run main.go &
[1] 81662
❯ time curl -v 'localhost:1202'
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 1202 (#0)
> GET / HTTP/1.1
> Host: localhost:1202
> User-Agent: curl/7.64.1
> Accept: */*
>
* Empty reply from server
* Connection #0 to host localhost left intact
curl: (52) Empty reply from server
* Closing connection 0
curl -v 'localhost:1202'  0.00s user 0.00s system 0% cpu 2.241 total
[1]  + 81662 exit 124   timeout 3 go run main.go
```
결과를 보시면 서버와의 연결이 커맨드 실행을 수동으로 했기 때문에 약간의 오차가 있지만 3초(2.241초) 후에 종료 되었고 OK 메시지를 수신하지 못한 것을 확인할 수 있습니다.

### Graceful Shutdown이 적용된 HTTP Server
Go에서는 기본적으로 graceful shutdown를 추가적인 라이브러리 없이 구현 가능합니다.
```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

var logger = log.New(os.Stdout, "", log.LstdFlags)

func main() {
	h := &HTTPHandler{}
	server := &http.Server{Addr: ":1202", Handler: h}

	// goroutine으로 서버 실행
	go func() {
		logger.Println("Start Server...")
		if err := server.ListenAndServe(); err != nil {
			logger.Printf("error: %v\n", err)
		}
	}()
	// OS SIGNAL을 수신할 chanel 생성
	signalChan := make(chan os.Signal, 1)
	// KILL, INTERRUPT, QUIT, TERM SIGNAL 수신 등록
	signal.Notify(
		signalChan,
		os.Interrupt,
		os.Kill,
		syscall.SIGQUIT, // kill -SIGQUIT XXXX
		syscall.SIGTERM,
	)
	// SIGNAL 수신
	sig := <-signalChan
	logger.Printf("Received SIGNAL: %v\n", sig)
	// timeout을 위한 context 생성
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	logger.Println("Close Server...")
	// context에 지정한 timeout만큼 shutdown 지연
	if err := server.Shutdown(ctx); err != nil {
		logger.Printf("error: %v\n", err)
	}
}

type HTTPHandler struct{}

func (h *HTTPHandler) ServeHTTP(res http.ResponseWriter, req *http.Request) {
	logger.Println("Receive Request...")
	time.Sleep(5 * time.Second)
	res.WriteHeader(http.StatusOK)
	res.Write([]byte("OK\n"))
	logger.Println("Send Response...")
}
```
변경된 코드에 대한 설명은 주석으로 간략하게 표현했습니다. 
핵심적인 차이는 Server를 goroutine으로 실행하고 OS로부터 종료 SIGNAL을 수신하면 종료하는 부분입니다.

이번에도 위와 동일하게 테스트를 해보겠습니다.
```bash
❯ timeout 3 go run main.go
2020/11/09 09:18:06 Start Server...
2020/11/09 09:18:07 Receive Request...
2020/11/09 09:18:08 Received SIGNAL: terminated
2020/11/09 09:18:08 Close Server...
2020/11/09 09:18:08 error: http: Server closed

❯ time curl -v 'localhost:1202'
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 1202 (#0)
> GET / HTTP/1.1
> Host: localhost:1202
> User-Agent: curl/7.64.1
> Accept: */*
>
2020/11/08 20:40:23 Receive Request...
2020/11/08 20:40:25 Received SIGNAL: terminated
2020/11/08 20:40:25 Close Server...
2020/11/08 20:40:25 error: http: Server closed
[1]  + 35087 exit 124   timeout 3 go run main.go
2020/11/08 20:40:28 Send Response...
< HTTP/1.1 200 OK
< Date: Sun, 08 Nov 2020 11:40:28 GMT
< Content-Length: 3
< Content-Type: text/plain; charset=utf-8
< Connection: close
<
OK
* Closing connection 0
curl -v 'localhost:1202'  0.00s user 0.00s system 0% cpu 5.012 total
```
`curl`의 로그와 섞여서 복잡하지만 로그를 보시면 서버는 이미 종료 되었지만 5초가 지난 후 성공적으로 응답을 수신한 것을 알 수 있습니다.


## Dockerfile 작성
위에서 작성한 서버를 Docker image로 만들어 보겠습니다.
```dockerfile
# first.Dockerfile
FROM golang
LABEL maintainer=tkddlf59@gmail.com
COPY server /usr/bin/server
ENTRYPOINT "server"
```

```bash
# build docker image 
> docker build -t graceful_shutdown .
# run docker container
> docker run -d -p 1202:1202 --name graceful_shutdown .
# stop container
> docker stop graceful_shutdown & time curl -v 'localhost:1202'
# log 확인
> docker log graceful_shutdown
2020/11/08 23:53:35 Start Server...
2020/11/08 23:53:41 Receive Request...
2020/11/08 23:53:46 Send Response...
```
로그를 확인 해보면 도커는 안정적인 컨테이너를 종료를 위해 디폹트로 10초의 지연이 있습니다. 컨테어너 종료 지연 덕분에 응답은 정상적으로 반환 했지만 SIGNAL을 수신받지 못한 것을 알 수 있습니다.
이런 경우 발생할 수 있는 문제는 컨테이너가 완전히 종료되기 전까지는 요청을 계속해서 수신한다는 것입니다.

그럼 이번에 Dokerfile의 `ENTRYPOINT` 부분을 조금 수정 해보겠습니다.
```dockerfile
# second.Dockerfile
FROM golang
LABEL maintainer=tkddlf59@gmail.com
COPY server /usr/bin/server
# 수정된 부분
ENTRYPOINT ["server"]
```
새롭게 빌드 후 동일하게 테스트하면 서버가 정상적으로 종료 SIGNAL을 수신하고 서버를 종료하기 위해 Client의 요청을 Block하는 것을 확인할 수 있습니다.

docker에서는 `ENTRYPOINT`와 `CMD`를 **exec-form(JSON array)**, **shell-form(문자열)**으로 처리합니다. exec-form은 `exec` 커맨드를 통해 실행 되지만 shell-form은 `sh -c` 커맨드를 통해 실행되기 때문에 SIGNAL을 전달 받을 수 없습니다. 그 외의 차이는 [문서](https://docs.docker.com/engine/reference/builder/#entrypoint)를 통해 확인하시면 되겠습니다.

## 정리
이번 포스트에서는 Docker 컨테이너로 만든 Go 서버의 우아하게 종료하는 방법에 대해서 알아 보았습니다. 
핵심은 서버가 Graceful Shutdown을 지원하더라도 Dockerfile을 작성할 때 `ENTRYPOINT`와 `CMD`를 exec-form으로 작성해야만 SIGNAL을 수신 받을 수 있다는 것입니다.
해당 코드는 [Graceful Shutdown이 적용된 Go HTTP server · GitHub](https://gist.github.com/psi59/3c6d71367a3f3f03d170c4655ee055a4)에서 확인가능합니다.

## References
- [Why Your Dockerized Application Isn’t Receiving Signals · Homepage of Hynek Schlawack](https://hynek.me/articles/docker-signals/)
- [Dockerfile feference - Entrypoint](https://docs.docker.com/engine/reference/builder/#entrypoint)
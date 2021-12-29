---
title: "Golang TestMain이란?"
tags: ["golang", "testing"]
date: 2021-12-29T20:34:00+09:00
cover:
    image: "/images/covers/golang.png"
---

이 포스트는 `TestMain`가 무엇이며, 어떤 식으로 사용할 수 있는지에 대해서 정리한 내용입니다.

<!--more-->

## TestMain

> 이번 포스트에서 말하는 TestMain은 main 함수의 테스트가 아닙니다.

가끔 테스트를 작성하다보면 테스트에 필요한 setup이나 teardown이 필요할 때가 있습니다.

Go에서는 테스트 패키지에 따로 setup이나 teardown을 제공하고 있지 않기 때문에 `TestMain(m *testing.M)`을 통해서 해당 작업을 해야 합니다.

예를 들어, 테스트 전에 임시 DB를 생성하고 테스트가 완료되면 임시 DB를 지워야 하는 경우가 있는데 Go 언어에서는 을 통해서 테스트 전후로 필요한 작업들을 할 수 있습니다.

## 테스트 코드

코드는 매우 간단합니다.

```go
func TestMain(m *testing.M) {
    // 테스트 전처리 작업
    
    exitCode := m.Run()

    // 테스트 후 처리 작업

    os.Exit(exitCode)
}
```

아래에 테스트를 위해 작성한 간단한 코드를 통해서 `TestMain`이 어떻게 동작하는지 확인해보겠습니다.

```go
package main

import (
    "fmt"
    "os"
    "testing"
)

func init() {
    fmt.Println("init 함수 실행")
}

func TestMain(m *testing.M) {
    setup()
    code := m.Run()
    teardown()
    os.Exit(code)
}

func Test_execute1(t *testing.T) {
    execute1()
}

func Test_execute2(t *testing.T) {
    execute2()
}

func execute1() {
    fmt.Println("TEST-1 테스트 중입니다.")
}

func execute2() {
    fmt.Println("TEST-2 테스트 중입니다.")
}

func setup() {
    fmt.Println("setup...")
}

func teardown() {`
    fmt.Println("teardown...")
}
```

```bash
go test  

init 함수 실행
setup...
TEST-1 테스트 중입니다.
TEST-2 테스트 중입니다.
PASS
teardown...
```

테스트 코드를 실행해보면 `Test_execute1`과 `Test_execute2`를 실행 전후로 `setup`, `teardown`가 함수가 실행되는걸 확인할 수 있습니다.

실제로 Go언어로 작성된 [mattermost의 서버](https://github.com/mattermost/mattermost-server)는 db 테스트 시 `TestMain`을 이용하여 테스트 전후로 테스트 DB를 만들고 삭제하고 있습니다.

가끔 패키지에서 테스트에 자원할당이나 해제와 같은 작업들이 공통적으로 필요할 때 사용하면 매우 유용할 거 같습니다.

## References

- https://pkg.go.dev/testing#hdr-Main
- http://cs-guy.com/blog/2015/01/test-main/

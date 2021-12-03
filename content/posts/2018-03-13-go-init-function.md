---
title: "Go의 init 함수 간단 소개!!"
tags: ['golang']
date: 2018-03-13T07:52:02+09:00
cover:
    image: "/images/covers/golang.png"
---

오늘은 Go언어의 init function에 대해 얘기해보려 합니다.

## init 함수?
`init` 함수는 이름에서 느껴지듯이 무언가를 초기화하는 함수입니다. 
일반적으로 `init` 함수는 다음과 같이 쓸 수 있습니다.  
* 일반적익 초기화 표현식으로 변수를 초기화 할 수 없는 경우.
* 일회성 계산
* 프로그램 상태 확인 / 수정
* 등등...

그럼 `init` 함수가 대체 뭐길래 위의 사항과 같이 쓸 수 있는지 설명드리겠습니다.
go에서 `init` 함수는 프로그램 실행시 변수 초기화 다음 호출되는 함수입니다.
각 파일 별로 선언할 수 있지만 직접 호출하거나 참조할 수 없다는 점 이외에 일반 함수들과 같습니다.

## Example

감이 잘 안오신다면 예제와 함께 설명 드리겠습니다.
뭐든 헷갈리시면 직접 해보시는게 제일 좋습니다.  
결과를 보시기 전에 한번 예측해보시기 바랍니다.

### Example 1

```go
// main.go
package main

import "fmt"

var v = variableInit()

func main() {
    fmt.Print("called main\n")
}

func init() {
    fmt.Print("called init\n")
}

func variableInit() int {
    fmt.Print("called variableInit\n")
    return 0
}

```

위의 예제를 보면 어떤 결과 값이 예상 되십니까??

**결과값** 

```go
called variableInit
called init
called main
```

위의 결과를 보면 `init`을 호출하지 않아도 변수 초기화 후에 호출되는 것을 확인하실 수 있습니다.  
그럼 다른 예제도 함께 확인해 볼까요?? 사실 제가 궁금해서.. 해보는겁니다..

### Example 2

이 예제는 다른 패키지(`exam`)의 함수를 `main` 함수에서 호출 할 때 어떤 결과가 나오는지 예측해보는 예제입니다.

```
// 파일 구조
// init_func
// exam
//   exam.go
//   exam2.go
// init.go
```

위와 같은 프로젝트 구조를 가진 예제입니다.

```go
// exam/exam.go
package exam

import "fmt"

var exam = variableInit()

func init() {
    fmt.Println("called init in 'exam/exam.go'")
}

func Exam() {
    fmt.Println("called first_exam in 'exam/exam.go'")
}

func variableInit() int {
    fmt.Println("called variableInit in 'exam/exam.go'")
    return 0
}
```

```go
// exam/exam2.go
package exam

import "fmt"

func init() {
    fmt.Println("called init in 'exam/exam2.go'")
}
```

```go
// main.go
package main

import (
    "fmt"

    "github.com/tkddlf59/init_func/exam"
)

var v = variableInit()

func main() {
    fmt.Println("called main")
    exam.Exam()
}

func init() {
    fmt.Println("called init")
}

func variableInit() int {
    fmt.Println("called variableInit")
    return 0
}
```

**결과값**
```
called variableInit in 'exam/exam.go'
called init in 'exam/exam.go'
called init in 'exam/exam2.go'
called variableInit
called init
called main
called first_exam in 'exam/exam.go'
```

저는 사실 다른 결과 값을 생각했었는데.. 생각해보니 제가 잘못 이해했던거 같습니다. 하하하... 이러면서 배우는 거져..
위의 결과값을 보면 패키지부터 초기화 한 후 `main` 패키지가 초기화 되는 걸 확인 할 수 있습니다.

## Conclusion

위의 `2.2 Example`의 소스를 보면 약간 의문을 가질 수 있습니다. 같은 패키지 안에 두 개의 `init` 함수가 두번 선언된 것에 의문을 품을 수 있지만 처음에 설명 드렸듯이 `init` 함수는 각각의 파일마다 선언할 수 있습니다. 따라서 `init` 함수는 위에서 설명 드렸듯이 _일반적인 초기화식으로 초기화 할 수 없는 변수를 초기화 하거나, 일회성 계산, 프로그램 시작시 패키지의 상태나 프로그램의 상태를 체크할 때_ 사용 할 수 있습니다.

이제 `init`이 어떤 순서로 호출되고 이걸 가지고 어떻게 응용할 수 있을지 뭔가 느낌이 오지 않습니까??
그 느낌이 온다면.. 이 포스트는 200% 성공했습니다.

사실 그냥 개발할 수도 있지만 이런 특수 함수들을 잘 활용해서 개발하시면 좀 더 편하게 개발할 수 있을거 같습니다.
위의 예제는 [golang-init-func(repository)](https://github.com/realsangil/golang-init-func)에서 확인하실 수 있습니다.

위의 내용에서 궁금하신 점이나 질문 혹은 잘못된 부분이 있으시면 댓글이나 메일을 보내주세요.    
 
감사합니다.
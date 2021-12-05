---
title: "Concurrency와 Parallelism의 차이"
tags: ["golang", "concurrency", "parallelism"]
date: 2020-02-06T09:30:00+09:00
---

최근에 golang을 처음부터 다시 공부하면서 헷갈렸던 Concurrency와 Parallelism의 차이점과 관계에 대해 개인적으로 공부한 내용을 간단한 예시와 함께 정리한 글입니다.

<!-- more -->

> "Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once."  
> \- Rob Pike, Concurrency is not Parallelism

Go언어 주요 개발자 중 한명인 '롭 파이크'는 말을 직역하면 '동시성은 한번에 여러가지 일을 다루는 것이다. 병렬성은 한번에 여러가지 일을 하는 것이다.'  

## 동시성(Concurrency)

| ![concurrency](/images/2020-02-06-concurrency-parallelism/concurrency.jpeg) |
| :--: |
| Concurrency |

동시성은 앞에서 언급했듯이 많은 일을 한번에 다루는 걸 의미합니다.  
  
이해를 돕기 위해 간단한 예시를 준비해봤습니다.
  
철수가 줄넘기 30회, 푸쉬업 15회, 스쿼트 15회씩 세 가지 운동을 한다고 가정해 봅시다.
물론 세 가지 운동들을 차례대로 수행할 수 있겠지만 동시성을 이해하기 위해 3세트씩 나눠서 진행해 보겠습니다.
  
1세트는 줄넘기 10회, 푸쉬업 5회, 스쿼트 5회 순으로 진행 했습니다.   
2세트는 푸쉬업 5회, 스쿼트 5회, 줄넘기 10회 순으로 진행 했습니다.  
3세트는 스쿼트 5회, 푸쉬업 5회, 줄넘기 10회 순으로 진행 했습니다.  

동시성은 ~~작업들을 동시에 실행~~하는 것이 아니라 위의 예시와 그림같이 작업들을 쪼개서 번갈아 가며 수행하는 것을 의미합니다.  
컴퓨터에서는 굉장히 빠른 속도로 처리하기 때문에 동시에 처리되는 것처럼 느껴지지만 <mark>시간의 관점으로는 동시가 아닙니다.</mark>  
이러한 관점으로 본다면 싱글코어 컴퓨터가 어떻게 멀티태스킹이 가능한지 알 수 있습니다.  
  
현실세계에서 병렬적으로 일어나는 일들을 컴퓨터 세계에서 구현하기 위한 방법 중 하나가 '동시성'입니다.

## 병렬성(Parallelism)

| ![parallelism](/images/2020-02-06-concurrency-parallelism/parallelism.jpeg) |
| :--: |
| Parallelism |

병렬성은 동시에 많은 일을 실행하는 것을 의미합니다.  
<mark>시간의 관점으로도 완벽한 동시가 맞습니다.</mark>  
병렬성은 싱글 코어에서는 불가능합니다. 멀티 코어이거나 싱글 코어 컴퓨터가 둘 이상 존재할 때 가능합니다.  
예를 들어 글 작성 기능 중 이미지 업로드 기능을 비동기로 구현한다고 가정해봅시다. 이때 리사이징이나 화질 저하 같은 작업들이 서버에서 처리한다면 병렬성이라고 할 수 있습니다. 

## Conclusion

이제는 롭 파이크가 동시성과 병렬성에 차이에 대해 말한 'Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.'가 이해가 됩니다.

시간의 관점을 보면 둘은 명확하게 다르지만 여러 작업을 다룬다는 관점에서는 같습니다.  
둘의 차이에 대해 공부하면서 저는 Go언어에서 goroutine을 많이 사용했지만 병렬성과 동시성에 대해서 너무 막연한 생각을 가지고 있었던 것 같습니다.  

궁금하신 점이나 잘못된 정보들은 메일이나 댓글을 남겨주세요. 
감사합니다.🙏

## Reference
 - [https://blog.golang.org/concurrency-is-not-parallelism](http://homoefficio.github.io/2019/02/02/Back-to-the-Essence-Concurrency-vs-Parallelism/?fbclid=IwAR0FCMcnSK4RbdnSy8TAuAO6q3eJbgJiCawD9zYQk_9RXJIg4ogFlesCmNg)
 - [http://homoefficio.github.io/2019/02/02/Back-to-the-Essence-Concurrency-vs-Parallelism/?fbclid=IwAR0FCMcnSK4RbdnSy8TAuAO6q3eJbgJiCawD9zYQk_9RXJIg4ogFlesCmNg](http://homoefficio.github.io/2019/02/02/Back-to-the-Essence-Concurrency-vs-Parallelism/?fbclid=IwAR0FCMcnSK4RbdnSy8TAuAO6q3eJbgJiCawD9zYQk_9RXJIg4ogFlesCmNg)
 - [http://tutorials.jenkov.com/java-concurrency/concurrency-vs-parallelism.html](http://tutorials.jenkov.com/java-concurrency/concurrency-vs-parallelism.html)
 - [https://medium.com/@tilaklodha/concurrency-and-parallelism-in-golang-5333e9a4ba64](https://medium.com/@tilaklodha/concurrency-and-parallelism-in-golang-5333e9a4ba64)
 - [https://wiki.haskell.org/Parallelism_vs._Concurrency](https://wiki.haskell.org/Parallelism_vs._Concurrency)
 - [https://stackoverflow.com/a/1050257](https://stackoverflow.com/a/1050257)
 - [https://light-tree.tistory.com/25](https://light-tree.tistory.com/25)
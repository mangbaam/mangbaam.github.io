---
layout: post
title: 
subtitle: 
categories: 
tags: []
---

## ⭐

> 본 글은 [코루틴 공식 가이드 읽고 분석하기 - Part1](https://myungpyo.medium.com/reading-coroutine-official-guide-thoroughly-part-1-98f6e792bd5b)을 읽고 공부한 내용을 정리하는 글입니다.

## 코루틴은 호출 스레드를 블록하지 않는다

---

```kotlin
package com.smp.coroutinesample.basic

import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch


fun main(args: Array<String>) {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    Thread.sleep(2000L)
}
```

위 예제의 마지막 라인에는 2초간 정지하는 코드가 있다.

`GlobalScope.launch {}` **코루틴 빌더**로 생성된 코루틴은 자신을 호출한 스레드를 블록하지 않기 때문에 위 예제에서 마지막 라인이 없다면 *World!* 를 출력하지 못하고 프로그램이 종료될 것이다.

위 예제에서 마지막 라인에 스레드를 중단하는 함수를 `중단 함수(Blocking function)`이라고 하는데 좀 더 명시적으로 나타내기 위해 다음 예제와 같이 `runBlocking {}` 블록을 사용할 수도 있다.

```kotlin
fun main(args: Array<String>) {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    runBlocking {
        delay(2000L)
    }
}
```

`runBlocking {}` 블록도 **코루틴 빌더**인데 블록이 완료될 때까지 현재 스레드를 멈추는 코루틴을 생성한다.

> 하지만 코루틴 안에서 `runBlocking {}` 사용은 권장되지 않는다

```kotlin
fun main(args: Array<String>) = runBlocking {
    GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    delay(2000L)
}
```

위 예제와 같이 main 함수 자체를 `runBlocking {}`으로 만들면 `delay()` 중단 함수를 더 자연스럽게 작성할 수 있다.

> `delay()`는 중단 함수이고, 모든 중단 함수는 코루틴 안에서만 호출될 수 있다.

## 코루틴과 타이밍 싸움 하지 말자

---

위 예제에서는 *World!* 를 출력하기 위해 2초라는 임의의 시간을 잠시 멈추었다. 하지만 실제 코딩할 때 이러한 방법으로 코딩하는 것은 ***굉장히*** 좋지 않은 방법이다. 부모 코루틴에서 자식 코루틴의 작업이 얼마나 걸릴지 예상하기 힘들 뿐더러 긴 시간을 중단하게 되면 사용성도 안 좋아 질테니까.

이런 문제를 해결하기 위해 `Job 인스턴스`를 이용할 수 있다. `GlobalScope.launch{}`는 `Job 인스턴스`를 반환한다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = GlobalScope.launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
    job.join()
}
```

위 예제의 마지막 라인에 `job.join()`이 있다. 여기서 `job`은 `GlobalScope.launch {}`가 반환한 `Job 인스턴스`이고, `join()`을 하면서 `job`이 종료될 때까지 대기한 후 `runBlocking {}`으로 되어있는 main() 함수가 종료될 수 있다.

만약 자식 코루틴이 여러 개 존재한다면 모든 자식 코루틴이 반환하는 `Job 인스턴스`를 가지고 있다가 부모 코루틴이 종료되는 시점에 모두 `join()`하여 자식 코루틴들의 종료를 기다려야 할 것이다. 이는 매우 번거로운 일이 될 수 있다. 이럴 때 필요한 것이 **코루틴 스코프(Scope)** 이다.

## 코루틴 스코프(Scope)

---

모든 코루틴은 각자의 스코프를 갖는다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    launch {
        delay(1000L)
        println("World!")
    }
    println("Hello,")
}
```

위 예제와 같이 `launch {}` **코루틴 빌더** 를 사용해서 새로운 코루틴을 생성하면 부모 코루틴에 `join()`을 명시적으로 호출할 필요 없이 모든 자식 코루틴들이 실행되고 종료될 때까지 대기할 수 있다.

## runBlocking vs coroutineScope

---

만약 어떤 코루틴을 위한 사용자 정의 스코프가 필요하다면 `coroutineScope {}` 빌더를 이용할 수 있다. 이 빌더로 생성된 코루틴은 모든 자식 코루틴이 끝날 때까지 종료되지 않는 스코프를 정의하는 코루틴이다.

앞서 살펴본 `runBlocking {}` 빌더와 `coroutineScope {}`의 차이는 `runBlocking {}`과 달리 `coroutineScope {}`는 자식들의 종료를 기다리는 동안 현재 스레드를 블록하지 않는다는 점이다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    launch {
        delay(200L)
        println("Task from runBlocking")
    }

    coroutineScope {
        launch {
            delay(500L)
            println("Task from nested launch")
        }
        delay(100L)
        println("Task from coroutine scope")
    }
    println("Coroutine scope is over")
}
```

```text
Task from coroutine scope
Task from runBlocking
Task from nested launch
Coroutine scope is over
```

## 중단 함수 추출

---

```kotlin
fun main(args: Array<String>) = runBlocking {
    launch {
        doWorld()
    }
    println("Hello,")
}

suspend fun doWorld() {
    delay(1000L)
    println("World!")
}
```

코루틴에서 함수로 추출하는 것은 비슷하지만 추출된 함수에 `suspend`라는 키워드가 붙게 된다. 이 `suspend`라는 키워드가 붙으면 중단 함수임을 뜻하게 된다.

중단 함수이기 때문에 코루틴 컨텍스트에서 동작하게 되고, 앞서 살펴본 `delay()`와 같은 함수도 사용할 수 있게 된다.

## 코루틴은 가볍다 (lightweight)

---

```kotlin
fun main(args: Array<String>) = runBlocking {
    repeat(100_000) {
        launch {
            delay(1000L)
            print(".")
        }
    }
}
```

위 코드는 10만 개의 코루틴을 만들어서 1초 후 `.`을 찍는 코드이다. 만약 이 동작을 코루틴이 아닌 스레드에서 동작했다면 많은 메모리를 사용하고 메모리 부족 예외를 발생시킬 수도 있다.

```kotlin
fun main(args: Array<String>) = run {
    repeat(100_000) {
        thread {
            Thread.sleep(1000L)
            print(".")
        }
    }
}
```

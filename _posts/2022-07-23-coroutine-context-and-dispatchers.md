---
layout: post
title: 
subtitle: 
categories: Kotlin
tags: [coroutine,dispatcher,context]
---

## ⭐

코루틴은 항상 CoroutineContext 내에서 수행된다. 그리고 코루틴 컨텍스트는 다양한 요소(Elements)들로 구성된다.

## 디스패처와 스레드

---

코루틴 컨텍스트는 연관된 코루틴들이 실행될 때 사용할 스레드나 스레드 풀을 결정하는 코루틴 디스패처를 포함한다. 특정 스레드에서 동작하게 하거나, 특정 스레드 풀로 전달하거나, 스레드 제한 없이 실행되도록 할 수도 있다.

모든 코루틴 빌더를 사용할 때 파라미터로 CoroutineContext를 넘겨줄 수 있고, 이를 이용해 새로운 코루틴들을 위한 디스패쳐나 그 외의 컨텍스트 요소를 지정할 수 있다.

```kotlin
fun launch1(): Unit = runBlocking {
    launch {
        println("main runBlocking - thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Unconfined) {
        println("Unconfined - thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) {
        println("Default - thread ${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("NewThread")) {
        println("newSingleThreadContext - thread ${Thread.currentThread().name}")
    }
}
```

![result1](https://user-images.githubusercontent.com/44221447/180615270-9c196944-84e3-4fd1-b9de-0d2647c2c0cf.png)

첫 번째 launch 에서는 파라미터를 지정하지 않았다. 이처럼 파라미터로 디스패처를 지정하지 않은 경우 실행된 코루틴 스코프로부터 상속 받은 컨텍스트를 사용한다. 여기서는 runBlocking의 컨텍스트를 사용한다.

두 번재 launch 에서는 `Unconfined`를 넘겨줬다. 이는 아래서 살펴보고자 한다.

세 번재 launch 에서는 `Default`를 넘겨줬다. 이는 GlobalScope에서 실행될 경우 사용되며, 공통으로 사용되는 백그라운드 스레드 풀을 사용한다. 즉 `launch(Dispathcers.Defult) { ... }` 와 `GlobalScope.launch { ... }`는 동일한 디스패처를 사용한다.

네 번째 launch 에서는 `newSingleThreadContext`를 넘겨줬다. 이는 항상 새로운 스레드를 생성한다. 이처럼 전용 스레드를 생성하는 작업은 더 이상 사용되지 않을 때는 close() 함수로 해제하거나 Top-level 변수에 저장해 어플리케이션 전반에서 사용되어야 한다.

### Unconfined vs confined dispatcher

위 코드에서 두 번째에 `Unconfined`를 넘겨줬다. `Dispathcers.Unconfined`는 호출 스레드에서 코루틴을 시작하고, 처음으로 중단 지점을 만난 후 다시 재개될 때는 중단 함수를 재개한 스레드에서 수행된다.

Unconfined 디스패처는 코루틴이 CPU 시간을 소모하지 않거나 UI를 업데이트 하지 않는 등의 상황에서 특정 스레드에 국한된 작업이 아닌 경우에 적절하다.

```kotlin
fun main(): Unit = runBlocking {
    launch(Dispatchers.Unconfined) {
        println("Unconfined - thread ${Thread.currentThread().name}")
        delay(500)
        println("Unconfined- thread ${Thread.currentThread().name}")
    }
    launch {
        println("main runBlocking - thread ${Thread.currentThread().name}")
        delay(1000)
        println("main runBlocking After delay - thread ${Thread.currentThread().name}")
    }
}
```

![result2](https://user-images.githubusercontent.com/44221447/180615558-ee2a07aa-5d0d-4c6e-b17a-26d2d23e2b5d.png)

runBlocking { ... } 컨텍스트를 상속한 코루틴은 메인 스레드에서 수행되지만 unconfined 코루틴은 delay 함수가 사용되는 `DefaultExecutor` 스레드에서 실행되는 것을 확인할 수 있다.

> 여기서 Unconfined에 대해 소개하기는 했지만 잘못 사용하는 경우 예상치 못한 동작을 할 수 있기 때문에 특수한 경우를 제외하고는 사용을 지양해야 한다.

## 코루틴, 스레드 디버깅

---

코루틴이 언제 어디서 수행 중이었는지를 파악하기는 쉽지 않다. 일반적으로 로그를 사용해서 확인하게 되는데 위 예제들처럼 스레드 이름만을 사용하면 구분하기 쉽지 않을 수 있다.

```kotlin
import kotlinx.coroutines.async
import kotlinx.coroutines.runBlocking

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun main() = runBlocking {
    val a = async {
        log("async A return 6")
        6
    }
    val b = async {
        log("async B return 7")
        7
    }
    log("The answer is ${a.await() * b.await()}")
}
```

![debug result1](https://user-images.githubusercontent.com/44221447/180616165-ec810c6e-d722-4042-a5ac-c7d084fb6de9.png)

현재는 위와 같이 main 이라는 스레드 이름만이 나오게 된다.

다음 방법으로 로그에서 코루틴의 이름까지 출력할 수 있다. (출처 : [스택오버플로우](https://stackoverflow.com/questions/52049150/dkotlinx-coroutines-debug-not-working-in-intellij-idea-jvm-options))

*한 번 실행 시켜야 설정할 수 있다.*

1. Run - Edit Configurations

![setting1](https://user-images.githubusercontent.com/44221447/180616417-e7815811-7c5f-49d6-8635-83df322d033c.png)

2. VM options

![configurations](https://user-images.githubusercontent.com/44221447/180616326-839b8264-a6f5-4f5a-83be-98cb660e50c6.png)

> -Dkotlinx.coroutines.debug=on

3. Result

![debug result 2](https://user-images.githubusercontent.com/44221447/180616471-b1fd8328-f59f-4b15-9e84-d0888e071515.png)

위와 동일한 코드를 실행해보면 main 만 나오는 것이 아니라 코루틴 정보까지 나와서 동일한 코루틴에서 동작하고 있는지 아닌지를 확인할 수 있다.

좀 더 많은 정보는 [여기](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/new-coroutine-context.html)서 확인할 수 있다.

## 스레드 전환

---

```kotlin
import kotlinx.coroutines.newSingleThreadContext
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.withContext

fun main() = runBlocking {
    newSingleThreadContext("Ctx1").use { ctx1 ->
        newSingleThreadContext("Ctx2").use { ctx2 ->
            runBlocking(ctx1) {
                log("Started in ctx1")
                withContext(ctx2) {
                    log("Working in ctx2")
                }
                log("Back to ctx1")
            }
        }
    }
}
```

![trans result1](https://user-images.githubusercontent.com/44221447/180616672-f3ff3638-829a-45ad-b77e-3dee68b00e60.png)

`newSingleThreadContext`는 새로운 스레드를 생성한다고 설명했고, 반드시 해제해주어야 한다고 했다. 코틀린의 `use` 문을 사용하면 사용 후 자동으로 해제해주기 때문에 이 상황에서 유용하게 사용된다.

코드와 결과를 보면 "Started in ctx1" 부분은 두 번째 생성된 코루틴이라서 **@coroutine#2** 에서 실행되지만 runBlocking의 인자로 들어온 ctx1 컨텍스트에서 실행된다.

하지만 동일한 코루틴에 있는 "Working in ctx2" 부분은 withContext()로 인해서 ctx2 컨텍스트에서 실행된다. 그리고 코루틴을 보면 기존 코루틴(@coroutine#2)을 유지하고 있다.

## context의 Job 확인

---

코루틴의 Job은 그 컨텍스트의 일부이다. `coroutineContext[Job]`으로 Job을 확인할 수 있다.

```kotlin
import kotlinx.coroutines.Job
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    println("My job is ${coroutineContext[Job]}")
}
```

![job result](https://user-images.githubusercontent.com/44221447/180616989-ebc5220d-41fe-42d8-8965-b965f14081f1.png)

## 코루틴의 자식

---

어떤 코루틴(부모 코루틴) 스코프 안에서 다른 코루틴(자식 코루틴)을 생성하고 실행하면 부모 코루틴의 CoroutineScope.coroutinecontext 를 통해 컨텍스트를 상속하고, 새롭게 생성된 자식 코루틴의 Job은 부모 코루틴 Job의 자식으로 생성된다.

만약 부모 코루틴이 취소되면 모든 자식 코루틴 역시 재귀적으로 취소된다.

예외적으로 GlobalScope에서 실행된 코루틴은 어디서 실행되건 독립적으로 동작한다.

```kotlin
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val request = launch {
        GlobalScope.launch {
            println("job1: I run in GlobalScope and execute independently!")
            delay(1000)
            println("job1: I am not affected by cancellation of the request")
        }
        launch {
            delay(100)
            println("job2: I am a child of the request coroutine")
            delay(1000)
            println("job2: I will not execute this line if my parent request is cancelled")
        }
    }
    delay(500)
    request.cancel()
    delay(1000)
    println("main: Who has survived request cancellation?")
}
```

![global result](https://user-images.githubusercontent.com/44221447/180617184-47ae14b8-b4f3-4532-b09e-6debe24c41df.png)

request가 취소된 후에도 GlobalScope에서 실행된 코루틴은 살아있는 것을 확인할 수 있다.

## 부모 코루틴의 의무

---

부모 코루틴은 모든 자식 코루틴이 실행 완료 될 때까지 기다린다. 이때 `Job.join()` 함수를 사용할 필요가 없다.

```kotlin
fun main() = runBlocking {

    val request = launch {
        repeat(3) { i -> // launch a few children jobs
            launch  {
                delay((i + 1) * 200L) // variable delay 200ms, 400ms, 600ms
                println("Coroutine $i is done")
            }
        }
        println("request: I'm done and I don't explicitly join my children that are still active")
    }
}
```

![wait result](https://user-images.githubusercontent.com/44221447/180617380-1724a5d1-09de-403c-b1c0-c128e47be06b.png)

request는 3개의 자식 코루틴을 만들고 launch를 하지 않았음에도 모든 자식 코루틴이 실행된 후에 종료되었다.

## 디버깅을 위해 코루틴 이름 지어주기

---

디버깅 할 때 기본적으로 자동 할당된 id를 확인할 수 있는데 어떤 코루틴인지 확실하게 알기 위해서 이름을 지정하면 좋다.

CoroutineName 이라는 컨텍스트 요소를 통해 코루틴 이름을 지어줄 수 있다. 디버깅 모드일 때 CoroutineName이 수행 중인 스레드 이름과 함께 코루틴 이름도 나타난다.

```kotlin
import kotlinx.coroutines.CoroutineName
import kotlinx.coroutines.async
import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    log("Started main coroutine")
    val v1 = async(CoroutineName("v1coroutine")) {
        delay(500)
        log("Computing v1")
        252
    }
    val v2 = async(CoroutineName("v2coroutine")) {
        delay(1000)
        log("Computing v2")
        6
    }
    log("The answer for v1 / v2 = ${v1.await() / v2.await()}")
}
```

![naming result](https://user-images.githubusercontent.com/44221447/180617523-ace76166-c8a1-49e8-b58e-90fe343fb453.png)

## 컨텍스트 요소 병합

---

코루틴 컨텍스트 요소를 두개 이상 동시에 지정해야 하는 경우 +(plus) 연산자를 통해 병합할 수 있다. 예를 들어 디스패처를 지정하면서 코루틴 이름을 지정할 수 있다.

```kotlin
import kotlinx.coroutines.CoroutineName
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

fun main() = runBlocking<Unit> {

    launch(Dispatchers.Default + CoroutineName("test")) {
        println("I'm working in thread ${Thread.currentThread().name}")
    }
}
```

![combine result](https://user-images.githubusercontent.com/44221447/180617629-9b6fae1a-4bd4-4694-b8d4-30e1be3581fa.png)

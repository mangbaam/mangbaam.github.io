---
layout: post
title: Kotlin Coroutine Cancellation
subtitle: 코루틴의 취소
categories: Kotlin
tags: [kotlin,coroutine]
---

## 모든 중단 함수는 취소 요청에 응답해야 한다

---

코루틴에서 실행되는 모든 중단 함수는 취소 요청에 응답하도록 구현되어야 한다. 중단 함수의 실행 중 취소 가능한 구간마다 취소 요청이 있었는지 확인하고, 취소 요청이 있었다면 즉시 실행을 취소하도록 구현되어야 한다. 취소가 가능한 지점마다 현재 코루틴이 취소 여부를 확인하다가 만약 취소되었다면 `CancellationException` 을 발생시키며 종료한다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispathcers.Default) {
        for (i in 1..10) {
            println("$i")
            Thread.sleep(500L)
        }
    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}
```

```text
1
2
3
취소 요청
4
5
6
7
8
9
10
종료
```

위 코드에서는 취소되지 않고 10까지 출력하게 된다. 이 코드를 취소 요청에 친화적인 코드로 만들기 위해서는 취소 가능한 지점마다 Continuation에 실행 시간을 양보하는 `yield()` 함수를 호출하거나 CoroutineScope 에 정의된 `isActive` 속성을 사용해 코루틴이 비활성 상태인 경우 작업을 중단하도록 작성할 수 있다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispathcers.Default) {
        for (i in 1..10) {
            yield() // <--
            println("$i")
            Thread.sleep(500L)
        }
    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}
```

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispathcers.Default) {
        for (i in 1..10) {
            if (!isActive) { // <--
                break
            }
            println("$i")
            Thread.sleep(500L)
        }
    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}
```

```text
1
2
3
취소 요청
종료
```

두 개의 코드 모두 위와 같이 취소 요청 후 바로 취소할 수 있게 된다.

취소가 되면 **CancellationException**이 발생하는데 이를 처리하기 위해 `try ~ finally` 구문이나 Kotlin의 `use()` 함수를 사용할 수 있다.

*try ~ finally*

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispatchers.Default) {
        try {
            for (i in 1..10) {
                println("$i")
                delay(500L)
            }
        } finally {
            println("실행 완료")
        }

    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}
```

*use()*

```text
1
2
3
취소 요청
실행 완료
종료
```

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispatchers.Default) {
        CancellationTest().use {
            it.repeatedPrint(1000)
        }
    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}

class CancellationTest : Closeable {

    suspend fun repeatedPrint(times: Int) {
        repeat(times) { i ->
            println("$i")
            delay(500L)
        }
    }

    override fun close() {
        println("CancellationTest - close()")
    }
}
```

```text
0
1
2
취소 요청
CancellationTest - close()
종료
```

## 취소된 코루틴에서 중단 함수 호출

---

취소된 코루틴의 finally 블록 안에서 중단 함수를 호출하면 **CancellableException**이 발생한다.

보통 리소스를 정리하는 함수들은 Non-Blocking으로 동작하기 때문에 큰 문제가 되지는 않지만 만약 취소된 코루틴 안에서 동기적으로 어떤 중단 함수를 호출해야 하는 경우에는 `withContext { }` 코루틴 빌더에 `NonCancellable` 컨텍스트를 전달해서 처리할 수 있다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = launch(Dispatchers.Default) {
        try {
            repeat(1000) { i ->
                println("$i")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                delay(1000L)
                println("실행 완료")
            }
        }

    }

    delay(1300L)
    println("취소 요청")
    job.cancelAndJoin()
    println("종료")
}
```

```text
0
1
2
취소 요청
실행 완료
종료
```

## 타임 아웃

---

코루틴 실행을 취소하는 많은 이유는 수행 시간이 허용할 수 있는 시간보다 길어진 경우이다. 이 경우 코루틴에 Timeout을 지정하고, 이 시간을 넘어서면 해당 작업을 취소하도록 구현할 수 있다.

```kotlin
fun main(args: Array<String>): Unit = runBlocking {
    val job = launch(Dispatchers.Default) {
        try {
            repeat(1000) { i ->
                println("$i")
                delay(500L)
            }
        } finally {
            println("실행 완료")
        }
    }

    launch {
        delay(1300L)
        println("취소 요청")
        if (job.isActive) {
            job.cancelAndJoin()
        }
    }
}
```

```text
0
1
2
취소 요청
실행 완료
```

위와 같이 코드를 짜면 예상한 대로 동작하겠지만 별도의 코루틴에서 취소를 처리하는 작업을 매번 해줘야 하기 때문에 좋은 방법은 아니다. 이때 `withTimeout()` 함수를 사용할 수 있다.

```kotlin
fun main(args: Array<String>): Unit = runBlocking {
    withTimeout(1300L) {
        launch {
            try {
                repeat(1000) { i ->
                    println("$i")
                    delay(500L)
                }
            } finally {
                println("실행 완료")
            }
        }
    }
}
```

![timeout](https://user-images.githubusercontent.com/44221447/179161265-5ef9b688-1654-4fc5-9cf6-0c76718276ef.png)

TimeoutCancellationException이 발생했는데 이는 메인 함수에서 바로 실행됐기 때문이다.

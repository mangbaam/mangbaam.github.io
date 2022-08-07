---
layout: post
title: 비동기 플로우에 대해서 알아보자
subtitle: Asynchronous Flow
categories: Kotlin
tags: [kotlin,coroutine,flow]
---

## ⭐

중단 함수를 사용할 때는 하나의 값만 반환한다. 만약 여러 개의 값을 비동기적으로 반환하고 싶다면 우리는 Flow 를 사용할 수 있다.

## 여러 값을 표현하기

---

여러 개의 값은 collections 를 사용해서 표현될 수 있다. 예를 들어 3 개의 숫자를 담고 있는 List 를 반환하는 `simple` 이라는 함수로 리스트를 받아와서 `forEach` 를 사용해 출력해 볼 수 있다.

```kotlin
fun simple(): List<Int> = listOf(1, 2, 3)

fun main() {
    simple().forEach { println(it) }
}
```

```text
1
2
3
```

### 시퀀스

각각 100ms 동안 CPU 를 사용해 계산해야 하는 경우 Sequence 를 사용해서 숫자들을 표현할 수 있다.

```kotlin
fun simpleSequence(): Sequence<Int> = sequence {
    repeat(3) { i ->
        Thread.sleep(100)
        yield(i + 1)
    }
}

fun main() {
    simpleSequence().forEach { println(it) }
}
```

```text
1
2
3
```

이 코드는 위에서 List 를 사용한 것과 같이 3 개의 숫자를 출력하지만 각각 100 ms 만큼을 기다린다는 차이가 있다.

### 중단 함수

위의 Sequnce 를 사용했던 코드에서는 메인 스레드를 차단해버린다. 이러한 값들이 비동기적으로 계산되어야 한다면 `suspend` 키워드로 메인 스레드를 차단하지 않고 값을 리스트로 반환할 수 있다

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.runBlocking

suspend fun simpleSuspend(): List<Int> {
    delay(1000)
    return listOf(1, 2, 3)
}

fun main() = runBlocking {
    simpleSuspend().forEach { println(it) }
}
```

```text
1
2
3
```

위 코드는 1초 뒤 출력된다.

### Flows

`List<Int>` 를 리턴 타입으로 사용한다는 것은 모든 값을 오직 한 번만 반환하겠다는 뜻이다. 비동기 적으로 계산되는 값들을 stream 으로 표현하기 위해서는 위에서 살펴본 `Sequnece<Int>` 타입과 같이 `Flow<Int>` 타입을 사용할 수 있다.

```kotlin
fun simpleFlow(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking<Unit> {
    launch {
        for (i in 1..3) {
            println("$i 를 차단하지 않았다!!")
            delay(100)
        }
    }
    simpleFlow().collect { value -> println(value) }
}
```

```text
1 를 차단하지 않았다!!
1
2 를 차단하지 않았다!!
2
3 를 차단하지 않았다!!
3
```

이 코드는 메인 스레드를 차단하지 않으면서 각 숫자를 출력할 때 100ms 를 기다렸다가 출력한다. simpleFlow() 의 Flow 가 실행되는 동안 main 함수의 "n 를 차단하지 않았다!!" 가 출력되면서 메인 함수가 차단되지 않았음을 알 수 있다

위 코드에서 알 수 있는 사실은

- Flow 함수는 flow 를 통해서 만들어진다
- `flow { ... }` 빌더 블록 내부에서는 중단될 수 있다
- `suspend` 키워드 없이 가능하다
- `emit` 함수를 통해 값을 방출할 수 있다
- `collect` 함수를 통해 값을 수집할 수 있다

## Flow 는 Cold 로 동작한다

---

Flow 는 시퀀스와 비슷하게 cold 로 동작한다. cold 로 동작한다는 것은 flow 가 수집될 때까지는 flow 빌더 내부가 동작하지 않는다는 뜻이다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking

fun coldFlow(): Flow<Int> = flow {
    println("Flow 시작")
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    println("coldFlow() 함수 호출")
    val flow = coldFlow()
    println("수집 시작")
    flow.collect { value -> println(value) }
    println("다시 수집 시작")
    flow.collect { value -> println(value)}

}
```

```text
coldFlow() 함수 호출
수집 시작
Flow 시작
1
2
3
다시 수집 시작
Flow 시작
1
2
3
```

바로 이러한 점이 `suspend` 키워드를 붙이지 않아도 되는 핵심적인 이유이다. `coldFlow` 함수는 빠르게 반환되어 버리고 무엇도 기다리지 않는다.
플로우는 수집될 때마다 시작하기 때문에 우리가 `collect` 를 사용할 때마다 "Flow 시작" 이라는 문구를 볼 수 있었던 이유이다.

## Flow 취소

---

플로우는 코루틴의 일반적인 협력적 취소 매커니즘을 준수한다. 플로우 수집은 플로우가 delay 같은 취소 가능한 중단 함수에서 중단될 수 있다.
다음 예제는 플로우가 `withTimeoutOrNull` 블록을 실행할 때 어떻게 취소가 되는지 보여준다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.withTimeoutOrNull

fun flowCancel(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        println("Emit $i")
        emit(i)
    }
}

fun main() = runBlocking {
    withTimeoutOrNull(250) {
        flowCancel().collect { value -> println(value) }
    }
    println("실행 완료")
}
```

```text
Emit 1
1
Emit 2
2
실행 완료
```

## Flow 빌더

---

`flow { ... }` 빌더가 가장 기본적인 빌더 중 하나다. 플로우를 쉽게 만들 수 있는 다른 빌더들도 있다.

- flowOf : 고정된 값들의 집합을 방출하는 flow를 만든다
- `.asFlow()` : 다양한 콜렉션들과 시퀀스는 `.asFlow()` 확장 함수를 통해서 플로우 변경될 수 있다.

```kotlin
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    (1..3).asFlow().collect { value -> println(value) }
}
```

## Flow 중간 연산자

---

플로우는 연산자로 변환될 수 있다. 중간 연산자는 업스트림 플로우에 적용되어 다운스트림 플로우를 반환한다. 이 연산자들은 여느 flow 처럼 cold 로 동작한다. 연산자로 호출하는 것은 자체적으로는 중단 함수가 아니며, 새로운 변환된 플로우를 즉시 반환한다.

기본 연산자는 map 이나 filter 같은 익숙한 이름도 있다. 시퀀스와의 중요한 차이점은 이 연산자 안에 있는 코드 블록에서 중단 함수를 호출할 수 있다는 점이다.

예를 들어, 요청된 플로우에 대해 map 연산자를 이용해 원하는 결과 값으로 매핑할 수 있으며, 요청 작업이 긴 시간을 소모하는 중단 함수인 경우에도 성공적으로 동작한다

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.runBlocking

suspend fun performRequest(request: Int): String {
    delay(1000)
    return "return $request"
}

fun main() = runBlocking {
    (1..3).asFlow()
        .map { request -> performRequest(request) }
        .collect { response -> println(response) }
}
```

```text
return 1
return 2
return 3
```

### 변환 연산자

플로우 변환 연산자 중에서 가장 일반적인 것이 `transform` 이다. 이 연산자는 `map` 이나 `filter` 같은 단순한 변환이나 혹은 복잡한 다른 변환들을 구현하기 위해 사용된다. `transform` 연산자를 사용해서 우리는 임의의 시간에 임의의 값을 방출할 수 있다.

예를 들어, `tranform` 을 사용해 오래 걸리는 비동기 요청을 수행하기 전에 기본 문자열을 방출하고 응답이 도착하면 그 결과를 방출할 수 있다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.transform
import kotlinx.coroutines.runBlocking

suspend fun performRequest(request: Int): String {
    delay(1000)
    return "return $request"
}

fun main() = runBlocking {
    (1..3).asFlow()
        .transform { request ->
            emit("$request 요청 보내기")
            emit(performRequest(request))
        }
        .collect { response -> println(response) }
}
```

```text
1 요청 보내기
return 1
2 요청 보내기
return 2
3 요청 보내기
return 3
```

### 크기 제한 연산자

`take` 같은 크기 제한 중간 연산자는 정의된 임계치에 도달하면 실행을 취소한다. 코루틴 취소는 예외를 발생시키는 방식으로 수행되며, 그렇기 때문에 `try ~ finally` 같은 자원 관리형 함수들이 정상적으로 동작할 수 있게 한다.

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.take
import kotlinx.coroutines.runBlocking

fun numbers(): Flow<Int> = flow {
    try {
        emit(1)
        emit(2)
        println("여기 부터는 실행되지 않을 겁니다")
        emit(3)
    } finally {
        println("finally 에서 실행 됨")
    }
}

fun main() = runBlocking {
    numbers()
        .take(2)
        .collect { value -> println(value) }
}
```

```text
1
2
finally 에서 실행 됨
```

두 번째 수를 방출하고 멈춘 것을 볼 수 있다.

## Flow 종단(terminal) 연산자

---

플로우 종단 연산자는 플로우 수집을 시작하는 중단 함수이다. `collect` 연산자가 가장 대표적이고, 다음과 같은 다른 종단 연산자들도 있다.

- `toList` 나 `toSet` 같은 다양한 컬렉션으로의 변환
- `first` 로 첫 번째 값만 방출하거나 `single` 로 단일 값만 방출함을 보장
- 플로우를 `reduce` 나 `fold` 를 이용하여 값으로 변환

예를 들어

```kotlin
package flow

import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.reduce
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val sum = (1..5).asFlow()
        .map { it * it }
        .reduce { accumulator, value -> accumulator + value }

    println(sum)
}
```

위 코드는 **55** 를 반환한다.

## 플로우는 순차적이다

---

플로우의 독립된 각 컬렉션들은 다중 플로우가 사용되는 특별한 연산자가 사용되지 않은 이상 순차적으로 수행된다. 수집은 종단 연산자를 호출한 코루틴에서 직접 수행되며 기본적으로 새로운 코루틴을 생성하지는 않는다. 각각 방출된 값은 업스트림의 모든 중간 연산자들에 의해 처리되어 다운스트림으로 전달되며 마지막으로 종단 연산자로 전달된다.

다음은 짝수 만을 필터링해서 문자열로 변환하는 예제이다.

```kotlin
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.filter
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    (1..5).asFlow()
        .filter {
            println("Filter $it")
            it % 2 == 0
        }.map {
            println("Map $it")
            "문자열 $it"
        }.collect {
            println("Collect $it")
        }
}
```

```text
Filter 1
Filter 2
Map 2
Collect 문자열 2
Filter 3
Filter 4
Map 4
Collect 문자열 4
Filter 5
```

## Flow 컨텍스트

---

플로우 수집은 항상 호출한 코루틴의 컨텍스트 안에서 수행된다. 이러한 특성을 Context preservation (컨텍스트 보존) 이라고 한다.

컨텍스트가 보존되기 때문에 호출자를 블록하지 않고 실행 컨텍스트에 관계 없이 비동기 작업을 할 수 있는 것이다.

### withContext 사용 주의

보통 `withContext` 는 코루틴의 컨텍스트를 전환하기 위해 사용되는데 `flow { ... }` 빌더 내부의 코드는 컨텍스트 보존 특성을 지켜야하기 때문에 다른 컨텍스트에서 값을 방출하는 것이 허용되지 않는다.

```kotlin
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking
import kotlinx.coroutines.withContext

fun wrong(): Flow<Int> = flow {
    withContext(Dispatchers.Default) {
        for ( i in 1..3) {
            Thread.sleep(100)
            emit(i)
        }
    }
}

fun main() = runBlocking {
    wrong().collect { value -> println(value) }
}
```

```text
Exception in thread "main" java.lang.IllegalStateException: Flow invariant is violated:
		Flow was collected in [CoroutineId(1), "coroutine#1":BlockingCoroutine{Active}@14d02ae6, BlockingEventLoop@5bd0e297],
		but emission happened in [CoroutineId(1), "coroutine#1":DispatchedCoroutine{Active}@7f824d68, Dispatchers.Default].
		Please refer to 'flow' documentation or use 'flowOn' instead
 at kotlinx.coroutines.flow.internal.SafeCollector_commonKt.checkContext (SafeCollector.common.kt:85) 
 at kotlinx.coroutines.flow.internal.SafeCollector.checkContext (SafeCollector.kt:106) 
 at kotlinx.coroutines.flow.internal.SafeCollector.emit (SafeCollector.kt:83) 
 ```

 위 에러 메시지를 잘 보면 `flowOn` 을 사용해보라고 알려주고 있다.

### flowOn 연산자

 ```kotlin
 package flow

import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.flowOn
import kotlinx.coroutines.runBlocking

fun log(msg: String) = println("[${Thread.currentThread().name}] $msg")

fun flowOnTest(): Flow<Int> = flow {
    for ( i in 1..3) {
        Thread.sleep(100)
        log("Emitting $i")
        emit(i)
    }
}.flowOn(Dispatchers.Default)

fun main() = runBlocking { 
    flowOnTest().collect { value ->
        log("Collected $value")
    }
}
```

```text
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 1
[main @coroutine#1] Collected 1
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 2
[main @coroutine#1] Collected 2
[DefaultDispatcher-worker-1 @coroutine#2] Emitting 3
[main @coroutine#1] Collected 3
```

위에서 `withContext` 를 사용한 예제와 비슷한 로직이지만 `flowOn` 을 사용했을 때는 컨텍스트가 원하는대로 변전되어 사용할 수 있는 것을 확인할 수 있다.

그리고 하나 더 확인해야 할 부분은 코드를 실행하고 있는 코루틴이다. collect 를 하고 있는 main 은 coroutine#1 에서 실행되고 있지만 emit 하고 있는 flowOnTest 는 새로운 코루틴인 coroutine#2 가 생성되어 실행되고 있다.

이는 플로우의 기본적인 특성인 순차성을 일부 포기했다고 볼 수도 있다.

## 버퍼링

---

플로우 로직을 여러 코루틴에서 수행하는 것은 플로우 수집 시간 관저에서는 도움이 될 수 있다.

```kotlin
package flow

import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.buffer
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking
import kotlin.system.measureTimeMillis

fun longJob(): Flow<Int> = flow {
    for ( i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun withNoBuffer() = runBlocking {
    val time = measureTimeMillis {
        longJob()
            .collect { value ->
                delay(300)
                println(value)
        }
    }
    println("$time ms 에 수집 됨")
}

fun withBuffer() = runBlocking {
    val time = measureTimeMillis {
        longJob()
            .buffer()
            .collect { value ->
                delay(300)
                println(value)
        }
    }
    println("$time ms 에 수집 됨")
}

fun main() {
    println("버퍼 없이 실행")
    withNoBuffer()
    
    println("\n버퍼 추가 후 실행")
    withBuffer()
}
```

위 코드를 실행 시켰을 때 결과는 다음과 같다

```text
버퍼 없이 실행
1
2
3
1218 ms 에 수집 됨

버퍼 추가 후 실행
1
2
3
1037 ms 에 수집 됨
```

같은 로직에 buffer 를 추가하지 않았던 코드에서는 약 1200 ms 가 소요된 반면 buffer 를 추가한 코드에서는 약 1000 ms 가 소요되었다.

buffer 를 추가하면 새로운 코루틴을 만들어서 처리를 하고, 각 결과는 channel 을 통해 주고 받으며 최종적으로 시간을 줄여줄 수 있는 것이다.

위 코드에서는 첫 번째 수를 처리하기 위해 100ms 를 기다리고, 각각의 수를 처리하기 위해 300ms 씩 기다리면서 1000ms 만을 소요하게 되었다.

위에서 flowOn 에 대해서 설명했는데 flowOn 에서도 마찬가지로 새로운 코루틴이 만들어진다고 했는데, 컨텍스트가 전환 되는지 여부만 다를 뿐 flowOn 에서도 동일한 버퍼링 매커니즘을 사용한다.

### conflate

어떤 플로우가 방출하는 속도보다 그 플로우를 소모(수집)해서 처리하는 시간이 더 오래걸리는 경우 플로우가 방출하는 중간 내용들을 전부 처리하지 않고 최종 결과만을 취하고 싶을 때 `conflate` 연산자를 사용하면 된다.

```kotlin
package flow

import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.conflate
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking
import kotlin.system.measureTimeMillis

fun conflate(): Flow<Int> = flow { 
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking { 
    val time = measureTimeMillis { 
        conflate()
            .conflate()
            .collect {  value ->
                delay(300)
                println(value)
            }
    }
    println("수행 시간: $time")
}
```

```text
1
3
수행 시간: 767
```

위 예제에서 첫 번째 값(1)을 처리하는 동안 2, 3 이 방출되는데, 이때 `conflate` 를 사용하여 처리하고 있던 첫 번째 값을 처리한 후 방출된 값 중 가장 최신 값인 3을 처리하면서 1을 방출할 때까지 걸린 시간 100ms + 1과 3을 수집하여 처리하는 데까지 걸린 시간 300 x 2 = 약 700 ms 만큼의 시간이 소요된 것이다.

### 최신 값 처리

바로 위에서 살펴본 conflate 는 방출과 수집이 모두 느릴 경우 중간 값들을 삭제함으로서 수행하는데 그 대안으로는 `xxxLatest` 연산자가 있다.

`xxxLastest` 연산자는 새로운 값이 방출될 때마다 느린 수집기를 취소하고 다시 시작하도록 동작한다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.collectLatest
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking
import kotlin.system.measureTimeMillis

fun latest(): Flow<Int> = flow {
    for (i in 1..3) {
        delay(100)
        emit(i)
    }
}

fun main() = runBlocking {
    val time = measureTimeMillis {
        latest()
            .collectLatest {  value ->
                println("Collecting $value")
                delay(300)
                println(value)
            }
    }
    println("수행 시간: $time")
}
```

```text
Collecting 1
Collecting 2
Collecting 3
3
수행 시간: 688
```

위 예제를 보면 Collecting n 은 값을 수집할 때마다 출력했지만 수집한 값을 처리하는데 300 ms 가 소요되면서 앞선 값들은 모두 취소되고 마지막 값인 3 만 끝까지 처리되어 출력되는 것을 알 수 있다.

## 다중 플로우 합성

---

### Zip

```kotlin
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.flow.zip
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val nums = (1..3).asFlow()
    val strs = flowOf("하나", "둘", "셋")

    nums.zip(strs) { i, s -> "$i -> $s"}
        .collect { println(it) }
}
```

```text
1 -> 하나
2 -> 둘
3 -> 셋
```

### Combine

플로우가 어떤 변수나 연산의 가장 최신의 값을 표현할 때 해당 플로우의 가장 최근 값에 계산을 하고, 업스트림 플로우 증 하나가 새로운 값을 방출하면 다시 계산해야 할 수 있다. 이와 관련된 연산자들을 `combine` 이라고 부른다.

예를 들어 위 예제(zip)에서 숫자가 300ms 마다 업데이트되고, 문자열이 400ms 마다 업데이트 되면 zip 으로 압축했을 때 더 긴 연산인 400ms 마다 출력된다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.flowOf
import kotlinx.coroutines.flow.onEach
import kotlinx.coroutines.flow.zip
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("하나", "둘", "셋").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    nums.zip(strs) { a, b -> "$a -> $b" }
        .collect { value ->
            println("$value 수행 시간 : ${System.currentTimeMillis() - startTime} ms")
        }
}
```

```text
1 -> 하나 수행 시간 : 435 ms
2 -> 둘 수행 시간 : 835 ms
3 -> 셋 수행 시간 : 1237 ms
```

`zip` 대신 `combine` 연산자를 사용한 예제는 다음과 같다

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun main() = runBlocking {
    val nums = (1..3).asFlow().onEach { delay(300) }
    val strs = flowOf("하나", "둘", "셋").onEach { delay(400) }
    val startTime = System.currentTimeMillis()
    nums.combine(strs) { a, b -> "$a -> $b" }
        .collect { value ->
            println("$value 수행 시간 : ${System.currentTimeMillis() - startTime} ms")
        }
}
```

```text
1 -> 하나 수행 시간 : 439 ms
2 -> 하나 수행 시간 : 641 ms
2 -> 둘 수행 시간 : 841 ms
3 -> 둘 수행 시간 : 942 ms
3 -> 셋 수행 시간 : 1242 ms
```

`zip` 과는 다르게 출력되었는데, 결과를 보면 `nums` 나 `strs` 가 각각 방출하고 마지막에 병합된 결과가 출력된 것을 볼 수 있다.

(다음 게시글에서 계속...)

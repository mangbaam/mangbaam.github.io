---
layout: post
title: 비동기 플로우에 대해서 알아보자 (2)
subtitle: Asynchronous Flow
categories: Kotlin
tags: [kotlin,coroutine,flow]
---

## ⭐

[이전 게시글](https://mangbaam.github.io/kotlin/2022/07/30/kotlin-flow-basic-1)에 이어지는 내용이다

## Flow 플래트닝 (Flattening)

---

플로우는 비동기로 수신되는 값들의 시퀀스를 나타낸다. 그렇기 때문에 각 값이 다른 값 시퀀스를 요청하는 것이 쉬워진다.

예를 들어 다음과 같이 500ms 간격으로 두 문자열을 방출하는 플로우를 만들 수 있다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.flow.map

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i : Second")
}

fun main() {
    (1..3).asFlow().map { requestFlow(it) }
}
```

1, 2, 3 을 차례로 방출하는 플로우로 각각의 정수가 방출될 때마다 `requestFlow` 를 호출할 수 있다.
(물론 출력 문이 없으니까 아직 출력되는 결과는 없다)

위 코드의 결과로 평탄화가 필요한 플로우의 플로우(`Flow<Flow<String>>`)를 얻을 수 있다.

컬렉션과 시퀀스들은 flatten 과 flatMap 이라는 연산자를 가지고 있는 반면 플로우는 비동기적인 특성으로 인해 다른 모드의 플래트닝이 필요해서 별도의 플래트닝 연산자가 정의되어 있다.

### flatMapConcat

연결(Concatenation) 모드는 `flatMapConcat`, `flattenConcat` 연산자에 의해 구현된다. 이 연산자들은 위에서 말한 시퀀스의 연산자들과 유사하게 동작하는 연산자들이다.

```kotlin
package flattening

import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i : Second")
}

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()

    (1..3).asFlow().onEach { delay(100) }
        .flatMapConcat { requestFlow(it) }
        .collect { value ->
            println("$value : ${System.currentTimeMillis() - startTime} ms 만큼 걸림")
        }
}
```

```text
1: First : 131 ms 만큼 걸림
1 : Second : 634 ms 만큼 걸림
2: First : 736 ms 만큼 걸림
2 : Second : 1238 ms 만큼 걸림
3: First : 1339 ms 만큼 걸림
3 : Second : 1840 ms 만큼 걸림
```

### flatMapMerge

`flatMapMerge` 와 `flattenMerge` 를 사용해서 들어오는 모든 플로우들을 동시에 수집하고, 그 값들을 하나의 플로우로 합쳐서 값들이 최대한 빠르게 방출되도록 하는 모드를 설정할 수 있다.

이 두 연산자는 모두 옵션으로 `concurrency` 라는 파라미터로 동시에 수집 가능한 플로우의 개수를 제한할 수 있다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i : Second")
}

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()

    (1..3).asFlow().onEach { delay(100) }
        .flatMapMerge { requestFlow(it) }
        .collect { value ->
            println("$value : ${System.currentTimeMillis() - startTime} ms 만큼 걸림")
        }
}
```

```text
1: First : 163 ms 만큼 걸림
2: First : 266 ms 만큼 걸림
3: First : 368 ms 만큼 걸림
1 : Second : 663 ms 만큼 걸림
2 : Second : 767 ms 만큼 걸림
3 : Second : 870 ms 만큼 걸림
```

결과를 보면 `flatMapMerge` 에서 `requestFlow` 를 순차적으로 호출하지만 반환된 플로우들은 동시에 `collect` 하는 것을 확인할 수 있다. 순차적으로 `map { requestFlow(it) }` 를 하고 그 결과에 `flattenMerge` 를 하는 것과 동일하다.

### flatMapLatest

[이전 게시글](https://mangbaam.github.io/kotlin/2022/07/30/kotlin-flow-basic-1.html#h-%EC%B5%9C%EC%8B%A0-%EA%B0%92-%EC%B2%98%EB%A6%AC)에서 살펴본 `collectLatest` 연산자와 비슷한 동작을 하는 플래트닝 모드 연산자가 있다.

`flatMapLatest` 연산자는 새로운 플로우가 방출될 때마다 직전의 플로우를 취소시킨다.

```kotlin
package flattening

import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i : Second")
}

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()

    (1..3).asFlow().onEach { delay(100) }
        .flatMapLatest { requestFlow(it) }
        .collect { value ->
            println("$value : ${System.currentTimeMillis() - startTime} ms 만큼 걸림")
        }
}
```

```text
1: First : 175 ms 만큼 걸림
2: First : 281 ms 만큼 걸림
3: First : 382 ms 만큼 걸림
3 : Second : 882 ms 만큼 걸림
```

위 예제에서 requestFlow 호출 자체가 빠르고, 중단되지 않고, 취소되지도 않기 때문에 모두 실행은 된다. 하지만 다음과 같이 중단에 delay 함수가 들어가면 requestFlow 호출 자체를 취소할 수도 있다.

```kotlin
package flattening

import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun requestFlow(i: Int): Flow<String> = flow {
    emit("$i: First")
    delay(500)
    emit("$i : Second")
}

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()

    (1..3).asFlow().onEach { delay(100) }
        .flatMapLatest {
            delay(500)
            requestFlow(it)
        }
        .collect { value ->
            println("$value : ${System.currentTimeMillis() - startTime} ms 만큼 걸림")
        }
}
```

```text
3: First : 880 ms 만큼 걸림
3 : Second : 1381 ms 만큼 걸림
```

## 플로우 예외

---

플로우 collect 는 emit 로직이나 연산자 안의 코드에서 예외가 발생되면 예외 발생 상태로 종료될 수 있다. 이러한 예외를 다루기 위한 다양한 방법들이 있다.

### 수집기에서 try ~ catch

수집기는 예외를 다루기 위해 코틀린의 `try/catch` 문을 사용할 수 있다.

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.runBlocking

fun simple(): Flow<Int> = flow {
    (1..3).forEach { i ->
        println("$i 방출")
        emit(i)
    }
}

fun main() = runBlocking {
    try {
        simple().collect { value ->
            println(value)
            check(value <= 1) { "$value 수집 "}
        }
    } catch (e: Throwable) {
        println("catch $e")
    }
}
```

```text
1 방출
1
2 방출
2
catch java.lang.IllegalStateException: 2 수집 
```

위 코드에서 예외를 발생시키기 위해 `check` 함수를 사용했다. 조건에 일치하지 않으면  `IllegalStateException` 예외를 발생시키는 함수이다.

코틀린의 try / catch 만으로도 성공적으로 예외를 잡아내고, 그 후에는 어떠한 값도 방출하지 않는다.

위 예제는 방출하는 부분이나 수집하는 부분 등 모든 예외를 잡는다. 상황에 따라서 예외를 처리해야할 경우도 있다.

## 예외 투명성

---

플로우는 예외에 있어서는 반드시 투명해야 한다. `flow {...}` 블록 안에서 try/catch 문으로 예외를 처리한 후에 값을 방출하는 것은 예외 투명성을 위반하는 것이다.

emitter 은 이러한 예외 투명성을 보존하기 위해서 `catch` 연산자를 사용할 수 있고, 이를 통해서 예외 캡슐화가 가능하다.

`catch` 연산자의 구현 블록은 예외의 타입에 따라 각각 다른 대응이 가능하다.

- `throw` 던지기
- `catch` 에서 `emit` 을 사용해 값 방출
- 다른 코드를 통해 예외 무시, 로깅, 기타 처리

```kotlin
package exception

import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun simple(): Flow<String> = flow {
    (1..3).forEach { i ->
        println("$i 방출")
        emit(i)
    }
}.map { value ->
    check(value <= 1) { "$value 에서 예외 발생" }
    "문자열 - $value"
}

fun main() = runBlocking {
    simple()
        .catch { e -> emit("catch $e") }
        .collect { value -> println(value) }
}
```

```text
1 방출
문자열 - 1
2 방출
catch java.lang.IllegalStateException: 2 에서 예외 발생
```

### catch 예외 투명성

예외 투명성을 지키는 `catch` 중간 연산자는 오직 업스트림(`catch` 연산자 위의 모든 연산자) 예외에서만 동작하며 다운 스트림에서 발생한 예외는 처리하지 않는다.

```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun simple(): Flow<Int> = flow {
    (1..3).forEach { i ->
        println("$i 방출")
        emit(i)
    }
}

fun main() = runBlocking {
    simple()
        .catch { e -> println("catch $e") }
        .collect { value ->
            check(value <= 1) { "$value 수집" }
            println(value)
        }
}
```

![image](https://user-images.githubusercontent.com/44221447/183281302-4407c515-ac78-4a31-8337-d5fee3e6a56c.png)

catch 다음에 발생한 예외에 대해서는 처리하지 않는 것을 확인할 수 있다.

```kotlin
package exception

import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun simple(): Flow<Int> = flow {
    (1..3).forEach { i ->
        println("$i 방출")
        emit(i)
    }
}

fun main() = runBlocking {
    simple()
        .onEach { value ->
            check(value <= 1) { "$value 수집" }
            println(value)
        }
        .catch { e -> println("catch $e") }
        .collect()
}
```

```text
1 방출
1
2 방출
catch java.lang.IllegalStateException: 2 수집
```

위와 같은 방식으로 모든 예외를 처리하면서 투명성을 지킬 수 있다.

## 플로우 종료

---

플로우 수집이 정상적으로든 예외가 발생해서든 종료된 후 어떠한 동작을 수행해야 하는 경우 두 가지 방식으로 가능하다. -> Imperative / Declarative

### Imperative finally 블록

`collect` 가 완료될 때의 코드를 try/catch 문과 함께 `finally` 블록에 작성할 수 있다.

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.runBlocking

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking {
    try {
        simple().collect { value -> println(value) }
    } finally {
        println("완료!")
    }
}
```

```text
1
2
3
완료!
```

### Declarative handling

선언적인 접근으로는 플로우에 `onCompletion` 중간 연산자를 추가해서 플로우가 완전히 수집되었을 때 실행될 로직을 작성할 수 있다.

위 예제에서 `onCompletion` 을 추가해도 같은 결과를 얻을 수 있다.

```kotlin
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.flow.onCompletion
import kotlinx.coroutines.runBlocking

fun simple(): Flow<Int> = (1..3).asFlow()

fun main() = runBlocking {
    simple()
        .onCompletion { println("완료!") }
        .collect { value -> println(value) }
}
```

```text
1
2
3
완료!
```

`onCompletion` 을 사용했을 때의 장점은 람다에 nullable 한 throwable 파라미터를 통해 플로우 수집이 성공적으로 종료되었는지 여부를 알 수 있다는 것이다.

```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun simple() = flow {
    emit(1)
    throw RuntimeException()
}

fun main() = runBlocking {
    simple()
        .onCompletion { cause ->
            if (cause != null) println("플로우가 예외적으로 완료됨!")
        }
        .catch { cause -> println("catch exception") }
        .collect { value -> println(value) }
}
```

```text
1
플로우가 예외적으로 완료됨!
catch exception
```

`onCompletion` 연산자는 `catch` 와는 달리 예외를 처리하지 않고 다운 스트림으로 전달한다. 결국 예외는 `catch` 연산자에서 처리된다.

### 성공적 종료

`catch` 와 또 다른 차이점은 `onCompletion` 이 모든 예외를 확인하고 업스트림 Flow 가 취소나 실패없이 성공적으로 완료된 경우에만 `null` 예외를 수신한다는 점이다.

```kotlin
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.runBlocking

fun simple() = (1..3).asFlow()

fun main() = runBlocking {
    simple()
        .onCompletion { cause -> println("플로우가 예외적으로 완료됨! : $cause") }
        .collect { value ->
            check(value <= 1) { "$value 수집" }
            println(value)
        }
}
```

![image](https://user-images.githubusercontent.com/44221447/183287608-8dccbb5a-9181-4e21-9928-3796c36aad09.png)

위 결과로 Flow 가 다운스트림 예외 때문에 성공적으로 완료되지 못 했을 때 cause 가 null 이 아니게 된다는 것을 알 수 있다.

## Imperative(명령적) vs Declarative(선언적)

---

플로우를 수집하는 방법과 예외를 명령적 / 선언적 방식으로 다루는 방법에 대해서 알아보았다.

둘 중 어느 방식이 더 좋은 방법인가? 라는 질문에 대해서는 (라이브러리로써) 어느 특정 접근이 더 낫다기 보다는 선호하는 코딩 스타일에 따라 선택하면 된다고 한다.

## 플로우 실행

---

어떤 소스로부터 발생하는 비동기적인 이벤트를 표현할 때 플로우를 사용하면 쉽게 사용할 수 있다.

이런 경우에서 일반적으로 들어오는 이벤트들에 대응하는 처리 코드를 `addEventListener` 를 통해 등록하고 이후 필요한 일을 진행하는 방식을 사용하기도 한다.

플로우에서는 `onEach` 연산자가 이런 역할을 담당하고 있다. 하지만 `onEach` 는 중간 연산자이기 때문에 플로우 주십을 시작하기 위해서는 [종단 연산자](https://mangbaam.github.io/kotlin/2022/07/30/kotlin-flow-basic-1.html#h-flow-%EC%A2%85%EB%8B%A8terminal-%EC%97%B0%EC%82%B0%EC%9E%90)가 필요하다.

`onEach` 연산자 이후에 `collect` 종단 연산자를 사용하면 플로우가 모두 수집될 때까지 코드는 대기하게 된다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.collect
import kotlinx.coroutines.flow.onEach
import kotlinx.coroutines.runBlocking

fun events() = (1..3).asFlow().onEach { delay(100)  }

fun main() = runBlocking {
    events()
        .onEach { event -> println("Event: $event") }
        .collect()

    println("완료!")
}
```

```text
Event: 1
Event: 2
Event: 3
완료!
```

`launchIn` 종단 연산자를 사용하면 플로우 수집을 다른 코루틴에서 진행할 수 있고, 이후 코드들이 곧바로 실행될 수 있기 때문에 유용하게 사용될 수 있다.

```kotlin
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.asFlow
import kotlinx.coroutines.flow.launchIn
import kotlinx.coroutines.flow.onEach
import kotlinx.coroutines.runBlocking

fun events() = (1..3).asFlow().onEach { delay(100)  }

fun main() = runBlocking {
    events()
        .onEach { event -> println("Event: $event") }
        .launchIn(this)

    println("완료!")
}
```

```text
완료!
Event: 1
Event: 2
Event: 3
```

`launchIn` 연산자는 플로우를 수집할 코루틴의 CoroutineScope 를 파라미터로 전달받는다. 위 예제에서는 runBlocking 코루틴 빌더의 스코프가 전달되었다. 그래서 플로우가 실행되는 동안 runBlocking 스코프는 자식 코루틴의 종료를 기다리게 되고 결국 메인 함수가 반환되어 프로그램이 종료되는 것을 방지한다.

안드로이드와 같이 실제 애플리케이션에서는 생명 주기를 갖는 스코프를 전달해서 스코프가 취소되면 스코프에 속한 플로우도 수집을 취소하도록 할 수 있다.

`onEach { ... }.launchIn(scope)` 와 `addEventListener` 은 동일하게 수행되지만 차이점이라고 하면 `addEventListener` 은 `removeEventListener` 가 필요하다는 점이다.

`launchIn` 도 `Job` 을 반환하기 때문에 스코프를 취소하거나 특정 Job 에 대한 조인을 할 필요가 없다.

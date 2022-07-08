---
layout: post
title: Coroutine suspend function
subtitle: 코루틴 중단 함수의 원리
categories: Kotlin
tags: [kotlin,android,coroutine]
---

## ⭐

> 본 글은 [suspend 함수란 무엇이고 어떻게 동작할까?](https://myungpyo.medium.com/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B3%B5%EC%8B%9D-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%9D%BD%EA%B8%B0-part-1-dive-3-b174c735d4fa)을 읽고 공부한 내용을 정리하는 글입니다.

코루틴을 사용해봤다면 `suspend` 키워드가 낯설지 않을 것이다. 중단(suspend)함수 없는 코루틴은 상상할 수 없다.

일반 함수(순차 함수)가 호출된 후 함수의 모든 내용을 모두 수행한 후 반환되는 것에 반해 중단 함수는 함수 내용을 수행 중 잠시 중단되어 다른 로직을 수행한 뒤 다시 돌아올 수 있다는 특징이 있다.

단순히 `suspend` 키워드를 붙이는 것으로 어떻게 중단 함수가 만들어지고, 동작하는지 알아보고자 한다.

## CPS: Continuation-Passing-Style

---

`suspend` 키워드가 붙은 함수는 코틀린 컴파일러에 의해 **CPS(Continuation-Passing-Style)** 로 동작하게 된다.

> CPS: `Continutation` 파라미터가 함수의 마지막 인자로 추가되며 반환 값이 `Any?` 로 변경됨

CPS로 변환된 중단 함수는 코루틴 스코프나 다른 중단 함수 내에서만 호출될 수 있지만 코루틴 실행 흐름을 일시 중단하는 분절점이 될 수 있다.

`suspend` 키워드를 붙여서 중단 함수로 만들 수 있는 함수는 **탑 레벨 함수**, **확장 함수**, **멤버 함수(메서드)**, **로컬 함수**, **연산자(operator) 함수** 등 ***모든 함수***가 가능하다.

### Continuation

```kotlin
@SinceKotlin("1.3")
public interface Continuation<in T> {
    /**
     * The context of the coroutine that corresponds to this continuation.
     */
    public val context: CoroutineContext

    /**
     * Resumes the execution of the corresponding coroutine passing a successful or failed [result] as the
     * return value of the last suspension point.
     */
    public fun resumeWith(result: Result<T>)
}
```

## 중단 과정

---

중단 함수가 코루틴에서 호출되면 그 시점의 실행 정보들을 `Continutation` 객체로 만들어서 캐시해 두었다가 실행이 재개되면 저장된 실행 정보를 기반으로 실행을 이어 나간다.

```kotlin
suspend fun sum(num1: Int, num2: Int): Int {
    delay(2000)
    return num1 + num2
}
```

위 코드처럼 2초가 걸리는 작업이 있다고 해보자. 이 함수는 중단 함수이기 때문에 코루틴이나 다른 중단 함수 내부에서 호출되어야 한다.

```kotlin
fun main(args: Array<String>) = runBlocking {
    val job = GlobalScope.launch {
        val result = sum(100, 200)
        println("Result : $result")
    }
    job.join()
}
```

GlobalScope.launch으로 생성된 코루틴 내에서 `sum()` 중단 함수가 호출되었다.

작성된 코드를 바이트 코드로 바꿔서 보면 다음과 같은 형식으로 만들어지는 것을 볼 수 있다. ([바이트 코드 보는 법](#바이트-코드-확인하는-법))

`INVOKESTATIC coroutine/SuspendKt.sum (IILkotlin/coroutines/Continuation;)Ljava/lang/Object;`

![byte](https://user-images.githubusercontent.com/44221447/177835615-6e382893-8b27-462a-951c-524936cb2bc0.png)

`INVOKESTATIC`은 정적 메서드를 실행하는 [자바 가상 머신 명령](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html)이다. sum 함수가 자바 코드로 변환되면 정적 메서드로 변환되기 때문이다.

`coroutine/SuspendKt.sum` 은 coroutine 패키지의 suspend.kt 파일에 있는 sum 함수를 뜻한다. 자바 코드로 변환하면 suspend.kt 파일은 SuspendKt 클래스로 변환되고, sum 함수는 SuspendKt 클래스의 정적 메서드로 변환된다.

`IILkotlin/coroutines/Continuation;` 부분은 sum 함수 파라미터 형식을 나타낸다. **I: Integer, I: Integer, L: Class Instance** 즉, `sum(int, int, kotlin.coroutines.Continuation)` 으로 변환될 것을 의미한다.

그리고 컴파일러가 함수의 마지막 파라미터에 `Continuation` object를 추가한 것을 볼 수 있다.

![디컴파일](https://user-images.githubusercontent.com/44221447/177836872-6f3a6132-f0de-47b9-9cdc-d96960bf7aa9.png)

바이트 코드를 디컴파일 해보면 위 사진과 같이 `SuspendKt`라는 클래스가 만들어지고 sum 함수가 정적(static) 메서드로 변환된 것을 확인할 수 있다. 또한 마지막 파라미터에 `Continuation` 객체가 추가된 것도 보인다.

`Continutaion`이 추가되면서 **CPS** 방식으로 함수가 사용될 수 있게 되었고, 코루틴 프레임워크는 이를 통해 중단/재개 전환 동작을 수행할 수 있게 된다.

### 중단 예제

```kotlin
fun main(args: Array<String>) = runBlocking {
    (1..2).forEach { num ->
        launch {
            longRunningTask(num, num + 1)
        }
    }
}

suspend fun longRunningTask(p1: Int, p2: Int): Int {
    log("계산 시작: p1: $p1, p2: $p2")
    delay(2000)
    val intermediateResult = p1 + p2
    log("계산 중간 결과 (p1 + p2) = $intermediateResult")
    delay(2000)
    val finalResult = intermediateResult * 2
    log("계산 종료 (result x 2) : $finalResult")
    return finalResult
}

private fun log(message: String) {
    println("[${Thread.currentThread().name}] : $message")
}
```

위 예제에서는 2개의 코루틴을 만들어 2번의 delay() 함수를 호출하는 `longRunningTask`가 main 함수에서 2번 호출된다.

만약 longRunningTask 함수가 일반 함수였다면 longRunningTask(1, 2)가 4초간 모든 계산 결과를 마친 후에 longRunningTask(2, 3)이 수행되어 8초 이상이 소요됐을 것이다.

하지만 중단 함수로 만들어진 예제를 실행시켜 보면 다음과 같은 결과를 얻을 수 있다.

```text
[main] : 계산 시작: p1: 1, p2: 2
[main] : 계산 시작: p1: 2, p2: 3
[main] : 계산 중간 결과 (p1 + p2) = 3
[main] : 계산 중간 결과 (p1 + p2) = 5
[main] : 계산 종료 (result x 2) : 6
[main] : 계산 종료 (result x 2) : 10
```

실행 결과를 보아 첫 번째 중단 함수가 delay()를 만난 동안 두 번째 중단 함수가 수행되어 번갈아가면 수행되는 것을 알 수 있다. delay() 를 만난 시점에서 잠시 중단되었다고 할 수 있는데, 이 지점을 중단점(suspension point)라고 하자.

중단 함수가 중단점을 만나면 다른 함수에게 실행 기회가 주어진다. 이런 식으로 하나의 스레드 안에서 실행 시간을 분할해가며 수행할 수 있다.

## 중첩된 중단 함수

---

그렇다면 중단 함수 안에서 또 다른 중단 함수를 호출하면 어떻게 될까?

![stack](https://user-images.githubusercontent.com/44221447/177974256-24e79159-da3e-4472-ab2a-c78b631097ee.png)

위 그림을 보면 일반적으로 순차 함수 안에서 다른 함수를 호출할 때 스택에 쌓이는 모습과 미슷하게 동작하게 된다.

다만 차이점이 있다면 일반 함수 호출은 OS에서 콜 스택을 관리해주지만 중첩 코루틴이나 중첩 중단 함수에서는 코루틴 프레임워크가 CPS 방식으로 호출 정보(Continuation)를 스택 형태로 유지하다가 호출 스택의 가장 마지막 함수가 실행을 종료할 때 결과 값이 직전 호출 함수들로 전파되며 직전 함수를 재개하게 된다.

만약 스택 상에서 어떤 함수가 예외를 발생시키면 이 예외는 최초 호출 함수까지 Continuation을 통해 전파된다.

## 부록

---

### 바이트 코드 확인하는 법

젯브레인 IDE(인텔리제이, 안드로이드 스튜디오 등)에서 할 수 있는 방법이다.

![bytecode](https://user-images.githubusercontent.com/44221447/177835433-c5dc8ac0-1951-4b82-a571-bb93e2856779.png)

> 도구 - Kotlin - Kotlin 바이트코드 표시

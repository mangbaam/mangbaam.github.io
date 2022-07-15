---
layout: post
title: Coroutine withContext + NonCancellable
subtitle: 
categories: Kotlin
tags: [kotlin,coroutine]
---

## ⭐

> 본 글은 [withContext 코루틴 빌더가 하는 일은 무엇인가?](https://myungpyo.medium.com/%EC%BD%94%EB%A3%A8%ED%8B%B4-%EA%B3%B5%EC%8B%9D-%EA%B0%80%EC%9D%B4%EB%93%9C-%EC%9E%90%EC%84%B8%ED%9E%88-%EC%9D%BD%EA%B8%B0-part-2-dive-1-4c468828319)을 읽고 공부한 내용을 정리하는 글입니다.

## 지난 내용

[이전 글](https://mangbaam.github.io/kotlin/2022/07/15/coroutine-cancellation.html)에서 중단된 코루틴의 `finally { }` 블록에서 중단 함수를 사용할 때 withContext를 사용하는 것에 대해 다뤘다. 간단히 살펴보면 다음과 같다.

```kotlin
fun main(args: Array<String>): Unit = runBlocking {
    val job = launch {
        try {
            repeat(1000) { i ->
                println("$i")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                delay(1000L) // <-- 중단 함수
                println("실행 완료")
            }
        }
    }

    delay(1300L)
    println("중단 시도")
    job.cancelAndJoin()
    println("종료")
}
```

`cancelAndJoin()`으로 job이 취소되면 try { } 블록에서 CancelledException이 발생하면서 finally { } 블럭으로 진입하게 된다. 이때 withContext 없이 중단 함수인 delay()를 호출하면 현재 코루틴 컨텍스트는 이미 취소된 상태이기 때문에 CancelledException이 다시 발생한다.

이를 withContext { } 와 NonCancellable 컨텍스트 요소의 조합으로 해결할 수 있었다.

## withContext의 반환

---

```kotlin
public suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
): T {
    contract {
        callsInPlace(block, InvocationKind.EXACTLY_ONCE)
    }
    return suspendCoroutineUninterceptedOrReturn sc@ { uCont ->
        // compute new context
        val oldContext = uCont.context
        val newContext = oldContext + context
        // always check for cancellation of new context
        newContext.ensureActive()
        // FAST PATH #1 -- new context is the same as the old one
        if (newContext === oldContext) {
            val coroutine = ScopeCoroutine(newContext, uCont)
            return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
        }
        // FAST PATH #2 -- the new dispatcher is the same as the old one (something else changed)
        // `equals` is used by design (see equals implementation is wrapper context like ExecutorCoroutineDispatcher)
        if (newContext[ContinuationInterceptor] == oldContext[ContinuationInterceptor]) {
            val coroutine = UndispatchedCoroutine(newContext, uCont)
            // There are changes in the context, so this thread needs to be updated
            withCoroutineContext(newContext, null) {
                return@sc coroutine.startUndispatchedOrReturn(coroutine, block)
            }
        }
        // SLOW PATH -- use new dispatcher
        val coroutine = DispatchedCoroutine(newContext, uCont)
        block.startCoroutineCancellable(coroutine, coroutine)
        coroutine.getResult()
    }
}
```

withContext 함수는 로직이 담긴 코드 블록과 이것이 실행될 코루틴 컨텍스트를 매개 변수로 받는다.

```kotlin
@kotlin.SinceKotlin @kotlin.internal.InlineOnly public suspend inline fun <T> suspendCoroutineUninterceptedOrReturn(crossinline block: (kotlin.coroutines.Continuation<T>) -> kotlin.Any?): T { contract { /* compiled contract */ }; /* compiled code */ }
```

그리고 [suspendCoroutineUninterceptedOrReturn()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.intrinsics/suspend-coroutine-unintercepted-or-return.html) 함수를 반환하는데 이것은 전달된 코드 블럭에서 호출 코루틴(Continuation) 정보에 접근할 수 있도록 해준다. 전달된 코드 블록에서 COROUTINE_SUSPENDED 라는 사전 정의 된 상수 값을 반환 할 경우 코루틴이 처리를 위해 시간이 필요하며 즉시 값을 반환하지 않고 처리가 완료되면 continuation 파라미터를 통해 결과를 전달할 것을 나타내며, 그 이외의 값을 반환할 경우 중단 없이 바로 결과 값을 반환할 것을 나타낸다.

## withContext 3가지 방식

---

withContext 함수로 전달된 컨텍스트에 따라 3가지 방식으로 처리가 된다.

- Function Caller - Callee 코루틴 컨텍스트가 완전히 동일한 경우 (===)
  - 현재 컨텍스트에서 그대로 코드 블럭을 실행해도 무방하기 때문에 ScopeCoroutine을 만들어 바로 실행한다. ScopeCoroutine은 코루틴 간 컨텍스트가 동일해서 바로 작업을 수행할 수 있을 경우 사용됨
- Function Caller - Callee 코루틴 컨텍스트가 서로 다른 부분이 있지만 Dispatcher 컨텍스트 요소는 동일한 경우 (==)
  - UndispatchedCoroutine을 만들고 새로 만들어진 컨텍스트 안에서 실행 (withContext)
- 그 외의 경우
  - 코루틴 컨텍스트 간에 dispatcher를 포함한 변경 사항이 있는 것이기 때문에 새로운 컨텍스트를 이용해 적절한 스레드로 dispatch 되어 실행될 수 있도록 DispathcedCoroutine을 생성해서 실행

## NonCancellable

---

지난 내용에서 NonCancellable을 withContext로 넘기는 부분이 있었다. NonCancellable은 어떻게 취소된 코루틴 블록에서 예외를 발생시키지 않고 작업을 이어나갈 수 있을까?

내부를 살펴보자.

```kotlin
public object NonCancellable : AbstractCoroutineContextElement(Job), Job {

    private const val message = "NonCancellable can be used only as an argument for 'withContext', direct usages of its API are prohibited"

    /**
     * Always returns `true`.
     * @suppress **This an internal API and should not be used from general code.**
     */
    @Deprecated(level = DeprecationLevel.WARNING, message = message)
    override val isActive: Boolean
        get() = true

    /**
     * Always returns `false`.
     * @suppress **This an internal API and should not be used from general code.**
     */
    @Deprecated(level = DeprecationLevel.WARNING, message = message)
    override val isCompleted: Boolean get() = false

    /**
     * Always returns `false`.
     * @suppress **This an internal API and should not be used from general code.**
     */
    @Deprecated(level = DeprecationLevel.WARNING, message = message)
    override val isCancelled: Boolean get() = false

    // ...
```

NonCancellable은 AbstractCoroutineContextElement를 상속하는 컨텍스트 요소이다. 작업을 이어나갈 수 있는 이유는 isActive가 항상 true를 반환하기 때문이다.

그렇다면 withContext(NonCancellable)은 [withContext 3가지 방식](#withcontext-3가지-방식) 중 어느 방식으로 진행될까?

정답은 두 번째 경로이다.

코드에서 oldContext는 현재 코루틴의 컨텍스트이고, newContext는 oldContext + NonCancellable 이다. NonCancellable에는 dispatcher 컨텍스트 요소에 대한 정의가 들어있지 않으므로 둘을 plus 하게 되면 oldContext의 dispatcher 요소를 상속하여 사용하게 된다.

> NonCancellable은 withContext 에서만 사용되어야 한다.

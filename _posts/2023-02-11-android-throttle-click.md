---
layout: post
title: Android throttle click with Flow and without Flow
subtitle:
categories: Android
tags: [android,flow,throttle,xml]
---

## Throttle 이란?

안드로이드와 같은 UI 기반 프레임워크는 클릭과 같은 사용자 이벤트를 다룰 일이 많다.

**Throttle**이란 일정 시간 내에 들어온 이벤트 중 하나의 이벤트만 처리하는 기술이다. 이 이벤트는 Flow나 RxJava와 같은 Stream에서는 계속 반복되어 결국 일정한 주기로 이벤트를 처리하게 되고, 클릭과 같은 단발성 이벤트의 경우 한 번만 이벤트를 처리하게 된다.

이때 일정 시간 내의 이벤트 중 가장 먼저 들어온 이벤트만 처리하는 것을 `ThrottleFirst`, 가장 마지막에 들어온 이벤트를 처리하는 것을 `ThrottleLast`라고 한다. 일반적으로 Throttle이라고 하면 `ThrottleFirst`를 칭한다.

### ThrottleFirst

![](https://i.imgur.com/RNIETz6.png)

Stream을 관찰하고 있을 때, 첫 번째로 방출된 이벤트를 받고, 그 다음 이벤트는 windowDuration(특정 간격)이 지날 때까지 오지 않는다.
즉, 이벤트가 방출될 때 windowDuration이 지났는지 확인하고, 지났다면 방출하고, 지나지 않았다면 버린다.
**주로 클릭 이벤트를 처리할 때 사용한다.** 예를 들어 회원 가입 버튼이 연속으로 눌려서 API 요청이 중복으로 들어가면서 발생하는 오류를 원천적으로 방지할 수 있는 방법이 될 수 있다.

### ThrottleLast

![](https://i.imgur.com/CbigbGn.png)

throttleLast는 intervalDuration(특정 시간)동안 들어온 이벤트 중 가장 최근의 데이터를 방출한다.
**주로 타이머와 같은 곳에 활용된다.**

### Debounce (Throttle with timeout)

Throttle을 얘기할 때 흔히 **Debounce**가 비교 대상이 된다.

RxJava에서는 `throttleWithTimeout`과 `debounce`가 있는데, 둘은 동일한 동작을 한다고 한다. (Debounce 라는 용어를 더 많이 사용하므로 이하 Debounce라고 부르겠다)

![](https://i.imgur.com/mKFJCpp.png)

Debounce는 throttleLast와 비슷하지만 동적인 타이머를 가진다. throttleLast가 일정 시간동안 반복되는 것에 반해, Debounce는 일정 시간 내에 새로운 이벤트가 발생하면 타이머가 리셋된다. 그래서 일정 시간 간격 내에 한 번씩은 실행되는 throttleLast와 달리 Debounce는 일정 시간 간격 내에 새로운 이벤트가 계속 발생된다면 이전의 이벤트들은 모두 손실된다.

**가장 많이 사용되는 곳이 텍스트 자동 완성이나 쿼리를 날릴 때 이다.**

텍스트 필드에 텍스트를 입력할 때 자동 완성 기능을 만들거나 검색 쿼리를 작성하는 경우 글자가 입력되는 동안은 이벤트를 발생시키지 않고, 입력을 마치고 일정 시간이 지나고 나서야 자동 완성을 위한 탐색을 하거나 검색 쿼리를 날리는 동작을 할 수 있다.

## Throttle Click with Kotlin Flow

```kotlin
fun View.clickFlow(): Flow<Unit> = callbackFlow {  
    setOnClickListener { trySend(Unit) }  
    awaitClose { setOnClickListener(null) }  
}
```

클릭을 플로우로 만들어주기 위해 `callbackFlow`를 사용했다. 그리고 클릭이 될 때 이벤트를 보내기 위해 `setOnClickListener { trySend(Unit) }` 을 작성했다.

`awaitClose()` 에서는 `setOnClickListenr`를 해제해주고 있는데, 그렇지 않으면 Flow 수집기가 완료된 후에도 계속 동작하여 메모리 누수가 발생한다.

```kotlin
fun <T> Flow<T>.throttleFirst(duration: Long): Flow<T> = flow {  
    var lastTime = 0L  
    collect {  
        val currentTime = System.currentTimeMillis()  
        if (currentTime - lastTime > duration) {  
            lastTime = currentTime  
            emit(it)  
        }  
    }  
}
```

throttleFirst를 작성하는 것은 생각보다 간단하다. 지난 방출되었던 시간 이후로 얼마나 지났는지 확인한 후에 duration보다 크다면 lastTime을 갱신하고, Flow를 방출한다.

```kotlin
fun View.onThrottleClick(  
    scope: CoroutineScope,  
    duration: Long = 300L,  
    onClick: (v: View) -> Unit,  
) {  
    clickFlow()  
        .throttleFirst(duration)  
        .onEach { onClick(this) }  
        .launchIn(scope)  
}
```

위에서 만든 `clickFlow()`와 `throttleFirst()`를 활용해 **throttleClick** 동작을 만들 수 있다.

```kotlin
class MainActivity : AppCompatActivity() {
    fun onCreate() {
        super.onCreate()

        binding.btnLogin.onThrottleClick(lifecycleScope) {  
            // Do Something  
        }
    }
}
```

위와 같이 사용할 수 있다.

## Throttle Click without Kotlin Flow

```kotlin
class OnThrottleClickListener(  
    private val clickListener: View.OnClickListener,  
    private val interval: Long = 300L,  
) : View.OnClickListener {  
  
    private var clicked = false  
    override fun onClick(v: View?) {  
        if (!clicked) {  
            clicked = true  
            v?.run {  
                postDelayed(  
                    { clicked = false },  
                    interval,  
                )  
                clickListener.onClick(v)  
            }  
        } else {  
            // 생략 가능
            Log.d("로그", "OnThrottleClickListener_onClick: miss!!")  
        }  
    }  
}
```

View.OnClickListener 타입의 클래스인 `OnThrottleClickListener` 를 작성했다. 또한 프로퍼티로 View.OnClickListener 타입의 `clickListener`를 받는다.

View.OnClickListener를 구현하기 때문에 `onClick()`을 오버라이드 해야하는데, 이 메서드는 클릭이 될 때 동작하는 메서드이고, 이 내부에서 clickListener의 onClick을 동작시킨다.

이때 throttle을 구현하기 위해서 `postDelayed`를 사용했다. postDelayed는 **일정 시간 이후에 실행시킬 동작**을 할 수 있는데, 위 코드를 보면 interval 시간 이후에 clicked 값을 false로 변경하고 있다.

즉, 최초에 clicked가 false로 설정되어 있기 때문에 clicked를 true로 바꾼 후 clickListener.onClick(v)이 실행되고, interval 시간 이후에 clicked를 false로 바꾸기 때문에
interval 시간이 지나 clicked가 false로 바뀌기 전까지는 클릭 처리가 되지 않는 것이다.

이를 눈으로 확인하기 위해 else 블록에 Log를 찍어 보았다.

```kotlin
fun View.onThrottleClick(  
    onClick: (v: View) -> Unit,  
) {  
    val listener = View.OnClickListener { onClick(it) }  
    setOnClickListener(OnThrottleClickListener(listener))  
}  
  
fun View.onThrottleClick(  
    interval: Long,  
    onClick: (v: View) -> Unit,  
) {  
    val listener = View.OnClickListener { onClick(it) }  
    setOnClickListener(OnThrottleClickListener(listener, interval))  
}
```

위에서 작성한 OnThrottleClickListener를 사용한 코드이다.

View의 확장함수로 만들었으며, View의 setOnClickListener에 앞서 만든 OnThrottleClickListener를 등록한다. (OnThrottleClickListener가 View.OnClickListener를 구현했기 때문에 가능하다)

```kotlin
binding.btnLogin.onThrottleClick {   
    // Do Something  
}

binding.btnLogin.onThrottleClick(1000) {  
    // Do Something  
}
```

위와 같이 활용 가능하다.

## BindingAdapter에서 사용하기

```kotlin
@BindingAdapter("onThrottleClick", "clickInterval", requireAll = false)  
fun applyThrottleClick(view: View, listener: View.OnClickListener, interval: Long? = 300L) {  
    val throttleListener = interval?.let { time ->  
        OnThrottleClickListener(listener, time)  
    } ?: OnThrottleClickListener(listener)  
    view.setOnClickListener(throttleListener)  
}
```

간단하게 위와 같이 작성할 수 있다.

```xml
<com.google.android.material.button.MaterialButton  
    android:id="@+id/btn_signup"
    android:layout_width="wrap_content"  
    android:layout_height="wrap_content"
    app:onThrottleClick="@{() -> viewmodel.onNextButtonClick()}"
    app:clickInterval="@{3000}"
    tools:text="다음" />
```

위와 같이 활용할 수 있으며, `app:clickInterval`은 생략할 수 있다.

---

#### 참고

- https://proandroiddev.com/throttling-in-rxjava-2-d640ea5f7bf1
- https://blog.yena.io/studynote/2019/12/26/Android-Kotlin-ClickListener.html
- https://m1nzi.tistory.com/2
- chatGPT

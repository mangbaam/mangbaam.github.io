---
layout: post
title: CoroutineContext and CoroutineScope in Kotlin and Android
subtitle: 코루틴 컨텍스트와 코루틴 스코프 딥 다이브
categories: Kotlin
tags: [kotlin,android,coroutine]
---

## ⭐

> 본 글은 [CoroutineContext 와 CoroutineScope 란 무엇인가?](https://myungpyo.medium.com/reading-coroutine-official-guide-thoroughly-part-1-7ebb70a51910)을 읽고 공부한 내용을 정리하는 글입니다.

## CoroutineContext.kt

---

다음 코드들은 **CoroutineContext.kt** 파일에 있는 코드들이다.

### CoroutineContext 인터페이스

```kotlin
public interface CoroutineContext {
  /**
   * Returns the element with the given [key] from this context or `null`.
   * Keys are compared _by reference_, that is to get an element from the context the reference to its actual key
   * object must be presented to this function.
   */
  public operator fun <E : Element> get(key: Key<E>): E?
  /**
   * Accumulates entries of this context starting with [initial] value and applying [operation]
   * from left to right to current accumulator value and each element of this context.
   */
  public fun <R> fold(initial: R, operation: (R, Element) -> R): R
  /**
   * Returns a context containing elements from this context and elements from  other [context].
   * The elements from this context with the same key as in the other one are dropped.
   */
  public operator fun plus(context: CoroutineContext): CoroutineContext = ...impl...
  /**
   * Returns a context containing elements from this context, but without an element with
   * the specified [key]. Keys are compared _by reference_, that is to remove an element from the context
   * the reference to its actual key object must be presented to this function.
   */
  public fun minusKey(key: Key<*>): CoroutineContext
}
```

**CoroutineContext** 인터페이스에는 4개의 메서드가 있다.

- `get()`: 주어진 key에 해당하는 컨텍스트 요소를 반환한다.
- `fold()`: 초기값(`initial`)을 시작으로 제공된 병합 함수(`operation`)를 사용해 대상 컨텍스트 요소들을 병합한 결과를 반환한다. 예를 들어 초기값에 `EmptyCoroutineContext`를 주고 특정 컨텍스트 요소들만 찾아 추가하는 함수를 주면 해당 요소만으로 구성된 코루틴 컨텍스트를 만들 수 있다.
- `plus()`: 현재 컨텍스트와 매개변수로 받은 `context`가 갖는 모든 요소들을 포함하는 컨텍스트를 반환한다. 현재 컨텍스트 요소와 `context`의 요소 중 중복되는 것은 현재 컨텍스트에서 중복 제거 한다.
- `minusKey()`: 현재 컨텍스트에서 주어진 키를 갖는 요소들을 제외한 새로운 컨텍스트를 반환한다.

### Key 인터페이스

```kotlin
/**
 * Key for the elements of [CoroutineContext]. [E] is a type of element with this key.
 * Keys in the context are compared _by reference_.
 */
public interface Key<E : Element>

```

**Key**는 `Element` 타입을 제네릭 타입으로 가진다.

### Element 인터페이스

```kotlin
/**
 * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
 */
public interface Element : CoroutineContext {
  /**
   * A key of this coroutine context element.
   */
  public val key: Key<*>
  
  ...overrides...
}
```

`Element`는 `CoroutineContext`를 상속하고, key를 멤버 속성으로 갖는다.

`CoroutineContext`를 구성하는 `Element`의 예로는 `CoroutineId`, `CoroutineName`, `CoroutineDispatcher`, `ContinuationInterceptor`, `CoroutineExceptionHandler` 등이 있다. 이런 요소(element)들은 각각의 key를 기반으로 **CoroutineContext**에 등록된다.

### 요약

코루틴 컨텍스트(CoroutineContext)에는 코루틴 컨텍스트를 상속한 요소(Element) 들이 있고, 각 요소들이 등록될 때 요소의 고유한 키를 기반으로 등록된다는 것이다.

### CoroutineContext의 구현체

`CoroutineContext` 인터페이스를 구현한 구현체는 다음 3가지가 있다.

- `EmptyCoroutineContext`: 컨텍스트가 명시되지 않은 경우 이 singleton 객체가 사용된다.
- `CombinedContext`: 두 개 이상의 컨텍트스가 명시되면 컨텍스트 간 연결을 위한 컨테이너 역할을 하는 컨텍스트이다.
- `Element`: 컨텍스트의 각 [요소](#element-인터페이스)들도 `CoroutineContext`를 구현한다.

### 예시

![코루틴컨텍스트](https://user-images.githubusercontent.com/44221447/177522788-218646a5-6f77-4f59-895a-b27d92793b80.png)

위 그림은 `GlobalScope.launch {}`를 수행할 때 `launch` 함수의 첫 번째 인자인 `CoroutineContext`에 어떤 값을 넘기는 지에 따라 변화하는 코루틴 컨텍스트의 상태를 보여준다.

하나 하나 살펴보면 `CoroutineId`, `ContinuationIntercepter`, `CoroutineName`, `CoroutineExceptionHandler`가 보이는데 이것들 각각이 요소이다.

인자로 넘겨줄 때 `+` 연산자를 사용해 연결하고 있는데 이는 [CoroutineContext 인터페이스](#coroutinecontext-인터페이스)가 `plus` 연산자(operator)를 구현하고 있기 때문이다.

이로 인해 각 요소가 병합되어 `CombinedContext`(병합된 CoroutineContext)가 만들어진다.

다시 그림을 보면 주황색 테두리로 묶여 있는 것을 확인할 수 있는데 이는 `CoroutineContext`와 `Element`가 묶여서 하나의 `CoroutineContext`(CombinedContext)가 된다는 것을 나타낸 것이다. 그리고 마지막에는 항상 `ContinuationInterceptor`가 존재하는데 이는 인터셉터로의 빠른 접근을 위해서라고 한다.

#### 코드 확인

![런치](https://user-images.githubusercontent.com/44221447/177527365-a142555d-6d2b-4f7a-807d-9bc104c76dfb.png)

*`launch`의 첫 번째 인자로 코루틴 컨텍스트를 받는 것을 확인할 수 있다. 기본 값은 `EmptyCoroutineContext`이다.*

## CoroutineScope

---

```kotlin
public interface CoroutineScope {
    /**
     * Context of this scope.
     */
    public val coroutineContext: CoroutineContext
}
```

`CoroutineScope`는 기본적으로 `CoroutineContext` 하나만 멤버 속성으로 정의하고 있는 인터페이스다.

우리가 사용하는 모든 코루틴 빌더(launch, async, coroutineScope, withContext 등)는 `CoroutineScope`의 확장 함수로 정의된다. 즉, 이 빌더들은 CoroutineScope의 함수들이고, 이것들로 코루틴을 생성할 때 소속된 CoroutineScope에 정의된 CoroutineContext를 기반으로 필요한 코루틴을 생성하게 된다.

### 예제 - 안드로이드 Activity

```kotlin
class MyActivity : AppCompatActivity(), CoroutineScope {
  lateinit var job: Job
  override val coroutineContext: CoroutineContext
  get() = Dispatchers.Main + job

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    job = Job()
  }
  
  override fun onDestroy() {
    super.onDestroy()
    job.cancel()
  }

  fun loadDataFromUI() = launch {
    val ioData = async(Dispatchers.IO) {
      // blocking I/O operation
    }

    // do something else concurrently with I/O
    val data = ioData.await() // wait for result of I/O
    draw(data) // can draw in the main thread
  }
}
```

위 코드는 안드로이드의 Activity에서 CoroutineScope를 구현하는 코드이다. CoroutineScope의 멤버인 CoroutineContext를 구현해야하는데 여기서는 `Dispatchers.Main + job`으로 정의했다. 이렇게 하면 MyActivity에서 생성되는 코루틴은 메인 스레드(UI Thread)로 디스패치 되고, 액티비티에서 생성한 Job 객체를 parent로 하는 Job들이 생성되면서 액티비티 **Job의 생명 주기에 존속되게 된다**. 즉, *parent의 Job을 cancel 하면 모든 자식 Job들도 cancel 된다.*

### GlobalScope.launch 가 의미하는 것

CoroutineScope는 CoroutineContext를 가지는 인터페이스라는 사실을 알았고, 이를 통해서 범위를 지정할 수 있다는 것도 [예제](#예제---안드로이드-activity)를 통해 알 수 있었다.

그렇다면 `GlobalScope.launch {}` 코루틴 빌더는 무엇일까?

위 예제처럼 사용자 정의 스코프를 만들 도 있지만 편의를 위해 코루틴 프레임워크에 미리 정의된 스코프들이 있는데 `GlobalScope`가 그 중 하나이다.

```kotlin
// -- in CoroutineScope.kt
object GlobalScope : CoroutineScope {
    override val coroutineContext: CoroutineContext
        get() = EmptyCoroutineContext
}
```

```kotlin
// -- in CoroutineContextImpl.kt
@SinceKotlin("1.3")
public object EmptyCoroutineContext : CoroutineContext, Serializable {
    private const val serialVersionUID: Long = 0
    private fun readResolve(): Any = EmptyCoroutineContext

    public override fun <E : Element> get(key: Key<E>): E? = null
    public override fun <R> fold(initial: R, operation: (R, Element) -> R): R = initial
    public override fun plus(context: CoroutineContext): CoroutineContext = context
    public override fun minusKey(key: Key<*>): CoroutineContext = this
    public override fun hashCode(): Int = 0
    public override fun toString(): String = "EmptyCoroutineContext"
}
```

위 코드는 `GlobalScope`의 구현 부분과 GlobalScope 내부에서 사용하는 `EmptyCoroutineContext`의 구현 부분이다.

`GlobalScope`는 **Singleton object**로써 `EmptyCoroutineContext`를 컨텍스트로 가진다.

`EmptyCoroutineContext`는 `CoroutineContext`에 대한 기본 구현만 한 컨텍스트이고, 어떤 생명주기에 바인딩 된 Job이 정의되지 않았기 때문에 **애플리케이션 프로세스와 동일한 생명주기를 가지게 된다.**

## 안드로이드의 CoroutineScope

---

안드로이드는 주로 Activity, Fragment, ViewModel 등의 컴포넌트에 작성이 되고, 이들은 각자의 생명주기를 가진다. 그리고 `androidx.lifecycle`에는 각 생명주기에 맞는 사전 정의된 스코프들이 있다.

### lifecycleScope

lifecycleScope는 LifecycleOwner의 생명주기를 따른다. LifecycleOwner는 Activity, Fragment 등 다양한 컴포넌트에서 구현하고 있는 인터페이스이다. Activity, Fragment 등이 destroy 되면 이 스코프도 cancel 된다.

```kotlin
public val LifecycleOwner.lifecycleScope: LifecycleCoroutineScope
    get() = lifecycle.coroutineScope
```

위 코드는 lifecycleScope의 구현 부분이고, LifecycleOwner의 확장 함수로 구현된 것을 알 수 있다.

`lifecycle`은 `LifecycleOwner`에서 생명주기를 불러오는 부분이다. (자바에선 `getLifecycle()`)

```kotlin
public abstract class LifecycleCoroutineScope internal constructor() : CoroutineScope {
    internal abstract val lifecycle: Lifecycle

    public fun launchWhenCreated(block: suspend CoroutineScope.() -> Unit): Job = launch {
        lifecycle.whenCreated(block)
    }

    public fun launchWhenStarted(block: suspend CoroutineScope.() -> Unit): Job = launch {
        lifecycle.whenStarted(block)
    }

    public fun launchWhenResumed(block: suspend CoroutineScope.() -> Unit): Job = launch {
        lifecycle.whenResumed(block)
    }
}
```

위 코드는 `LifecycleCoroutineScope`의 코드이다. 추상 클래스이고, 반환 값을 보면 `CoroutineScope`로 되어있는 것을 알 수 있다.

그리고 `launchWhenCreated`, `launchWhenStated`, `launchWhenResumed`가 보인다. 각각은 이름 그대로 생명주기의 create, start, resume 상태에서 block을 수행하는 코루틴을 생성한다.

![caution](https://user-images.githubusercontent.com/44221447/177607525-84163ad9-ed45-4b19-8ebc-928be20113b0.png)

> 위 3개의 메서드에는 사진과 같은 경고성 주석이 달려있다. 특정 상황에 리소스를 낭비할 수 있으니 사용을 추천하지 않으며 미래에는 이 API들이 제거될 것이라고 한다. 대신 `Lifecycle.repeatOnLifecycle`을 사용하라고 한다.

```kotlin
fun LifecycleOwner.repeatOnStarted(block: suspend CoroutineScope.() -> Unit) {
    lifecycleScope.launch {
        lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED, block)
    }
}
```

좀 더 편하게 사용하기 위해 위와 같이 확장 함수로 만들어서 사용하기도 한다.

### viewModelScope

viewModelScope는 AAC ViewModel의 생명주기를 따른다. 즉, ViewModel의 `onCleared`가 호출되는 시점에 이 스코프도 cancel 된다는 의미이다.

```kotlin
private const val JOB_KEY = "androidx.lifecycle.ViewModelCoroutineScope.JOB_KEY"

/**
 * This scope is bound to Dispatchers.Main.immediate
 */
public val ViewModel.viewModelScope: CoroutineScope
    get() {
        val scope: CoroutineScope? = this.getTag(JOB_KEY)
        if (scope != null) {
            return scope
        }
        return setTagIfAbsent(
            JOB_KEY,
            CloseableCoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
        )
    }

internal class CloseableCoroutineScope(context: CoroutineContext) : Closeable, CoroutineScope {
    override val coroutineContext: CoroutineContext = context

    override fun close() {
        coroutineContext.cancel()
    }
}
```

위 코드는 `ViewModelScope`의 구현 부분이다. ViewModel의 확장 함수로 되어 있는 것을 알 수 있다. 그렇기 때문에 ViewModel의 `getTag`와 `setTagIfAbsent` 메소드를 사용할 수 있다.

로직을 살펴보면 `getTag`를 통해 ViewModel에 이미 `JOB_KEY`라는 키로 CoroutineScope가 저장되어 있었다면 해당 스코프를 반환하고, 저장되어 있지 않았다면 저장 후 (저장 한)스코프를 반환한다.

```java
public abstract class ViewModel {
    @Nullable
    private final Map<String, Object> mBagOfTags = new HashMap<>();
    @Nullable
    private final Set<Closeable> mCloseables = new LinkedHashSet<>();
    private volatile boolean mCleared = false;

    // ...

    <T> T setTagIfAbsent(String key, T newValue) {
        T previous;
        synchronized (mBagOfTags) {
            previous = (T) mBagOfTags.get(key);
            if (previous == null) {
                mBagOfTags.put(key, newValue);
            }
        }
        T result = previous == null ? newValue : previous;
        if (mCleared) {
            closeWithRuntimeException(result);
        }
        return result;
    }

    <T> T getTag(String key) {
        if (mBagOfTags == null) {
            return null;
        }
        synchronized (mBagOfTags) {
            return (T) mBagOfTags.get(key);
        }
    }
}
```

좀 더 자세히 살펴보자.

위 코드는 `ViewModel`의 구현 부분의 일부이다. ViewModel은 추상 클래스로 되어 있으며, `mBagOfTags`와 `mCloseables`, `mCleared` 멤버 속성을 가지고 있다.

ViewModelScope에서 `setTagIfAbsent`에 `JOB_KEY`라는 상수 값과 `SupervisorJob() + Dispatchers.Main.immediate`를 매개변수로 넘겨주는 부분이 있었다.

`getTag`는 `mBagOfTags`에 key로 저장된 값이 없다면 null을, 있다면 해당 값을 반환한다. `setTagIfAbsent`는 key로 저장된 값이 없다면 newValue를 저장하고, 있다면 newValue는 무시된다. 그리고 저장된 값이 있었다면 저장되어 있던 값을, 없었다면 newValue를 result에 담아 반환하는데 그 전에 뷰모델이 clear 된 경우 result를 정리 후 반환하게 된다.

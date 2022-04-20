---
layout: post
title: Effective Kotlin_아이템5. 예외를 활용해 코드에 제한을 걸어라
subtitle: 1장. 안정성 - 아이템5. 예외를 활용해 코드에 제한을 걸어라
categories: Book
tags: [book, effective_kotlin]
---

![이펙티브 코틀린 북커버](/assets/images/EffectiveKotlinCover.png)

[내용 요약(Notion)][notion]

## TIL (Today I Learned)
코틀린에서 코드의 동작에 제한을 걸 때 사용할 수 있는 방법들


## 오늘 읽은 범위
- 1부. 좋은 코드
  - 1장. 안정성
    - 아이템5. 예외를 활용해 코드에 제한을 걸어라

## Content
### 아규먼트
---

`require` 함수를 사용한다. `require` 함수는 제한을 확인하고, 제한을 만족하지 못할 경우 예외를 throw한다.

```kotlin
fun factorial(n: Int): Long {
	require(n >= 0)
	return if (n <= 1) 1 else factorial(n - 1) * n
}
...
```

위 코드와 같이 입력 유효성 검사 코드는 함수의 가장 앞 부분에 배치되므로, 쉽게 확인할 수 있다.

`require` 함수는 조건을 만족하지 못 할 때 무조건적으로 `IlleagalArgumentException`을 발생시키므로 제한을 무시할 수 없다.

### 상태
---

- 어떤 객체가 미리 초기화되어 있어야만 처리할 수 있는 함수
- 사용자가 로그인했을 때만 처리할 수 있는 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

위와 같이 어떤 구체적인 조건을 만족할 때만 함수를 사용하게 제한하고 싶다면 `check` 함수로 상태를 제한할 수 있다.

```kotlin
fun speak(text: String) {
	check(isInitialized)
	// ...
}

fun getUserInfo(): UserInfo {
	checkNotNull(token)
	// ...
}

fun next(): T {
	check(isOpen)
	// ...
}
```

`check` 함수는 `require`와 비슷하지만 지정된 예측을 만족하지 못할 때 `IllegalStateException`을 throw한다.

함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 `require` 블록 뒤에 배치하고 `check`를 뒤에 한다.

### Assert 계열 함수 사용
---

함수를 잘못 구현했거나 누군가 변경해서 제대로 작동하지 않는 경우를 예방하려면 단위 테스트를 사용하는 것이 좋다.

```kotlin
class StackTest {
	@Test
	fun 'Stack pops correct number of elements'() {
		val stack = Stack(20) { it }
		val ret = stack.pop(10)
		assertEquals(10, ret.size)
	}
	// ...
}
```

위 코드는 크기가 20인 스택에서 10개의 요소를 pop 했을 때 10개의 요소가 나오는지 테스트한다. 만약 모든 pop 호출 위치에서 제대로 동작하는지 확인해보기 위해선 다음과 같이 pop 함수 내부에서 Assert 계열의 함수를 사용할 수 있다.

```kotlin
fun pop(num: Int = 1): List<T> {
	//...
	assert(ret.size == num)
	return ret
}
```

위와 같은 테스트 코드는 테스트 할 때만 활성화되기 때문에 프로덕션 환경에서는 오류가 발생하지 않는다. 만약 심각한 오류나 결과를 초래할 수 있다면 `check`를 사용하는 것이 좋다.

단위 테스트 대신 함수에서 `assert`를 사용했을 때 장점

- `Assert` 계열의 함수는 코드를 자체 점검하며 더 효율적으로 테스트할 수 있게 해준다
- 특정 상황이 아닌 모든 상황에 대한 테스트를 할 수 있다
- 실행 시점에 정확하게 어떻게 되는지 확인할 수 있다
- 실제 코드가 더 빠른 시점에 실패하게 만든다. 다라서 예상하지 못 한 동작이 언제 어디서 실행되었는지 쉽게 찾을 수 있다

*참고로 이를 활용해도 여전히 단위 테스트는 따로 작성해야 한다*

사실 `assert`는 파이썬에서 주로 사용하고, 자바에서 주로 사용하진 않는다. 코틀린에서는 코드를 안정적으로 만들고 싶을 때 양념처럼 사용할 수 있다.

### nullability와 스마트 캐스팅
---

코틀린에서 `require`와 `check` 블록으로 어떤 조건을 확인해서 true가 나왔다면, 해당 조건은 이후로도 true일 거라고 가정한다. 따라서 이를 활용해서 타입 비교를 한다면, 스마트 캐스트가 작동한다.

null 확인

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
	require(person.email != null)
	val email: String = person.email
	// ...
}
```

`requireNotNull`, `checkNotNull` 사용

```kotlin
class Person(val email: String?)
fun validateEmail(email: String) { /*...*/ }

fun sendEmail(person: Person, text: String) {
	val email = requireNotNull(person.email)
	validateEmail(email)
	// ...
}

fun sendEmail(person: Person, text: String) {
	requireNotNull(person.email)
	validateEmail(person.email)
	// ...
}
```

nullability를 목적으로 **Elvis 연산자** 활용

```kotlin
fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: return
	// ...
}
```

프로퍼티에 문제가 있어서 `null`일 때 `run` 함수와 조합

```kotlin
fun sendEmail(person: Person, text: String) {
	val email: String = person.email ?: run {
		log("Email not sent, no email address")
		return
	}
	// ...
}
```

## 정리

이번 내용에서 얻을 수 있는 이점

- 제한을 훨씬 더 쉽게 확인할 수 있다
- 애플리케이션을 더 안정적으로 지킬 수 있다
- 코드를 잘못 쓰는 상황을 막을 수 있다
- 스마트 캐스팅을 활용할 수 있다

이를 위해 활용한 매커니즘

- `require`블록: 아큐먼트 제한
- `check` 블록: 상태와 관련된 동작을 제한
- `assert`블록: 어떤 것이 true인지 확인 가능. `assert` 블록은 테스트 모드에서만 작동
- `return` 또는 `throw`와 함께 활용하는 `Elvis` 연산자


[notion]: https://mangbaam.notion.site/5-9945572b0027499986e6b6cb3bd2ef7a
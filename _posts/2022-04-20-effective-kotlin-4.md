---
layout: post
title: Effective Kotlin_아이템4. inferred 타입으로 리턴하지 말라
subtitle: 1장. 안정성 - 아이템4. inferred 타입으로 리턴하지 말라
categories: Book
tags: [book, effective_kotlin]
---

![이펙티브 코틀린 북커버](/assets/images/EffectiveKotlinCover.png)

[내용 요약(Notion)][notion]

## TIL (Today I Learned)


## 오늘 읽은 범위
- 1부. 좋은 코드
  - 1장. 안정성
    - 아이템 4: inferred 타입으로 리턴하지 말라

## Content

할당 때 inferred 타입은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다. 절대 슈퍼클래스 또는 인터페이스로는 설정되지 않는다.

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal = Zebra()
	animal = Animal() // 오류: Type mismatch
}
```

이런 경우 타입을 명시적으로 지정해 해결할 수 있다

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
	var animal:Animal = Zebra()
	animal = Animal()
}
```

하지만 직접 라이브러리(혹은 모듈)를 조작할 수 없는 경우 간단하게 해결하기 힘들다. 그리고 이런 경우 inferred 타입을 노출하면 위험한 일이 발생할 수 있다.

```kotlin
interface CarFactory {
	fun produce(): Car
}
```

```kotlin
val DEFAULT_CAR: Car = Fiat126P() // 지정하지 않았을 때 디폴트로 생성되는 자동차

interface CarFactory {
	fun produce() = DEFAULT_CAR
}
```

DEFAULT_CAR는 Car로 명시적으로 지정되어 있으므로 따로 필요 없다고 판단해서 함수의 리턴 타입을 제거한 경우이다.

이 경우 DEFAULT_CAR는 타입 추론에 의해 자동으로 타입이 지정될 것이므로, Car를 명시적으로 지정하지 않아도 된다고 생각해서 다음과 같이 코드를 변경할 위험이 있다.

```kotlin
val DEFAULT_CAR = Fiat126P()
```

그러면 CarFactory에서는 이제 Fiat126P 이외의 자동차를 생산하지 못하게 된다.

외부 API라면 문제를 쉽게 해결하기 어렵다.

따라서 리턴 타입은 외부에서 확인할 수 있게 명시적으로 지정해 주는 것이 좋다.

## 정리

-   타입을 확실하게 지정해야 하는 경우 명시적으로 타입을 지정해야 한다
-   안전을 위해 외부 API를 만들 때는 반드시 타입을 지정하고, 지정한 타입은 특별한 이유와 확인 없이 제거하지 않아야 한다

[notion]: https://mangbaam.notion.site/4-inferred-85608d0b29a14a86a9faebddfb1d8ba7
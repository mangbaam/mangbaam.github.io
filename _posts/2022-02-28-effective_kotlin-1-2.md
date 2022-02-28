---
layout: post
title: Effective Kotlin_1장. 안정성 / 아이템2. 변수의 스코프를 최소화하라
subtitle: 아이템2. 변수의 스코프를 최소화하라
categories: Book
tags: [book, effective_kotlin]
---

![클린 코드 책 커버](/assets/images/EffectiveKotlinCover.png)

[내용 요약(Notion)][notion]

## TIL (Today I Learned)
상태를 정의할 때 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

## 오늘 읽은 범위
- 1부. 좋은 코드
  - 1장. 안정성
    - 아이템2. 변수의 스코프를 최소화하라

## 책에서 기억하고 싶은 내용을 써보세요.
상태를 정의할 때는 변수와 프로터피의 스코프를 최소화하는 것이 좋다.
- 프로퍼티보다는 지역 변수를 사용
- 최대한 좁은 스코프를 갖게 변수를 사용
  - 반복문 내부에서만 변수가 상용된다면 변수를 반복문 내부에서 작성

" 스코프: 요소를 볼 수 있는 *컴퓨터 프로그래밍 영역*이다. 코틀린의 스코프는 기본적으로 중괄호로 만들어지며, 내부 스코프에서 외부 스코프에 있는 요소에만 접근할 수 있다.

```kotlin
// 나쁜 예
var user: User
for (i in users.indices) {
	user = users[i]
	print("User at $i is $user")
}

// 조금 더 좋은 예
for (i in users.indices) {
	val user = users[i]
	print("User at $i is $user")
}

// 제일 좋은 예
for ((i, user) in users.withIndex()) {
	print("User at $i is $user")
}
```
- 첫 번째 예에서 변수 user는 for 반복문 스코프 내부뿐만 아니라 외부에서도 사용할 수 있다.


#### 스코프를 좁게 만들어야 하는 이유
1. 프로그램을 추적하고 관리하기 쉽다
2. 스코프 범위가 너무 넓으며 다른 개발자에 의해서 변수가 잘못 사용될 수 있다.

#### 변수는 정의할 때 초기화되는 것이 좋다.
```kotlin
// 나쁜 예
val user: User
if (hasValue) {
    user = getValue()
} else {
    user = User()
}

// 조금 더 좋은 예
val user: User = if(hasValue) {
    getValue()
} else {
    User()
}
```

#### 여러 프로퍼티를 한꺼번에 설정해야 하는 경우에는 구조분해 선언(destructuring declaration)을 활용하는 것이 좋다.

```kotlin
// 나쁜 예
fun updateWeather(degress: Int) {
    val description: String
    val color: Int
    if (degress < 5) {
        description = "cold"
        color = Color.BLUE
    } else if (degress < 23) {
        description = "mild"
        color = Color.YELLOW
    } else {
        description = "hot"
        color = Color.RED
    }
    // ...
}

// 조금 더 좋은 예
fun updateWeather(degress: Int) {
    val (description, color) = when {
        degress < 5 -> "cold" to Color.BLUE
        degress < 23 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
    // ...
}
```


[result1]: https://user-images.githubusercontent.com/44221447/155886112-7308d5c6-c4b4-4524-896f-29bd6c6f35f0.png
[result2]: https://user-images.githubusercontent.com/44221447/155886153-09874237-e883-45e4-8cac-d5743276e568.png
[result3]: https://user-images.githubusercontent.com/44221447/155977107-8056ad6d-6faa-4eeb-88b8-95258390ac88.png
### 캡처링
다음은 ***에라토스테네스의 체***를 구현하는 방법이다.

1. 2부터 시작하는 숫자 리스트를 만든다.
2. 첫 번째 요소를 선택한다. 이는 소수이다.
3. 남아 잇는 숫자 중에서 2번에서 선택한 소수로 나눌 수 있는 모든 숫자를 제거한다.

##### 간단한 구현
```kotlin
// 간단한 구현
var numbers = (2..100).toList()
val primes = mutableListOf<Int>()
while (numbers.isNotEmpty()) {
    val prime = numbers.first()
    primes.add(prime)
    numbers = numbers.filter { it % prime != 0 }
}
print(primes)
```
![result1]

##### 시퀀스 구현
```kotlin
// 시퀀스 구현
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }
    
    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}
print(primes.take(10).toList()) 
```
![result2]

만약 위 코드에서 `prime`을 `var`로 선언하고 반복문 내부에서 계속 생성하는 것이 아니라, 반복문에 진입하기 전에 한 번만 생성하는 형태로 짜면 다음과 같다.

##### 잘못된 시퀀스 구현
```kotlin
// 잘못된 스코프 설정
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }
    
    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}
print(primes.take(10).toList())
```
![result3]

하지만 실행 결과는 위와 같이 이상하게 나온다.

그 이유는 `prime`라는 변수를 **캡처**했기 때문이다. 반복문 내부에서 `filter`를 활용해서 `prime`으로 나눌 수 있는 숫자를 필터링한다. 그런데 시퀀스를 활용하므로 필터링이 지연되고, 따라서 최종적인 `prime`값으로만 필터링된 것이다.

`prime`이 2로 설정되어 있을 때 필터링된 4를 제외하면, `drop`만 동작하므로 그냥 연속된 숫자가 나와버린 것이다.

가변성을 피하고 스코프 범위를 좁게 만들면 이런 문제를 간단하게 피할 수 있다.

### 정리
- 여러 이유로 변수의 스코프는 좁게 만들어서 활용하는 것이 좋다.
- `var`보다는 `val`을 사용하는 것이 좋다.
- 람다에서는 변수를 캡처한다.


## 오늘 읽은 소감은? 떠오르는 생각을 가볍게 적어보세요
변수나 프로퍼티의 스코프를 작게 만드는 습관은 오래 전부터 들여와서 이번 챕터의 내용을 받아들이는데 어렵지 않았다.

그러나 (캡처링, 시퀀스와 같은) 몇몇 생소한 개념들이 등장해 별도의 학습이 필요했다.

## 궁금한 내용이 있거나, 잘 이해되지 않는 내용이 있다면 적어보세요.
이번 챕터의 마지막 부분에 등장한 캡처링은 처음 들어본 개념이었다. 더군다나 이 캡처링 상황을 설명하는 시퀀스 개념도 처음 들어본 개념이어서 관련된 내용 전체를 이해하기 힘들었다.

### 캡처링
자바의 람다에서 접근할 수 있는 변수는 **지역 변수, 멤버 변수, static 변수**가 있다. 

함수의 지역 변수의 경우 **stack 메모리 영역**에 위치하고, 함수가 종료되면 stack에서 메모리를 날려버린다.(정확히는 해당 주소를 참조할 수 없게 된다.) 그렇기 때문에 람다 내부에서 지역 변수에 접근하기 위해서는 `final`이어야 접근할 수 있다 (자바에서).

> `final`로 선언된 상수 값은 stack이 아닌 heap에 저장되어서 괜찮은건가? 궁금증이 생겨서 찾아보니 [스택오버플로우]에서 원하는 답을 찾을 수 있었다.
>
>> `final` variable also stored in stack but the copy that variable which a inner class have stored in `heap`.

이처럼 람다 안에서 람다 외부에 있는 지역 변수를 사용하는 것을 ***lambda capturing***이라고 한다.

하지만 코틀린의 경우엔 꼭 `final`이 아니더라도 제약없이 사용할 수 있다.

`final`로 선언된 경우 자바에서와 동일하게 값을 복사하여 람다 내부에서 사용할 수 있고, `final`이 아니라면 특별한 `wrapper` 클래스로 감싸 그 `wrapper`를 `final` 변수에 담아 람다 내에서 사용한다.

### 시퀀스
시퀀스에 대한 내용을 이 곳에서 다루기에는 너무 방대한 양이라 여기서는 간단한 특징에 대해 설명하고자 한다.

Kotlin의 `sequence`는 Java8의 `stream`에 대응되는 개념으로 **lazy evaluation**으로 처리된다. (이에 반해 `Collection`은 연산에 대해 **eager evaluation**으로 처리한다)

`Collection`을 chain call 할 때는 중간 컬렉션(intermediate collection)이 생성된다. 하지만 `sequence`는 중간 결과가 컬렉션이 아닌 first collection에 대한 reference와 어떠한 연산이 수행되는지 저장해둔 sequence object가 반환된다. 그리고 결과가 필요한 시점에 연산을 수행하여 최종 결과만을 반환한다.


[위 코드](#시퀀스-구현)에서 `generateSequence`로 2부터 시작하는 자연수 목록이 만들어지고, `take(10)`을 수행하면서 10개의 결과 만을 계산할 수 있다. `yield`를 통해 값을 저장할 수 있다.



---
##### 참고 자료
시퀀스: https://medium.com/@mook2_y2/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%84%B0%EB%94%94-15-sequences-52cfca1805c8

final: https://stackoverflow.com/questions/29225745/where-is-the-local-final-variable-in-method-stored-stack-heap





[notion]: (https://www.notion.so/mangbaam/Effective-Kotlin-2a5acacaa11d443e8da3a00a75b85450#fda32eb08a564b7c825ca79bafb9aede)

[스택오버플로우]: https://stackoverflow.com/questions/29225745/where-is-the-local-final-variable-in-method-stored-stack-heap
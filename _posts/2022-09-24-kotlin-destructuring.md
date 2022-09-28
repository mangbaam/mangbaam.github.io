---
layout: post
title: Kotlin Array, List, Data Class destructuring and ComponentN max count
subtitle: 코틀린 Array, List, Data Class 의 구조분해와 ComponentN 사용
categories: Kotlin
tags: [kotlin]
---

## Destructuring (구조 분해)

[공식 문서](https://kotlinlang.org/docs/destructuring-declarations.html)

코틀린에서 `리스트`나 `배열`, `Data Class` 등의 값을 각각 변수에 담고 싶을 때 구조 분해를 사용할 수 있다.

```kotlin
fun main() {
	val li = listOf(1, 2, 3)
    val (n1, n2, n3) = li
    println("$n1, $n2, $n3") // 1, 2, 3
    
    val arr = charArrayOf('a', 'b', 'c')
    val (c1, c2, c3) = arr
    println("$c1, $c2, $c3") // a, b, c
    
    data class DoubleClass(val d1: Double, val d2: Double, val d3: Double)
    val dc = DoubleClass(3.2, 1.5, 5.43)
    val (d1, d2, d3) = dc
    println("$d1, $d2, $d3") // 3.2, 1.5, 5.43
}
```

파이썬을 사용해 본 사람이 있다면 `unpack` 과 비슷한 개념이라고 생각하면 될 것 같다. (파이썬과 다르게 코틀린에서는 괄호로 감싸야 함)

```kotlin
fun main() {
	val fruitPrice = mapOf("melon" to 10000, "banana" to 3000, "tomato" to 1000)
    for ((fruit, price) in fruitPrice) {
        println("$fruit 가격은 $price 원")
    }
    /*
    melon 가격은 10000 원
    banana 가격은 3000 원
    tomato 가격은 1000 원
    */
}
```

위와 같이 for 문으로 접근할 수도 있다. 코틀린에서 `Map` 은 각각의 **Key-Value** 셋을 `Pair` 로 관리한다.

이 말은 즉, `Pair` 도 구조 분해가 가능하고, 비슷한 타입인 `Triple` 역시 구조 분해가 가능하다.

```kotlin
fun main() {
	val science = Pair("science", 100)
    val pe = Triple("pe", 100, 80)
    
    val (subject1, score) = science
    val (subject2, testScore, activityScore) = pe
    
    println("$subject1 점수는 $score 점") // science 점수는 100 점
    println("$subject2 시험 점수는 $testScore, 활동 점수는 $activityScore 점") // pe 시험 점수는 100, 활동 점수는 80 점
}
```

```kotlin
fun main() {
    val pe = Triple("pe", 100, 80)
    
    val (subject2, _, activityScore) = pe
    
    println("$subject2 활동 점수는 $activityScore 점") // pe 활동 점수는 80 점
}
```

사용하지 않을 값은 `_` 로 하면 값을 구조분해 하지 않고 넘어간다.

그 외에도 `.map` 같은 확장 함수에서도 활용할 수 있다.

## 구조 분해 최대 개수

구조 분해도 상황에 따라 무한히 할 수 있는 것은 아니다.

- Pair - 2 개
- Triple - 3 개
- List, Array - 5 개
- Data Class - 제한 없음

`Pair` 나 `Triple` 은 당연한 말이고, `List`, `Array`, `Data Class` 는 다른 개수를 가지고 있다.

내부를 확인해보자

## Data Class - ComponentN

`Data Class` 를 만들어서 [자바 코드로 확인](#코틀린-코드를-자바-코드로-decompile-하기)해보자.

```kotlin
data class Person(
    val name: String,
    val age: Int,
    val married: Boolean
)
```

![image](https://user-images.githubusercontent.com/44221447/192093490-1818591e-8123-4d7f-ba66-be5c14cf0a9e.png)

디컴파일 결과 다양한 내용이 있지만 표시해 둔 `component1, 2, 3` 이 있는 것을 볼 수 있다. 그리고 각각은 멤버 변수가 선언된 순서대로 각각을 반환하는 역할을 한다.

그렇다면 이것을 몇 개 까지 만들 수 있을까?

궁금해서 많은 수의 멤버 변수를 가지는 `Data Class` 를 만들어 보았다.

![image](https://user-images.githubusercontent.com/44221447/192093593-ef687f99-12aa-40e5-a21e-59ad2c66c2bb.png)

...

![image](https://user-images.githubusercontent.com/44221447/192093597-3d2955ac-0f65-4473-a4b4-65deec8d61d5.png)

총 510 개의 멤버 변수를 가지고 있다. 디컴파일 해보자.

![image](https://user-images.githubusercontent.com/44221447/192093738-44604350-4b8a-47a5-b651-5c90a8b70a14.png)

...

![image](https://user-images.githubusercontent.com/44221447/192093726-b2916979-4d31-4b82-8428-3190bc482876.png)

예상했던대로(?) 선언한 510 개 모두에 대해서 `componentN()` 메서드가 생긴다. 그리고 좀 더 알아본 결과 시스템이 허용하는 한 무제한 생길 수 있다.

왜 갑자기 `componentN` 이야기를 꺼냈냐면 `componentN 이 바로 구조 분해를 하기 위한 메서드`이기 때문이다.

확인해보자.

```kotlin
data class Person(
    val name: String,
    val age: Int,
    val married: Boolean
)

fun personTest() {
    val me = Person("mangbaam", 27, false)
    val (name, age, married) = me
}
```

위와 같이 코드를 작성하고 디컴파일 해보면 다음과 같이 나온다.

![image](https://user-images.githubusercontent.com/44221447/192093888-f08c9d94-511a-4dcf-bb0c-bd86ad216e83.png)

`name` 과 `age` 가 `var1`, `var2` 로 표현되긴 했지만 `me.component1, 2, 3` 을 통해서 값을 할당하는 것을 확인할 수 있다.

```kotlin
val me = Person("mangbaam", 27, false)
me.component1()
me.component2()
me.component3()
```

`componentN` 은 `public` 메서드이기 때문에 위와 같이 직접 접근도 가능하다.

> 구조 분해는 componentN() 을 통해서 이루어진다

### (보너스) Pair와 Triple 이 구조 분해가 가능한 이유

![image](https://user-images.githubusercontent.com/44221447/192094757-13f65468-f4c7-4939-8f9d-869205bfe78b.png)

![image](https://user-images.githubusercontent.com/44221447/192094786-d4e076f9-e0b4-4f82-b949-b0e0eef12138.png)

`Pair` 와 `Triple` 의 구현체는 `Data Class` 이다. 그래서 당연하게도 구조 분해가 가능한 것이다.

## List, Array - ComponentN

![image](https://user-images.githubusercontent.com/44221447/192093157-406e2870-d7a5-45ba-ab2f-ca0689dba71f.png)

![image](https://user-images.githubusercontent.com/44221447/192093192-fc0f3fe0-c8b6-41a9-a578-326f037fa185.png)

`List` 와 `Arr` 모두 `Data Class` 와 마찬가지로 `componentN()` 가 있다.

그런데 이상한 점은 `li`, `arr` 모두 값을 8개나 가지고 있는데 `component`는 5개만 존재한다.

![image](https://user-images.githubusercontent.com/44221447/192094229-38f5f19a-073c-4b0d-bec4-9b8db1fc4c0a.png)

그래서 구조 분해를 시도하니 위에서 확인했던 것과는 다르게 에러가 발생하고 있는 것을 볼 수 있다.

에러 내용을 보면 `component6()`, `component7()`, `component8()` 이 없다는 것 같다.

### 디컴파일 해보자

```kotlin
val li = listOf(1, 2, 3, 4, 5, 6, 7, 8)

val (n1, n2, n3, n4, n5) = li
```

컴파일 에러가 발생하면 디컴파일을 할 수 없기 때문에 구조 분해 개수를 줄이고 디컴파일을 해보았다.

![image](https://user-images.githubusercontent.com/44221447/192094372-d21a91ab-764b-488e-84b7-e2044e15e86a.png)

이상한 점이 있다. 분명 `Data Class` 를 디컴파일 했을 때는 `componentN()` 으로 값을 할당하고 있었는데 `List` 를 구조 분해 해보니 `get()` 을 통해서 값을 할당하고 있다. 하지만 분명 `component1()` 과 같이 접근할 수는 있었다.

### 사실은 확장함수

![image](https://user-images.githubusercontent.com/44221447/192094493-0a7863fd-3e9d-48d8-9efe-11e071ff3177.png)

![image](https://user-images.githubusercontent.com/44221447/192094525-ef35e316-2c8e-4033-ba6f-ba52e1e154da.png)

사실 List 나 Array 는 자체적으로 componentN 메서드를 가지고 있는 것이 아니라 확장 함수로써 가지고 있다.

그리고 그 내용은 `get()` 을 통해서 가져오고 있기 때문에 디컴파일 해보면 `get()` 으로 나타나고 있던 것이다.

확장 함수가 정의된 곳은 **List** 는 `_Collections.kt`, **Array** 는 `_Arrays.kt` 이다.

`Data Class` 와 `List`, `Array` 가 다르게 작성된 이유는 아마도 `List` 와 `Array` 는 기존에 자바에서 제공되는 것을 사용하기 때문이고, `Data class` 는 코틀린에서 새롭게 설계되었기 때문에 더 유연한 것이 아닌가 생각한다.

## componentN 직접 만들기

`List`와 `Array` 에서 기본으로 만들어지는 `componentN` 은 1 ~ 5 이지만 더 많은 값을 구조 분해 하고싶은 경우가 있을 것이다.

코틀린 기본 라이브러리에서 1 ~ 5 번을 만든 것처럼 우리도 확장 함수를 사용해서 만들어볼 수 있다.

```kotlin
inline operator fun <T> List<T>.component6() = this[5]
operator fun <T> List<T>.component7() = this[6]
```

두 가지 방식으로 만들어 보았다. (`inline` 의 유무)

그리고 구조 분해를 해보자.

![image](https://user-images.githubusercontent.com/44221447/192095102-40b65b89-76c4-49ec-ae37-5edb37b9e788.png)

에러가 발생하지 않는다. 그럼 구조 분해를 해보자.

![image](https://user-images.githubusercontent.com/44221447/192095236-abf072b4-d952-45e8-944d-8a11aefceca9.png)

구조 분해 된 코드에서 var6(원래 코드에서 n6) 와 n7 의 차이를 확인해보자.

`var6`는 앞선 동작과 같이 `get()`을 통해서 값을 가져오지만 `var7`는 `component7()`을 호출하면서 값을 가져온다.

약간 주제에서 벗어난 이야기일 수 있지만 `inline` 키워드를 붙이면 함수를 호출하는 것이 아니라 함수에 정의된 기능을 코드에 직접 삽입하는 방식으로 이루어진다. 참조가 줄어들기 때문에 시스템 리소스 낭비를 줄일 수 있는 경우도 생긴다.

![image](https://user-images.githubusercontent.com/44221447/192095286-6db73474-0cdb-4aba-8840-75bdc697cef5.png)

이번에는 똑같이 만드는데 자연스럽게 붙였던 `operator` 키워드를 빼고 만들어 보았다.

보이는 것과 같이 컴파일 에러가 뜨면서 `operator` 를 붙이라는 힌트를 보여준다.

구조 분해를 위해서는 `operator` 키워드가 필요하기 때문이다.

원하는 만큼 추가해서 사용해보자.

---

## 코틀린 코드를 자바 코드로 Decompile 하기

1. 코틀린 코드를 작성하고 [Tools] - [Kotlin] - [Show Kotlin Bytecode] 를 선택한다

    ![image](https://user-images.githubusercontent.com/44221447/192093291-176ca852-2d78-4ab0-ae56-fec26ed0429c.png)

2. 오른쪽 창에 바이트 코드가 나타난다

    ![image](https://user-images.githubusercontent.com/44221447/192093318-8b306119-9796-48e8-b3c7-ef8a4af683f7.png)

3. 바이트 코드 창에서 상단에 있는 [Decompile] 버튼을 누른다. 그러면 새 창으로 디컴파일 된 자바 코드가 보인다.

    ![image](https://user-images.githubusercontent.com/44221447/192093356-94eb6d93-dea5-4b53-89c1-fa4d4dc37173.png)

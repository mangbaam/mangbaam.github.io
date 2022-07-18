---
layout: post
title: How to sort List<Pair<Int, Int>> in kotlin
subtitle: Kotlin 여러 기준으로 정렬하는 법
categories: Kotlin
tags: [basic,tip,kotlin]
---

## ⭐

이번에 `List<Pair<Int, Int>>` 타입을 정렬해 줄 경우가 생겨서 알아보게 된 내용이다. 정확히는 `Map<Int, Int>` 자료 구조를 value - key 순으로 정렬하고자 했는데 Map의 아이템으로 Pair가 사용되기 때문에 같은 내용으로 생각해도 될 것 같다.

Pair는 2개의 아이템이 하나의 쌍으로 되어있는 자료 구조인데, 첫 번째 아이템을 first, 두 번째 아이템을 second 라고 하자. (`first`, `second`로 각각 첫 번째, 두 번째 아이템에 접근할 수 있다)

이때 `second`를 기준으로 정렬 후 `second` 값이 같은 것끼리는 `first`를 기준으로 정렬하고자 한다.

## sortedWith

---

많이 사용하는 `sortedBy`와 달리 `sortedWith`는 **comparator**를 지정해서 다중 조건을 둘 수 있다는 장점이 있다. 이러한 특징 때문에 Pair의 두 아이템을 각각 기준으로 만들어 주기 위해서 `sortedWith`를 사용했다.

방법은 간단하다.

```kotlin
fun main() {
    val simpleMap = mutableMapOf<Int, Int>(
        1 to 10,
        2 to 20,
        3 to 30,
        4 to 30,
        5 to 30
    )
    val sortedMap = simpleMap.toList().sortedWith(compareBy({ it.second }, { it.first }))
    println(sortedMap) // [(1, 10), (2, 20), (3, 30), (4, 30), (5, 30)]
}
```

정렬된 값 중 가장 큰 값을 구하는 방법은 다음과 같다.

```kotlin
fun main() {
    val simpleMap = mutableMapOf<Int, Int>(
        1 to 10,
        2 to 20,
        3 to 30,
        4 to 30,
        5 to 30
    )
    val largestMap = simpleMap.toList().sortedWith(compareBy({ it.second }, { it.first })).last()
    println(largestMap) // (5, 30)
}
```

그냥 마지막 아이템만 취해주면 된다.

내림차순으로 정렬하기 위한 방법은 다음과 같다.

```kotlin
fun main() {
    val simpleMap = mutableMapOf<Int, Int>(
        1 to 10,
        2 to 20,
        3 to 30,
        4 to 30,
        5 to 30
    )
    val reverseSortedMap = simpleMap.toList().sortedWith(compareBy({ it.second }, { it.first })).asReversed()
    println(reverseSortedMap) // [(5, 30), (4, 30), (3, 30), (2, 20), (1, 10)]
}
```

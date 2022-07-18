---
layout: post
title: How to get longest continuous list in kotlin
subtitle: 코틀린에서 가장 긴 연속된 리스트 구하기 with fold
categories: Kotlin
tags: [kotlin,tip,basic]
---

## ⭐

이번에 주어진 정수형 배열에서 가장 긴 연속된 배열을 찾아야하는 작업이 있어서 그 방법에 대해 찾아보게 되었다.

## fold vs reduce

---

1, 2, 3, 5, 7, 8, 9, 10, 11, 13 을 가진 리스트를 위 코드로 수행하면 1, 2, 3 / 5 / 7, 8, 9, 10, 11 / 13 으로 쪼갤 수 있다.

kotlin의 `fold`는 `reduce`와 비슷한 동작을 하는데 reduce는 초기 값을 받지 않는 반면 fold는 초기 값을 받게 된다.

*reduce*

```kotlin
public inline fun <S, T : S> Iterable<T>.reduce(
    operation: (S, T) -> S
): S
```

*fold*

```kotlin
public inline fun <T, R> Iterable<T>.fold(
    initial: R,
    operation: (R, T) -> R
): R
```

reduce는 초기 값을 받지 않는 대신 컬렉션의 **첫 번째 요소**를 초기 값으로 사용하고, 자연스럽게 반환 타입도 **컬렉션의 자료형**이 된다.

반면 fold는 초기 값을 받게 되면서 반환 타입은 **초기 값의 자료형**이 된다.

## do it

```kotlin
fun main() {
    val simpleList = mutableListOf(1, 2, 3, 5, 7, 8, 9, 10, 11, 13)
    val continuous = simpleList.fold(mutableListOf<MutableList<Int>>()) { acc, i ->
        if (acc.isEmpty() || acc.last().last() != i - 1) {
            acc.add(mutableListOf(i))
        } else acc.last().add(i)
        acc
    }
    println(continuous) // [[1, 2, 3], [5], [7, 8, 9, 10, 11], [13]]
}
```

fold의 초기 값으로 빈 이중 리스트를 줬다. 이때 acc는 초기 값으로 받은 이중 리스트를 의미하고, i는 simpleList의 요소를 순차 접근한다.

그래서 맨 처음에 acc가 비어있을 때와 앞선 요소가 연속되지 않은 경우 acc에 새 리스트에 i를 담은 것을 추가하고, 앞선 요소가 연속되는 경우는 acc의 마지막 리스트의 맨 끝에 i를 추가한다.

이 중에서 가장 긴 리스트를 구하고, 가장 긴 리스트가 여러 개 있다면 마지막 요소가 가장 큰 리스트를 구해보려면 다음과 같이 하면 된다.

```kotlin
fun main() {
    val simpleList = mutableListOf(1, 2, 3, 5, 7, 8, 9, 10, 11, 13)
    val longest = simpleList.fold(mutableListOf<MutableList<Int>>()) { acc, i ->
        if (acc.isEmpty() || acc.last().last() != i - 1) {
            acc.add(mutableListOf(i))
        } else acc.last().add(i)
        acc
    }.sortedWith(compareBy({ it.size }, { it.last() })).last()
    println(longest) // [7, 8, 9, 10, 11]
}
```

2개의 조건으로 정렬하는 방법은 [이전 게시글](https://mangbaam.github.io/kotlin/2022/07/18/kotlin-sortwith.html)에서 다뤘다.

크기와 마지막 요소를 기준으로 정렬한 후에 마지막 아이템(리스트)를 취하면 된다.

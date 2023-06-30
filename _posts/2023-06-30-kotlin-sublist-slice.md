---
layout: post
title: Kotlin subList vs slice
subtitle:
categories: Kotlin
tags: [kotlin,list,collections]
---

## subList

`kotlin.collections.List` 에 정의된 함수이다.

> Returns a view of the portion of this list between the specified [fromIndex](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html#kotlin.collections.List$subList(kotlin.Int,%20kotlin.Int)/fromIndex) (inclusive) and [toIndex](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html#kotlin.collections.List$subList(kotlin.Int,%20kotlin.Int)/toIndex) (exclusive). The returned list is backed by this list, so non-structural changes in the returned list are reflected in this list, and vice-versa.
>
> Structural changes in the base list make the behavior of the view undefined.

```text
리스트의 fromIndex부터 toIndex 이전까지의 뷰 일부를 반환한다.
반환된 리스트는 원래 리스트를 투영하기 때문에 반환된 리스트의 비구조적 변경은 원래 리스트에 반영되고, 원래 리스트의 비구조적 변경도 마찬가지로 반환된 리스트에 반영된다.

원래 리스트의 구조적 변경은 반환된 뷰가 제대로 동작하지 않도록 만들 수 있다.
```

## slice

`kotlin.collections`에 정의된 함수이다.

`kotlin.collections.collections`에도 정의되어 있다.

> Returns a list containing elements at indices in the specified indices range.

```text
리스트에서 주어진 범위(indices)만큼을 포함하는 리스트를 반환한다.
```

## 공통점

subList와 slice 모두 리스트의 일부를 가져오기 위해 사용한다.

<iframe id='frame' src='https://pl.kotl.in/VTn6ztyFw?from=2&to=7' frameborder='0' scrolling='no' style='width: 100%;' onload="this.style.height=(this.contentWindow.document.body.scrollHeight+20)+'px';"></iframe>

![](https://i.imgur.com/3hqJMOC.png)

## 차이점

### case1: 원래 (정수)리스트 수정

<iframe id='frame' src='https://pl.kotl.in/-K-YyShgC?from=2&to=13' frameborder='0' scrolling='no' style='width: 100%;' onload="this.style.height=(this.contentWindow.document.body.scrollHeight+20)+'px';"></iframe>

![](https://i.imgur.com/5SEEbDp.png)

원래 리스트의 1번 인덱스 값을 2 → 200 으로 변경했더니, `subList`는 변경이 반영되었고, `slice`는 반영되지 않았다.

### case2: 원래 (객체)리스트 수정

<iframe id='frame' src='https://pl.kotl.in/L-aJxIq3c?from=2&to=17' frameborder='0' scrolling='no' style='width: 100%;' onload="this.style.height=(this.contentWindow.document.body.scrollHeight+20)+'px';"></iframe>

![](https://i.imgur.com/JWNCjkG.png)

이번에는 원래 리스트의 1번 인덱스에 있는 데이터클래스의 프로퍼티를 변경했더니, `subList`와 `slice` 모두 변경이 반영되었다.

### 중간 결과

```kotlin
public fun <T> List<T>.slice(indices: IntRange): List<T> {  
	if (indices.isEmpty()) return listOf()  
	return this.subList(indices.start, indices.endInclusive + 1).toList()  
}
```

`slice`는 내부적으로 반환 전 마지막에 `asList()` 혹은 `toList()`를 사용한다.

```kotlin
public fun <T> Iterable<T>.toList(): List<T> {  
	if (this is Collection) {  
		return when (size) {  
			0 -> emptyList()  
			1 -> listOf(if (this is List) get(0) else iterator().next())  
			else -> this.toMutableList()  
		}  
	}  
	return this.toMutableList().optimizeReadOnlyList()  
}
```

`toList()`는 다시 `this.toMutableList()`를 반환한다.

```kotlin
public fun <T> Collection<T>.toMutableList(): MutableList<T> {  
	return ArrayList(this)  
}
```

`toMutableList()`는 결국 ArrayList의 생성자로 사용되며 결과적으로 **원래 리스트의 일부 영역을 얕은 복사 한 효과**를 가진다.

> 얕은 복사는 객체의 주소 값을 복사하는 것이기 때문에 리터럴인 정수는 원래 리스트에서 변경되어도 반영이 안되었지만, 데이터클래스를 사용한 리스트에서는 데이터클래스의 객체 주소 값이 복사되었기 때문에 원래 리스트와 `slice()` 한 리스트 모두 같은 객체를 참조하고 있는 것이다.

그렇다면, `subList()`는 어떻게 동작하길래 항상 변경이 반영되는 것일까?

주석을 살펴보면 그 이유를 알 수 있다.

예시를 통해 마저 알아보자.

### case3: 원래 리스트의 구조적 변경

<iframe id='frame' src='https://pl.kotl.in/HXxPnLyYD?from=2&to=18' frameborder='0' scrolling='no' style='width: 100%;' onload="this.style.height=(this.contentWindow.document.body.scrollHeight+20)+'px';"></iframe>

![](https://i.imgur.com/FAngFMi.png)

원래 리스트의 마지막 요소를 제거해보았다. (구조적 변경)

그랬더니 `slice`는 예상한대로 출력이 되지만, `subList`는 **ConcurrentModificationException**을 발생시킨다.

즉, `subList`는 원래 리스트를 참조하고 있고, 원래 리스트에 구조적인 변경이 발생하면 예외를 발생시킨다. 주석의 “뷰의 일부(view of the portion of this list)” 라는 표현이 이런 동작을 잘 설명하고 있는 것 같다.

## 결론

- `subList`는 **원래 리스트를 참조**하기 때문에 원래 리스트나 subList의 결과 리스트의 변경이 동기화되며, 원래 리스트에 구조적 변경이 발생되면 ConcurrentModificationException을 발생시킨다.
- `slice`는 원래 리스트의 **일부 아이템들을 얕은 복사**하기 때문에 원래 리스트의 구조적 변경에 영향을 받지 않는다.

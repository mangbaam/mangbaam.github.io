---
layout: post
title: Effective Kotlin_아이템3. 최대한 플랫폼 타입을 사용하지 말라
subtitle: 1장. 안정성 - 아이템3. 최대한 플랫폼 타입을 사용하지 말라
categories: Book
tags: [book, effective_kotlin]
---

![이펙티브 코틀린 북커버](/assets/images/EffectiveKotlinCover.png)

[내용 요약(Notion)][notion]

## TIL (Today I Learned)
상태를 정의할 때 변수와 프로퍼티의 스코프를 최소화하는 것이 좋다.

## 오늘 읽은 범위
- 1부. 좋은 코드
  - 1장. 안정성
    - 아이템3. 최대한 플랫폼 타입을 사용하지 말라

## Content

### null-safety 매커니즘이 없는 언어와 코틀린의 결합

> null-safety 매커니즘이 없는 자바, C 등의 프로그래밍 언어와 코틀린을 연결해서 사용할 때는 NPE가 발생할 수 있다.

만약 자바에서 String 타입을 리턴하는 메서드가 있다고 했을 때

- `@Nullable` 어노테이션이 붙어있다면 nullable로 추정 ->  `String?`으로 변경
- `@NotNull` 어노테이션이 붙어있다면 ->  `String`으로 변경
- 아무 어노테이션이 붙지 않았다면
  - 최대한 안전하게 접근한다면 -> nullable로 가정
  - `null`을 리턴하지 않는 것이 확실하다면 -> not-null 단정을 나타내는 `!!`을 붙인다


### 문제점

nullable과 관련하여 문제가 발생하는 부분은 자바의 **제네릭 타입**이다.

자바 API에서 `List<User>`를 리턴하고 어노테이션이 따로 붙지 않은 경우, <u>코틀린이 디폴트로 모든 타입을 nullable로 다룬다면</u> 우리는 이를 사용할 때 이러한 리스트와 리스트 내부의 `User` 객체들이 null이 아니라는 것을 알아야 한다. 따라서 리스트 자체만 null 확인하는 것이 아니라, 그 내부에 있는 것들도 null인지 확인해야 한다.

그러면 다음과 같이 null을 처리하는 코드를 작성해야 한다.

```java
// 자바
public class UserRepo {
	
	public List<User> getUsers() {
		// ...
	}
}
```
```kotlin
// 코틀린
val users: List<User> = UserRepo().users!!.filterNotNull()
```

만약 `List<List<User>>`를 리턴한다면 훨씬 복잡해질 것이다.

```kotlin
val users: List<List<User>> = UserRepo().groupedUsers!!.map { it!!.filterNotNull() }
```

List는 적어도 `map`과 `filterNotNull` 등의 메서드를 제공하지, 다른 제네릭 타입이라면 null을 확인하는 것 자체가 정말 복잡한 일이 된다.

그래서 코틀린은 자바 등의 <u>다른 프로그램이 언어에서 넘어온 타입</u>들을 특수하게 다룬다. 이러한 타입을 **플랫폼 타입**이라고 부른다.

### 플랫폼 타입
플랫폼 타입이란 다른 프로그래밍 언어에서 전달되어서 <u>nullable인지 아닌지 알 수 없는 타입</u>을 말한다.

플랫폼 타입은 `String!` 처럼 타입 이름 뒤에 `!` 기호를 붙여서 표기한다. 물론 이러한 어노테이션이 *직접적으로 코드에 나타나지는 않는다.* 

```java
// 자바
public class UserRepo {
	public User getUsers() {
		// ...
	}
}
```
``` kotlin
// 코틀린
val repo = UserRepo()
val user1 = repo.user        // user1의 타입은 User!
val user2: User = repo.user  // user2의 타입은 User
val user3: User? = repo.user // user3의 타입은 User?
```

코틀린에서 플랫폼 타입을 지원하므로 다음 코드와 같이 내부까지 null 체크를 하는 복잡한 코드를 작성하지 않아도 된다.

##### 플랫폼 타입 예제
```kotlin
val users = UserRepo().users
val users = UserRepo().groupedUsers
```

하지만 null일 가능성이 없어지는 것은 아니라서 여전히 플랫폼 타입을 사용할 때는 항상 주의를 기울여야 한다.

만약 자바 코드를 직접 조작할 수 있다면 가능한 `@Nullable`과 `@NotNull` 어노테이션을 사용하자.

### NPE(Null Point Exception)의 발생
```kotlin
// 자바
public class JavaClass {
    public String getValue() {
        return null;
    }
}

// 코틀린
fun statedType() {
    val value: String = JavaClass().value // NPE
    // ...
    println(value.length)
}

fun platformType() {
    val value = JavaClass.value
    // ...
    println(value.length) // NPE
}
```

자바로 쓰여진 `JavaClass` 라는 클래스의 `value`를 사용하는 코틀린 코드인데 둘 다 NPE가 발생한다. 하지만 NPE가 발생하는 위치가 다르다.

**statedType** 에서는 자바에서 <u>값을 가져오는 위치에서 NPE가 발생</u>한다. 이는 코드를 쉽게 수정할 수 있다.

**platformType**에서는 <u>값을 활용할 때 NPE가 발생</u>한다. 플랫폼 타입으로 지정된 변수는 nullable일 수도 있고, 아닐 수도 있기 때문에 변수를 한두 번 안전하게 사용했더라도 이후에 다른 사람이 사용할 때는 NPE가 발생될 수도 있다. 이런 문제는 타입 검사기에서 검출해 줄 수 없다. 그렇기에 오류를 찾는데 굉장히 오랜 시간이 걸리게 될 것이다.

## 정리
- 다른 프로그래밍 언어에서 와서 nullable 여부를 알 수 없는 타입을 **플랫폼 타입**이라고 부른다.

- 플랫폼  타입을 사용하는 코드는 해당 부분만 위험한 것이 아니라, 이를 활용하는 곳까지 영향을 줄 수 있는 위험한 코드이다.

- 따라서 이런 코드를 사용하고 있다면 빨리 해당 코드를 제거하는 것이 좋다.

- 또한 연결되어 있는 자바 생성자, 메서드, 필드에 nullable 여부를 지정하는 **어노테이션을 활용하자.**

---
## 추가

[이 부분](#플랫폼-타입-예제) 코드가 책에는 다음과 같이 되어 있었다.

```kotlin
val users: List<User> = UserRepo().users
val users: List<List<User>> = UserRepo().groupedUsers
```

책의 코드에서는 타입을 지정했는데, 이는 플랫폼 타입이 사용되는 예제로 적당하지 않다고 생각되어 자체적으로 코드를 수정하였다. (물론 위 코드와 같이 명시적으로 NotNull 타입을 지정한 경우에도 null 체크를 하지 않아도 된다.)



[notion]: https://mangbaam.notion.site/3-fc1532f883fe49c9bfc243ee78d2f454
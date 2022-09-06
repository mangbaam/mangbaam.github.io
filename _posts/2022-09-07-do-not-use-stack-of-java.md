---
layout: post
title: DO NOT USE java.util.Stack
subtitle: why does Stack extend Vector
categories: Java
tags: [java,kotlin]
---

## ⭐

[![image](https://user-images.githubusercontent.com/44221447/188686040-51a571bd-0868-4203-b845-26cdf1a1dc48.png)](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)

위 내용은 Stack 클래스의 JDoc 내용이다. 공식적으로 이 클래스를 사용하기 보다는 `Deque` 사용을 권장하고 있다.

그 이유와 대안에 대해서 알아본다.

## Stack 은 Vector 를 상속한다

---

![image](https://user-images.githubusercontent.com/44221447/188682512-22066dca-24a3-4ecc-838f-17f636ab0383.png)

`java.util.Stack` 의 구현부를 보면 `Vector` 를 상속하고 있는 것을 알 수 있다.

Stack 을 사용하지 않아야 하는 이유는 바로 여기에 있다.

그 중에서도 2 가지 이유로 나뉜다.

### 1. Vector 의 메서드를 사용할 수 있다

Stack 에서는 push, pop, peek, empty, search 메서드를 제공한다. 하지만 Stack 은 Vector 를 상속 받았기 때문에 이 5개의 메서드 외에도 Vector 에서 제공되는 여러 메서드들도 사용할 수 있다.

문제는 Stack 은 LIFO 구조를 유지해야 하지만 Vector 의 특성 상 인덱스를 사용해 중간의 값에 접근하거나 심지어는 조작할 수도 있는 상황이 발생한다.

![image](https://user-images.githubusercontent.com/44221447/188689719-2f3bd29a-1ca0-479c-a15c-0e02a4b692c3.png)

실제로 remove 나 indexOf 와 같은 메서드를 사용해도 ide 에서 전혀 에러 메시지를 표시하지 않는다. (정상적인 코드니까)

애초부터 Stack 은 잘못 설계된 클래스라는 것이다. 왜 Vector 를 상속하게 되었는지는 밑에서 살펴보자.

### 2. Vector 사용 자체를 지양해야 한다

Stack 을 사용하지 않아야 할 두 번째 이유는 기본적으로 Vector 처럼 동작하기 때문이다. 그리고 Vector 사용을 지양하기 때문이다.

Vector 와 ArrayList 의 차이점을 알고 있는가?

두 컬렉션의 차이는 thread-safe 여부이다. Vector 는 멀티스레드 환경에서 안전하게 사용하기 위해 등장했다.

![image](https://user-images.githubusercontent.com/44221447/188694000-422240c8-7682-475a-8451-8418ff1edc6d.png)

![image](https://user-images.githubusercontent.com/44221447/188694538-8c3c096b-63c7-4c4d-ad94-a413eaa0f120.png)

![image](https://user-images.githubusercontent.com/44221447/188694110-d0e05681-3c31-400b-9102-045a13c689c1.png)

실제로 add, get, remove 등의 메서드를 보면 모두 `synchronized` 키워드가 붙어있는 것을 확인할 수 있다.

그러면 멀티스레드 환경에서 Vector 를 사용하면 될까? - 음... 권장하지 않는다.

`synchronized` 키워드가 붙어 있으면 특정 상황에서 성능을 상당히 저하시킬 위험이 있다.

예를 들어 Vector 를 순차적으로 탐색한다고 했을 때 전체 과정 중 한 번만 locking 을 해도 될 것을 get() 이 호출되는 매 순간 locking 과정이 발생해 필요 없는 오버헤드가 발생하게 된다.

그럼 멀티스레드 환경에서 Vector 대신 어떤 것을 사용해야 할까? - 아래 메서드 중 하나를 사용하면 된다.

![image](https://user-images.githubusercontent.com/44221447/188697762-6ed13e35-f2fc-4ae7-a78b-df53bcf210a9.png)

위 메서드들은 `Collections` 에 포함되어 있는 메서드들이다. 이 `Collections.synchronizedXXX` 메서드를 사용하면 thread-safe 한 컬렉션을 만들 수 있다.

```kotlin
val li = Collections.synchronizedList(mutableListOf<T\>()) // li : (Mutable)List<String!>!
```

## Stack 은 왜 Vector 를 상속했을까

---

![image](https://user-images.githubusercontent.com/44221447/188702519-01a7e028-26d2-4448-83a9-2ed53e07a6cf.png)

![image](https://user-images.githubusercontent.com/44221447/188702567-f77fb66b-8798-4734-8db8-42767dbd98df.png)

`Stack`과 `Vector` 는 Java 의 첫 번째 릴리즈(Java 1.0)에 포함된 것들이다. Java 는 당시에 인터넷 열풍으로 인해 시장에 일찍 출시되었다.

이와 비슷하게 날짜 및 시간과 관련된 클래스의 초기 버전들도 잘 설계되지 않은 채 개발되었다.

![image](https://user-images.githubusercontent.com/44221447/188701777-2e3250b8-010c-4aa2-bfca-57a32f6d4796.png)

![image](https://user-images.githubusercontent.com/44221447/188702356-9e7a9f88-34f4-4e3e-b0d1-f9e6b856c9e5.png)

이후 [Java Collection 프레임워크](https://en.wikipedia.org/wiki/Java_collections_framework)에서 Vector 와 Stack 을 대체되었고, 마찬가지로 java.time 프레임워크는 java.util.Date, java.util.Calendar 와 같은 클래스로 대체되었지만 쉽게 없애버리지 못하는 이유는 기존 앱과의 하위 호환성을 위해서이다.

객체지향 프로그래밍에서 특히 더 설계에 주의해야 한다는 것을 잘 보여주는 사례인 것 같다.

## Stack 대신 어떤 것을 사용해야 할까

---

주석에 나와 있듯이 `ArrayDeque` 같은 Deque 관련 컬렉션을 사용하면 된다. 물론 Deque 의 특성 상 양쪽 끝에서 접근할 수 있지만 적어도 Stack 클래스와 같은 문제는 없다.

직접 만들어서 사용하는 방법도 좋을 것 같다.

만약 멀티스레드 환경에서 사용해야 한다면 `ConcurrentLinkedDeque` 을 사용하면 된다.

## 출처

---

### Thread-safe method

https://stackoverflow.com/questions/66169482/are-kotlin-mutable-collections-thread-safe

### Why Stack extends Vector in JDK

https://stackoverflow.com/questions/37314298/why-stack-extends-vector-in-jdk

### 자바에서 Vector 와 Stack 컬렉션이 쓰이지 않는 이유

https://aahc.tistory.com/8

### Java 의 Stack 대신 Deque

https://tecoble.techcourse.co.kr/post/2021-05-10-stack-vs-deque/

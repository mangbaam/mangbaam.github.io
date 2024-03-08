---
layout: post
title: 안드로이드에서 섹션 구조로 유연한 앱 만들기
subtitle: Android section structured app
categories: Android
tags: [android,architecture]
---

앱의 전시 영역을 섹션 구조로 변경하여 유연하게 만든 경험에 대해 기술하고자 합니다.

간단하게 섹션 구조에 대해 소개하고, 안드로이드에서는 어떻게 구현했는지 소개하겠습니다.

## 섹션 구조

사실 섹션이라는 용어가 업계에서 통상적으로 사용되는 용어인지는 모르겠지만, 아마도 자주 봤던 구성일 것입니다.

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/140f5cf5-2647-4ffa-9800-f1505354119a" />

### 헤더 영역

주로 섹션을 설명하는 타이틀과 더보기와 같은 액션 버튼이 들어가기도 합니다.

버튼은 생략될 수 있으며, 헤더 자체도 생략될 수 있습니다.

### 컨텐츠 영역

섹션의 컨텐츠가 들어가는 영역입니다. 컨텐츠는 다양한 방식으로 렌더링 될 수 있습니다. 예를 들어 1열 세로 스크롤로 배치될 수도 있고, 2행 가로 스크롤로 배치될 수도 있고, 그 형태는 다양합니다.

카테고리 같은 영역이 추가되는 경우도 있지만 컨텐츠 영역의 주된 컨텐츠들은 같은 타입을 가지는 경우가 많습니다.

### 푸터 영역

섹션의 보조적인 정보를 표시하거나 더보기와 같은 액션 버튼이 들어가기도 합니다.

주로 헤더에 들어가는 버튼보다 더 크게 표시하는 경우가 많이 때문에 사용자들이 더 많은 액션을 하기를 원할 때 활용하는 편입니다.

헤더와 마찬가지로 생략될 수 있습니다.

### 예시

실제 전시 영역을 가지는 여러 서비스에서 위에서 설명한 섹션 구조를 가지고 있는 것을 어렵지 않게 발견할 수 있습니다.

*푸터가 있는 경우*

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/0152de40-de91-4338-93f6-1b1fbc41ecd1" />

*헤더에 액션 버튼이 있는 경우*

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/c476abb1-fdb1-4fab-9eda-b6601e91c441" />

*푸터가 없는 경우*

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/a995359b-ba62-4aaa-a0f9-d01b36633595" />

이렇게 섹션 구조로 만들게 되면 여러 장점이 있지만 가장 유용한 점은 앱의 업데이트 없이 섹션 간의 순서를 바꾸거나, 미 노출 하거나 레이아웃을 바꿔보거나 AB 테스트에도 용이하다는 점이 있습니다.

## 섹션 구조 구상하기

섹션 구조로 변경하려면 앱(프론트), 디자인, 서버, cms(백오피스) 등 대부분의 파트에서 얼라인이 맞아야 합니다. 디자인은 적어도 섹션 간, 컨텐츠 간 간격 등을 일정하게 만들어 공통화 해야하고, 서버는 섹션 구조로 묶어서 내려줄 수 있어야 하며 cms 도 전시 영역을 원하는대로 조작할 수 있도록 지원되어야 합니다. 앱 역시 서버에서 내려주는 섹션 구조의 데이터를 적절히 조작해 렌더링 할 수 있어야 합니다.

## 안드로이드에서 섹션 다루기

섹션에서는 동일한 아이템 타입을 가진다고 생각하고 구성했기 때문에 제네릭을 사용해서 Section 클래스를 작성했습니다.

```kotlin
data class Section<T : Component>(
    val hasNext: Boolean? = null,
    val header: Header? = null,
    val option: String? = null,
    val items: List<T> = emptyList(),
    override val margin: Margin = Margin(),
    override val type: DisplayType = DisplayType.UNKNOWN,
    override val log: Log = emptyLog(),
) : Serializable, Component {
    data class Header(
        val id: String = "",
        val title: String = "",
        val desc: String? = null,
        val more: More? = null,
        override val margin: Margin = Margin(),
        override val type: DisplayType = DisplayType.UNKNOWN,
        override val log: Log = emptyLog(),
    ) : Component {
        data class More(
            val id: String,
            val title: String?,
            val desc: String?,
            val action: String?
        )
    }
}
```

여기서 짚고 넘어가야 할 부분이 몇 가지 있습니다.

우선 Section 은 `Component` 타입의 제네릭을 받도록 되어있습니다. 즉, 섹션의 컨텐츠로 올 수 있는 것은 `Component` 타입이며 Section 자체도 `Component` 입니다. (자세한 내용은 밑에서 다시 설명하겠습니다)

`Component` 는 아래와 같은 인터페이스 입니다.

```kotlin
interface Component : Loggable {
    val margin: Margin
    val type: DisplayType
}
```

margin 과 type 이 보이는데, margin 은 이 컴포넌트가 가질 마진을 의미합니다. 레이아웃이 변경될 때나 컴포넌트의 포지션에 따라 마진 값이 달라질 수 있기 때문이죠.

DisplayType 도 눈에 띕니다. DisplayType 은 앱을 더 풍성하게 만들어주기 위해 필요합니다. 섹션에 표시할 컨텐츠를 어떤 형태로 렌더링 할 지 결정하기 때문입니다.

컴포넌트의 타입으로 구분하면 되지 않느냐? 라고 할 수 있지만, **동일한 데이터 구조를 가지더라도 다양한 형태로 표현될 수 있기 때문에** 필요한 필드입니다.

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/57032513-4cb7-4424-86af-e3d9ab1b2fee" />

위 사진과 같이 동일한 데이터이지만 다양한 형태로 표현할 수 있습니다.

Section 도 Component 여야 하는 이유는 다음과 같습니다.

전시 영역은 이중 리스트가 될 수도 있습니다. 전시 영역은 세로 스크롤 되는 영역이며, 각 섹션에서는 items 라는 Component 리스트를 가질 수 있기 때문입니다.

대표적인 예가 섹션 안에서 가로 스크롤이 되는 경우입니다.

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/aa5ad6e8-e882-43e3-bfef-2f422fb1b0e9" />

반대로 섹션의 컴포넌트가 세로 스크롤이 된다면 하나의 섹션의 길이가 아주 길어질 수도 있습니다.

## 섹션 펼치기

안드로이드에서 구현한다고 하면 (컴포즈를 사용하지 않는다면) RecyclerView 로 구현하게 될텐데, 전시 영역을 Section 의 리스트로 구상하게 되면 문제가 발생합니다.

위에서 언급한대로 Secion 하나의 길이가 아주 길어질 수 있기 때문에 RecyclerView 의 재사용성을 사용할 수 없게 된다는 점이 있고, 스크롤이나  이벤트 처리, 로깅 등을 할 때도 아주 복잡해질 수 있습니다.

그래서 고민 후에 섹션으로 받은 데이터를 일부 펼치기로 했습니다. 사진과 함께 살펴보겠습니다.

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/15d510ea-5904-4952-a207-95d5d5971461" />

빨간색으로 표시한 부분이 Section 이고, 초록색으로 표시한 부분이 Section 에서 펼쳐낸 Component 입니다.

섹션으로 펼쳐내는 조건은 단순합니다. `리스트에서 단독으로 존재할 수 있는가`

- 맨 위의 추천태그 섹션의 경우 각 태그들이 가로 스크롤 되며, 타이틀을 가지고 있기 때문에 Section 이 더 적합합니다.
- 그림 문장은 단독으로 존재할 수 있기에 Section 으로부터 펼쳐졌습니다. (현재 서버에서는 Section 의 리스트로 내려주고 있기 때문에 원래는 Section 에 감싸진 형태였기에 펼쳤다고 표현했습니다.)
- 세 번째에 있는 Section 역시 3개의 문장이 (뷰 상으로) 연결되어 있고, 타이틀을 가지고 있기 때문에 Section 이 더 적합합니다.
- 마지막 문장들은 단독으로 존재할 수 있기 때문에 Section 에 감싸진 컨텐츠들을 펼쳤습니다.

## 어댑터 작성

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/5ac6e94b-b73c-4448-8747-c38b112c9032">

Component 타입을 받는 ComponentAdapter 를 만들었으며, 앞서 살펴본 `DisplayType` 에 따라 각각 뷰홀더를 작성하였습니다.

또한 각 뷰에서 필요한 옵션이나 리스너를 담고 있는 `xxxOption` 타입을 만들어서 다음과 같이 `ComponentAdapter` 를 생성했습니다.

<img width="80%" alt="image" src="https://github.com/Nexters/Boolti/assets/44221447/8a7e8b1e-607f-4d3f-9abd-973db4c6d8ae">

이렇게 만들어진 `ComponentAdapter` 는 섹션 구조를 사용하고 DisplayType 으로 정의할 수 있는 뷰로 구성된 화면이라면 어디서든 사용할 수 있었습니다.

## 마치며

이번 글에서 다룬 내용은 실제로는 생략된 부분이 정말 많지만 섹션 구조에 대한 이해가 생긴다면 구현 방법은 완전히 달라질 수도 있고, 팀 마다, 프로덕트 마다, 플랫폼 마다 더욱 적합한 방법이 모두 다를 수 있기 때문에 더 자세히 설명하기 보단 이 정도로 마무리 하려고 합니다.

섹션 구조로 변경하면서 정말 많은 고민을 했고, 다양한 시도를 해서 현재의 상태까지 도달했지만 아직 완성형이라는 생각은 아닙니다. 앞으로 더 개선할 부분도 있고, 발전시키고 싶은 부분들도 있기에 항상 조금 더 고민해보고 조금씩 발전시켜 나가고자 합니다.

추가로 ComponentAdapter 를 만들게 되면서, 거의 모든 화면에서 비슷한 컴포넌트를 노출하는 저희 서비스에서 개발 속도가 정말 많이 빨라졌고, 기존에 있던 문제점인 화면마다 동일한 컴포넌트이지만 조금씩 다른 디자인 디테일과 동작들을 모두 공통화해서 잡을 수도 있었습니다.

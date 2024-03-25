---
layout: post
title: Now In Android is not Clean Architecture
subtitle: Now In Android 는 클린아키텍처와 SOLID 를 따르지 않는다
categories: Architecture
tags: [android,architecture,solid]
---

최근 Now in android 의 Discussions 에서 [Now in android 가 클린 아키텍처와 SOLID 원칙을 따르지 않는다](https://github.com/android/nowinandroid/discussions/1273)는 논쟁이 화두에 올랐습니다.
이 논쟁에서 아키텍처를 비롯한 다양한 유용한 내용들이 등장해서 이를 정리해보고자 글을 작성해보았습니다.

*[원본](https://github.com/android/nowinandroid/discussions/1273) 내용이 길기 때문에 핵심 내용 위주로 요약했으며, 의역된 부분들이 원본의 의도를 담지 못했을 수도 있습니다. 자세한 내용은 [원본](https://github.com/android/nowinandroid/discussions/1273)을 참고해주시면 좋을 것 같습니다.*

## Now in android 는 클린 아키텍처와 SOLID를 따르지 않는다

#### Yazon2006

Now in android(이하 NIA)가 클린 아키텍처와 SOLID 원칙을 위반하는 것 같습니다. 특히 domain 계층이 data 계층에 의존하고 있기 때문에 순수한 비즈니스 로직이 아닙니다.
이로 인해 프로젝트 구조가  혼란스럽고, 잘못된 방향을 제공할 수도 있습니다.
domain 계층이 data 계층에 의존하도록 리팩터링하거나 README에 해명과 설명을 첨부해주십시오.

#### dturner (Maintainer)

안녕하세요 저는 Google Android DevRel 팀의 엔지니어이자 NIA의 테크 리더입니다. 좀 더 명확하게 설명해보겠습니다.

**클린 아키텍처를 사용하지 않는 이유:**

NIA는 [공식 아키텍처 가이드(](https://developer.android.com/topic/architecture/intro)테스트, UI 디자인 등)를 구현하고 있습니다. 공식 가이드는 두 가지 측면에서 "클린 아키텍처"와 다릅니다.

- domain 계층은 선택 사항이다.
- domain 계층은 data 계층에 의존하고, data 계층은 domain 계층에 의존하지 않는다.

**구글이 클린 아키텍처를 권장하지 않는 이유**

많은 앱이 domain 계층이 필요할 만큼 복잡하지 않기 때문에 선택사항으로 두고 있고, UI 계층이 data 계층에 의존하도록 앱을 구축한 다음 필요에 따라 나중에 domain 계층을 도입할 수도 있기 때문에 공식 가이드가 대부분의 안드로이드 앱에 가장 적합한 접근 방식을 제공한다고 생각합니다.
가이드는 가이드일 뿐이고, 개발자가 본인들의 앱이 ui -> (선택적 domain) -> data 구조가 적합한지 판단해야 합니다.

**안드로이드 공식 가이드의 장점은?**

- 덜 엄격하다. UI 및 데이터 계층으로 앱을 시작하고 필요할 때 domain 계층을 추가할 수 있다.
- 데이터가 저장되는 위치를 변경하기 쉽다.(ex. Preference -> Room or local -> cloud) data 계층이 다른 계층에 의존하지 않기 때문이다.

**클린 아키텍처의 장점은?**

- 비즈니스 로직을 변경하기 쉽다. 초기부터 로직을 캡슐화할 수 있다. UI 또는 데이터 계층에는 비즈니스 로직이 없다.
- 클린 아키텍처를 엄격하게 따르는 앱은 비즈니스 로직이 안드로이드 종속성이 없는 평범한 JVM 라이브러리 모듈로 구성되기 때문에 크로스 플랫폼에 유용할 수 있다.

SOLID 원칙도 위반했다고 하는데 어디서 위반했나요?

#### Yazon2006

다른 아키텍처(e.g. 클린 아키텍처 + 도메인 주도 설계)에서 domain 계층은 비즈니스 규칙과 도메인 객체를 포함하는 계층을 의미합니다. 비즈니스 규칙은 repository 에서 사용하는 라이브러리 정보나 호출하는 서버와 같은 구체적인 세부사항에 의존하면 안됩니다. 따라서 domain은 추상화여야 합니다.
NIA의 data 계층 및 다른 아키텍처 설명에는 정확한 라이브러리 등을 사용하는 정확한 구현이 포함되어 있기 때문에 추상화는 세부사항에 의존하면 안되며, 세부사항은 추상화에 의존해야 한다는 SOLID 의 다섯 번째 원칙에 위배합니다.

공식 가이드의 domain 계층의 정의는 복잡한 클래스 집합의 간소화된 인터페이스를 생성하는데 사용하는 퍼사드 패턴인 것 같습니다.

![](https://i.imgur.com/VkXTUe9.png)

이전 권장 아키텍처를 보면 몇몇 repository에 대한 엑세스 뿐만아니라 다른 매퍼/유틸리티 클래스에 대한 엑세스를 제공하는 선택적 [Facade](https://refactoring.guru/ko/design-patterns/facade) 클래스를 추가할 수 있다는 걸 알 수 있습니다. 이건 Gradle 모듈의 의미에서 추가적인 아키텍처 계층이 필요하지 않다는 걸 의미합니다.

> 많은 앱은 도메인 레이어가 필요할 만큼 복잡하지 않으며, 심지어 많은 앱은 NIA 의 [공식 아키텍처](https://developer.android.com/topic/architecture/intro)를 따를 만큼 복잡하지 않습니다. domain이 data에 의존하는 이유에 대해 논의할 것도 없이 단일 gradle 모듈로 구현할 수 있습니다.

가장 중요한 것은 UseCase/Interactor가 클린 아키텍처 및 기타 추상화 계층에서 선택 사항이라는 것입니다.
많은 개발자들이 ViewModel 에서 repository 를 호출하는 프록시 패턴 기반의 UseCase 를 만드는 걸 알지만 이는 클린 아키텍처의 문제가 아니라 잘못된 이해의 문제입니다. domain 계층에 속하는 repository 인터페이스를 사용해 ViewModel 에서 직접 데이터에 접근할 수 있습니다.

선택 사항이 될 수 없는 것은 비즈니스 엔터티 모델 체계입니다. 이는 공식 아키텍처 가이드에도 설명되어 있지 않은 **core** 모듈/레이어에 위치합니다.
클린 아키텍처에서는 해당 모델은 도메인의 일부이며, 데이터는 엔터티 모델이 포함된 인터페이스를 따르는 repository 를 제공하는 도메인에 따라 달라집니다.

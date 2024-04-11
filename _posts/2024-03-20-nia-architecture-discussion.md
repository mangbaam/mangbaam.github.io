---
layout: post
title: Now In Android is not Clean Architecture
subtitle: Now In Android 는 클린아키텍처와 SOLID 를 따르지 않는다
categories: Architecture
tags: [android,architecture,solid]
---
최근 Now in android 의 Discussions 에서 [Now in android 가 클린 아키텍처와 SOLID 원칙을 따르지 않는다](https://github.com/android/nowinandroid/discussions/1273)는 논쟁이 화두에 올랐습니다.
이 논쟁에서 아키텍처를 비롯한 다양한 유용한 내용들이 등장해서 이를 정리해보고자 글을 작성해보았습니다.

## 요약

Yazon2006 이라는 개발자가 Now in android 프로젝트가 클린 아키텍처와 SOLID 원칙 중 의존 역전의 원칙을 위배한다고 주장하며 이에 Now in android 의 테크 리더인 dturner 와 몇몇 안드로이드 개발자들이 이에 대한 반박을 하면서 시작된 논쟁입니다.

Yazon2006은 Now in android 가 클린 아키텍처를 따르지 않고 있어서 Now in android 를 참고하는 안드로이드 주니어 개발자들에게 혼동을 줄 수 있으니 클린 아키텍처를 따르도록 프로젝트 구조를 변경하거나 최소한 Now in android 는 클린 아키텍처를 따르지 않는다고 명시해야 한다고 주장합니다.

결과적으로 Now in android 의 구조는 변경되지 않았으며, 문서에 `The official Android architecture is different from other architectures, such as "Clean Architecture".` 라는 글이 추가되었습니다.

자세한 논쟁 내용은 밑에 정리한 내용을 살펴보거나 원본을 참고해주시면 좋겠습니다.

*[원본](https://github.com/android/nowinandroid/discussions/1273) 내용이 길기 때문에 핵심 내용 위주로 요약했으며, 일부 생략된 부분이 있고, 의역된 부분들이 원본의 의도를 담지 못했을 수도 있습니다. 자세한 내용은 [원본](https://github.com/android/nowinandroid/discussions/1273)을 참고해주시면 좋을 것 같습니다.*

## Now in android 는 클린 아키텍처와 SOLID를 따르지 않는다

#### Yazon2006

Now in android(이하 NIA)가 클린 아키텍처와 SOLID 원칙을 위반하는 것 같습니다. 특히 domain 계층이 data 계층에 의존하고 있기 때문에 순수한 비즈니스 로직이 아닙니다.
이로 인해 프로젝트 구조가  혼란스럽고, 잘못된 방향을 제공할 수도 있습니다.
domain 계층이 data 계층에 의존하도록 리팩터링하거나 README에 해명과 설명을 첨부해주세요.

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
클린 아키텍처에서에서 해당 모델은 도메인의 일부이며, 데이터는 엔터티 모델을 포함하는 repository 를제공하는 도메인에 의존합니다.

초급 개발자나 심지어 중급 개발자들이 프로젝트를 만들 때 "안드로이드 앱 아키텍처"를 구글링해서 [공식 아키텍처 가이드](https://developer.android.com/topic/architecture/intro)를 찾고, [NIA 예제](https://github.com/android/nowinandroid)를 찾아서 권장 사항을 맹목적으로 따릅니다. 왜냐하면 그들은 충분한 경험이 부족하여 유용한 것을 선택하고 버릴 수 없기 때문입니다.

이 경우 두 가지 결과를 예상할 수 있습니다.

1. 프로젝트가 충분히 작은 경우 권장 아키텍처가 필요 이상으로 복잡할 것입니다.
2. 프로젝트가 충분히 크거나 복잡하여 확장 가능하고 신뢰할 수 있으며 보편적으로 인정받는 아키텍처가 필요한 경우, domain 레이어를 포함한 독립적인 entity/interface/relations 등을 가지는 클린 아키텍처를 따르는 것이 좋습니다.

## 정말 DIP 를 위반하나요?

#### arkivanov

제 생각에, domain 레이어의 클래스 A가 data 레이어의 B 인터페이스를 의존하는데, B 인터페이스와 B 인터페이스의 구현체인 BImpl도 모듈에 함께 있는 상황에 대해 "의존성 역전 원칙"을 위반하는지는 논란의 여지가 있는 것 같습니다.
구체 클래스 A는 여전히 추상화된 B에 의존하며, BImpl 이 B 옆에 있더라도 바뀌는 것은 없습니다. 제 관점에서는 괜찮아 보이며, B의 다른 구현이 필요한 상황처럼 필요에 따라 구현을 다른 모듈로 쉽게 옮길 수 있어서 충분히 유연해 보입니다.

반면 (클린 아키텍처에서) data 모듈의 구현에는 domain 모듈의 모든 UseCase가 필요하지 않은데 data 레이어 모듈이 domain 레이어 모듈에 의존하는 이유는 무엇인가요?

#### Yazon2006

[상위 수준의 모듈은 하위 수준의 모듈의 어떤 것도 참조해선 안됩니다.](https://en.wikipedia.org/wiki/Dependency_inversion_principle) 우리의 경우 [domain](https://en.wikipedia.org/wiki/Domain_(software_engineering))은 상위 수준의 모듈이고, data는 하위 수준의 모듈입니다.
domain 모듈이 data 모듈에 의존하는 경우 의도했든 아니든 이 원칙을 위반할 수 있습니다.
data가 domain 에 의존하는 경우(즉, domain 이 data 에서 무언가를 참조하는 경우) 이런 상황은 절대 발생하지 않습니다.
모든 Gradle 모듈의 목적은 관심사 분리이며, 그렇지 않으면 모든 것을 하나의 gradle 모듈에 넣고 DIP의 B 부분만 따를 수 있습니다.

> A: High-level modules should not import anything from low-level modules. Both should depend on abstractions (e.g., interfaces).
> 
> B: Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.

data 모듈이 domain 모듈을 의존하는 이유는 간단히 말하면 NIA의 경우 domain 계층이 주로 core.model로 표시되기 때문입니다.

#### arkivanov

동의하기 힘듭니다. [동일한 위키](https://en.wikipedia.org/wiki/Dependency_inversion_principle)에서 다음과 같은 설명이 있습니다.

`따라서 하위 수준 모듈 구현 세부 사항과 관계없이 상위 수준 모듈을 렌더링합니다.`

이것이 바로 NIA가 구축되는 방식입니다. domain 모듈의 클래스는 data 모듈에서 구현 세부 정보를 가져오지 않습니다.

또한 일반적인 용어인 `module` 은 `gradle module`을 의미하지 않습니다. 모듈은 클래스일 수 있습니다.

domain 모듈의 엔터티가 data 모듈에서 세부 구현체를 가져오지 않는 한, data 모듈에 두 종류의 엔터티(인터페이스, 구현체)가 모두 포함되어 있다고 해서 SOLID를 위반하는 것은 아닙니다.

파일의 저장 위치, 폴더 구조, gradle 모듈성은 크게 중요하지 않습니다. data gradle 모듈을 두 개, 즉 data-api와 data-impl로 분할하고 도메인 모듈이 data-api에만 의존하도록 하는 것과 큰 차이가 없습니다.

#### Yazon2006

> module 은 gradle module을 의미하지 않습니다.

domain 계층은 database, datastore, network를 포함하는 data 모듈에 대한 의존성을 전이적으로 추가합니다. UseCase의 로직은 우리가 Ktor를 쓰던 Retrofit을 쓰던 의존하지 않기 때문에 의미가 없습니다.

> 큰 차이가 없습니다

극적인 차이가 있습니다. data를 `data-api`, `data-impl`로 쪼개면

- `data-api`는 인터페이스만 있고, `core.model`에 의존합니다.
- `core.domain`은 `data-api`와 `core.model`에 의존합니다.

그러면 `core.model` + `core.domain` + `data-api`는 고전적인 정의에서 domain 계층을 효과적으로 형성하고 `data-impl`은 그 일부에 의존합니다.
이는 "데이터를 도메인에 따라 결정"하는 측면에서는 동일하지만 관심사 분리 측면에서는 훨씬 더 적절합니다.

domain 레이어의 가장 간단한 예시는 하나의 모듈에 모델  + 인터페이스 + usecase가 있는 경우입니다. 그러나 성장하는 프로젝트에서는 이 모듈의 복잡성이 선형적으로 증가하여 확장하기 어려운 복잡한 형태가 되기 때문에 위험합니다.
`-api`/`impl`/등의 모듈 개념을 사용하면 도메인 지식의 일부를 별도의 `-api` 피쳐 모듈로 추출하여 정확한 구현 없이 다른 피쳐에서 재사용할 수 있습니다.
더 나아가 `-model`을 별도의 모듈로 추출할 수도 있습니다. 항상 동일한 규칙이 있어야 하며 `-impl`은 `api`에 따라 달라지지만 `api`는 `-impl`에 영향받지 않습니다.

> domain 모듈의 엔터티가 data 모듈에서 세부 구현체를 가져오지 않는 한

NIA 는 구글의 예제이며 안드로이드 개발 커뮤니티의 신성한 스크립트입니다. 이 프로젝트에서는 DTO -> domain 매퍼 생성을 피하기 위해 DTO 모델을 Usecase 로 만들기 쉬운 구조를 가지고 있습니다. 또한 core.model 의 모델이 아니라 data-domain-presentation 계층에서 DTO 모델을 어디서든 사용할 수도 있습니다.

## 정말 구글 권장 아키텍처가 클린 아키텍처보다 데이터 저장을 변경하기 수월한가요?

(dturner 가 구글 권장 아키텍처는 클린 아키텍처보다 데이터 저장을 변경하기 수월하다는 코멘트를 남x김)

#### briandr97

동의할 수 없습니다. 왜 클린 아키텍처에서는 데이터 저장을 변경하기 어렵다고 생각하는 건가요? 클린 아키텍처를 사용하면 데이터 계층의 DataSource 에서 데이터 저장 위치를 변경하는 것은 쉽다고 생각합니다.
data 계층에서 무슨 변화가 생기든, data 레이어는 domain 계층의 인터페이스를 구현할 뿐입니다. 그러므로 다른 계층에 데이터 계층의 변화가 전파되지 않습니다.

그러나 안드로이드 가이드에서는 data 계층은 완전히 독립적이며, domain, ui와 같은 다른 계층은 data 계층에 종속되어야 합니다. 따라서 data 계층은 안정적이어야 합니다. 구성요소가 많은 다른 구성요소에 의존한다면 변경하는 것이 어렵습니다.

data 계층과 다른 계층 사이에 인터페이스가 존재하기 때문에 인터페이스를 변경하지 않고도 DataSource 를 변경할 수 있다고 할 수 있다고 하더라도 클린 아키텍처가 Android 가이드보다 데이터 저장 위치를 변경하는 것이 더 어렵다고 할 수 없습니다.

#### dtuner

네, 정확한 지적입니다. 좋은 설계 예제를 따른다면 클린 아키텍처나 공식 가이드 모두 data 계층을 변경하는 것에 차이가 없어야 합니다.
인터페이스를 사용하지 않고 구체적인 구현에 의존하도록 설계된 data 계층이 있는 경우에 차이가 발생합니다. 가이드를 잘 따른다면 구현이 domain 계층에 종속되지 않기 때문에 data 계층을 변경하는 것이 더 쉽습니다.

#### briandr97

계층 간에 인터페이스가 존재하면 데이터 저장 위치를 변경하는 측면에서 두 아키텍처 간에 큰 차이가 없다는 점에 동의합니다.
하지만 인터페이스가 없다는 가정은 납득하기 어렵습니다. 클린 아키텍처에서는 인터페이스를 통한 의존성 역전이 필수적이기 때문입니다.
대신 클린 아키텍처가 계층 간 의존성 역전에 대한 이해가 필요하기 때문에 구글 공식 가이드보다 진입 장벽이 높다고 말하는 거라면 동의할 수 있습니다.

## 클린 아키텍처와 구글 가이드는 어떻게 다른가요?

#### dturner

![](https://i.imgur.com/eYRrVxT.png)

차이점을 빨간 색으로 표시했습니다.

클린 아키텍처를 엄격하게 준수하려면 domain 계층에서 래퍼 함수를 작성해 ui -> data 계층 종속성을 제거해야 합니다.

#### Yazon2006

순수 안드로이드 프로젝트에서는 의미 없습니다. ui와 data는 너무 결합되어 있어 분리하려고 하면 종속성을 전이하게 되고 결국 아무런 이점 없이 더 많은 보일러 플레이트를 가져오게 됩니다.

#### arkivanov

제 관점에선 domain 계층에 Repository 인터페이스가 있는 것은 좋지 않습니다. 예를 들어 domain 계층에 Repository 인터페이스와 Usecase 클래스가 있는 경우, domain 에 의존하는 내 data 계층이 Usecase 를 인식하는 것은 말이 안 됩니다.

저는 구글 공식 가이드가 더 좋아 보입니다.
![](https://i.imgur.com/433wujf.png)

정말 필요한 경우 data 계층을 data-api와 data-impl 두 개로 쪼개고 domain 계층이 data-api 에 의존하도록 할 수 있습니다. 따라서 data-api에는 데이터 접근을 위한 API 만 포함되는데, 이는 정확히 doamain 계층에 필요한 것입니다.

#### dturner

네, 제 요점은 Repository 인터페이스가 domain 계층에 있어야 하는 클린 아키텍처를 채택하는 경우 모든 Repository 메서드에 대해 Usecase를 제공할 필요가 없다는 겁니다.

domain 계층에 data repository interface 를 갖는 것이 이상하게 느껴지는 데에는 동의하지만, 요점은 클린 아키텍처에서는 다른 무엇보다 비즈니스 로직을 우선시한다는 것입니다.
repository 인터페이스는 모든 데이터 공급자가 지켜야 하는 규칙이므로 domain 계층에 있어야 합니다.

이건 저에게는 인터페이스와 구현을 분할하는 한 다음과 상관 없이 사소한 것입니다.
- 동일한 모듈 내에 있는 것(NIA에서 수행하는 작업, e.g. `:core:data`)
- `api`와 `impl` 모듈을 분리하는 것 (좋은 제안입니다)
- 도메인 모듈에 위치하는 것 (클린 아키텍처)

실제로 중요한 것은 사람들이 프로젝트의 아키텍처를 이해하고 올바르게 사용하는 것입니다.

내 (편향된) 의견에 따르면 공식 아키텍처는 개념적 수준에서 파악하기가 더 쉽습니다. 의존성 화살표가 모두 같은 방향으로 진행되기 때문입니다.

#### Yazon2006

공식 아키텍처는 개념적으로 이해하기 쉽습니다. 이 접근 방식에 반대하지는 않지만 구글이 무언가를 추천한다면 이러한 접근 방식의 단점을 최소한 사용자에게 알려주면 친절할 것 같습니다.

우리는 분명 의존성 방향을 바꾸면 독립적인 domain 계층을 얻을 수 있으며 로직이 정확한 구현에 의존하는 상황이 불가능해질 것입니다.

경험 많은 개발자에겐 NIA 하나의 예제에 불과하겠지만 공부하는 사람들, 강한 의견이 없는 사람들에게는 이 예제는 진리의 유일한 소스(Single Source of Truth)입니다. 그들은 이 프로젝트를 자신의 작은 프로젝트의 기반으로 삼을 것입니다. 그리고 정확한 데이터 소스에 의존하여 인터페이스와 로직을 구축할 것입니다. domain 계층이 있기 때문에 클린 아키텍처라고 생각할 것입니다.

이 프로젝트는 이해하기 어렵지 않지만 의존성 방향 변경은 할 만한 가치가 있습니다.

#### dturner

data 계층의 인터페이스는 내부 구현이 아니라 domain 계층과의 공개 계약을 나타냅니다. 예를 들어, Android의 경우 Retrofit에서 Ktor로 변경하려면 단순히 [NiaNetworkDataSource](https://github.com/android/nowinandroid/blob/main/core/network/src/main/kotlin/com/google/samples/apps/nowinandroid/core/network/NiaNetworkDataSource.kt)를 구현하는 `KtorDataSource`를 추가함으로써 변경할 수 있습니다. UseCases나 repository 구현은 변경되지 않습니다.

분명히 말하자면, 우리는 여전히 의존성 역전을 사용하고 권장합니다([공식 아키텍처 가이드의 모듈화 섹션](https://developer.android.com/topic/modularization/patterns#dependency_inversion)에서 설명된대로). 그것은 아키텍처적 레이어 수준이 아니라 모듈/객체 수준에서 적용됩니다.

#### Yazon2006

알겠습니다. 당신의 의견을 이해했습니다. 저는 이미 내 생각을 모두 말했습니다.
만약 당신이 가이드 문서의 도메인 정의가 괜찮다고 주장한다면 그렇게 되도록 하겠습니다.
데이터에 대한 도메인 의존성이 괜찮다고 생각한다면 저는 그에 동의합니다.

그냥 문서와 레포지토리 README에 어떤 해명을 추가해주세요.
예를 들어, **"저희가 정의하는 '도메인'은 로버트 C. 마틴의 Clean Architecture나 제프리 팔머로의 Onion Architecture에서 사용하는 '도메인'의 정의와 다를 수 있습니다."** 이렇게 말이죠. 이것이 제가 요청하는 전부입니다. 이해하셨으면 좋겠습니다.

P.S. 이러한 해명은 대부분의 아키텍처적 특이성을 설명할 수 있습니다.

#### dturner

프로젝트 아키텍처 문서에 메모를 추가했습니다. (승인되면 merge 합니다) [#1304](https://github.com/android/nowinandroid/pull/1304)

-> 현 시점 merge 되어 [NIA 문서](https://github.com/android/nowinandroid/blob/main/docs/ArchitectureLearningJourney.md)에서 확인할 수 있습니다.
![](https://i.imgur.com/shgMcQo.png)

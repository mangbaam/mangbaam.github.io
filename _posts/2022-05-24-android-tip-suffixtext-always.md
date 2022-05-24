---
layout: post
title: Android. How to always display **Suffix Text** on EditText
subtitle: Using TextInputLayout & TextInputEditText
categories: Android
tags: [tip, toolbar]
---

## 🔥해결하고자 하는 문제

---

![design](https://user-images.githubusercontent.com/44221447/170051583-5eb36c81-f268-4b79-bbbf-8ba05242b684.png)

위와 같은 디자인을 받았다. 현재 진행 중인 프로젝트에서는 학교 이메일로만 가입하는 정책이라서 뒤에 항상 동일한 메일 도메인을 표시한다.

저렇게 지정하는 것은 Material 디자인의 `TextInputLayout`과 `TextInputEditText`로 가능하다.

![default_xml](https://user-images.githubusercontent.com/44221447/170088134-dd8e4497-b492-4951-90cd-ffdb02a356a5.png)

`TextInputLayout`에 `android:hint` 속성에 힌트로 보여줄 텍스트를 지정하고 `app:hintEnabled` 속성에 true를 주면 다음과 같이 힌트가 보여진다.

![default_video](https://user-images.githubusercontent.com/44221447/170059824-83d19b34-baeb-4a20-b8dc-c9cbcb53c51e.mp4)

그런데 차이점이 있다면 `TextInputLayout`은 포커즈를 받지 않은 상태에서는 **Suffix Text**가 보이지 않고, 포커즈를 받으면 hint가 위로 올라가면서 **Suffix Text**가 보이게 된다.

처음에는 커스텀 뷰를 만드려고 했는데 간단히 해결할 수 있는 방법을 찾게 되어 공유하려고 한다.

## 🔥어떻게 고쳐야 할까

---

![layout_inspector](https://user-images.githubusercontent.com/44221447/170061800-66c0a697-82cc-481f-8f3c-4ee19da0c2d6.png)

layout inspector로 확인을 해보니 힌트 부분이 처음에 전체 공간을 차지하고 있는 것을 확인했다.

![focused_layout](https://user-images.githubusercontent.com/44221447/170062812-3c70ae8c-1ee4-4b5a-b9a8-a84962e7839b.png)

포커즈를 받은 상태를 확인해보니 애초에 없었던 **Suffix Text**가 등장했다. 이걸 보아 hint에 가려져서 보이지 않았던 것이 아니라 내부적으로 포커즈를 받았을 때만 보이도록 만들어진 것 같다.

우리가 해결해야 하는 부분은 두 가지이다.

1. 포커즈를 받으면 힌트가 위로 올라가는 것이 아니라 **보이지 않아야 한다.**
2. **포커즈를 받지 않은 상태에서도** **Suffix Text**가 보여야 한다.

## 🔥해결해보자

---

### 글자가 입력되면 힌트 없애기

![solve_1](https://user-images.githubusercontent.com/44221447/170088259-aaf0ff95-07b6-4ccf-990d-c0a459e13d84.png)

이 부분은 간단하다. 힌트를 `TextInputLayout`가 아니라 `TextInputEditText`에서 표시하면 된다. `android:hint` 속성을 `TextInputEditText`로 옮기고, `TextInputLayout`에서 `app:hintEnabled` 속성을 제거하면 된다. (기본 값이 false이다)

![solve_1_video](https://user-images.githubusercontent.com/44221447/170065210-f3663154-9538-4cf9-8956-cc2359a97206.mp4)

입력 창을 클릭하면 힌트가 위로 올라가지 않고, 글자를 입력하면 힌트가 보이지 않는다.

### 항상 **Suffix Text** 표시하기

![updateSuffixTextVisibility](https://user-images.githubusercontent.com/44221447/170070246-07ea1cbf-c147-4161-987e-a6b2347a753d.png)

`TextInputLayout.java` 파일을 뒤져봤다. `updateSuffixTextVisibility()` 함수를 찾았는데 이 함수는 **Suffix Text**가 존재하고, 힌트가 확장되지 않는 상태일 때 **Suffix Text**를 보이게 하는 동작을 하는 것으로 생각된다.

힌트가 확장된 상태라고 하면 포커즈를 받지 않은 상태에서 힌트가 입력창 전체 부분을 덮고 있는 상태를 의미하는 것 같다.

![description](https://user-images.githubusercontent.com/44221447/170020093-ceb93cf2-93bd-4226-9eb3-25bb742d75d6.png)

또 `TextInputLayout`에 `expandedHintEnabled` 라는 속성이 있는데 이 속성의 주석을 보면 다음과 같이 적혀있다.

```text
텍스트 필드가 채워지지 않고 포커즈를 받지 않은 경우 힌트가 입력 영역을 차지해야 하는지 여부
```

즉 `expandedHintEnabled` 속성으로 글자가 입력되지 않고, 포커즈를 받지 않은 상태에서는 힌트가 전체 입력 영역을 차지할지 말지 옵션을 줄 수 있다는 말이다.

## 🔥결론

---

![complete_video](https://user-images.githubusercontent.com/44221447/170086239-452324cb-a1ad-4572-b3fa-7cd3b2d3b278.mp4)

![complete_xml](https://user-images.githubusercontent.com/44221447/170086485-2277cab1-c41a-4900-9770-e90d6c3fe53f.png)

1. TextInputLayout에 `app:expandedHintEnabled="false"` 속성 설정
2. TextInputEditText에 `android:hint` 속성 설정

## 🔥추가 사항

---

![comment](https://user-images.githubusercontent.com/44221447/170086891-751d4064-06e4-4514-9adc-6906dcd44e5c.png)

주석에는 위와 같은 설명이 적혀있었다.

힌트를 TextInputLayout이 아니라 TextInputLayout에 달으라는 말이다. 나중에 힌트를 수정할 때 문제가 생길 수 있다는 설명이다.

우리 프로젝트의 요구사항에서는 힌트가 변경되지 않기 때문에 문제가 되지 않는 부분이라고 생각하고, 코드를 좀 더 살펴보니 EditText에 힌트가 설정된 경우 이 힌트를 TextInputLayout으로 옮기는 과정이 있는 것을 확인했다.

![reason_to_use_hint_at_layout](https://user-images.githubusercontent.com/44221447/170069189-482cdd52-6a78-4bfa-b6f2-ad77870be08f.png)

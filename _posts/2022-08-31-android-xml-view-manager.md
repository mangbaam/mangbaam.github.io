---
layout: post
title: Android TextView 를 중첩(계층)할 수 없는 이유
subtitle: Android ViewManager
categories: Android
tags: [android,xml,inside]
---

## TextView 를 계층 구조로 만들 수 있을까?

---

![image](https://user-images.githubusercontent.com/44221447/187700176-a349d63f-1e50-4564-9a8d-619fee880668.png)

안드로이드 스튜디오에서 새 프로젝트를 만들면 activity_main.xml 파일에 위와 같이 TextView 하나가 생성되어 있다.

자세히 보면 ConstraintLayout 태그 안에 TextView 단일 태그가 있는 형태이다.

그럼 TextView 도 ConstriantLayout 처럼 태그를 열어서 다른 뷰를 넣을 수 없을까?

![image](https://user-images.githubusercontent.com/44221447/187699938-b7dd32b1-7753-43e1-9334-0392fb34cc82.png)

그래서 위 코드와 같이 TextView 를 중첩해서 작성해보았다. 그랬더니 다음 사진과 같이 렌더링 오류가 발생했다.

![image](https://user-images.githubusercontent.com/44221447/187699686-aec3d842-1bc2-41ec-92cf-23d661715277.png)

TextView 는 하위 계층으로 다른 뷰를 추가할 수 없다는 것인데... 왜 그럴지 알아보았다.

## View 와 ViewGroup

---

ConstraintLayout 과 TextView 는 둘 다 View 이다. View 라고 부르는 것은 실제로 View 라는 클래스를 상속하고 있기 때문이다.

하지만 차이가 있다면 ConstraintLayout 은 View 를 상속하는 ViewGroup 을 상속하고 있고, TextView 는 View 를 직접 상속하고 있다는 점이다.

ConstraintLayout 뿐만 아니라 FrameLayout, LinearLayout, RelativeLayout 등 xxxLayout 으로 이름 지어진 뷰들은 ViewGroup 을 상속하고 있고, 태그를 열어서 다른 뷰들을 포함할 수 있는 특별한 뷰라는 특징이 있다.

[![image](https://user-images.githubusercontent.com/44221447/187703738-1470bf0d-de3a-47a1-87f2-5812b029da53.png)](https://developer.android.com/guide/topics/ui/declaring-layout?hl=ko)

위 그림은 안드로이드 공식 문서에 있는 그림인데 마치 ViewGroup 이 최상위 계층인 것으로 착각할 수 있는데 저 그림이 상속 관계는 아니고 어떠한 UI 레이아웃의 한 예시이다. ViewGroup 이 View 를 포함할 수 있다는 것 정도로 이해하면 될 듯 하다.

## ViewParent와 ViewManager

---

![image](https://user-images.githubusercontent.com/44221447/187707010-d81ab9b5-b4b9-4a60-9ad6-cf8cc86bf23e.png)

ViewGroup 은 View 를 상속하고 있지만 ViewParent와 ViewManager 인터페이스도 구현하고 있다.

### ViewParent

ViewParent 인터페이스는 부모 뷰로서의 책임을 정의한다. 어떠한 자식 뷰가 부모와 상호작용 하기 위해 사용하는 메서드들이 정의되어 있다.

예를 들면 포커즈를 요청하기 위한 메서드나 뷰를 맨 위(Z축)로 올리기 위한 메서드 등이 있다.

[공식 문서](https://developer.android.com/reference/android/view/ViewParent)에서 자세히 확인할 수 있다.

### ViewManager

ViewManager 는 자식 뷰를 추가하거나 제거할 수 있도록 한다.

총 3개의 메서드가 제공되는데 `addView`, `removeView`, `updateViewLayout` 이다.

ViewManager 를 상속하는 클래스는 ViewGroup 이 유일하다.

[공식 문서](https://developer.android.com/reference/android/view/ViewManager)에서 자세히 확인할 수 있다.

## 결론

---

어떠한 View 가 자식 View 를 가지려면 자식 View 가 배치되고 부모 뷰와 상호작용 해야 하는데 이때 ViewParent 에 정의된 메서드들과 ViewManager 에 정의된 메서드들이 모두 필요하다. 그리고 이 두 인터페이스를 구현하고 있는 유일한 클래스가 ViewGroup 클래스이기 때문에 자식 뷰를 가지기 위해서는 ViewGroup 을 상속하거나 ViewGroup 을 상속하는 다른 클래스를 상속해야 한다.

예를 들어 ScrollView 는 ViewGroup 을 상속하는 FrameLayout 을 상속하고, ViewPager2 는 ViewGroup 을 직접 상속한다.

결국 TextView 는 ViewGroup 을 상속하고 있지 않기 때문에 하위 뷰를 둘 수 없는 것을 알 수 있었고, 커스텀 뷰를 만들 때 하위에 뷰를 두고자 한다면 ViewGroup 을 상속하거나 ViewGroup 을 상속하는 다른 클래스를 상속해서 만들어야 한다는 점도 알 수 있었다.

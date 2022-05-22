---
layout: post
title: 미립자 팁. 테마에 앱바 스타일 적용하기
subtitle: Change style of AppBarLayout in the entire theme
categories: Android
tags: [tip, toolbar]
---

## 해결 할 문제

많은 프로젝트에서 `ActionBar`가 없는 테마를 사용하고, 상황에 맞게 직접 앱바를 추가해서 사용할 것이다.

나 역시도 `NoActionBar` 를 사용해 직접 xml에서 `AppBar`를 추가해서 사용한다.

```xml
<style name="Theme.MyTheme" parent="Theme.MaterialComponents.Light.NoActionBar">
...
</style>
```

![before](https://user-images.githubusercontent.com/44221447/169691837-5a9e8c3e-7c22-4169-9716-9b00bb54cbec.png)

```xml
<com.google.android.material.appbar.AppBarLayout
    android:id="@+id/appbar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:navigationIcon="@drawable/ic_back">
        // ...
    </androidx.appcompat.widget.Toolbar>
</com.google.android.material.appbar.AppBarLayout>
```

기존에는 기본적으로 AppBarLayout을 만들면 배경 색상이 `themes.xml > colorPrimary` 의 색을 따랐다. 그래서 모든 AppBar에 background 속성을 지정해서 사용해야 하는 것에 불편함을 느꼈다.

## 해결

![after](https://user-images.githubusercontent.com/44221447/169692009-febe318f-04cb-4ab8-9a30-fc6dfa7fa0cf.png)

*themes.xml*

```xml
<style name="Theme.Uswtimetable" parent="Theme.MaterialComponents.Light.NoActionBar">
    // ...
    <item name="appBarLayoutStyle">@style/SuwikiAppbar</item>
</style>

<style name="SuwikiAppbar" parent="Widget.AppCompat.Toolbar">
    <item name="background">@color/background</item>
</style>
```

1. `Widget.AppCompat.Toolbar`를 상속한 스타일을 만든다.
2. 현재 사용하고 있는 테마의 속성으로 `<item name="appBarLayoutStyle">@style/1번의스타일</item>`을 지정한다

정말 특이하게 동작했다.

appBarLyaoutStyle을 지정해주기 위해 처음에는 `Widget.Design.AppBarLayout`를 상속한 스타일을 만들었는데 동작하지 않는 것이다.

그래서 NoActionBar를 사용해서 그런가보다 하고 `Widget.AppCompat.Toolbar`를 상속한 스타일을 만들어서 현재 사용하고 있는 테마의 속성으로 `android:toolbarStyle`와 `toolbarStyle`에도 지정해봤는데 동작하지 않았다.

그러다 우연히 `appBarLayoutStyle`에 `Widget.AppCompat.Toolbar`를 상속한 스타일을 적용했더니 적용이 된 것이다.

혹시나 해서 `ToolBar`에 `margin`을 줘서 `AppBar`를 확인해봤는데 원했던 대로 스타일이 적용된 것을 확인했다.

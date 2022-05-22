---
layout: post
title: 미립자 팁. 앱바 밑에 그림자 없애기
subtitle: Disable shadow below appbar
categories: Android
tags: [tip, toolbar]
---

## 이전

![before](https://user-images.githubusercontent.com/44221447/169690473-801cc0e2-f7f4-4464-bb7e-b3f70bf534c0.png)

```xml
<com.google.android.material.appbar.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:elevation="0dp">
```

`android:elevation="0dp"`를 넣어도 그림자가 보인다.

## 해결

![after](https://user-images.githubusercontent.com/44221447/169690060-1c8c1ed8-d6f2-4f08-b1e4-e3167c181837.png)

```xml
<com.google.android.material.appbar.AppBarLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:elevation="0dp">
```

`android` -> `app` 으로 바꾸면 그림자가 사라진다.

-> `app:elevation="0dp"`

---
참고: https://blogdeveloperspot.blogspot.com/2018/04/toolbar-appbar-shadow.html

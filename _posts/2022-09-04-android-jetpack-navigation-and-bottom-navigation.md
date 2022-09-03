---
layout: post
title: Android BottomNavigationView with Jetpack Navigation
subtitle: 
categories: Android
tags: [android,jetpack,navigation]
---

## ⭐

---

<img src="https://user-images.githubusercontent.com/44221447/188276445-e4f18957-57d8-4f9d-a24e-6533c8fa36f8.png" width="30%"/>

앱에서 하단 네비게이션은 흔히 볼 수 있는 UI 이다. 아마도 가장 익숙한 것이 카카오톡의 하단 메뉴인 것 같다. (물론 카카오톡이 Jetpack Navigation 으로 작성되었다는 말은 아니다)

이번 글에서는 `Jetpack Navigation` 을 사용해서 하단 네비게이션을 연동하는 방법에 대해서 설명한다.

## Jetpack Navigation 이란

---

[공식 문서](https://developer.android.com/guide/navigation)

Jetpack Navigation(이하 *Navigation* 혹은 *네비게이션*)은 앱의 컨텐츠를 탐색하거나 상호작용을 구현할 때 더욱 쉽고 편하고 안전하게 할 수 있도록 해주는 Jetpack 라이브러리이다. 주로 프래그먼트 간의 이동이나 데이터 전달 등의 기능을 사용할 때 좋다.

개발자에게 아주 편한 기능을 제공하지만 제공되지 않는 기능은 커스텀해서 사용해야 하기 때문에 규모가 큰 앱에서는 잘 사용하지 않는다고도 한다. 물론 기능이 제공되는 부분에 한해 선택적으로 사용할 수도 있겠다.

그래서 FragmentManager 를 통해서 직접 프래그먼트를 다루는 방법을 반드시 알고는 있어야 한다. 이 글에서는 Navigation 에 대해서만 설명한다.

## Navigation 의 장점

---

- 프래그먼트 트랜젝션 처리
- 'Up'과 'Back' 이벤트를 적절히 처리
- 애니메이션, 전환 등의 리소스 제공
- 딥 링크 구현 및 처리 가능
- Navigation Drawer나 Bottom Navigation 등의 UI 패턴을 쉽게 연동 가능
- [Safe Args](https://developer.android.com/guide/navigation/navigation-pass-data#Safe-args) 제공 (안정적으로 데이터를 전달할 수 있는 기능)
- ViewModel 지원
- GUI([Navigation Editor](https://developer.android.com/guide/navigation/navigation-getting-started))를 통한 쉬운 조작

## Navigation 의 3가지 요소

---

- `Navigation Graph`(이하 NavGraph)
  - Navigation 관련 기능이 모여 있는 XML 파일
  - 대상(앱 내의 컨텐츠나 화면)간 경로도 이 파일에서 관리됨
  - GUI([Navigation Editor](https://developer.android.com/guide/navigation/navigation-getting-started))로 조작 가능
- `NavHost`
  - NavGraph 에서 대상(프래그먼트 뷰)을 표시하는 빈 컨테이너
  - `NavHostFragment` 를 사용하는 것을 권장함
  - 예전에는 `FrameLayout` 등으로 사용했음
- `NavController`
  - `NavHost` 에서 앱 탐색을 할 수 있도록 하는 객체
  - 대상 간 이동하는 경우 `NavController` 를 통해 이루어진다
  - 전환이 이루어지면 `NavHost` 에 이동한 프래그먼트를 띄우는 역할

## Build.gradle 추가

---

```groovy
dependencies {
  def nav_version = "2.5.1"

  // Java language implementation
  implementation "androidx.navigation:navigation-fragment:$nav_version"
  implementation "androidx.navigation:navigation-ui:$nav_version"

  // Kotlin
  implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
  implementation "androidx.navigation:navigation-ui-ktx:$nav_version"

  // Feature module Support
  implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"

  // Testing Navigation
  androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"

  // Jetpack Compose Integration
  implementation "androidx.navigation:navigation-compose:$nav_version"
}
```

![image](https://user-images.githubusercontent.com/44221447/188277621-1ab03b35-b563-48f9-936e-2a44a0d3d48c.png)

필요한 것만 사용하면 된다. 특히 자바와 코틀린 중 사용하는 언어만 추가해도 된다.

## UI 작성

---

![image](https://user-images.githubusercontent.com/44221447/188277753-93ed089c-9dd0-4264-a9d9-b83cd1bdd73b.png)

이런 UI 를 작성해보려고 한다.

### Menu 추가

*bottom_nav.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/newsListFragment"
        android:icon="@drawable/ic_news"
        android:title="Top News" />
    <item
        android:id="@+id/categoriesFragment"
        android:icon="@drawable/ic_category"
        android:title="Categories" />
    <item
        android:id="@+id/savedFragment"
        android:icon="@drawable/ic_save"
        android:title="Saved" />
</menu>
```

![image](https://user-images.githubusercontent.com/44221447/188277895-01e3e1b3-a3e2-4a06-92fb-3f8294cce0f2.png)

하단 메뉴에 추가할 메뉴를 res/menu 폴더에 추가한다.

### Fragment 추가

![image](https://user-images.githubusercontent.com/44221447/188277958-5190c087-2d0b-4a58-936e-2fcf483c3b33.png)

*categories.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ED9292"
    tools:context=".CategoriesFragment">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:textStyle="bold"
        android:textSize="40dp"
        android:gravity="center"
        android:text="Categories" />

</FrameLayout>
```

하단 네비게이션에 연결할 3개의 프래그먼트를 만들었다. (CategoryFragment, NewsListFragment, SavedFragment)

xxxFragment.kt 파일에는 자동으로 생성된 코드만 존재하고, 뷰는 배경 색과 글자만 빼고 모두 동일하게 작성했다.

### NavGraph 추가

![image](https://user-images.githubusercontent.com/44221447/188278036-870df5ce-fd8d-41a9-bba5-1c31c320a1ff.png)

우측 상단에서 디자인 탭으로 보면 아래와 같이 UI 로 조작할 수 있다.

![image](https://user-images.githubusercontent.com/44221447/188278121-f4a932cd-2f95-41d7-9192-a729907be38b.png)

빨갛게 표시해놓은 `+` 버튼을 누르면 생성한 프래그먼트나 액티비티를 추가할 수 있다.

![image](https://user-images.githubusercontent.com/44221447/188278161-35d5be96-c1f7-441d-94e4-418071dc9c47.png)

앞서 만든 3개의 프래그먼트를 마우스로 클릭하면 다음과 같이 추가가 된다.

![image](https://user-images.githubusercontent.com/44221447/188278222-a6093052-cb07-47f4-ae5b-fcc6a5dd8fc3.png)

그리고 가장 최초에 NavHost 에 보여줄 프래그먼트는 StartDestination 이라고 부르는데 위에 있는 집모양 버튼을 누르면 지정할 수 있다.

![image](https://user-images.githubusercontent.com/44221447/188278266-5425bab1-ea2b-49f8-a6e0-56a5316c4c48.png)

우측 상단에서 split 탭이나 코드 탭에서 코드를 확인해보면 다음과 같이 추가된 것을 확인할 수 있다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/newsListFragment">
    <fragment
        android:id="@+id/categoriesFragment"
        android:name="mangbaam.practice.wanted_preonboarding_android.CategoriesFragment"
        android:label="fragment_categories"
        tools:layout="@layout/fragment_categories" />
    <fragment
        android:id="@+id/newsListFragment"
        android:name="mangbaam.practice.wanted_preonboarding_android.NewsListFragment"
        android:label="fragment_news_list"
        tools:layout="@layout/fragment_news_list" />
    <fragment
        android:id="@+id/savedFragment"
        android:name="mangbaam.practice.wanted_preonboarding_android.SavedFragment"
        android:label="fragment_saved"
        tools:layout="@layout/fragment_saved" />
</navigation>
```

- `type`: 위 코드에서 `fragment` 에 해당함
- `label`: 사용자에게 표시될 수 있는 이름. `setUpWithNavController()`로 `Toolbar`에 연결하면 UI에 이 값이 표시될 수 있기 때문에 리소스 문자열 사용 권장
- `id`: 코드에서 대상을 참조할 때 필요한 ID
- `layout`: 표시할 레이아웃

여기서 중요한 것은 `id` 와 res/menu 의 메뉴 `id` 가 일치해야 한다는 것이다.

[menu 를 추가하는 부분](#menu-추가)으로 돌아가보면 `id` 값이 일치하는 것을 알 수 있다.

## NavHost와 BottomNavigationView 추가

---

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/container"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:layout_constraintBottom_toTopOf="@+id/navigation"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/nav_graph" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:menu="@menu/bottom_nav" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

<img src="https://user-images.githubusercontent.com/44221447/188278411-2c4397b3-d798-4985-8ca2-35e8fcb66ac5.png" width="33%" /> <img src="https://user-images.githubusercontent.com/44221447/188278422-5d1f94cf-1124-4d3a-a0be-1100780f5536.png" width="33%" />

NavHost 에 해당하는 것이 FragmentContainerView 이고, Bottom Navigation이 BottomNavigationView 이다.

NavHost 에서 중요한 속성은 다음과 같다

- `android:name` : NavHost 구현 클래스 이름
- `defaultNavHost : true` : true 일 때 NavHostFragment 가 시스템 뒤로 버튼을 가로채는 등 적절한 동작을 한다. 여러 호스트가 있는 경우 하나만 설정 가능
- `navGraph` : NavHostFragment 를 NavGraph 와 연결. 위에서 만든 NavGraph 를 연결해주면 된다

BottomNavigatinoView 의 중요한 속성은 다음과 같다

- `menu` : 하단 네비게이션에 추가할 메뉴를 추가한다

## NavController 로 BottomNavigation 과 연결

---

*MainActivity.kt*

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initNavigation()
    }

    private fun initNavigation() {
        val navHostFragment = supportFragmentManager.findFragmentById(R.id.container) as NavHostFragment
        val navController = navHostFragment.navController
        findViewById<BottomNavigationView>(R.id.navigation).setupWithNavController(navController)
    }
}
```

1. NavHostFragment 를 찾는다
2. NavController 를 찾는다
3. BottomNavigation 에 NavController 를 연결한다

위 과정을 따른다.

참고로 NavController 는 주로 4가지 방법 중 하나로 찾을 수 있다.

1. NavHostFragment.navController (위에서 사용한 방법)
2. Fragment.findNavController()
3. View.findNavController()
4. Activity.findNavController()

## 결과

---

![untitled](https://user-images.githubusercontent.com/44221447/188280066-ff3463cd-374e-4d1c-903c-f247bbef8f4c.gif)

Jetpack Navigation 을 사용하면 뒤로가기 버튼을 눌렀을 때 StartDestination 으로 돌아온다.

StartDestination 에서 뒤로가기 버튼을 한 번 더 눌러야 앱이 종료된다.

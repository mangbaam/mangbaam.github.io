---
layout: post
title: 안드로이드에서 FCM 사용해보기-1
subtitle: 파이어베이스 프로젝트 추가 및 설정
categories: Android
tags: [android, firebase, fcm]
---

[prev_posting]: https://mangbaam.github.io/android/2022/05/16/fcm-starter.html

## ⭐

> 이번 게시글에서는 파이어베이스를 사용해 본 적 없거나 FCM을 사용해 본 적 없는 분들을 위해 파이어베이스 프로젝트를 생성하고 설정한 후에 Android 프로젝트를 등록하는 방법에 대해 단계별로 자세히 설명하려고 합니다.

> FCM에 대해서 궁금하다면 [이전 게시글][prev_posting]을 참고해보세요!

## 🔥새 안드로이드 프로젝트 생성

---
![new_android_project1](https://user-images.githubusercontent.com/44221447/168643722-86740084-6d2c-4db9-a4c0-af32c1fe9bbb.png)

![new_android_project2](https://user-images.githubusercontent.com/44221447/168644073-d5123e34-ac06-4449-be47-f61a44bd5689.png)

### 앱 ID 확인

![android_id](https://user-images.githubusercontent.com/44221447/168645836-a2d44009-618f-4508-9d22-ada12bd31c4e.png)

`build.gradle(App)`에서 `applicationId`를 확인한다. 이 값은 뒤에서 파이어베이스를 설정할 때 사용된다.

[console]: https://console.firebase.google.com/

## 🔥Firebase 콘솔

---
[![firebase_console](https://user-images.githubusercontent.com/44221447/168642858-d66f068e-338c-417d-b96f-886bbbbbc1c1.png)][console]

### 1. 프로젝트 추가

[Firebase 콘솔][console]에서 `프로젝트 추가`를 눌러 프로젝트를 추가한다.

#### 1. 프로젝트 이름 설정

![new_project1](https://user-images.githubusercontent.com/44221447/168646051-9a6a1ccc-a725-4a02-89ea-81c467b907c6.png)

파이어베이스 프로젝트 이름을 지정한다.

#### 2. 선택 옵션

![new_project2](https://user-images.githubusercontent.com/44221447/168646175-be22e19c-a463-444a-af66-021f082e551c.png)

파이어베이스는 `구글 애널리틱스`를 활용한 `A/B 테스팅` 등의 기능을 제공한다. 당장 안 쓰더라도 굳이 옵션을 끌 필요는 없으니 체크된 상태로 넘어간다.

#### 3. 계정 선택

![new_project3](https://user-images.githubusercontent.com/44221447/168646241-801dbb3c-1797-433f-8654-db8d1461c344.png)

계정을 선택한다.

#### 4. 프로젝트 생성 완료

![new_project4](https://user-images.githubusercontent.com/44221447/168646418-01b9941b-7bfb-42e5-94e7-2931c83d8c40.png)

파이어베이스 프로젝트가 성공적으로 생성되었다.

### 2. 앱 추가

하나의 파이어베이스 프로젝트에는 하나의 앱만 등록하는 것을 권장한다. 예를 들어 **A 쇼핑몰**이 `안드로이드`, `iOS`, `웹` 등의 플랫폼을 지원할 때는 하나의 파이어베이스 프로젝트에 각각의 플랫폼을 추가해도 되지만 **A 쇼핑몰**과 *관련 없는* **B 쇼핑몰**을 추가하는 것은 **지양하는 것이 좋다**.

#### 1. 안드로이드 추가

![fcm_console](https://user-images.githubusercontent.com/44221447/168646664-faea0e12-2c0f-43e8-97c1-6a1c0732d2fa.png)

하나의 프로젝트에서 다양한 플랫폼을 추가할 수 있다. 안드로이드를 선택한다.

#### 2. 패키지 이름 지정

![regist_app1](https://user-images.githubusercontent.com/44221447/168646834-3a54912e-c14d-44f2-be4c-8b2e023c8763.png)

[앞서 확인했던](#앱-id-확인) 안드로이드 패키지 이름을 지정한다.

> `build.gradle(App)`의 `applicationId`

#### 3. google-services.json

![regist_app2](https://user-images.githubusercontent.com/44221447/168646930-fe16cd68-59cb-4129-b327-0b3119952082.png)

`google-services.json`파일을 다운로드 한다. 이 파일의 내용은 민감한 정보를 포함하고 있으니 주의해서 다뤄야 한다.

#### 4. google-services.json 붙여넣기

![change_to_project](https://user-images.githubusercontent.com/44221447/168647292-fa41a0bb-de3d-4a4d-ae81-293c1e2ecbfe.png)

안드로이드 스튜디오 프로젝트 탭에서 `Android`로 되어있는 것을 `Project`로 변경한다.

![paste_google-services.json](https://user-images.githubusercontent.com/44221447/168647487-983c791c-983c-4479-a836-cf953809d09c.png)

`app`에서 <kbd>Ctrl</kbd> + <kbd>v</kbd>로 붙여넣기 한다.

![paste_google-services.json2](https://user-images.githubusercontent.com/44221447/168647619-4b6d4eb4-a53b-47a8-8360-b770b4f43929.png)

`OK`를 눌러 완료한다.

![add_to_git](https://user-images.githubusercontent.com/44221447/168647693-e76f48ff-abde-49e5-be87-b68eaeca5ca8.png)

만약 `Git`을 사용하고 있다면 이 파일은 커밋에서 제외한다.

![add_to_gitignore](https://user-images.githubusercontent.com/44221447/168647946-85958119-6daa-4431-a384-a3889ea8e06b.png)

`.gitignore`파일에 `google-services.json`을 추가해 실수로 커밋하지 않도록 하자.

### 3. Firebase SDK 추가

다음은 Firebase SDK를 추가하는 과정이다. Firebase에서는 다음과 같이 안내하고 있지만 안드로이드 스튜디오 버전에 따라 설정 위치가 달라질 수 있다. 아래 2개의 사진은 참고만 하고 그 **밑에서 다른 방법을 소개하려고 한다.**

![add_sdk1](https://user-images.githubusercontent.com/44221447/168648976-fc0f4aa7-5367-44af-90a7-446a9c8bd390.png)

![add_sdk2](https://user-images.githubusercontent.com/44221447/168649044-420d17a9-9eca-4ac8-b1d2-f9d5b2e92106.png)

우리는 다음 과정을 따라해보자.

![firebase_menu](https://user-images.githubusercontent.com/44221447/168649662-d3f0da9c-6436-4ccc-93f1-228068dfd70d.png)

안드로이드 스튜디오의 상단 메뉴의 `Tools`에는 `Firebase` 메뉴가 존재한다. 이것을 선택하면 다음 화면이 뜬다.

![fcm_menu](https://user-images.githubusercontent.com/44221447/168650031-913c0ca4-8560-40be-b86b-49246257f4f5.png)

파이어베이스의 여러 기능이 보이는데 그 중 `Cloud Messaging`을 선택하고 Kotlin을 사용할 것이므로 **두 번째 옵션**을 선택한다.

![fcm_config1](https://user-images.githubusercontent.com/44221447/168650432-1aa4c63c-7c7a-42df-9451-39f7dff30995.png)

그러면 FCM을 구성하고 사용하기 위한 다양한 도움말이 나오는데 우린 이미 *1.앱 연결*을 했기 때문에 *2.앱에 FCM 추가* 버튼을 선택한다.

![fcm_config2](https://user-images.githubusercontent.com/44221447/168650524-889d31fa-200b-44a7-8400-66a5eb4d5ed4.png)

버튼을 누르면 위와 같이 변경될 내용이 표시된다. 하단의 `Accept Changes` 버튼을 눌러 설정을 완료한다.

만약 수동으로 추가했다면 `Sync now` 버튼을 눌러 동기화한다.

![completed](https://user-images.githubusercontent.com/44221447/168656164-52a9af63-5eb0-4af1-af99-abce39f46696.png)

여기까지 잘 따라왔다면 설정이 성공적으로 완료된다.

![completed_register](https://user-images.githubusercontent.com/44221447/168656757-01bb4423-9162-4f14-848f-6ebb720b405e.png)

정상적으로 추가되었다면 위 사진처럼 등록된 앱을 확인할 수 있다.

![menu_fcm](https://user-images.githubusercontent.com/44221447/168656454-329da649-b397-4f6c-af9e-f7bd228169ff.png)

좌측의 메뉴에서 아래로 내려보면 `Cloud Messaging` 메뉴가 있다. 이 곳에서 FCM을 다룰 수 있다.

## 🔥마무리

---
> 여기까지 **파이어베이스 프로젝트를 추가**하고, **안드로이드 프로젝트도 생성**하고, 파이어베이스에서 **안드로이드를 사용하기 위한 설정하는 방법**에 대해 알아봤습니다.

> 안드로이드에서 `FCM`를 사용하는 예제를 확인하고 싶다면 다음 게시글(포스팅 예정)을, `FCM`에 대해서 알고싶다면 [이전 게시글][prev_posting]를 참고하세요!

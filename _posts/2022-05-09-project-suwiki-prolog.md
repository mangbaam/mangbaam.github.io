---
layout: post
title: "[SUWIKI] Introduce"
subtitle: 🎈SUWIKI를 소개합니다!
categories: Project
tags: [suwiki]
---
## 🐥 무슨 프로젝트 인가요?

SUWIKI는 수원대학교 시간표를 만들 수 있는 시간표 앱입니다. 기존에 시간표 앱으로 많이 사용하던 `애브리타임`이 더 이상 수원대학교의 시간표를 지원해주지 않으면서 수원대학교 학생들이 시간표 관리로 겪을 불편함을 줄이고자 시작된 프로젝트입니다.

시간표 기능에 더불어 모든 강의에 대한 강의 평가를 사용자들이 서로 공유하고, 시험 관련 정보도 공유할 수 있도록 커뮤니티 기능을 제공합니다.

`SUWIKI`는 기존에 시간표 기능 만을 제공하던 `수타` 앱에서 시작하여 **자체 로그인, 강의 평가, 시험 정보 공유, 포인트 관리 등** 다양한 편의 기능을 추가해 기존의 안드로이드 앱 뿐만 아니라 웹, iOS까지 확장하여 최대한 많은 수원대학교 학생들이 SUWIKI 서비스를 사용할 수 있도록 확장하고 있습니다.

서비스 배포 계획은 2학기가 시작하기 전인 **7월 중**으로 계획하고 있으며 현재는 디자이너도 합류하여 좀 더 좋은 사용자 경험을 주고자 노력하고 있습니다.

## 👨‍👩‍👧‍👦프로젝트 팀원 구성은 어떻게 되나요?

현재 `안드로이드 2명`, `iOS 1명`, `웹 프론트엔드 3명`, `백엔드 2명`으로 구성되어 있습니다.

저는 안드로이드 파트를 맡고 있고, `SUWIKI`의 전선이 되는 `수타` 개발자와 함께 안드로이드 앱을 확장 개발하고 있습니다.

## 😎프로젝트 내에서 맡은 역할은?

기존의 `수타`에서 시간표 기능이 제공되었기 때문에 저는 시간표 외의 파트를 맡게 되었습니다. 주로 회원 관리와 관련된 파트입니다. **회원 가입, 로그인, 로그 아웃, 아이디 찾기, 비밀번호 찾기, 비밀번호 변경, 회원 탈퇴, 내가 쓴 글(강의 평가, 시험 정보), 시험 정보 구매 이력 관리 등**을 비롯해서 **공지 사항, 문의하기 등**과 같은 부분까지 담당하고 있습니다.

## 🚀기존 프로젝트에서 개선한 사항

기존의 `수타`는 시간표 기능 만을 제공했기 때문에 대부분이 Activity로 구성되어 있었습니다. 앞서 언급한 다양한 기능을 확장하고 사용자들이 제공되는 기능들을 좀 더 쉽게 접근하기 위해서 `Jetpack Navigation`을 사용한 하단 네비게이션 바 구조로 변경했습니다.

그리고 파이어베이스 만으로 시간표를 저장 및 제공했었는데 다양한 기능이 추가되고 서버 개발 인원이 추가 됨에 따라 Retrofit2로 네트워크 통신하는 부분을 추가했습니다.

혼자 개발했던 앱을 이제는 협업을 하며 키워 나가야 했기 때문에 `Git commit convention`을 정하고 `Branch 전략` 등을 함께 세웠고, 더 좋은 확장성과 유지 보수성을 고려해서 `MVVM`을 지향하는 아키텍처를 적용했습니다.

## ⭐Android에서 사용된 주요 기술 및 라이브러리

    굵게 표시한 항목은 직접 다룬 항목입니다

- **Jetpack Navigation**
- Jetpack Room
- **Retrofit2**
- **Gson converter**
- **OkHttp3**
  - Authenticator
  - Interceptor
- **Coroutine**
- **JWT**
- Firebase firestore
- Custom View
- **Repository Pattern**
- **MVVM Architecture**
- **EncryptedSharedPreference**
- Admob

## ⚒️사용하고 있는 협업 툴

- Git
- [GitHub][GitHub]
- JANDI
- Trello
- Whale On
- Gather Town

## 🖥️소스 코드도 확인해보세요

[🔗소스 코드 보러가기 §(*￣▽￣*)§][GitHub]

[GitHub]: https://github.com/uswLectureEvaluation/Android

## 🔒~~앱 설치해보기~~ (현재 배포 전)

[SUWIKI](https://play.google.com/store/apps/details?id=com.kunize.uswtimetable) (현재 `수타`)

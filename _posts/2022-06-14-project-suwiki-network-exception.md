---
layout: post
title: "[SUWIKI] 문제 해결 - 네트워크 에러 핸들링"
subtitle: Sandwich를 사용한 네트워킹 핸들링 🥪
categories: Project
tags: [suwiki,network,exception,error]
---

## 문제 발생

---

잘 동작하던 앱이 갑자기 켜자마자 앱이 죽어버리는 버그가 발생했다. 로그를 확인해보니 네트워킹을 하던 중 죽은 것 같은데 응답을 받지 못한 상태로 터진 것을 확인했다.

원인을 파악해보니 Base URL로 사용하던 주소가 충돌을 일으키면서 `UnKnownHostException`가 발생한 것이다.

Suwiki 안드로이드 앱은 앱을 켜면 자동 로그인 여부를 확인한 후 `true`라면 사용자 정보를 받아온다. 또한 시간표 버전을 확인하기 위해서도 서버와 통신을 한다. 어쨌는 앱을 켤 때 통신을 하게 되면서 바로 앱이 죽어버리는 것이었다. 즉 통신 중 예외가 발생했을 때에 대한 대처가 안 되어 있었던 것이다.

## 해결 방법 찾아보기

---

배포 전에 이런 문제가 발생해서 다행이라고 생각하고 해결 방법을 찾아보고자 했다.

이미 선배 개발자들이 수없이 겪은 문제이고 최적의 방법으로 해결했을 거라 기대하고 구글링을 했다.

그 중 한 [블로그]가 눈에 띄었다. (사실 이전에 한 번 읽어봤던 글이었다) 바로 그 유명한 skydoves 님의 글이었다. 해당 글에서는 `Retrofit`을 사용한 통신 중 발생하는 에러나 예외를 처리할 때의 문제점이나 아키텍처 관점의 문제점, 보일러 플레이트 등에 대해 설명하고, 이러한 문제를 해결하기 위해 `Sandwich`라는 라이브러리를 개발했다고 설명한다. 나는 `sandwich`를 사용해 Suwiki의 네트워킹 문제도 해결하고자 했다.

## 해결해보자

---

### 의존성 추가

```gradle
dependencies {
    implementation "com.github.skydoves:sandwich:1.2.5"
}
```

### 응답 래퍼 클래스 변경 (Response -> ApiResponse)

기존의 대부분의 API 응답은 `Retrofit`의 `Response`로 감싸져 있었다. 이를 `sandwich`에서 제공하는 `ApiResponse`로 변경한다.

`ApiResponse`는 아주 간단한 형태로 통신 성공 상황에서의 body data나 예외 상황에서의 페이로드 등을 처리할 수 있도록 해준다.

내부를 살펴보면 다음과 같이 sealed class로 만들어져 있다. (추상화 된 코드)

```kotlin
public sealed class ApiResponse<out T> {
  public data class Success<T>(val response: Response<T>) : ApiResponse<T>() {
    val data: List<Poster>? = response.data
    val statusCode: StatusCode = response.statusCode
    val headers: Headers = response.headers
  }
  public sealed class Failure<T> {
    public data class Error<T>(val response: Response<T>) : ApiResponse<T>() {
      val message: String = response.message()
      val errorBody: ResponseBody? = response.errorBody
      val statusCode: StatusCode = response.statusCode
      val headers: Headers = response.headers
    }
    public data class Exception<T>(val exception: Throwable) : ApiResponse<T>() {
      val message: String? = exception.localizedMessage
    }
  }
  // ...
}
```

Success와 Failure가 있고, Failure는 다시 Error와 Exception으로 구분된다. Error와 Exception을 sealed class로 묶음으로써 Error와 Exception을 한 번에 처리할 수 있는 장점도 챙겼다. 👍👍

내가 직면한 문제는 Exception 에서 처리 되어야 할 것이다.

### Call Adapter Factory 추가

```kotlin
.addCallAdapterFactory(ApiResponseCallAdapterFactory.create())
```

위 코드를 `Retrofit.Builder`에 추가한다.

![retrofit.builder](https://user-images.githubusercontent.com/44221447/173775163-5bfe8b29-0762-41fe-97b8-6f22e3fc4135.png)

### 기존 코드 대응

DataSource 인터페이스나 구현체인 RemoteDataSource, 그리고 Repository, ViewModel 등 기존의 Response로 응답 타입이 되어있던 부분을 ApiResponse로 변경해준다.

![dataSource](https://user-images.githubusercontent.com/44221447/173776416-b87953aa-b490-4655-bcc5-0bc18683e4f1.png)

성공, 실패 분기는 다음과 같이 두 가지 형태로 할 수 있다.

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    val response = IRetrofit.getInstance().getUserData()
    response.onSuccess {
        // 로그인
    }.onFailure {
        Log.e(TAG, "User - login() failed: $this")
        error.postValue("로그인에 실패했습니다")
    }
}
```

```kotlin
CoroutineScope(Dispatchers.IO).launch {
    val response = IRetrofit.getInstance().getUserData()
    when(response) {
        is Success -> { /* 로그인 */ }
        is ApiResponse.Failure.Error -> // 에러 응답 처리
        is ApiResponse.Failure.Exception -> // 예외 처리
    }
}
```

### 앱 구동 성공

네트워크 예외로 시작과 동시에 터지던 것과 달리 이제는 앱을 켤 수 있게 되었다. 물론 서버 문제가 해결된 것은 아니라서 아직 API 요청이 불가해 제대로 된 서비스를 이용할 수는 없지만 최소한 앱을 그냥 죽여버리는 것이 아닌 어떤 상황인지 유저에게 알릴 수 있게 되었으며, 예상치 못한 상황에서도 정상적으로 동작할 수 있는 기능들은 계속해서 제공할 수 있는 버그 픽스가 되었다.

서버 통신이 불가한 상황에서 어떤 식으로 사용자에게 알리고, 기능을 제공해 줄 지에 대한 방법을 고민해야 할 것이다.

## 결론

---

sandwich 라이브러리를 사용해서 간단하게 네트워킹 실패를 처리할 수 있었다. 여기서 소개한 내용은 아주 아주 기본적인 내용만 있지만 sandwich는 Flow를 사용하는 환경 등 아주 다양한 상황을 커버할 수 있도록 정교하게 만들어진 라이브러리이다 보니 좀 더 공부하고 최적의 방법을 적용해보려 한다. 물론 편리한 라이브러리 사용에 실제 네트워킹이 어떻게 이루어지고 어떻게 에러를 핸들링해야 할 지 고민해보는 것을 덮어두지 않으리라 다짐하며 글을 마친다.

## 참고

[https://velog.io/@skydoves/retrofit-api-handling-sandwich][블로그]
[https://github.com/skydoves/sandwich][깃헙]

[블로그]: https://velog.io/@skydoves/retrofit-api-handling-sandwich
[깃헙]: https://github.com/skydoves/sandwich

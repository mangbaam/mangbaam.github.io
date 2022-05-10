---
layout: post
title: "[SUWIKI] Networking"
subtitle: 🎈네트워킹은 이렇게 구현했어요
categories: Project
tags: [suwiki]
comments: true
---
## Intro

`SUWIKI`에서 서버 통신을 위한 구성을 어떻게 했는지 포스팅합니다.

네트워킹을 위해 사용한 기술은 `Retrofit2` 입니다.

API는 `REST API`를 따릅니다. (이에 대한 내용은 백엔드와 관련된 내용이므로 자세히 설명하지 않습니다)

> 이하 내용에서는 **편한 말투**로 진행하겠습니다 😉

---

## Dependencies 추가


``` gradle
/* Retrofit */
def retrofit_version = "2.9.0"
implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
implementation "com.squareup.okhttp3:logging-interceptor:4.9.3"
implementation "com.jakewharton.retrofit:retrofit2-kotlin-coroutines-adapter:0.9.2"
implementation "com.squareup.retrofit2:converter-scalars:2.9.0"
```

*build.gradle (app)*

네트워크 통신을 위한 `Retrofit2`와 JSON 데이터 응답 값을 객체로 변환하기 위한 `Gson Converter`, 통신 중간에 가로채는 `Loggin Intercepter`, 네트워크 통신을 Coroutine을 사용해 비동기 처리하기 위한 `Coroutine adatper` 등을 추가했다.

## Retrofit 인터페이스

*IRtrofit.kt*
![IRetrofit](https://user-images.githubusercontent.com/44221447/167475994-935e1ee7-4638-4cda-836d-79bc222901c6.png)

위 사진을 보면 `@POST`와 `@GET`을 확인할 수 있다. API 명세에 따라 POST 방식을 사용할 것인지, GET 방식을 사용할 것인지 구분한다. 주로 서버로부터 데이터를 내려 받을 때는 `GET`을 사용하고, 데이터를 확인하거나 요청 내용을 숨기기 위해 `POST`를 사용했다. `GET` 방식을 사용하면 요청 쿼리가 주소창에 노출되기 때문이다.

사진에서도 알 수 있듯이 `POST` 방식은 `Body`에 필요한 파라미터를 넣는다.

다음으로는 각 함수 앞에 붙은 `suspend` 키워드를 볼 수 있다. `suspend` 키워드는 `Coroutine`을 사용하기 위해 붙은 키워드이다. 네트워크 통신은 상당히 비용이 큰 작업이다. 만약 안드로이드의 메인 스레드인 `UI 스레드`에서 IO 작업을 하게 되면 작업이 진행되는 동안 사용자가 앱을 사용할 수 없어서 불편할 뿐만 아니라 5초가 지나게 되면 `ANR(Application Not Responding)`이 발생하면서 앱이 죽게 된다. 이러한 이유로 네트워킹과 같은 IO 작업은 비동기로 이루어지며 주로 `RxJava`나 `Coroutine`을 사용해서 비동기 작업을 하게 된다.

*Constants.kt*
![constant](https://user-images.githubusercontent.com/44221447/167480543-a5981510-6c00-4bc9-b6e8-3b28e39d2f54.png)
다음으로 알 수 있는 것은 요청 파라미터를 `SIGN_UP`과 같은 방식으로 상수로 관리했다. 이렇게 하면 API 요청 파라미터가 변경되었을 때 빠르게 찾아서 변경할 수 있고, 의존성을 낮출 수 있다. 또한 캡슐화를 통해 어떠한 기능인지 더 명확히 알기 쉽고 세부 내용을 감출 수 있다.

응답은 `Response`로 감싸서 받는다. `Response`로 감싸면 **응답 코드와 메시지, 통신 성공 여부**를 알 수 있다. 통신에 실패했을 때 사용자에게 적절히 알리고 예외 처리하기 위해 `Response`를 사용했다.

## 인스턴스 만들기

이 부분은 별도로 분리해서 만들 수도 있지만 이 프로젝트에서는 `IRetrofit.kt`에 인터페이스와 같이 작성했다. `companion object`에 작성되었다.

API 요청을 할 때마다 `Retrofit`의 인스턴스를 새로 만드는 것은 굉장히 큰 비용이 소모되기 때문에 주로 한 번 만든 인스턴스를 재활용하는 싱글톤 방식으로 사용한다.

`SUWIKI`에서는 2개의 인스턴스를 만들었다. 하나는 JWT 토큰이 필요 없는 API 작업에 사용하는 인스턴스이고, 다른 하나는 API 요청 시 헤더에 JWT 토큰을 요구할 때 사용하는 인스턴스이다.

예를 들어 공지사항 확인과 같이 로그인이 필요 없는 요청을 할 때와 시험 정보를 포인트로 구매 하는 등 인증 및 인가가 필요한 작업이 서로 다른 인스턴스를 사용하게 된다.

물론 하나의 인스턴스를 만들어서 JWT 토큰이 필요한 경우에 위에서 작성한 인터페이스에 직접 넣어줘도 되지만 대부분의 API 요청이 토큰을 요구하므로 별도의 인스턴스를 만들게 되었다.

### 인스턴스 생성

*IRetrofit.kt*

```kotlin
companion object{
    private var retrofitService: IRetrofit? = null
        private var retrofitServiceWithNoToken: IRetrofit? = null

        fun getInstance(): IRetrofit {
            if (retrofitService == null) {
                val client = getClient()
                retrofitService = client.create(IRetrofit::class.java)
            }
            return retrofitService!!
        }

        fun getInstanceWithNoToken(): IRetrofit {
            if (retrofitServiceWithNoToken == null) {
                val client = getClientWithNoToken()
                retrofitServiceWithNoToken = client.create(IRetrofit::class.java)
            }
            return retrofitServiceWithNoToken!!
        }

    // ...
}
```

토큰을 필요로 하지 않는 `retrofitServiceWithNoToken`와 토큰을 헤더에 넣어주는 `retrofitService`에 인스턴스를 생성하는 코드이다. 두 부분의 차이라고 한다면 `client` 부분이다.

### Retrofit 객체 생성

*IRetrofit.kt*

```kotlin
companion object {
    // ...

    private fun getClient(): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(getOkHttpClient(TokenAuthenticator()))
            .addConverterFactory(gsonConverterFactory())
            .build()
    }

    private fun getClientWithNoToken(): Retrofit {
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(getOkHttpClient(null))
            .addConverterFactory(gsonConverterFactory())
            .build()
    }

    // ...
}
```

이 부분에서는 API의 베이스 URL을 넣어서 `Retrofit` 객체를 생성한다. `client`에는 `OkHttpClient` 객체를 넣어줬고, `ConverterFactory`는 `Gson Converter`를 사용했다. 서버에서 내려준 `JSON` 타입의 데이터를 자바 객체로 변환해주는 컨버터이다.
(서버에서 내려주는 타입이 `JSON` 타입이 아닌 경우 `ScalarsConverter`를 사용했는데 내가 작성한 부분이 아니라 제외했다)

`getClient()`와 `getClientWithNoToken()`의 차이는 `client`를 추가하는 부분에서 `getOkHttpClient()` 파라미터가 `null`인지 아닌지 차이가 있다.

하나씩 살펴보도록 하자.

### OkHttpClient

서버와의 통신에서 발생하는 request, response 로그를 가로채서 확인할 수 있도록 `OkHttpClient`의 `HttpLoggingInterceptor`를 사용했다.

*IRetrofit.kt*

```kotlin
companion object {
    // ...

    private fun getOkHttpClient(
            authenticator: TokenAuthenticator?
    ): OkHttpClient {
        val loggingInterceptor = HttpLoggingInterceptor { message ->
            when {
                message.isJsonObject() ->
                    Log.d(TAG, JSONObject(message).toString(4))
                message.isJsonArray() ->
                    Log.d(TAG, JSONArray(message).toString(4))
                else ->
                    Log.d(TAG, "CONNECTION INFO -> $message")
            }
        }
        loggingInterceptor.level = HttpLoggingInterceptor.Level.BODY

        val client = OkHttpClient.Builder().addInterceptor(loggingInterceptor)
        authenticator?.apply {
            client
                .addInterceptor(AuthenticationInterceptor())
                .authenticator(this)
        }

        return client.build()
    }

    //..
}

```

로그 메시지가 `JSON 객체` 형태일 때와 `JSON 배열` 형태일 때 각자의 타입에 맞게 출력하고, 그 외의 정보는 그냥 출력한다.

가로챈 로그는 문자열 타입인데 이를 어떤 형태인지 구분하기 위해 Kotlin의 확장 함수 기능을 사용해서 판별했다. 확장 함수를 사용한 부분은 다음과 같다.

*Extention.kt*

```kotlin
// 문자열 -> json 형태인지 json 배열 형태인지
fun String?.isJsonObject(): Boolean = this?.startsWith("{") == true && this.endsWith("}")
fun String?.isJsonArray(): Boolean = this?.startsWith("[") == true && this.endsWith("]")
```

*로그*
![log](https://user-images.githubusercontent.com/44221447/167576646-4d15d079-1f27-4374-8ef1-4ae02614f82a.png)

위와 같은 로그를 받아볼 수 있다. 위 로그는 현재 로그인 한 유저의 정보를 받아오는 API 요청에 대한 응답이다. 응답 코드로 200번을 받았고, 중간에 *intercept() called* 라는 부분이 확인할 수 있다. 그리고 `JSON 객체 타입`은 탭(4개의 Space) 되어서 찍히고, 그 외의 데이터는 평문으로 찍힌다.

이렇게 로그를 자세히 확인할 수 있는 것은 `LoggingInterceptor` 레벨을 `BODY`로 설정했기 때문이다.

여기까지 `LoggingInterceptor`를 만드는 과정이었고, `OkHttpClient.Builder()`를 사용해 `OkHttpClient`객체를 만들고 앞서 만든 `LoggingInterceptor`를 추가한다.

그리고 [위에서](#retrofit-객체-생성) `getClient()`와 `getClientWithNoToken()`의 차이가 `getOkHttpClient()` 메소드의 매개 변수(`authenticator`)로 `null`의 유무였다.

`authenticator`를 매개 변수로 받았다면 매개 변수로 받은 `authenticator`와 `AuthenticationInterceptor`의 값을 추가로 `OkHttpClient.Builder()`에 추가한다.

`TokenAuthenticator`(authenticator)와 `AuthenticationInterceptor`는 바로 밑에서 살펴보도록 하자.

### 헤더에 Access 토큰 삽입

`AuthenticationInterceptor` 클래스는 `OkHttp3`의 [Interceptor][okhttp3.interceptors] 클래스를 상속한다.

이 클래스는 API 요청을 가로채서 JWT의 액세스 토큰을 헤더에 삽입해서 요청을 보내주는 역할을 한다. 앞서 `getInstance()`와 `getInstanceWithNoToken()`의 차이가 바로 여기서 발생하는 것이다.

*IRetrofit.kt*

```kotlin
class AuthenticationInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): okhttp3.Response {
        val accessToken = TimeTableSelPref.encryptedPrefs.getAccessToken() ?: ""
        val request = chain.request().newBuilder()
            .addHeader(AUTH_HEADER, accessToken).build()
        Log.d(
            TAG,
            "AuthenticationInterceptor - intercept() called / request header: ${request.headers}"
        )
        return chain.proceed(request)
    }
}
```

이 프로젝트에서 JWT는 `EncyptedSharedPreference`를 사용해 저장되어 있다. 여기에 필요한 `Context`는 `ApplicationContext`를 사용했기 때문에 이 클래스에 `Context`를 별도로 주입하지 않고 사용할 수 있었다.

### Access 토큰 갱신

`TokenAuthenticator` 클래스는 `OkHttp3`의 [Authenticator][okhttp3.authenticator] 클래스를 상속한다.

이 클래스는 [헤더에 토큰을 싣고 요청](#authenticationinterceptor)했을 때 해당 토큰이 만료된 상태라면 Refresh 토큰을 사용해 서버로부터 Access 토큰을 재발급 받고, 재발급 된 토큰을 가지고 기존에 중단됐던 요청을 이어나갈 수 있는 역할을 한다.


*IRetrofit.kt*

```kotlin
class TokenAuthenticator : Authenticator {
    override fun authenticate(route: Route?, response: okhttp3.Response): Request? {

        val refresh = TimeTableSelPref.encryptedPrefs.getRefreshToken() ?: ""
        Log.d(TAG, "TokenAuthenticator - authenticate() called / 토큰 만료. 토큰 Refresh 요청: $refresh")
        val tokenResponse =
            IRetrofit.getInstanceWithNoToken().requestRefresh(refresh).execute()

        return if (handleResponse(tokenResponse)) {
            Log.d(TAG, "TokenAuthenticator - authenticate() called / 중단된 API 재요청")
            response.request
                .newBuilder()
                .removeHeader(AUTH_HEADER)
                .header(AUTH_HEADER, TimeTableSelPref.encryptedPrefs.getAccessToken() ?: "")
                .build()
        } else {
            null
        }
    }

    private fun handleResponse(tokenResponse: Response<Token>) =
        if (tokenResponse.isSuccessful && tokenResponse.body() != null) {
            TimeTableSelPref.encryptedPrefs.saveAccessToken(tokenResponse.body()!!.accessToken)
            TimeTableSelPref.encryptedPrefs.saveRefreshToken(tokenResponse.body()!!.refreshToken)
            true
        } else {
            User.logout()
            Log.d(TAG, "TokenAuthenticator - handleResponse() called / 리프레시 토큰이 만료되어 로그 아웃 되었습니다.")
            false
        }

    companion object {
        const val AUTH_HEADER = "Authorization"
    }
}
```

액세스 토큰이 만료된 경우 REST API에 따라 `401` 응답 코드를 내려준다. 응답 코드로 `401`을 받으면 `TokenAuthenticator`가 동작한다.

`EncryptedSharedPreference`에 저장된 `Refresh 토큰`을 가져와서 `Access 토큰`을 갱신하는 작업을 한 후에 중단되었던 API 요청을 재개한다.

만약 `Refresh 토큰`마저 만료되었다면 로그인 된 사용자를 **로그 아웃** 시키고, 중단된 작업을 재개하지 않는다.

*토큰 만료*
![token expired](https://user-images.githubusercontent.com/44221447/167582290-fc7eb335-c3db-46e7-af62-6c403d765baf.png)

*중단된 요청 재수행*
![resume api request](https://user-images.githubusercontent.com/44221447/167582766-0b72fe98-13e5-406d-82cb-446caa4055b6.png)

[okhttp3.authenticator]: https://square.github.io/okhttp/4.x/okhttp/okhttp3/-authenticator/
[okhttp3.interceptors]: https://square.github.io/okhttp/features/interceptors/

### LocalDateTime 다루기
[Retrofit 객체 생성 부분](#retrofit-객체-생성)의 `ConverterFactory`를 추가하는 부분에서 `gsonConverterFactory()` 라는 메소드를 사용했다.

이 부분은 원래 다음과 같이 작성되었었다.

*IRetrofit*

```kotlin
companion object {
    //...

    private fun getClient(): Retrofit {
        return Retrofit.Builder()
            .baseUrl( /* ... */ )
            .client( /* ... */ )
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    // ...
}
```

만약 서버의 응답 값으로 `LocalDateTime`을 받게 된다면 위의 방법으로는 `Gson`이 `String` 타입으로 변환해버려서 `LocalDateTime`으로 사용할 수 없게 된다.

*Error*
![LocalDateTime_Type_Error](https://user-images.githubusercontent.com/44221447/167625596-5a76daa2-7473-4b3e-b863-7fe0aa79e33f.png)

이를 해결하기 위해 별도의 메소드로 추출하게 되었다.

*IRetrofit*

```kotlin
companion object {
    // ...

    private fun getClient(): Retrofit {
        return Retrofit.Builder()
            .baseUrl( /* ... */ )
            .client( /* ... */ )
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    private fun gsonConverterFactory(): GsonConverterFactory {
        val gson = GsonBuilder()
            .registerTypeAdapter(LocalDateTime::class.java, object: JsonDeserializer<LocalDateTime> {
                override fun deserialize(
                    json: JsonElement?,
                    typeOfT: Type?,
                    context: JsonDeserializationContext?
                ): LocalDateTime {
                    return LocalDateTime.parse(json?.asString, DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm:ss"))
                }
            })
            .create()
        return GsonConverterFactory.create(gson)
    }
}
```

위 과정을 통해 `LocalDateTime`이 `String` 타입으로 변한 것을 `역 직렬화(Deserialize)`하여 다시 `LocalDateTime` 타입으로 사용할 수 있다.

> 직렬화: 자바 객체를 **JSON 타입으로 변환**
>
> 역 직렬화: JSON 타입 데이터를 **자바 객체로 변환**. 어떤 타입으로 변환할 지 명시해야 한다.

---

## API 호출 예제 (시험 정보 구매 이력 확인)

### Retrofit 인터페이스 선언

*IRetrofit.kt*

```kotlin
interface IRetrofit {
    @GET(PURCHASE_HISTORY) // PURCHASE_HISTORY에는 Base Url 이후의 주소가 들어감
    suspend fun getPurchaseHistory(): Response<PurchaseHistoryDto>
}
```

### DTO

*PurchaseHistoryDto.kt*

```kotlin
import java.io.Serializable
import java.time.LocalDateTime

data class PurchaseHistoryDto(
    val data: List<PurchaseHistory>
) : Serializable

data class PurchaseHistory(
    val id: Long,
    val lectureName: String,
    val professor: String,
    val majorType: String,
    val createDate: LocalDateTime
) : Serializable
```

*API 명세서*
![api spec](https://user-images.githubusercontent.com/44221447/167621638-e42c954d-e57e-4da2-8787-4fab960c4632.png)

API 명세서에 맞게 Data Class를 작성해야 한다. 위 명세서에서는 `data`라는 key 값으로 `JSON 배열` 형태의 value를 내려주는데 배열의 각 아이템을 이루는 **id, lectureName, professor, majorType, createDate** 에 맞춰서 `PurchaseHistory`라는 Data Class를 작성했다.

그리고 `PurchaseHistoryDto`라는 이름의 `PurchaseHistory` 리스트를 가지는 Data Class를 만들어 `PurchaseHistoryDto`로 응답을 받게 된다.

### ViewModel

*PurchaseHistoryViewModel.kt*

```kotlin
class PurchaseHistoryViewModel(private val repository: MyPostRepository) : ViewModel() {

    // ...
    private val _historyList = MutableLiveData<List<PurchaseHistory>>()
    val historyList: LiveData<List<PurchaseHistory>> get() = _historyList

    init {
        getHistory()
    }

    private fun getHistory() {
        viewModelScope.launch {
            val response = repository.getPurchaseHistory()
            if (response.isSuccessful) {
                if (response.body()?.data?.isEmpty() == true) {
                    // 데이터가 없을 때 처리
                }
                response.body()?.data?.let { _historyList.postValue(response.body()?.data) }
            } else {
                // 통신 에러 처리
            }
        }
    }
}
```

IO 작업을 비동기로 처리하기 위해 `Coroutine`을 사용했다. ViewModel에서는 ViewModel 생명 주기에 맞춰서 동작하는 `viewModelScope`를 사용할 수 있다.

MyPostRepository에 getPurchaseHistory()를 호출하는 부분을 확인할 수 있다. 응답은 Response로 감싼 값을 받으므로 `response.isSuccessful`로 통신 성공 여부를 확인할 수 있고, [`PurchaseHistoryDto`](#dto)의 `data` 키 값으로 데이터를 확인할 수 있다.

받아온 데이터가 있다면 라이브데이터에 받아온 데이터를 담는다.

액티비티나 프래그먼트에서 `historyList`의 값을 관찰하고 있다가 값이 변경되면 RecyclerView Adapter의 `submitList`로 데이터를 변경해주는 방식으로 처리한다.

### Repository

데이터는 지금처럼 네트워크 통신을 통해 가져올 수도 있고, SQLite DB에서 가져올 수도 있고, 내부 Asset 파일에서 가져올 수도 있고, 다양한 데이터 소스가 존재할 수 있다.

ViewModel에서는 비즈니스 로직을 수행하지만 데이터가 어디서 왔는지는 몰라도 되도록 Repository 패턴을 사용한다.

[*구글 개발자 문서*](https://developer.android.com/jetpack/guide?hl=ko#data-layer)
![repository_pattern](https://user-images.githubusercontent.com/44221447/167629187-db1a82f6-4831-458d-8a87-3c24a362dcdf.png)

*MyPostRepository*

```kotlin
class MyPostRepository(private val dataSource: MyPostRemoteDataSource) {
    // ...
    suspend fun getPurchaseHistory() = withContext(Dispatchers.IO) { dataSource.getPurchaseHistory() }
}
```

`MyPostRepository`에서 매개 변수로 dataSource를 주입 받는다. `MyPostRemoteDateSource`에서는 실질적으로 네트워크 통신을 통해 데이터를 받아오는 과정이 수행된다.

### Data Source

*MyPostRemoteDataSource*

```kotlin
class MyPostRemoteDataSource(private val apiService: IRetrofit): MyPostDataSource {
    // ...
    override suspend fun getPurchaseHistory() = apiService.getPurchaseHistory()
}
```

DataSource에서 매개 변수로 apiService를 주입 받는다. [`IRetrofit`](#retrofit-객체-생성)은 위에서 한참 설명했던 바로 그 인스턴스이다.

위 코드를 보면 `MyPostDataSource`를 상속하는 것을 볼 수 있다. `MyPostDataSource`는 다양한 데이터 소스를 통일화하기 위해 메서드를 강제하는 인터페이스이다.

*MyPostDataSource*

```kotlin
interface MyPostDataSource {
    // ...
    suspend fun getPurchaseHistory(): Response<PurchaseHistoryDto>
}
```

### 의존성 주입

A 클래스 내부에서 B 클래스의 객체가 필요할 때 A 클래스 내부에서 직접 B 클래스의 객체를 생성하면 두 클래스 간의 결합도가 올라가게 된다. 객체 지향 설계에서 클래스 간의 결합도를 낮추고 응집도를 높이는 설계는 중요하다.

이렇게 결합도가 높아지는 것을 방지하기 위해 **생성자 주입 방식, 필드 주입 방식, 수정자 주입 방식 등** 다양한 방법이 고안되었다.

`Dagger Hilt`와 같은 라이브러리도 많이 사용하는 걸로 알고 있다. (아직 Hilt에 대해 학습하지 않은 상태라 프로젝트에 적용되지는 않았다. 현재 프로젝트는 생성자 주입 방식을 채택했다)

현재 프로젝트에서 ViewModel을 사용할 때 ViewModel에 생성자로 apiService를 주입하기 위해서 `ViewModelFactory`를 사용해서 생성해주는 방식을 사용하고 있다.

*ViewModelFactory*

```kotlin
@Suppress("UNCHECKED_CAST")
class ViewModelFactory : ViewModelProvider.Factory {

    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return when {
            // ...
            modelClass.isAssignableFrom(PurchaseHistoryViewModel::class.java) -> {
                val apiService = IRetrofit.getInstance()
                val repository = MyPostRepository(MyPostRemoteDataSource(apiService))
                PurchaseHistoryViewModel(repository) as T
            }
        }
    }
}
```

## 전체 구조

![networking_structure](https://user-images.githubusercontent.com/44221447/167644878-81522ce3-df9e-44aa-b2fc-d8c04cb250fa.png)

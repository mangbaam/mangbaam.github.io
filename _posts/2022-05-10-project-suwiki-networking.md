---
layout: post
title: "[SUWIKI] Networking"
subtitle: πλ€νΈμνΉμ μ΄λ κ² κ΅¬ννμ΄μ
categories: Project
tags: [suwiki]
comments: true
---
## Intro

`SUWIKI`μμ μλ² ν΅μ μ μν κ΅¬μ±μ μ΄λ»κ² νλμ§ ν¬μ€νν©λλ€.

λ€νΈμνΉμ μν΄ μ¬μ©ν κΈ°μ μ `Retrofit2` μλλ€.

APIλ `REST API`λ₯Ό λ°λ¦λλ€. (μ΄μ λν λ΄μ©μ λ°±μλμ κ΄λ ¨λ λ΄μ©μ΄λ―λ‘ μμΈν μ€λͺνμ§ μμ΅λλ€)

> μ΄ν λ΄μ©μμλ **νΈν λ§ν¬**λ‘ μ§ννκ² μ΅λλ€ π

---

## Dependencies μΆκ°


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

λ€νΈμν¬ ν΅μ μ μν `Retrofit2`μ JSON λ°μ΄ν° μλ΅ κ°μ κ°μ²΄λ‘ λ³ννκΈ° μν `Gson Converter`, ν΅μ  μ€κ°μ κ°λ‘μ±λ `Loggin Intercepter`, λ€νΈμν¬ ν΅μ μ Coroutineμ μ¬μ©ν΄ λΉλκΈ° μ²λ¦¬νκΈ° μν `Coroutine adatper` λ±μ μΆκ°νλ€.

## Retrofit μΈν°νμ΄μ€

*IRtrofit.kt*
![IRetrofit](https://user-images.githubusercontent.com/44221447/167475994-935e1ee7-4638-4cda-836d-79bc222901c6.png)

μ μ¬μ§μ λ³΄λ©΄ `@POST`μ `@GET`μ νμΈν  μ μλ€. API λͺμΈμ λ°λΌ POST λ°©μμ μ¬μ©ν  κ²μΈμ§, GET λ°©μμ μ¬μ©ν  κ²μΈμ§ κ΅¬λΆνλ€. μ£Όλ‘ μλ²λ‘λΆν° λ°μ΄ν°λ₯Ό λ΄λ € λ°μ λλ `GET`μ μ¬μ©νκ³ , λ°μ΄ν°λ₯Ό νμΈνκ±°λ μμ²­ λ΄μ©μ μ¨κΈ°κΈ° μν΄ `POST`λ₯Ό μ¬μ©νλ€. `GET` λ°©μμ μ¬μ©νλ©΄ μμ²­ μΏΌλ¦¬κ° μ£Όμμ°½μ λΈμΆλκΈ° λλ¬Έμ΄λ€.

μ¬μ§μμλ μ μ μλ―μ΄ `POST` λ°©μμ `Body`μ νμν νλΌλ―Έν°λ₯Ό λ£λλ€.

λ€μμΌλ‘λ κ° ν¨μ μμ λΆμ `suspend` ν€μλλ₯Ό λ³Ό μ μλ€. `suspend` ν€μλλ `Coroutine`μ μ¬μ©νκΈ° μν΄ λΆμ ν€μλμ΄λ€. λ€νΈμν¬ ν΅μ μ μλΉν λΉμ©μ΄ ν° μμμ΄λ€. λ§μ½ μλλ‘μ΄λμ λ©μΈ μ€λ λμΈ `UI μ€λ λ`μμ IO μμμ νκ² λλ©΄ μμμ΄ μ§νλλ λμ μ¬μ©μκ° μ±μ μ¬μ©ν  μ μμ΄μ λΆνΈν  λΏλ§ μλλΌ 5μ΄κ° μ§λκ² λλ©΄ `ANR(Application Not Responding)`μ΄ λ°μνλ©΄μ μ±μ΄ μ£½κ² λλ€. μ΄λ¬ν μ΄μ λ‘ λ€νΈμνΉκ³Ό κ°μ IO μμμ λΉλκΈ°λ‘ μ΄λ£¨μ΄μ§λ©° μ£Όλ‘ `RxJava`λ `Coroutine`μ μ¬μ©ν΄μ λΉλκΈ° μμμ νκ² λλ€.

*Constants.kt*
![constant](https://user-images.githubusercontent.com/44221447/167480543-a5981510-6c00-4bc9-b6e8-3b28e39d2f54.png)
λ€μμΌλ‘ μ μ μλ κ²μ μμ²­ νλΌλ―Έν°λ₯Ό `SIGN_UP`κ³Ό κ°μ λ°©μμΌλ‘ μμλ‘ κ΄λ¦¬νλ€. μ΄λ κ² νλ©΄ API μμ²­ νλΌλ―Έν°κ° λ³κ²½λμμ λ λΉ λ₯΄κ² μ°Ύμμ λ³κ²½ν  μ μκ³ , μμ‘΄μ±μ λ?μΆ μ μλ€. λν μΊ‘μνλ₯Ό ν΅ν΄ μ΄λ ν κΈ°λ₯μΈμ§ λ λͺνν μκΈ° μ½κ³  μΈλΆ λ΄μ©μ κ°μΆ μ μλ€.

μλ΅μ `Response`λ‘ κ°μΈμ λ°λλ€. `Response`λ‘ κ°μΈλ©΄ **μλ΅ μ½λμ λ©μμ§, ν΅μ  μ±κ³΅ μ¬λΆ**λ₯Ό μ μ μλ€. ν΅μ μ μ€ν¨νμ λ μ¬μ©μμκ² μ μ ν μλ¦¬κ³  μμΈ μ²λ¦¬νκΈ° μν΄ `Response`λ₯Ό μ¬μ©νλ€.

## μΈμ€ν΄μ€ λ§λ€κΈ°

μ΄ λΆλΆμ λ³λλ‘ λΆλ¦¬ν΄μ λ§λ€ μλ μμ§λ§ μ΄ νλ‘μ νΈμμλ `IRetrofit.kt`μ μΈν°νμ΄μ€μ κ°μ΄ μμ±νλ€. `companion object`μ μμ±λμλ€.

API μμ²­μ ν  λλ§λ€ `Retrofit`μ μΈμ€ν΄μ€λ₯Ό μλ‘ λ§λλ κ²μ κ΅μ₯ν ν° λΉμ©μ΄ μλͺ¨λκΈ° λλ¬Έμ μ£Όλ‘ ν λ² λ§λ  μΈμ€ν΄μ€λ₯Ό μ¬νμ©νλ μ±κΈν€ λ°©μμΌλ‘ μ¬μ©νλ€.

`SUWIKI`μμλ 2κ°μ μΈμ€ν΄μ€λ₯Ό λ§λ€μλ€. νλλ JWT ν ν°μ΄ νμ μλ API μμμ μ¬μ©νλ μΈμ€ν΄μ€μ΄κ³ , λ€λ₯Έ νλλ API μμ²­ μ ν€λμ JWT ν ν°μ μκ΅¬ν  λ μ¬μ©νλ μΈμ€ν΄μ€μ΄λ€.

μλ₯Ό λ€μ΄ κ³΅μ§μ¬ν­ νμΈκ³Ό κ°μ΄ λ‘κ·ΈμΈμ΄ νμ μλ μμ²­μ ν  λμ μν μ λ³΄λ₯Ό ν¬μΈνΈλ‘ κ΅¬λ§€ νλ λ± μΈμ¦ λ° μΈκ°κ° νμν μμμ΄ μλ‘ λ€λ₯Έ μΈμ€ν΄μ€λ₯Ό μ¬μ©νκ² λλ€.

λ¬Όλ‘  νλμ μΈμ€ν΄μ€λ₯Ό λ§λ€μ΄μ JWT ν ν°μ΄ νμν κ²½μ°μ μμμ μμ±ν μΈν°νμ΄μ€μ μ§μ  λ£μ΄μ€λ λμ§λ§ λλΆλΆμ API μμ²­μ΄ ν ν°μ μκ΅¬νλ―λ‘ λ³λμ μΈμ€ν΄μ€λ₯Ό λ§λ€κ² λμλ€.

### μΈμ€ν΄μ€ μμ±

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

ν ν°μ νμλ‘ νμ§ μλ `retrofitServiceWithNoToken`μ ν ν°μ ν€λμ λ£μ΄μ£Όλ `retrofitService`μ μΈμ€ν΄μ€λ₯Ό μμ±νλ μ½λμ΄λ€. λ λΆλΆμ μ°¨μ΄λΌκ³  νλ€λ©΄ `client` λΆλΆμ΄λ€.

### Retrofit κ°μ²΄ μμ±

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

μ΄ λΆλΆμμλ APIμ λ² μ΄μ€ URLμ λ£μ΄μ `Retrofit` κ°μ²΄λ₯Ό μμ±νλ€. `client`μλ `OkHttpClient` κ°μ²΄λ₯Ό λ£μ΄μ€¬κ³ , `ConverterFactory`λ `Gson Converter`λ₯Ό μ¬μ©νλ€. μλ²μμ λ΄λ €μ€ `JSON` νμμ λ°μ΄ν°λ₯Ό μλ° κ°μ²΄λ‘ λ³νν΄μ£Όλ μ»¨λ²ν°μ΄λ€.
(μλ²μμ λ΄λ €μ£Όλ νμμ΄ `JSON` νμμ΄ μλ κ²½μ° `ScalarsConverter`λ₯Ό μ¬μ©νλλ° λ΄κ° μμ±ν λΆλΆμ΄ μλλΌ μ μΈνλ€)

`getClient()`μ `getClientWithNoToken()`μ μ°¨μ΄λ `client`λ₯Ό μΆκ°νλ λΆλΆμμ `getOkHttpClient()` νλΌλ―Έν°κ° `null`μΈμ§ μλμ§ μ°¨μ΄κ° μλ€.

νλμ© μ΄ν΄λ³΄λλ‘ νμ.

### OkHttpClient

μλ²μμ ν΅μ μμ λ°μνλ request, response λ‘κ·Έλ₯Ό κ°λ‘μ±μ νμΈν  μ μλλ‘ `OkHttpClient`μ `HttpLoggingInterceptor`λ₯Ό μ¬μ©νλ€.

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

λ‘κ·Έ λ©μμ§κ° `JSON κ°μ²΄` ννμΌ λμ `JSON λ°°μ΄` ννμΌ λ κ°μμ νμμ λ§κ² μΆλ ₯νκ³ , κ·Έ μΈμ μ λ³΄λ κ·Έλ₯ μΆλ ₯νλ€.

κ°λ‘μ± λ‘κ·Έλ λ¬Έμμ΄ νμμΈλ° μ΄λ₯Ό μ΄λ€ ννμΈμ§ κ΅¬λΆνκΈ° μν΄ Kotlinμ νμ₯ ν¨μ κΈ°λ₯μ μ¬μ©ν΄μ νλ³νλ€. νμ₯ ν¨μλ₯Ό μ¬μ©ν λΆλΆμ λ€μκ³Ό κ°λ€.

*Extention.kt*

```kotlin
// λ¬Έμμ΄ -> json ννμΈμ§ json λ°°μ΄ ννμΈμ§
fun String?.isJsonObject(): Boolean = this?.startsWith("{") == true && this.endsWith("}")
fun String?.isJsonArray(): Boolean = this?.startsWith("[") == true && this.endsWith("]")
```

*λ‘κ·Έ*
![log](https://user-images.githubusercontent.com/44221447/167576646-4d15d079-1f27-4374-8ef1-4ae02614f82a.png)

μμ κ°μ λ‘κ·Έλ₯Ό λ°μλ³Ό μ μλ€. μ λ‘κ·Έλ νμ¬ λ‘κ·ΈμΈ ν μ μ μ μ λ³΄λ₯Ό λ°μμ€λ API μμ²­μ λν μλ΅μ΄λ€. μλ΅ μ½λλ‘ 200λ²μ λ°μκ³ , μ€κ°μ *intercept() called* λΌλ λΆλΆμ΄ νμΈν  μ μλ€. κ·Έλ¦¬κ³  `JSON κ°μ²΄ νμ`μ ν­(4κ°μ Space) λμ΄μ μ°νκ³ , κ·Έ μΈμ λ°μ΄ν°λ νλ¬ΈμΌλ‘ μ°νλ€.

μ΄λ κ² λ‘κ·Έλ₯Ό μμΈν νμΈν  μ μλ κ²μ `LoggingInterceptor` λ λ²¨μ `BODY`λ‘ μ€μ νκΈ° λλ¬Έμ΄λ€.

μ¬κΈ°κΉμ§ `LoggingInterceptor`λ₯Ό λ§λλ κ³Όμ μ΄μκ³ , `OkHttpClient.Builder()`λ₯Ό μ¬μ©ν΄ `OkHttpClient`κ°μ²΄λ₯Ό λ§λ€κ³  μμ λ§λ  `LoggingInterceptor`λ₯Ό μΆκ°νλ€.

κ·Έλ¦¬κ³  [μμμ](#retrofit-κ°μ²΄-μμ±) `getClient()`μ `getClientWithNoToken()`μ μ°¨μ΄κ° `getOkHttpClient()` λ©μλμ λ§€κ° λ³μ(`authenticator`)λ‘ `null`μ μ λ¬΄μλ€.

`authenticator`λ₯Ό λ§€κ° λ³μλ‘ λ°μλ€λ©΄ λ§€κ° λ³μλ‘ λ°μ `authenticator`μ `AuthenticationInterceptor`μ κ°μ μΆκ°λ‘ `OkHttpClient.Builder()`μ μΆκ°νλ€.

`TokenAuthenticator`(authenticator)μ `AuthenticationInterceptor`λ λ°λ‘ λ°μμ μ΄ν΄λ³΄λλ‘ νμ.

### ν€λμ Access ν ν° μ½μ

`AuthenticationInterceptor` ν΄λμ€λ `OkHttp3`μ [Interceptor][okhttp3.interceptors] ν΄λμ€λ₯Ό μμνλ€.

μ΄ ν΄λμ€λ API μμ²­μ κ°λ‘μ±μ JWTμ μ‘μΈμ€ ν ν°μ ν€λμ μ½μν΄μ μμ²­μ λ³΄λ΄μ£Όλ μ­ν μ νλ€. μμ `getInstance()`μ `getInstanceWithNoToken()`μ μ°¨μ΄κ° λ°λ‘ μ¬κΈ°μ λ°μνλ κ²μ΄λ€.

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

μ΄ νλ‘μ νΈμμ JWTλ `EncyptedSharedPreference`λ₯Ό μ¬μ©ν΄ μ μ₯λμ΄ μλ€. μ¬κΈ°μ νμν `Context`λ `ApplicationContext`λ₯Ό μ¬μ©νκΈ° λλ¬Έμ μ΄ ν΄λμ€μ `Context`λ₯Ό λ³λλ‘ μ£Όμνμ§ μκ³  μ¬μ©ν  μ μμλ€.

### Access ν ν° κ°±μ 

`TokenAuthenticator` ν΄λμ€λ `OkHttp3`μ [Authenticator][okhttp3.authenticator] ν΄λμ€λ₯Ό μμνλ€.

μ΄ ν΄λμ€λ [ν€λμ ν ν°μ μ£κ³  μμ²­](#authenticationinterceptor)νμ λ ν΄λΉ ν ν°μ΄ λ§λ£λ μνλΌλ©΄ Refresh ν ν°μ μ¬μ©ν΄ μλ²λ‘λΆν° Access ν ν°μ μ¬λ°κΈ λ°κ³ , μ¬λ°κΈ λ ν ν°μ κ°μ§κ³  κΈ°μ‘΄μ μ€λ¨λλ μμ²­μ μ΄μ΄λκ° μ μλ μ­ν μ νλ€.


*IRetrofit.kt*

```kotlin
class TokenAuthenticator : Authenticator {
    override fun authenticate(route: Route?, response: okhttp3.Response): Request? {

        val refresh = TimeTableSelPref.encryptedPrefs.getRefreshToken() ?: ""
        Log.d(TAG, "TokenAuthenticator - authenticate() called / ν ν° λ§λ£. ν ν° Refresh μμ²­: $refresh")
        val tokenResponse =
            IRetrofit.getInstanceWithNoToken().requestRefresh(refresh).execute()

        return if (handleResponse(tokenResponse)) {
            Log.d(TAG, "TokenAuthenticator - authenticate() called / μ€λ¨λ API μ¬μμ²­")
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
            Log.d(TAG, "TokenAuthenticator - handleResponse() called / λ¦¬νλ μ ν ν°μ΄ λ§λ£λμ΄ λ‘κ·Έ μμ λμμ΅λλ€.")
            false
        }

    companion object {
        const val AUTH_HEADER = "Authorization"
    }
}
```

μ‘μΈμ€ ν ν°μ΄ λ§λ£λ κ²½μ° REST APIμ λ°λΌ `401` μλ΅ μ½λλ₯Ό λ΄λ €μ€λ€. μλ΅ μ½λλ‘ `401`μ λ°μΌλ©΄ `TokenAuthenticator`κ° λμνλ€.

`EncryptedSharedPreference`μ μ μ₯λ `Refresh ν ν°`μ κ°μ Έμμ `Access ν ν°`μ κ°±μ νλ μμμ ν νμ μ€λ¨λμλ API μμ²­μ μ¬κ°νλ€.

λ§μ½ `Refresh ν ν°`λ§μ  λ§λ£λμλ€λ©΄ λ‘κ·ΈμΈ λ μ¬μ©μλ₯Ό **λ‘κ·Έ μμ** μν€κ³ , μ€λ¨λ μμμ μ¬κ°νμ§ μλλ€.

*ν ν° λ§λ£*
![token expired](https://user-images.githubusercontent.com/44221447/167582290-fc7eb335-c3db-46e7-af62-6c403d765baf.png)

*μ€λ¨λ μμ²­ μ¬μν*
![resume api request](https://user-images.githubusercontent.com/44221447/167582766-0b72fe98-13e5-406d-82cb-446caa4055b6.png)

[okhttp3.authenticator]: https://square.github.io/okhttp/4.x/okhttp/okhttp3/-authenticator/
[okhttp3.interceptors]: https://square.github.io/okhttp/features/interceptors/

### LocalDateTime λ€λ£¨κΈ°
[Retrofit κ°μ²΄ μμ± λΆλΆ](#retrofit-κ°μ²΄-μμ±)μ `ConverterFactory`λ₯Ό μΆκ°νλ λΆλΆμμ `gsonConverterFactory()` λΌλ λ©μλλ₯Ό μ¬μ©νλ€.

μ΄ λΆλΆμ μλ λ€μκ³Ό κ°μ΄ μμ±λμμλ€.

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

λ§μ½ μλ²μ μλ΅ κ°μΌλ‘ `LocalDateTime`μ λ°κ² λλ€λ©΄ μμ λ°©λ²μΌλ‘λ `Gson`μ΄ `String` νμμΌλ‘ λ³νν΄λ²λ €μ `LocalDateTime`μΌλ‘ μ¬μ©ν  μ μκ² λλ€.

*Error*
![LocalDateTime_Type_Error](https://user-images.githubusercontent.com/44221447/167625596-5a76daa2-7473-4b3e-b863-7fe0aa79e33f.png)

μ΄λ₯Ό ν΄κ²°νκΈ° μν΄ λ³λμ λ©μλλ‘ μΆμΆνκ² λμλ€.

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

μ κ³Όμ μ ν΅ν΄ `LocalDateTime`μ΄ `String` νμμΌλ‘ λ³ν κ²μ `μ­ μ§λ ¬ν(Deserialize)`νμ¬ λ€μ `LocalDateTime` νμμΌλ‘ μ¬μ©ν  μ μλ€.

> μ§λ ¬ν: μλ° κ°μ²΄λ₯Ό **JSON νμμΌλ‘ λ³ν**
>
> μ­ μ§λ ¬ν: JSON νμ λ°μ΄ν°λ₯Ό **μλ° κ°μ²΄λ‘ λ³ν**. μ΄λ€ νμμΌλ‘ λ³νν  μ§ λͺμν΄μΌ νλ€.

---

## API νΈμΆ μμ  (μν μ λ³΄ κ΅¬λ§€ μ΄λ ₯ νμΈ)

### Retrofit μΈν°νμ΄μ€ μ μΈ

*IRetrofit.kt*

```kotlin
interface IRetrofit {
    @GET(PURCHASE_HISTORY) // PURCHASE_HISTORYμλ Base Url μ΄νμ μ£Όμκ° λ€μ΄κ°
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

*API λͺμΈμ*
![api spec](https://user-images.githubusercontent.com/44221447/167621638-e42c954d-e57e-4da2-8787-4fab960c4632.png)

API λͺμΈμμ λ§κ² Data Classλ₯Ό μμ±ν΄μΌ νλ€. μ λͺμΈμμμλ `data`λΌλ key κ°μΌλ‘ `JSON λ°°μ΄` ννμ valueλ₯Ό λ΄λ €μ£Όλλ° λ°°μ΄μ κ° μμ΄νμ μ΄λ£¨λ **id, lectureName, professor, majorType, createDate** μ λ§μΆ°μ `PurchaseHistory`λΌλ Data Classλ₯Ό μμ±νλ€.

κ·Έλ¦¬κ³  `PurchaseHistoryDto`λΌλ μ΄λ¦μ `PurchaseHistory` λ¦¬μ€νΈλ₯Ό κ°μ§λ Data Classλ₯Ό λ§λ€μ΄ `PurchaseHistoryDto`λ‘ μλ΅μ λ°κ² λλ€.

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
                    // λ°μ΄ν°κ° μμ λ μ²λ¦¬
                }
                response.body()?.data?.let { _historyList.postValue(response.body()?.data) }
            } else {
                // ν΅μ  μλ¬ μ²λ¦¬
            }
        }
    }
}
```

IO μμμ λΉλκΈ°λ‘ μ²λ¦¬νκΈ° μν΄ `Coroutine`μ μ¬μ©νλ€. ViewModelμμλ ViewModel μλͺ μ£ΌκΈ°μ λ§μΆ°μ λμνλ `viewModelScope`λ₯Ό μ¬μ©ν  μ μλ€.

MyPostRepositoryμ getPurchaseHistory()λ₯Ό νΈμΆνλ λΆλΆμ νμΈν  μ μλ€. μλ΅μ Responseλ‘ κ°μΌ κ°μ λ°μΌλ―λ‘ `response.isSuccessful`λ‘ ν΅μ  μ±κ³΅ μ¬λΆλ₯Ό νμΈν  μ μκ³ , [`PurchaseHistoryDto`](#dto)μ `data` ν€ κ°μΌλ‘ λ°μ΄ν°λ₯Ό νμΈν  μ μλ€.

λ°μμ¨ λ°μ΄ν°κ° μλ€λ©΄ λΌμ΄λΈλ°μ΄ν°μ λ°μμ¨ λ°μ΄ν°λ₯Ό λ΄λλ€.

μ‘ν°λΉν°λ νλκ·Έλ¨ΌνΈμμ `historyList`μ κ°μ κ΄μ°°νκ³  μλ€κ° κ°μ΄ λ³κ²½λλ©΄ RecyclerView Adapterμ `submitList`λ‘ λ°μ΄ν°λ₯Ό λ³κ²½ν΄μ£Όλ λ°©μμΌλ‘ μ²λ¦¬νλ€.

### Repository

λ°μ΄ν°λ μ§κΈμ²λΌ λ€νΈμν¬ ν΅μ μ ν΅ν΄ κ°μ Έμ¬ μλ μκ³ , SQLite DBμμ κ°μ Έμ¬ μλ μκ³ , λ΄λΆ Asset νμΌμμ κ°μ Έμ¬ μλ μκ³ , λ€μν λ°μ΄ν° μμ€κ° μ‘΄μ¬ν  μ μλ€.

ViewModelμμλ λΉμ¦λμ€ λ‘μ§μ μννμ§λ§ λ°μ΄ν°κ° μ΄λμ μλμ§λ λͺ°λΌλ λλλ‘ Repository ν¨ν΄μ μ¬μ©νλ€.

[*κ΅¬κΈ κ°λ°μ λ¬Έμ*](https://developer.android.com/jetpack/guide?hl=ko#data-layer)
![repository_pattern](https://user-images.githubusercontent.com/44221447/167629187-db1a82f6-4831-458d-8a87-3c24a362dcdf.png)

*MyPostRepository*

```kotlin
class MyPostRepository(private val dataSource: MyPostRemoteDataSource) {
    // ...
    suspend fun getPurchaseHistory() = withContext(Dispatchers.IO) { dataSource.getPurchaseHistory() }
}
```

`MyPostRepository`μμ λ§€κ° λ³μλ‘ dataSourceλ₯Ό μ£Όμ λ°λλ€. `MyPostRemoteDateSource`μμλ μ€μ§μ μΌλ‘ λ€νΈμν¬ ν΅μ μ ν΅ν΄ λ°μ΄ν°λ₯Ό λ°μμ€λ κ³Όμ μ΄ μνλλ€.

### Data Source

*MyPostRemoteDataSource*

```kotlin
class MyPostRemoteDataSource(private val apiService: IRetrofit): MyPostDataSource {
    // ...
    override suspend fun getPurchaseHistory() = apiService.getPurchaseHistory()
}
```

DataSourceμμ λ§€κ° λ³μλ‘ apiServiceλ₯Ό μ£Όμ λ°λλ€. [`IRetrofit`](#retrofit-κ°μ²΄-μμ±)μ μμμ νμ°Έ μ€λͺνλ λ°λ‘ κ·Έ μΈμ€ν΄μ€μ΄λ€.

μ μ½λλ₯Ό λ³΄λ©΄ `MyPostDataSource`λ₯Ό μμνλ κ²μ λ³Ό μ μλ€. `MyPostDataSource`λ λ€μν λ°μ΄ν° μμ€λ₯Ό ν΅μΌννκΈ° μν΄ λ©μλλ₯Ό κ°μ νλ μΈν°νμ΄μ€μ΄λ€.

*MyPostDataSource*

```kotlin
interface MyPostDataSource {
    // ...
    suspend fun getPurchaseHistory(): Response<PurchaseHistoryDto>
}
```

### μμ‘΄μ± μ£Όμ

A ν΄λμ€ λ΄λΆμμ B ν΄λμ€μ κ°μ²΄κ° νμν  λ A ν΄λμ€ λ΄λΆμμ μ§μ  B ν΄λμ€μ κ°μ²΄λ₯Ό μμ±νλ©΄ λ ν΄λμ€ κ°μ κ²°ν©λκ° μ¬λΌκ°κ² λλ€. κ°μ²΄ μ§ν₯ μ€κ³μμ ν΄λμ€ κ°μ κ²°ν©λλ₯Ό λ?μΆκ³  μμ§λλ₯Ό λμ΄λ μ€κ³λ μ€μνλ€.

μ΄λ κ² κ²°ν©λκ° λμμ§λ κ²μ λ°©μ§νκΈ° μν΄ **μμ±μ μ£Όμ λ°©μ, νλ μ£Όμ λ°©μ, μμ μ μ£Όμ λ°©μ λ±** λ€μν λ°©λ²μ΄ κ³ μλμλ€.

`Dagger Hilt`μ κ°μ λΌμ΄λΈλ¬λ¦¬λ λ§μ΄ μ¬μ©νλ κ±Έλ‘ μκ³  μλ€. (μμ§ Hiltμ λν΄ νμ΅νμ§ μμ μνλΌ νλ‘μ νΈμ μ μ©λμ§λ μμλ€. νμ¬ νλ‘μ νΈλ μμ±μ μ£Όμ λ°©μμ μ±ννλ€)

νμ¬ νλ‘μ νΈμμ ViewModelμ μ¬μ©ν  λ ViewModelμ μμ±μλ‘ apiServiceλ₯Ό μ£ΌμνκΈ° μν΄μ `ViewModelFactory`λ₯Ό μ¬μ©ν΄μ μμ±ν΄μ£Όλ λ°©μμ μ¬μ©νκ³  μλ€.

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

## μ μ²΄ κ΅¬μ‘°

![networking_structure](https://user-images.githubusercontent.com/44221447/167644878-81522ce3-df9e-44aa-b2fc-d8c04cb250fa.png)

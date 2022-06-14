---
layout: post
title: 안드로이드에서 FCM 사용해보기-2
subtitle: 테스트 메시지 보내기
categories: Android
tags: [android, firebase, fcm]
---

## 🔥프로젝트 준비

안드로이드에서 FCM (Firebase Cloud Messaging)을 사용하기 위한 세팅은 이전 게시글을 참고하세요

[안드로이드에서 FCM 사용해보기-1][setting_posting]

### ViewBinding 세팅

![setting_viewbinding](https://user-images.githubusercontent.com/44221447/169105515-b2f6e4a7-62ef-4c2d-b06a-08316dc6e2f4.png)

*build.gradle (:app)*

```gradle
android {
    // ...
    buildFeatures {
        viewBinding true
    }
}
```

뷰를 참조하기 위해 findViewById, ViewBinding, DataBinding 등의 방법을 사용할 수 있다. 
ViewBinding과 DataBinding은 findViewById를 대체하기 위한 수단으로 사용되고, DataBinding은 ViewBinding을 포함하고 있다. (잘못된 사실이라면 알려주세요!)

이번 예제에서는 간단한 뷰 참조만 할 것이라서 ViewBinding을 사용했다.

`build.gradle(:app)`에 `buildFeatrues` 부분을 추가하고 `Sync now`를 눌러 활성화한다.

## 🔥테스트 메시지를 보내보자 (실습)

---

### XML 뷰 작성

*activity_main.xml*

![activity_main.xml](https://user-images.githubusercontent.com/44221447/169109826-bcaf4c8b-cb86-4762-970e-c701bae45b72.png)

다음은 위와 같은 간단한 뷰를 만드는 부분이다. `activity_main.xml`에 작성한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/tv_title_token"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="토큰"
        android:textColor="@color/black"
        android:textSize="32sp"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <EditText
        android:id="@+id/et_token"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="16dp"
        android:inputType="text"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_title_token"
        tools:text="토큰이 들어가는 부분입니다." />

    <TextView
        android:id="@+id/tv_title_response"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:text="응답"
        android:textColor="@color/black"
        android:textSize="32sp"
        android:textStyle="bold"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/et_token" />

    <TextView
        android:id="@+id/tv_response"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="16dp"
        android:textColor="@color/black"
        android:textSize="16sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/tv_title_response"
        tools:text="결과 값입니다" />


</androidx.constraintlayout.widget.ConstraintLayout>
```

### MainActivity.kt 세팅

```kotlin
package mangbaam.practice.simplefcm

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.EditText
import android.widget.TextView
import com.google.firebase.messaging.FirebaseMessaging
import mangbaam.practice.simplefcm.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private val token: EditText by lazy { binding.etToken }
    private val response: TextView by lazy { binding.tvResponse }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

`ViewBinding`을 사용하면 `binding`을 통해 빠르게 뷰를 찾을 수 있다.

이 때 xml에서 **스네이크 케이스**로 되어 있던 id를 **카멜 케이스**로 접근할 수 있다. (ex: `tv_response` -> `tvResponse`)

### 파이어베이스 토큰 가져오기

우리는 파이어베이스 콘솔에서 기기로 메시지를 보낼 것이다. 이때 어떤 기기로 메시지를 보낼 것인지 구분하기 위해 파이어베이스 토큰을 사용한다.

앞서 token에 저장한 EditText에 토큰을 표시할 것이다.

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private val token: EditText by lazy { binding.etToken }
    private val response: TextView by lazy { binding.tvResponse }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        initFirebase()
    }

    private fun initFirebase() {
        FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                token.setText(task.result)
            }
        }
    }
}
```

[공식 문서](https://firebase.google.com/docs/cloud-messaging/android/client?hl=ko&authuser=0#retrieve-the-current-registration-token)에 따르면 파이어베이스 토큰은 `FirebaseMessaging.getInstance().token`에 `addOnCompleteListener`를 통해 비동기적으로 task를 가져올 수 있다. 이 task의 result에 파이어베이스 토큰이 담겨있다.

빌드를 해보면 다음 사진처럼 토큰을 확인할 수 있다. 이 토큰을 복사하자

![firebase_token](https://user-images.githubusercontent.com/44221447/169112615-8c25066a-9667-489e-a7bc-eb3fb4c9312b.png)

### 메시지를 보내보자

![console_firebase](https://user-images.githubusercontent.com/44221447/169114487-add3d470-87ca-4416-b547-8d4b17a8609d.png)

[파이어베이스 콘솔](https://console.firebase.google.com/project/simple-fcm-93980/notification)에 가서 앞서 생성한 프로젝트를 선택하고 왼쪽 메뉴 중 Cloud Messaging을 선택하면 위 화면을 볼 수 있다.

아직 파이어베이스 프로젝트를 생성하지 않았다면 [이 글][setting_posting]을 참고해서 프로젝트를 생성하자.

![btn_send_message](https://user-images.githubusercontent.com/44221447/169116464-6189c654-847d-4a53-979c-cdef9693aa95.png)

**Send your first message** 버튼을 누른다.

![sample_message](https://user-images.githubusercontent.com/44221447/169116922-f16148bd-9f56-4a22-9bbe-f7406844d8d1.png)

보낼 메시지의 제목과 텍스트를 적는다.

![send_test_message](https://user-images.githubusercontent.com/44221447/169117559-51bf0124-9659-4f1a-95c1-e431505ab31d.png)

우측의 **테스트 메시지 전송** 버튼을 누른다.

![set_token](https://user-images.githubusercontent.com/44221447/169117742-b5e6a727-5788-4e92-886e-2ef077efc89d.png)

아까 복사한 토큰을 붙여넣기 하고 + 버튼을 눌러 추가한다.

***테스트 버튼을 누르기 전에!!!***

> 메시지를 받는 앱이 `백그라운드`에 있어야 알림을 수신한다. 홈 버튼을 누르거나 앱을 꺼서 `백그라운드`로 보낸다

앱이 `백그라운드`에 있는 상태에서 **테스트**버튼을 누른다.

![test_success](https://user-images.githubusercontent.com/44221447/169118571-b8bd2039-3792-4d77-9d63-d41da99e753b.png)

노티를 성공적으로 수신했다. 🎉

노티를 클릭하면 우리의 앱이 `포그라운드` 상태로 전환된다. (꺼져 있던 상태라면 켜진다)

## 🔥토큰은 변경될 수 있다

---

앞서 진행한 [테스트 메시지 보내기](#테스트-메시지를-보내보자)는 직접 토큰을 콘솔에 입력해서 보내는 말 그대로 테스트 용도이다.

파이어베이스 토큰은 변경될 수 있는 값이기 때문에 토큰이 갱신될 때 새로운 토큰을 사용해야 한다.

토큰이 변경되는 경우는 [이전 게시글](https://mangbaam.github.io/android/2022/05/16/fcm-starter.html#h-%EB%93%B1%EB%A1%9D-%ED%86%A0%ED%81%B0-%EA%B4%80%EB%A6%AC)에서 설명했다.

이때 사용할 수 있는 것이 바로 `onNewToken()` 메서드이다. 토큰이 갱신되면 `onNewToken()` 콜백 메서드가 실행되고, 이때 서버에 저장된 기기의 토큰을 갱신하는 처리가 필요하다.

앱을 처음 시작해서 아직 토큰이 없는 경우에도 `onNewToken()`에서 토큰을 받아와서 서버에 보내주는 등의 처리를 해야 한다.

[![message_recieve_table](https://user-images.githubusercontent.com/44221447/169356437-59ee9d7d-f2b3-4c20-a482-228ab39af371.png)](https://firebase.google.com/docs/cloud-messaging/android/receive?authuser=0#handling_messages)

또한 [이전 게시글](https://mangbaam.github.io/android/2022/05/16/fcm-starter.html#h-%EB%A9%94%EC%8B%9C%EC%A7%80%EC%9D%98-%EC%A2%85%EB%A5%98)에서 메시지의 종류에는 `알림 메시지`와 `데이터 메시지`가 있다고 설명했는데, `알림 메시지`의 경우 백그라운드 상태에서는 작업 표시줄에 표시되고, `데이터 메시지`는 백그라운드 포그라운드 상관 없이 `onMessageReceived()` 메서드에서 수신한다.

이번에는 `onMessageReceived()`로 메시지를 수신하는 `데이터 메시지`를 실습해보도록 하자.



## Notification과 Service

---

### Service

안드로이드 초보라면 안드로이드 4대 주요 컴포넌트 중 대부분 액티비티만을 사용해서 개발했을 것이다. (나 역시도...)

안드로이드 4대 주요 컴포넌트라고 하면 `Activity`, `Service`, `Broadcast Receiver`, `Content Provider`를 뜻하는데 그 중 `Service`를 사용할 것이다.

`Service`는 `Activity`와 별개로 백그라운드에서 실행되는 컴포넌트이다. `Activity`와 다르게 화면을 작성하지 않는다. 대표적으로 음악 스트리밍 앱을 사용할 때 앱이 백그라운드에 있을 때도 음악을 계속해서 들을 수 있는 경우이다.

FCM을 사용할 때도 앱이 백그라운드에 있어도 메시지를 수신해야 하기 때문에 `Service`를 사용한다.

### 노티 채널

[![noti_channel_docs](https://user-images.githubusercontent.com/44221447/169479369-bb92eb90-fdda-4208-b7a6-7b24b30fea4c.png)](https://developer.android.com/guide/topics/ui/notifiers/notifications#ManageChannels)

안드로이드 공식 문서에 따르면 안드로이드 8.0부터는 모든 알림(노티)을 채널에 할당해야 한다고 설명되어 있다. 이를 따르기 위해서는 기기의 버전에 따라 분기를 해서 처리해야 한다.

### 노티 채널 우선순위(Priority)

[![channel_priority_table](https://user-images.githubusercontent.com/44221447/169481220-9a41c773-64b6-4665-93a2-76cfc22d7d2b.png)](https://developer.android.com/training/notify-user/channels#importance)

노티 채널을 만들 때 알림의 우선순위(중요도)를 설정해야 한다. 각각의 예시는 위 표와 같고, 한 채널에 포함된 노티들은 모두 같은 우선순위를 갖는다.

## 🔥(실습)

---

### Service 만들기

![new_file](https://user-images.githubusercontent.com/44221447/169359215-128aa970-8292-4037-bb0d-6169d7d40fae.png)

`FCMService.kt`라는 클래스를 생성했다.

그 내부에서는 다음과 같이 작성한다.

```kotlin
package mangbaam.practice.simplefcm

import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

class FCMService: FirebaseMessagingService() {
    override fun onNewToken(token: String) {
        super.onNewToken(token)

    }

    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)

    }
}
```

이 클래스는 `FirebaseMessagingService()`를 상속한다. 그러면 [위에서 언급](#토큰은-변경될-수-있다)했던 `onNewToken()`과 `onMessageReceived()`를 사용할 수 있다.

### Manifest에 등록하기

액티비티에서와 마찬가지로 Manifest에 등록을 해야한다.

![manifest.xml](https://user-images.githubusercontent.com/44221447/169359762-6b95c671-a915-44ac-bb50-9709640cdb39.png)

```kotlin
<service android:name=".FCMService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>
```

`com.google.firebase.MESSAGING_EVENT` 이름의 액션이 실행되면 우리의 `FCMService()`가 실행된다.

> 알림의 모양(아이콘, 색상 등)을 커스텀하려면 [공식 문서](https://firebase.google.com/docs/cloud-messaging/android/receive?authuser=0#edit-the-app-manifest)를 참고

### Notification 채널 만들기

```kotlin
class FCMService : FirebaseMessagingService() {
    override fun onNewToken(token: String) {
        super.onNewToken(token)

    }

    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)

        createNotificationChannel()
    }

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                CHANNEL_NAME,
                NotificationManager.IMPORTANCE_DEFAULT
            )
            channel.description = CHANNEL_DESCRIPTION

            (getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager)
                .createNotificationChannel(channel)
        }
    }

    companion object {
        private const val CHANNEL_NAME = "SimpleFCM"
        private const val CHANNEL_DESCRIPTION = "Simple FCM Example"
        private const val CHANNEL_ID = "SimpleFCM ID"
    }
}
```

노티 채널을 만드는 과정이다. 앞서 설명한대로 안드로이드8.0 부터는 노티들은 채널을 가져야 한다고 했는데 `Build.VERSION.SDK_INT`로 현재 기기의 버전을 가져와서 분기할 수 있다.

메시지를 받아서 알림으로 보여주기 전에 채널이 생성되어 있어야 해서 `onMessageReceived()` 안에서 채널을 생성했는데 동일한 아이디로 생성된 채널은 중복으로 생성되지 않기 때문에 메시지를 수신할 때마다 채널을 생성하지 않을까 고민하지 않아도 된다.

![unnecessary_branch](https://user-images.githubusercontent.com/44221447/169483625-197449be-ab17-4ca8-a43a-99530f64061c.png)

그러나 현재 프로젝트에서는 이런 분기가 필요 없다고 나오는데 우리가 프로젝트를 생성할 때 MinSDK를 26으로 설정했기 때문이다.

그래서 현재 프로젝트에서는 분기가 필요 없지만 MinSDK를 26보다 작게 설정된 프로젝트에서는 위와 같은 분기가 필요하다.

채널을 만들 때는 `NotificationChannel()`에 아이디, 채널 이름, 우선순위를 부여하고 설명을 추가할 수 있다. 그리고 `NotificationManager`로 노티 채널을 생성하는 부분을 작성하고 앱을 실행해보자.

그리고 [앞선 예제](#메시지를-보내보자))처럼 콘솔에서 **테스트 메시지를 보내보자.**

![chnnel_craeted](https://user-images.githubusercontent.com/44221447/169491877-7a4352f5-82cc-4913-aa3e-f4006b6b8a74.png)

앱의 정보를 확인해보면 우리가 생성한 채널이 보인다.

> 처음에는 "miscellaneous"라는 채널로 생성이 되어서 원인을 찾다가 앱을 재설치 후에 다시 실행해보니 성공했다

### FCM에서 받은 메시지를 노티로

```kotlin
class FCMService : FirebaseMessagingService() {

    // ...

    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)

        createNotificationChannel()

        val title = message.data["title"]
        val msg = message.data["message"]

        NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.pizza)
            .setContentTitle(title)
            .setContentText(msg)
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    }
}
```

FCM에서 받은 메시지는 `message`에 담겨 있다. 

[setting_posting]: https://mangbaam.github.io/android/2022/05/17/fcm-setting.html

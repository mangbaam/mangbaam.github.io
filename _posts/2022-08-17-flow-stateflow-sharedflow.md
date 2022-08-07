---
layout: post
title: StateFlow 와 sharedFlow
subtitle: StateFlow and SharedFlow
categories: Kotlin
tags: [kotlin,coroutine,flow]
---

## ⭐

본 글은 [StateFlow 와 SharedFlow](https://myungpyo.medium.com/stateflow-%EC%99%80-sharedflow-32fdb49f9a32) 를 보고 정리한 글입니다

## 상태를 나타내는 StateFlow

---

UI 를 다루는 상태를 생각해보자. 로그인 여부에 따라 "로그인", "로그아웃" 등 상태를 정의할 수 있으며 화면에 표시하는 데이터도 "로딩 중", "데이터 이용 가능", "오류" 등의 상태로 정의할 수 있다.

*그리고 한 가지 상태만을 가지고 있다.*

Kotlin 으로 상태를 표현하는 방법은 부가적인 데이터가 필요 없다면 Enum 으로, 부가적인 데이터가 필요하다면 Sealed class 로 정의하기도 한다.

Android 에서는 이런 상태들을 다루기 위한 LiveData 를 제공한다. LiveData 는 Android lifecycle 을 알고 있기 때문에 자연스럽게 연동된다. 하지만 LiveData 는 생명 주기가 있는 UI와 연동되도록 설계되었기 때문에 비지니스 레이어에서 사용하는 것은 무리가 있다. 특히 LiveData 는 AAC 에서 제공되는 기능이기 때문에 비지니스 레이어를 플랫폼 독립적으로 가져가기 위해서 걸림돌이 된다.

과거에는 이 점을 RxJava 를 통해 해결했는데 이제는 코틀린을 주 언어로 사용하면서 경량화된 Reactive stream 구현체라고 볼 수 있는 Flow 를 이용하여 이를 구현할 수 있다.

[Flow 글](https://mangbaam.github.io/kotlin/2022/07/30/kotlin-flow-basic-1.html#h-flow-%EB%8A%94-cold-%EB%A1%9C-%EB%8F%99%EC%9E%91%ED%95%9C%EB%8B%A4)에서 설명한 것처럼 플로우는 기본적으로 콜드 스트림이다. 하지만 `StateFlow`, `SharedFlow` 와 같은 핫스트림 플로우도 제공된다.

이 중에서 기본 값을 가지고 모든 구독자에게 최신 상태를 전달해주는 것은 StateFlow 이다.

```kotlin
// code by Myungpyo Shim
@HiltViewModel
class MainViewModel(
    private val refreshDataUseCase: RefreshDataUseCase,
    @Assisted private val savedStateHandle: SavedStateHandle,
) : ViewModel() {
    private val _dataState: MutableStateFlow<UiState<List<MembershipUiModel>>> 
        = MutableStateFlow(UiState.InProgress())
    val dataState = _dataState.asStateFlow()


    fun refreshData() = viewModelScope.launch {
        runCatching {
            refreshDataUseCase.execute()
        }.onSuccess { data ->
            _dataState.emit(UiState.Success(data))
        }.onFailure { throwable ->
            _dataState.emit(UiState.Fail(throwable))
        }
    }
}
```

_dataState 와 dataState 를 정의하는 부분을 보면 _dataState는 Mutable Flow 이고, dataStore 는 Immutable Flow 로 정의된다.

refreshData() 가 호출되면 ViewModel 스코프에서 코루틴이 생성되어 UseCase 의 중단 함수를 호출하고, 결과(성공/실패)에 따라서 적당한 데이터로 변환하여 Flow 를 전달하고 `dataState` 를 구독하고 있는 UI를 갱신할 수 있게 만든다.

## 이벤트를 나타내는 SharedFlow

---

위에서 살펴본 것과 같이 애플리케이션은 상태에 따라 전환하며 실행된다. 상태를 담고 있는 상태 머신은 항상 기본 상태를 갖고 동작하지만 이벤트는 기본 값 없이 특정 상황이 발생하면 구독자들에게 이벤트라는 형태로 전달하게 된다.

상태와 이벤트는 어떻게 다른 걸까

- 상태 : 기본 값이 있고, 신규 구독 시 가장 최근 값을 받는다
- 이벤트 : 기본 값이 없고, 구독 이후에 발생한 값을 받는다

이벤트는 일반적으로 UI Layer 에서 사용자와 뷰 간의 인터렉션에서 발생한 이벤트를 전달하거나 시스템에 메모리 부족 이벤트, 인증 오류 발생 이벤트 등 관련 컴포넌트들이 대응할 수 있도록 하기 위해서 사용된다.

```kotlin
// code by Myungpyo Shim
@HiltViewModel
class MainViewModel(
    @Assisted private val savedStateHandle: SavedStateHandle,
) : ViewModel() {
    private val _systemEvent: MutableSharedFlow<Unit> =
        MutableSharedFlow(replay = 0, extraBufferCapacity = 1, onBufferOverflow = BufferOverflow.DROP_OLDEST)
    val systemEvent = _systemEvent.asSharedFlow()
    
    init {
        viewModelScope.launch {
            systemEvent.collect { systemEvent ->
                when(systemEvent) {
                    is SystemEvent.MemoryWarning -> { TODO() }
                    is SystemEvent.StorageWarning -> { TODO() }
                    else -> { }
                }
            }
        }
    }
    
    fun reportSystemEvent(systemEvent: SystemEvent) {
        _systemEvent.emit(systemEvent)
    }
}
```

구조는 StateFlow 와 비슷하지만 MutableSharedFlow 를 만들 때 몇 가지 옵션을 통해 SharedFlow 동작을 재정의 한다.

- replay = 0 : 새로운 구독자에게 이전 이벤트를 전달하지 않음
- extraBufferCapacity = 1 : 추가 버퍼를 생성하여 emit 한 데이터가 버퍼에 유지되도록 함
- onBufferOverFlow = BufferOverflow.DROP_OLDEST : 버퍼가 꽉 찼을 때 오래된 데이터 제거

## 계층 구조

내부적으로 다음과 같은 계층 구조를 가진다.

> Flow < SharedFlow < StateFlow

SharedFlow 에서는 replayCache 와 크기를 정의하여 새로운 구독자에게 반복 전달할 값의 개수를 설정할 수 있고, 구독자들은 Slot 이라는 형태로 관리하여 값이 전달될 때 액티브한 모든 구독자에게 새로운 값이 전달되도록 한다.

SharedFlow 를 상속하는 StateFlow 는 기본 값을 가지고 있으며 replaceCache 는 가장 최근 값 하나를 갖는 리스트를 replayCache 로 재정의한다.

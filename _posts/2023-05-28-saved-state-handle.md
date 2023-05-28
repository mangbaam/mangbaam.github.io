---
layout: post
title: SavedStateHandle과 커스텀 SaveableStateFlow
subtitle: 
categories: Android
tags: [android,custom,state,configuration change,viewmodel,savedinstancestate,bundle,flow,stateflow]
---

![image](https://github.com/mangbaam/MySampleApps/assets/44221447/4a984a77-90a5-4caa-bf40-191bce080e17)

`ViewModel`은 화면 회전 같은 Configuration change 상황에서 사용할 수 있지만, 시스템에 의해 프로세스가 종료되는 상황을 대응할 때는 `SavedStateHandle`을 사용할 수 있다.

`SavedStateHandle`은 `ViewModel` 객체에서 생성자로 받을 수 있으며, 키-값 맵 형태로 상태를 저장하거나 저장된 상태를 가져올 수 있다.
`SavedStateHandle`에 저장된 상태들은 시스템에 의해 프로세스가 종료된 후에도 유지되지만 사용자에 의한 앱 강제 종료, 최근 메뉴에서 종료, 디바이스 재부팅 등의 상황에서는 유지할 수 없다. (Task Stack 에서 제거되면 함께 사라진다)

## 어떤 값을 저장해야 할까?

일반적으로 텍스트 필드의 입력 값, 스크롤 위치, 탐색 중이던 항목의 ID 등의 상태를 저장할 수 있다.
저장되는 상태들은 단순하고 가벼운 형태여야 하며, 저장할 값이 복잡하거나 큰 경우는 DB와 같은 곳에 저장하는 것이 좋다.

### 저장할 수 있는  타입

`SavedStateHandle`에 저장되는 데이터는 액티비티나 프래그먼트의 `SavedInstanceState`처럼 `Bundle`로 저장되기 때문에 `Bundle`에서 지원하는 값으로 저장할 수 있다.

| 타입/클래스     | 배열           |
| --------------- | -------------- |
| double          | double[]       |
| int             | int[]          |
| long            | long[]         |
| String          | String[]       |
| byte            | byte[]         |
| char            | char[]         |
| CharSequence    | CharSequence[] |
| float           | float[]        |
| Parcelable      | Parcelable[]   |
| Serializable    | Serializable[] |
| short           | short[]        |
| SparseArray     |                |
| Binder          |                |
| Bundle          |                |
| ArrayList       |                |
| Size (API 21+)  |                |
| SizeF (API 21+) |                |

## 사용 방법

### 뷰모델의 생성자에 SavedStateHandle 제공

**Activity 1.1.0**, **Fragment 1.2.0** 버전부터 `ViewModel`의 생성자에 `SavedStateHandle`을 사용할 수 있다.

뷰모델에서 다음과 같이 생성자를 선언하고

```kotlin
class SavedStateViewModel(
	private val state: SavedStateHandle
) : ViewModel() {
	... 
}
```

액티비티나 프래그먼트에서는 ViewModel 팩토리로 SavedStateHandle을 제공할 수 있다.

```kotlin
class MainFragment : Fragment() { 
	val vm: SavedStateViewModel by viewModels() 
	... 
}
```

`by viewModels()`가 아닌 커스텀으로 제공할 때는 `AbstractSavedStateViewModelFactory`를 확장하여 사용할 수도 있다.

### SavedStateHandle 사용법

`SavedStateHandle` 에서 사용할 수 있는 메서드
- `get(key: String)`: key 로 저장된 값을 반환
- `set(key: String, value: T?)`: key - value 를 저장
- `keys()`: SavedStateHandle에 포함된 모든 키를 반환
- `containts(key: String)`: key에 대한 값이 존재하는지 확인
- `remove(key: String)`: key에 대한 값을 제거
- `getLiveData(key: String)`: key에 대한 값을 LiveData로 반환 **(밑에서 설명)**
- `getStateFlow(key: String, initialValue: T)`: key에 대한 값을 StateFlow로 반환 **(밑에서 설명)**

### LiveData

`getLiveData()`를 사용하면 `SavedStateHandle`에서 `LiveData`에 래핑된 값을 받을 수 있다.

```kotlin
class SavedStateViewModel(
	private val savedStateHandle: SavedStateHandle
) : ViewModel() { 
	val filteredData: LiveData<List<String>> = 
	savedStateHandle.getLiveData<String>("query").switchMap { query -> 
		repository.getFilteredData(query) 
	} 
	
	fun setQuery(query: String) { savedStateHandle["query"] = query } 
}
```

### StateFlow

> lifecycle 2.5.0부터 지원

`getStateFlow()`를 사용하여 `SavedStateHandle`에서 `StateFlow`에 래핑된 값을 받을 수 있다.

```kotlin
class SavedStateViewModel(
	private val savedStateHandle: SavedStateHandle
) : ViewModel() { 
	val filteredData: StateFlow<List<String>> = 
	savedStateHandle.getStateFlow<String>("query") .flatMapLatest { query -> 
		repository.getFilteredData(query) 
	} 
	fun setQuery(query: String) { savedStateHandle["query"] = query } 
}
```

## SaveableMutableStateFlow

> 이 타입은 Android 에서 제공되는 것이 아닌 커스텀 한 타입입니다

ViewModel에서 StateFlow 에 상태를 저장해도 메모리가 부족하면 시스템에 의해 프로세스가 정리되고, ViewModel 객체 역시 메모리에 유지되던 것이기 때문에 ViewModel 객체가 정리되면서 상태 역시 잃어버릴 수 있다. 이를 방지하기 위해 SavedStateHandle 와 StateFlow 의 장점을 모두 활용하는 `SaveableMutableStateFlow` 타입을 만들어 보았다.

```kotlin
class SaveableMutableStateFlow<T>(
    private val savedStateHandle: SavedStateHandle,
    private val key: String,
    initialValue: T,
) : MutableStateFlow<T> {
    // ...
}
```

MutableStateFlow 를 상속해서 만들며, `SavedStateHandle`과 키, 초기 값을 받는다.

왠만하면 상속을 피하는 것이 좋지만 MutableStateFlow 를 상속한 이유는 SavedStateHandle 로 저장할 수 있는 타입이 한정적이기 때문에 SaveableMutableStateFlow 를 사용할 수 없는 경우 MutableStateFlow 로 사용할 수 있도록 동일한 타입으로 만들어주기 위해 상속을 사용했다.

```kotlin
private val _state = try {
    MutableStateFlow(savedStateHandle.getStateFlow(key, initialValue).value)
} catch (_: IllegalArgumentException) {
    MutableStateFlow(initialValue)
}
```

구현의 간소화를 위해 내부적으로 MutableStateFlow 를 사용했다. 이때 SavedStateHandle 에서 `getStateFlow`를 통해 StateFlow 를 가져오고, 만약 지원되지 않는 타입의 경우 직접 MutableStateFlow 객체를 생성해서 할당하게 된다.

```kotlin
override var value: T
get() = _state.value
set(value) {
    try {
        savedStateHandle[key] = value
    } catch (_: IllegalArgumentException) {
    }
    _state.value = value
}
```

MutableStateFlow 를 상속하면 구현해야 하는 추상 프로퍼티와 메서드가 여럿 있는데 그 중 value 도 있다.

value 를 설정할 때는 savedStateHandle 에도 저장하는 것이 SaveableMutableStateFlow 의 핵심 동작이다.

```kotlin
fun <T> SavedStateHandle.getSaveableMutableStateFlow(
    key: String,
    initialValue: T,
): SaveableMutableStateFlow<T> =
    SaveableMutableStateFlow(this, key, initialValue)
```

SavedStateHandle 에서 바로 SaveableMutableStateFlow 를 가져오기 편하도록 확장함수로 만들었다.

> 전체 구현은 [Github](https://github.com/mangbaam/MySampleApps/blob/master/SaveableMutableStateFlow/app/src/main/java/com/example/saveablemutablestateflow/SaveableStateFlow.kt)에서 볼 수 있습니다.

ViewModel 에서는 다음과 같이 사용할 수 있다

```kotlin
class MainViewModel(
    savedStateHandle: SavedStateHandle,
) : ViewModel() {
    private val _textSaveableStateFlow =
        savedStateHandle.getSaveableMutableStateFlow("userInput", "")
    val textSaveableStateFlow = _textSaveableStateFlow.asStateFlow()
    // ...
}
```

## 샘플 앱

### 화면 회전

> 화면을 회전하여 Configuration change 상황을 발생시켰다.

회전 전

<img width="33%" alt="image" src="https://github.com/mangbaam/MySampleApps/assets/44221447/61c54939-5bdb-44ff-a5da-e4e605d5cb84">

회전 후

<img width="33%" alt="image" src="https://github.com/mangbaam/MySampleApps/assets/44221447/bf0abf5c-ab6d-443f-abbb-b7ea11321e1a">

버튼 밑의 숫자들은 각각 액티비티의 전역 변수, MutableStateFlow, SaveableMutableStateFlow 로 저장되고 있다.

예상대로 액티비티의 전역 변수로 관리되는 첫 번째 숫자는 **화면 회전 후 0으로 초기화** 되었고, 나머지 두 개는 상태를 유지했다.

EditText 에 표시되는 글자들도 각각 액티비티의 전역 변수, MutableStateFlow, SaveableMutableStateFlow 로 저장되었는데, TextView 와는 다르게 **모두 상태를 유지했다.**

그 이유는 다음과 같다.

<img width="901" alt="image" src="https://github.com/mangbaam/MySampleApps/assets/44221447/b2d51c2a-ec46-4fba-9d3a-1f913d017638">

위 사진은 [공식 문서](https://developer.android.com/guide/components/activities/activity-lifecycle?hl=ko#instance-state)에서 발췌한 내용이다. 밑에 있는 **참고**와 함께 보면 `android:id` 속성이 부여된 경우 액티비티에 있는 View 객체 정보를 `Bundle`에 저장했다가 복원한다고 설명되어 있다. 그래서 EditText 내부에서 상태를 별도로 관리하고 있기 때문에 안드로이드 시스템이 이를 저장했다가 복원할 수 있었던 것이다.

### 시스템에 의한 프로세스 종료

> 터미널에서 ` adb shell am kill "com.example.saveablemutablestateflow"` 명령으로 프로세스를 kill 할 수 있다.

프로세스 kill 이전

<img width="33%" alt="image" src="https://github.com/mangbaam/MySampleApps/assets/44221447/0d466088-b3ed-44fb-bf13-5074ffbff6d9" />

프로세스 kill 이후

<img width="33%" alt="image" src="https://github.com/mangbaam/MySampleApps/assets/44221447/ba6353c0-f7a0-4b15-8c97-44a538a694c3">

EditText 에 표시되는 값은 앞서 설명했듯이 `Bundle`에 저장되어 복원되기 때문에 이번에도 역시 살아남았다.

TextView 에 표시되는 숫자의 경우는 `SaveableMutableStateFlow` 로 저장된 상태를 제외하고는 **모두 0으로 초기화 되었다.**

추가적으로 스크롤 상태도 유지하는 것을 확인할 수 있다.

샘플 프로젝트는 [Github](https://github.com/mangbaam/MySampleApps/tree/master/SaveableMutableStateFlow)에서 확인할 수 있다.

---
layout: post
title: Android Compose - Coil 로 불러온 이미지의 비율 알아내기
subtitle: Android Compose - Coil Image aspect ratio
categories: Compose
tags: [compose,coil,image,android]
---

> 비율 계산 코드 : [이동](#비율-계산)

## 문제 상황

![문제상황](https://i.imgur.com/tJF3e7a.gif)

이미지를 클릭하면 다이얼로그로 크롭되지 않은 원본 비율의 사진을 보여줘야 한다.

하지만 이미지가 불러와지기 전에 화면을 가득 채우는 현상이 있었다.
(그 와중에 태극기는 바로 불러와지는 ㅋㅋ 🇰🇷 펄럭)

### 문제 상황

```kotlin
Dialog(onDismissRequest = { }) {  
    Surface(  
        modifier = Modifier.fillMaxWidth().wrapContentHeight(),  // <--
        shape = RoundedCornerShape(8.dp),  
        color = MaterialTheme.colorScheme.background,  
    ) {  
        PhotoDetail(  
            photo = photo,
            // ... 
        )  
    }  
}
```

다이얼로그를 띄우는 부분이다. `Surface` 로 width를 가득 채우고, height는 `wrapContentHeight` 로 설정하였다.

```kotlin
Column(  
    modifier = modifier  
        .wrapContentHeight()  
        .padding(bottom = 30.dp),  
    horizontalAlignment = Alignment.CenterHorizontally,  
) {  
    Box(  
        modifier = Modifier  
            .fillMaxWidth()  
            .wrapContentHeight(),  
    ) {
	    // 사진이 들어가는 부분
	    AsyncImage(  
		    model = photo.loadUrlMedium,  
		    contentDescription = "Photo detail : ${photo.title}",  
		    contentScale = ContentScale.FillWidth,  
		    modifier = Modifier.fillMaxWidth(),  
		)
	}

	// 텍스트와 버튼 그리고 공백
	
```

Column 을 `wrapContentHeight` 로 만들었다.

사진이 로드되기 전 `wrapContentHeight` 로 높이를 계산하지 못해 생기는 문제였다

## 해결 방법

- 사진 목록을 표시할 때 이미 한 번 사진을 로드한다. 이때 비율을 계산할 수 있다
- 비율을 계산한다
- 클릭 이벤트에서 계산된 비율이 반영된 모델을 넘겨준다
- 넘겨 받은 비율로 다이얼로그의 사진 부분의 비율을 미리 설정한다

그 결과 불필요한 계산 작업(`wrapContentHeight`)을 하지 않고도 깔끔하게 다이얼로그를 띄울 수 있다

### Photo 모델에 `ratio` 프로퍼티 추가

```kotlin
data class PhotoUIModel(  
    val id: String = "",  
    val owner: String = "",  
    val secret: String = "",  
    val server: String = "",  
    val title: String = "",  
) {  
    val loadUrlSmall: String // 검색화면 표시용  
        get() = "https://live.staticflickr.com/$server/${id}_${secret}_n.jpg"  
    val loadUrlMedium: String // 다운로드 확인용  
        get() = "https://live.staticflickr.com/$server/${id}_$secret.jpg"  
    val loadUrlOriginal: String // 다운로드용  
        get() = "https://live.staticflickr.com/$server/${id}_${secret}_o.jpg"  
    val photoId: String  
        get() = "$owner/$id"  
  
    var ratio: Float = 1f  // <-- 새로 추가함
}
```

`ratio` 를 프로퍼티로 추가했고, **1f**로 비율을 초기화하였다.

### Coil 로 이미지 로드

Coil에서 이미지 UI를 표시하기 위해 보통 `AsyncImage` 컴포저블을 사용한다. 이렇게 하면 자체적으로 이미지를 표시할 수 있기 때문에 편리하지만 이미지의 속성이나 로딩 상태를 컨트롤 할 수 없다.

그래서 내부적으로 사용하는 [`rememberAsyncImagePainter`](https://coil-kt.github.io/coil/compose/#asyncimagepainter) 를 사용하게 되었다

*단, `rememberAsyncImagePainter` 는 low-level API이기 때문에 변경될 가능성이 있다*

### 비율 계산

```kotlin
val painter = rememberAsyncImagePainter(  
    model = photo.loadUrlSmall,  
    placeholder = painterResource(id = R.drawable.charlezzicon),  
)  
val imageRatio = remember(painter.state) {  
    val imageSize = painter.intrinsicSize  
    imageSize.width / imageSize.height  
}
```

`rememberAsyncImagePainter` 의 반환 값은 Coil의 `AsyncImagePainter` 타입이며, 이는 컴포즈의 `Painter` 클래스의 하위 타입이다.

<img src="https://i.imgur.com/tgW7Mqf.png" width="33%" />

`AsyncImagePainter` 는 이미지 로딩 **State** 를 가지고 있으며, **State** 는 다음의 속성들을 가지고 있다

imageRatio 에서는 remember 의 key 로 `patiner.state` 를 사용했다. 즉, 이미지 로딩이 완료되면 state가 **Success** 가 되고, 이때 imageRatio 를 계산한 값으로 업데이트 되기 때문에 최신의 비율을 받아볼 수 있게 된다.

### Coil AsyncImagePainter 의 State

```kotlin
sealed class State {  
  
    /** The current painter being drawn by [AsyncImagePainter]. */  
    abstract val painter: Painter?  
  
    /** The request has not been started. */  
    object Empty : State() {  
        override val painter: Painter? get() = null  
    }  
  
    /** The request is in-progress. */  
    data class Loading(  
        override val painter: Painter?,  
    ) : State()  
  
    /** The request was successful. */  
    data class Success(  
        override val painter: Painter,  
        val result: SuccessResult,  
    ) : State()  
  
    /** The request failed due to [ErrorResult.throwable]. */  
    data class Error(  
        override val painter: Painter?,  
        val result: ErrorResult,  
    ) : State()  
}
```

### 이미지 표시하기 및 비율 반영

```kotlin
Box(  
    modifier = Modifier  
        .width(minSize)  
        .aspectRatio(1f)  
        .fillMaxSize(),  
) {  
    Image(  
        painter = painter,  
        contentDescription = null,  
        contentScale = ContentScale.Crop,  
        modifier = Modifier  
            .fillMaxSize()  
            .clickable { onClick(photo.apply { ratio = imageRatio }) },  
    )  
}
```

`AsyncImagePainter` 는 앞서 말했듯이 컴포즈의 `Painter` 타입이기 때문에 `Image` 컴포저블의 `painter` 파라미터로 사용할 수 있다.

클릭이 일어날 때 해당 모델의 ratio 속성으로 앞서 계산한 imageRatio 값을 적용하고 넘겨주게 된다.

### 다이얼로그 비율 설정하기

```kotlin
Column(  
    modifier = modifier  
        .padding(bottom = 30.dp)  
        .wrapContentHeight(),  
    horizontalAlignment = Alignment.CenterHorizontally,  
) {  
    Box(  
        modifier = Modifier.aspectRatio(photo.ratio),  
    ) { 
		// 사진이 들어가는 부분
        AsyncImage(  
            model = photo.loadUrlMedium,  
            contentDescription = "Photo detail : ${photo.title}",  
            contentScale = ContentScale.FillWidth,  
            modifier = Modifier.fillMaxWidth(),  
        )
	}

	// 텍스트와 버튼 그리고 공백
}
```

사진을 감싸는 `Box` 의 `Modifier` 에 `aspectRatio` 를 `photo.ratio` 로 지정한다.

![결과](https://i.imgur.com/4D3r9xy.gif)

그러면 위와 같이 아직 사진이 불러와지지 않은 상황에서도 불러와질 사진의 비율대로 미리 다이얼로그의 크기가 설정되어 표시된다.

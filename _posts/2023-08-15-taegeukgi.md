---
layout: post
title: 안드로이드 태극기 커스텀 뷰
subtitle: Android Taegeukgi CustomView
categories: Android
tags: [android,customview]
---

2023년 8월 15일은 제 78주년 광복절이다.

광복절을 맞아 태극기를 안드로이드 커스텀뷰로 작성해보았다.

덤으로 한글을 사용한 코딩을 해보았다.

![](https://i.imgur.com/HB0dgwA.png)

## 태극기

태극기는 대한민국의 국기이며 가운데 태극문양과 네모서리의 건곤감리 4괘로 구성된다.

각각의 의미는 다음을 읽어보면 좋을 것 같다.

[![](https://i.imgur.com/k3gUnYV.png)](https://www.mois.go.kr/chd/sub/a05/birth/screen.do)

## 태극기 도안

![](https://i.imgur.com/jAZ6Hla.png)

태극기 도안은 [행정안전부 사이트](https://www.mois.go.kr/chd/sub/a05/birth/screen.do)에 공개되어 있다.

참고로 사이트 언어를 영어로 변경하면 조금 더 자세한 가이드가 나와있다.

![](https://i.imgur.com/kcJ94ni.png)

## 태극기 그리기

### 뷰 클래스 만들기

```kotlin
class 태극기(  
    context: Context,  
    attrs: AttributeSet? = null,  
) : View(context, attrs) {
	// ...
}
```

클래스 이름을 한글로 `태극기`로 지었다.

### 색상 정의

[![](https://i.imgur.com/jlKVySm.png)](https://www.mois.go.kr/frt/sub/a06/b08/nationalIcon_2_2/screen.do)
[행정안전부 사이트](https://www.mois.go.kr/frt/sub/a06/b08/nationalIcon_2_2/screen.do)에 태극기에서 사용되는 색상이 정의되어 있지만 CIE나 MUNSELL 색표기 방식이 생소하고, 자료도 많이 나오지 않았다. 그래서 시스템 컬러 피커를 사용하여 색을 추출했다.

```kotlin
class 태극기(  
    context: Context,  
    attrs: AttributeSet? = null,  
) : View(context, attrs) {
	// ...

	companion object {
		const val 하양 = Color.WHITE  
		val 검정 = Color.parseColor("#0D0D0D")  
		val 파랑 = Color.parseColor("#134A9D")  
		val 빨강 = Color.parseColor("#D0303C")
	}
}
```

### 비율 고정하기

```kotlin
class 태극기(  
    context: Context,  
    attrs: AttributeSet? = null,  
) : View(context, attrs) {  

    private var 너비 = 0  
    private var 높이 = 0  

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {  
        너비 = MeasureSpec.getSize(widthMeasureSpec)  
        높이 = (너비 * 2F / 3).toInt()  
        setMeasuredDimension(너비, 높이)  
    }  
  
    companion object {  
        // 색상 정의
    }  
}  
```

태극기의 비율은 가로:세로=3:2 비율을 가지기 때문에 고정적인 비율을 위해 `setMeasuredDimension`을 사용해 정의해주었다.

그리고 이때 뷰의 너비와 높이 값을 설정해준다.

### 타입 별명 지정

한글을 사용한 코딩을 하면서 Paint와 같은 타입도 한글로 작성해주고 싶었다. (하지만 모든 것을 다 한글로 바꾸면 가독성이 너무 안 좋아 질 것 같아서 Paint 정도만 한글로 바꿔주었다)

```kotlin
typealias 물감 = Paint
```

참고로 typealias 는 우리의 태극기 뷰가 아닌 최상단에 정의되어야 한다.

### 태극문양 그리기

#### 길이, 각도 계산

```kotlin
class 태극기(  
    context: Context,  
    attrs: AttributeSet? = null,  
) : View(context, attrs) {  
    private val 태극문양물감 = 물감()  
  
    private val 태극문양회전각도 = Math.toDegrees(atan(2.0 / 3)).toFloat()  
    private var 너비 = 0  
    private var 높이 = 0  
    private val 태극문양영역: RectF  
        get() = RectF(  
            태극반지름 * 2,  
            높이 / 2 - 태극반지름,  
            태극반지름 * 4,  
            높이 / 2 + 태극반지름,  
        )  
    private val 태극반지름: Float  
        get() = 너비 / 6F  
    private val 가로중앙  
        get() = 너비 / 2F  
    private val 세로중앙  
        get() = 높이 / 2F  

	// ...
}  
  
typealias 물감 = Paint
```

`너비`와 `높이`는 onMeasure에서 정의되었고, `태극반지름`, `가로중앙`, `세로중앙`은 태극기 도안을 보면 쉽게 구할 수 있다. (단, `너비`와 `높이`의 초기 값이 0이므로 `get()`을 사용하여 최신의 값을 계산해야한다.)

`태극문양영역`은 다음 영역을 의미한다

![](https://i.imgur.com/L4IZOO2.png)

`태극문양회전각도`는 다음의 각도를 의미한다

![](https://i.imgur.com/AM9wtsz.png)

`태극문양회전각도`는 위 사진과 같이 직각삼각형을 만들어 *tan*(탄젠트)를 이용하면 구할 수 있다.

큰 직각삼각형을 보면 밑변이 태극기 가로길이의 반을 차지하고, 높이가 태극기 세로길이의 반을 차지하기 때문에 2:3의 비율을 가지며, 탄젠트 공식에 의해 *tan(⍬) = ⅔* 이 된다. 결국 ⍬를 구하기 위해서 *atan*(아크탄젠트)를 사용할 수 있으며, `atan`은 코틀린에서 기본으로 제공되고 있기에 쉽게 구할 수 있다.

단, 제대로 각도를 구하기 위해서는 Degrees 표기법으로 변환해주어야 하며, 이것도 `Math.toDegrees`를 통해 제공된다.

`private val 태극문양회전각도 = Math.toDegrees(atan(2.0 / 3)).toFloat()`

#### 반원 그리기

![](https://i.imgur.com/xnBSSvc.png)

drawArc 를 통해 빨간 반원과, 파란 반원을 그릴 것이다.

```kotlin
class 태극기(  
    context: Context,  
    attrs: AttributeSet? = null,  
) : View(context, attrs) {  

    override fun onDraw(canvas: Canvas) {  
        super.onDraw(canvas)  
        canvas.drawColor(하양)  
  
        // 태극 문양  
        태극문양물감.apply {  
            color = 빨강  
            canvas.drawArc(태극문양영역, 180 + 태극문양회전각도, 180F, true, this)  
  
            color = 파랑  
            canvas.drawArc(태극문양영역, 태극문양회전각도, 180F, true, this)
        }  
    }  

	// ...
}  
```

![](https://i.imgur.com/BEoHenY.png)

`drawArc` 에서 시작 각도는 위 그림과 같다.

그래서 빨간 반원을 그릴 때는 `180 + 태극문양회전각도`를 한 것이다.

#### 내부에 작은 원 (빨강)

![](https://i.imgur.com/0vqVEPv.png)

위와 같은 그림을 그리기 위해서는 각각 *cos*과 *sin*을 사용하여 x, y 좌표를 구하고, 위치를 보정하여 작은 원을 그려줘야 한다.

색상을 바꿔서 보여주면 다음과 같다.

![](https://i.imgur.com/6xBDLc8.png)

(수학시간이 아니니 더 자세한 수학적 설명은 하지 않겠다)

```kotlin
// 태극 문양  
태극문양물감.apply {  
    color = 빨강  
    canvas.drawArc(태극문양영역, 180 + 태극문양회전각도, 180F, true, this)  
  
    color = 파랑  
    canvas.drawArc(태극문양영역, 태극문양회전각도, 180F, true, this)  
  
    color = 빨강  
    canvas.drawCircle(  
        가로중앙 + (태극반지름 / 2) * cos(Math.toRadians(태극문양회전각도 + 180.0)).toFloat(),  
        세로중앙 + (태극반지름 / 2) * sin(Math.toRadians(태극문양회전각도 + 180.0)).toFloat(),  
        태극반지름 / 2,  
        this,  
    )
```

#### 내부에 작은 원 (파랑)

비슷한 방식으로 파란 원도 그려준다.

```kotlin
// 태극 문양  
태극문양물감.apply {  
    color = 빨강  
    canvas.drawArc(태극문양영역, 180 + 태극문양회전각도, 180F, true, this)  
  
    color = 파랑  
    canvas.drawArc(태극문양영역, 태극문양회전각도, 180F, true, this)  
  
    color = 빨강  
    canvas.drawCircle(  
        가로중앙 + (태극반지름 / 2) * cos(Math.toRadians(태극문양회전각도 + 180.0)).toFloat(),  
        세로중앙 + (태극반지름 / 2) * sin(Math.toRadians(태극문양회전각도 + 180.0)).toFloat(),  
        태극반지름 / 2,  
        this,  
    )

	color = 파랑  
	canvas.drawCircle(  
	    가로중앙 + (태극반지름 / 2) * cos(Math.toRadians(태극문양회전각도.toDouble())).toFloat(),  
	    세로중앙 + (태극반지름 / 2) * sin(Math.toRadians(태극문양회전각도.toDouble())).toFloat(),  
	    태극반지름 / 2,  
	    this,  
	)
```

그럼 다음과 같이 태극 문양을 완성할 수 있다.

![](https://i.imgur.com/1vvKykt.png)

다른 색으로 표현하면 다음과 같은 모습이다

![](https://i.imgur.com/P2WGRtp.png)

### 괘 그리기

#### 괘 속성

```kotlin
private val 괘_물감 = 물감().apply {  
    color = 검정  
    style = Paint.Style.FILL  
}

private val 괘_높이  
    get() = 태극반지름 / 6  
private val 괘_간격  
    get() = 태극반지름 / 12  
private val 괘_너비  
    get() = 태극반지름  
private val 괘_중앙으로부터_거리  
    get() = 높이 * (3F / 8)
```

괘 속성 역시 태극기 도안에서 쉽게 수치를 알 수 있다.

#### 괘 그리는 과정

괘를 그릴 때는 각도를 사용하여 위치와 크기를 계산하고 그리는 것이 불가능은 아니지만 상당히 까다로운 작업이다.

하지만 다행히도 조금 더 편한 방법이 있다.

바로 캔버스를 조금 돌려서 그린 후 다시 되돌리는 방법이다.

이는 `canvas.store()` 를 통해 현재 캔버스 상태를 저장한 후 `canvas.rotate()`로 회전하여 그리기 작업을 완료한 후 `canvas.restore()` 를 통해 `store()`로 저장한 상태로 복구할 수 있다. 즉, 다시 원래대로 회전시킬 수 있다.

```kotlin
canvas.apply {  
    save()    
    rotate(90 + 태극문양회전각도, 가로중앙, 세로중앙)  
    // 건괘 그리기 
    restore()  
    
    save()    
    rotate(270 + 태극문양회전각도, 가로중앙, 세로중앙)  
    // 곤괘 그리기  
    restore()  
    
    save()    
    rotate(270 - 태극문양회전각도, 가로중앙, 세로중앙)  
    // 감괘 그리기    
    restore()  
    
    save()    
    rotate(90 - 태극문양회전각도, 가로중앙, 세로중앙)  
    // 이괘 그리기   
    restore()
}
```

위와 같은 방식으로 그릴 것이다.

건곤감리는 규칙성이 있어서 함수로 뺄 수 있을 것 같다.

- 건곤감리의 두께는 태극문양 지름의 1/12 로 모두 동일하다
- 건곤감리의 너비는 긴 것은 태극문양 지름의 ½, 짧은 것은 태극문양 지름의 ½ - 1/24 로 일정하다
- 각 괘는 3개의 막대기로 이루어져 있다
- 3개의 막대기는 동일한 간격을 가진다
- 건곤감리는 태극문양 지름의 ¼ 만큼 떨어진 곳부터 그려진다

위와 같은 규칙을 기반으로 다음의 함수를 작성해보았다.

```kotlin
private fun 괘_그리기(캔버스: Canvas, 순번: Int, 작은괘: Boolean = false) {  
    val 높이 = 괘_중앙으로부터_거리 + 순번 * 괘_간격 + (순번 + 1) * 괘_높이  
  
    if (작은괘) {  
        캔버스.drawRect(  
            가로중앙 - 괘_너비 / 2,  
            세로중앙 + 높이 + 괘_높이,  
            가로중앙 - 괘_간격 / 2,  
            세로중앙 + 높이,  
            괘_물감,  
        )  
        캔버스.drawRect(  
            가로중앙 + 괘_간격 / 2,  
            세로중앙 + 높이 + 괘_높이,  
            가로중앙 + 괘_너비 / 2,  
            세로중앙 + 높이,  
            괘_물감,  
        )  
    } else {  
        캔버스.drawRect(  
            가로중앙 - 괘_너비 / 2,  
            세로중앙 + 높이 + 괘_높이,  
            가로중앙 + 괘_너비 / 2,  
            세로중앙 + 높이,  
            괘_물감,  
        )  
    }  
}
```

`순번`은 태극문양과 가까운 순서대로 0, 1, 2 가 입력될 것이다.

`높이`는 태극문양과 가장 가까운 막대기가 그려지기 시작하는 위치이며, 태극기뷰의 `높이` 프로퍼티와는 다른 값이다.

`작은괘`가 `true`라면 2개의 작은 막대기를 그리고, `false`라면 1개의 긴 막대기를 그린다.

이 함수를 적용하면 다음과 같이 작성할 수 있다.

```kotlin
canvas.apply {  
    save()  
    // 건괘  
    rotate(90 + 태극문양회전각도, 가로중앙, 세로중앙)  
    repeat(3) {  
        괘_그리기(this, it)  
    }  
    restore()  
  
    // 곤괘  
    save()  
    rotate(270 + 태극문양회전각도, 가로중앙, 세로중앙)  
    repeat(3) {  
        괘_그리기(this, it, true)  
    }  
    restore()  
  
    // 감괘  
    save()  
    rotate(270 - 태극문양회전각도, 가로중앙, 세로중앙)  
    repeat(3) {  
        괘_그리기(this, it, it != 1)  
    }  
    restore()  
  
    // 이괘  
    save()  
    rotate(90 - 태극문양회전각도, 가로중앙, 세로중앙)  
    repeat(3) {  
        괘_그리기(this, it, it == 1)  
    }  
    restore()  
}
```

### 완성

![](https://i.imgur.com/CYVFqkn.png)

이렇게 건곤감리까지 그리면 멋진 태극기가 그려진다.

## 느낀 점

### 태극기 커스텀 뷰

우선 태극기를 이렇게 면밀히 살펴본 것은 거의 처음인 것 같다. 그리고 태극기가 굉장히 조화롭다고 느껴졌다.

구현 난이도는 생각보다 높았다. 오랜만에 삼각함수를 살펴보기도 했고, 각도에 대한 이해가 있어야 그릴 수 있었던 것 같다.

### 한글 코딩

파이썬이나 테스트코드의 이름에서는 종종 한글로 된 변수명이나 함수명을 사용하기도 했지만, 이렇게 사용한 것은 처음이다.

그래서 앞으로 한글 코딩을 종종 할 것인가? - 그렇지 않을 것 같다.

- 안드로이드 스튜디오에서 한글을 잘 지원하지 않는다. rename 중에 이상한 곳으로 이동한다거나 자동완성이 미흡하다.
- 경고가 많이 뜬다. Ascii 코드를 사용하라는 경고와 파스칼케이스 등의 컨벤션을 지키지 않았다는 경고가 계속해서 뜬다. (개인적으로 이런 경고는 참지 못한다…)
- 가독성이 생각보다 좋지 않다. (익숙함의 차이일수도 있겠다)
- 리소스 명으로는 아예 한글이 불가능하다.

하지만 재밌는 경험이었다.

## 전체 코드

[Github](https://github.com/mangbaam/CustomView/blob/main/%ED%83%9C%EA%B7%B9%EA%B8%B0/src/main/java/com/mangbaam/taegeukgi/%ED%83%9C%EA%B7%B9%EA%B8%B0.kt)에서 볼 수 있습니다~!

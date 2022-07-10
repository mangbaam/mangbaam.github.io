---
layout: post
title: AutoHotKey를 사용해 완전 편하게 개발자 키매핑 하기
subtitle: 
categories: TIP
tags: [autohotkey,keymapping,tip]
---

## ⭐

본 게시글에서 공유한 스크립트는 언제든 변경되거나 추가될 수 있습니다!

[깃허브](https://github.com/mangbaam/AutoHotKey)에서 최신의 내용을 확인할 수 있습니다. 복사해서 자유롭게 사용하세요!

더 좋은 방법이나 버그, 추가하면 좋을 키매핑 등 자유롭게 의견 주세요!

[완성된 키매핑](#완료된-모습)을 바로 보고싶다면 클릭하세요.

## 극한의 효율성

---

![annepro2](https://user-images.githubusercontent.com/44221447/178151444-dd6314ca-6c10-4d5c-9cfb-f823c5719258.png)

나는 **Anne Pro2** 키보드를 2년 간 사용하고 더 좋은 키감을 가진 키보드를 가지고 싶다는 생각으로 **Durgod K320KR** 저소음 적축 모델을 구매하게 되었다.

키감은 정말 만족스러웠다. 하지만 포커 배열(60% 배열)인 앤프로2를 사용할 때 두 손이 키보드를 절대 벗어나지 않도록 (개인적으로) 완벽한 키매핑을 했기에 텐키리스 키보드에서 방향키나 `HOME`, `END` 등의 키를 누르기 위해 손이 자리에서 벗어나서 오른쪽으로 이동해야 한다는 점이 너무나도 불편하게 느껴졌다.

물론 한 달 정도 현재 키보드로만 타이핑을 하다보니 생각보다 금방 익숙해졌지만 이미 극한의 효율을 맛 본 이후라 만족스럽지는 못했다.

![powertoys](https://user-images.githubusercontent.com/44221447/178151664-b1aaea74-ae9f-46c3-9e77-3932fcb6e800.png)

처음에는 **PowerToys**의 키매핑 기능으로 비교적 덜 사용되는 `Alt` 키를 가지고 조합을 시도했다. 참고로 앤프로2는 `CapsLock` 버튼을 `Fn키`로 사용하기 때문에 엄청난 효율을 챙길 수 있는 것이었는데, 키보드에 있는 `Fn키`는 공식 키매핑 프로그램이나 딥 스위치로 제공하지 않으면 옮기기 힘들고, `CapsLock`은 `Ctrl`, `Alt`, `Shift`와 같은 조합 키가 아니기 때문에 **PowerToys**의 기능으로는 `CapsLock` 위치에 `Alt` 키를 매핑한 후 `Alt`와 다른 키들의 조합으로 하는 것이 최선이었다.

하지만 나는 단축키를 사용하는 것을 즐기기에 생각보다 Alt 키를 사용하는 경우도 많이 있었고, 이때마다 조합해두었던 키들과 충돌을 일으켜 원하는 기능을 제대로 사용할 수 없는 경우가 허다했다.

그렇게 불편함을 감수하고 있다가 오늘 우연히 **Auto Hot Key** 라는 프로그램을 알게되어 기존의 앤프로2 키매핑과 거의 유사하게 스크립트를 짜서 적용해볼 수 있었다.

다른 사람들도 꼭 키보드에서 손이 벗어나지 않아도 되는 **극한의 효율성**을 느끼게 해주고 싶어서 스크립트와 방법을 공유하고자 한다.

> 앤프로 후기 및 키매핑 -> [https://latte-is-horse.tistory.com/89](https://latte-is-horse.tistory.com/89)

## 설치

---

AutoHotKey를 사용하려면 우선 설치를 해야한다.

[https://www.autohotkey.com/download/](https://www.autohotkey.com/download/)

위 공식 링크에서 설치 파일을 다운로드 할 수 있다.

<img src="https://user-images.githubusercontent.com/44221447/178151925-5261a38a-cb68-422c-a687-aba429789b62.png" width="50%" />

Express Installation 밑에 있는 경로에 그대로 설치하려면 Express Installation을, 다른 경로에 설치하려면 Custom Installation을 선택한다.

<img src="https://user-images.githubusercontent.com/44221447/178151968-4994f6a2-fc87-44c3-8b10-e50c0dc97eb3.png" width="50%" />

Exit을 누르고 설치를 종료한다.

## VS Code 설정

---

AutoHotKey는 일종의 스크립트인데 메모장으로도 편집이 가능하지만 개발자라면 대부분 설치되어 있을 VS Code에 설정하고 작성하는 방법을 공유하고자 한다.

![vscode](https://user-images.githubusercontent.com/44221447/178152038-802ad79a-b4ba-4779-99c9-4dc3cb43a405.png)

VS Code의 확장 프로그램에서 검색해보면 AutoHotKey 확장 프로그램이 있다. `설치`한다.

![코드러너](https://user-images.githubusercontent.com/44221447/178152095-1d8eb400-b509-46c4-a0e3-a47542d79f16.png)

아마 VS Code를 사용한다면 Code Runner는 대부분 설치되어 있을 것 같은데 Code Runner에서도 `.ahk` 확장자를 지원한다. 이 확장자가 AutoHotKey 파일의 확장자이다.

*settings.json*

```json
{
    "code-runner.executorMap": {
        // ...
        "ahk": "\"C:\\Program Files\\AutoHotkey\\AutoHotkey.exe\"",
        // ...
    },
    "code-runner.executorMapByFileExtension": {
		// ...
        ".ahk": "\"C:\\Program Files\\AutoHotkey\\AutoHotkey.exe\"",
        // ...
    },
	// ...
    "[ahk]": 
    {
        "files.encoding": "utf8",
    },
}
```

`F1` 이나 `Ctrl + p`를 눌러 `settings.json`을 열고 위와 같이 추가한다.

![경로](https://user-images.githubusercontent.com/44221447/178152298-18440981-ab07-445d-9e47-817879e22643.png)

`ahk`에 추가되는 경로는 실제 경로를 선택하자.

*keybindings.json*

```json
[
	// ...
	{ "key": "f5", "command": "code-runner.run","when": "editorTextFocus && editorLangId == 'ahk'"},
	{ "key": "f6", "command": "code-runner.stop" },
]
```

마찬가지로 Open Keyboard Shortcuts (JSON)에서 위 코드를 추가한다. `ahk` 파일에서 `F5`를 누르면 실행되도록 하는 단축키를 추가하는 부분이다.

![트레이](https://user-images.githubusercontent.com/44221447/178152373-c0ea2d30-4998-414a-a88a-2bcee236117a.png)

저장하고 `ahk` 파일에서 `F5` 키를 눌러보면 위와 같이 트레이에 추가되어 실행되는 것을 확인할 수 있다.

## 키매핑

---

![앤프로](https://user-images.githubusercontent.com/44221447/178152449-2e64fd3b-4803-4b77-b2e0-04f911a45f35.png)

목표는 기존의 앤프로와 유사하게 만드는 것이다.

### CapsLock을 Fn키로 사용하기

```ahk
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

#if fn
; 키매핑
```

`CapsLock`에 `fn` 이라는 변수를 할당했고, 누르면 1, 떼면 0으로 토글되도록 하였다.

가끔 캡스락을 사용하는 경우가 있기 때문에 `Win + Space` 키로 캡스락 기능을 하도록 만들었다.

`#if fn` 밑으로 키매핑을 하면 캡스락이 눌린 상태에서 동작하는 키를 만들 수 있다.

### 방향키

![방향키](https://user-images.githubusercontent.com/44221447/178152533-4a9b86a8-a3ab-465a-8479-baadb78ed57f.png)

```ahk
;***************************************
;   캡스락 설정
;***************************************

*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   방향키 설정
;***************************************
; 기본 WASD, IJKL
#if fn
j::Send, {Left} ; LEFT
a::Send, {Left}
k::Send, {Down} ; DOWN
s::Send, {Down}
l::Send, {Right} ; RIGHT
d::Send, {Right}
i::Send, {Up} ; UP
w::Send, {Up}

; Shift 조합
+j::Send, {ShiftDown}{Left} ; LEFT
+a::Send, {ShiftDown}{Left}
+k::Send, {ShiftDown}{Down} ; DOWN
+s::Send, {ShiftDown}{Down}
+l::Send, {ShiftDown}{Right} ; RIGHT
+d::Send, {ShiftDown}{Right}
+w::Send, {ShiftDown}{Up}
+i::Send, {ShiftDown}{Up} ; UP

; Ctrl 조합
^j::Send, {CtrlDown}{Left} ; LEFT
^a::Send, {CtrlDown}{Left}
^k::Send, {CtrlDown}{Down} ; DOWN
^s::Send, {CtrlDown}{Down}
^l::Send, {CtrlDown}{Right} ; RIGHT
^d::Send, {CtrlDown}{Right}
^i::Send, {CtrlDown}{Up} ; UP
^w::Send, {CtrlDown}{Up}

; Alt 조합
!j::Send, {AltDown}{Left} ; LEFT
!a::Send, {AltDown}{Left}
!k::Send, {AltDown}{Down} ; DOWN
!s::Send, {AltDown}{Down}
!l::Send, {AltDown}{Right} ; RIGHT
!d::Send, {AltDown}{Right}
!i::Send, {AltDown}{Up} ; UP
!w::Send, {AltDown}{Up}

; Ctrl Shift 조합
^+j::Send, {CtrlDown}{ShiftDown}{Left} ; LEFT
^+a::Send, {CtrlDown}{ShiftDown}{Left}
^+k::Send, {CtrlDown}{ShiftDown}{Down} ; DOWN
^+s::Send, {CtrlDown}{ShiftDown}{Down}
^+l::Send, {CtrlDown}{ShiftDown}{Right} ; RIGHT
^+d::Send, {CtrlDown}{ShiftDown}{Right}
^+i::Send, {CtrlDown}{ShiftDown}{Up} ; UP
^+w::Send, {CtrlDown}{ShiftDown}{Up}
```

위 사진과 같이 방향키를 설정했다. 방향키는 조합키과 사용하는 경우가 많아서 주로 사용되는 조합을 추가했다.

> 혹시 이렇게 일일이 지정하지 않아도 되는 방법이 있다면 알려주세요!!

### F 키

![f키](https://user-images.githubusercontent.com/44221447/178152825-f793b700-4341-49fe-96c1-edec6feb89d8.png)

```ahk
;***************************************
;   캡스락 설정
;***************************************
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   F키 설정
;***************************************
#if fn
1::Send, {F1}
2::Send, {F2}
3::Send, {F3}
4::Send, {F4}
5::Send, {F5}
6::Send, {F6}
7::Send, {F7}
8::Send, {F8}
9::Send, {F9}
0::Send, {F10}
-::Send, {F11}
=::Send, {F12}

; Shift 조합
+1::Send, {ShiftDown}{F1}
+2::Send, {ShiftDown}{F2}
+3::Send, {ShiftDown}{F3}
+4::Send, {ShiftDown}{F4}
+5::Send, {ShiftDown}{F5}
+6::Send, {ShiftDown}{F6}
+7::Send, {ShiftDown}{F7}
+8::Send, {ShiftDown}{F8}
+9::Send, {ShiftDown}{F9}
+0::Send, {ShiftDown}{F10}
+-::Send, {ShiftDown}{F11}
+=::Send, {ShiftDown}{F12}

; Ctrl 조합
^1::Send, {CtrlDown}{F1}
^2::Send, {CtrlDown}{F2}
^3::Send, {CtrlDown}{F3}
^4::Send, {CtrlDown}{F4}
^5::Send, {CtrlDown}{F5}
^6::Send, {CtrlDown}{F6}
^7::Send, {CtrlDown}{F7}
^8::Send, {CtrlDown}{F8}
^9::Send, {CtrlDown}{F9}
^0::Send, {CtrlDown}{F10}
^-::Send, {CtrlDown}{F11}
^=::Send, {CtrlDown}{F12}

; Ctrl + Shift 조합
^+1::Send, {CtrlDown}{ShiftDown}{F1}
^+2::Send, {CtrlDown}{ShiftDown}{F2}
^+3::Send, {CtrlDown}{ShiftDown}{F3}
^+4::Send, {CtrlDown}{ShiftDown}{F4}
^+5::Send, {CtrlDown}{ShiftDown}{F5}
^+6::Send, {CtrlDown}{ShiftDown}{F6}
^+7::Send, {CtrlDown}{ShiftDown}{F7}
^+8::Send, {CtrlDown}{ShiftDown}{F8}
^+9::Send, {CtrlDown}{ShiftDown}{F9}
^+0::Send, {CtrlDown}{ShiftDown}{F10}
^+-::Send, {CtrlDown}{ShiftDown}{F11}
^+=::Send, {CtrlDown}{ShiftDown}{F12}

; Alt 조합
!1::Send, {AltDown}{F1}
!2::Send, {AltDown}{F2}
!3::Send, {AltDown}{F3}
!4::Send, {AltDown}{F4}
!5::Send, {AltDown}{F5}
!6::Send, {AltDown}{F6}
!7::Send, {AltDown}{F7}
!8::Send, {AltDown}{F8}
!9::Send, {AltDown}{F9}
!0::Send, {AltDown}{F10}
!-::Send, {AltDown}{F11}
!=::Send, {AltDown}{F12}
```

### Home, End, PageUp, PageDown, Insert, Delete

![기능키](https://user-images.githubusercontent.com/44221447/178152937-848af9d4-fdab-41ad-b7d8-02d1e6e0037a.png)

코딩할 때 정말 유용하게 사용되는 키들이다.

```ahk
;***************************************
;   캡스락 설정
;***************************************
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   HOME, END, PGUP, PGDN 설정
;***************************************
#if fn
`;::Send, {Home} ; HOME
'::Send, {End} ; END
q::Send, {PgUp} ; PageUp
e::Send, {PgDn} ; PageDown

; Shift 조합
+`;::Send, {ShiftDown}{Home} ; HOME
+'::Send, {ShiftDown}{End} ; END
+q::Send, {ShiftDown}{PgUp} ; PageUp
+e::Send, {ShiftDown}{PgDn} ; PageDown

; Alt 조합
!`;::Send, {AltDown}{Home} ; HOME
!'::Send, {AltDown}{End} ; END
!q::Send, {AltDown}{PgUp} ; PageUp
!e::Send, {AltDown}{PgDn} ; PageDown

;***************************************
;   Insert, Delete 설정
;***************************************
.::Send, {Insert} ; INSERT
/::Send, {Delete} ; DELETE

; Alt 조합
!.::Send, {AltDown}{Insert} ; INSERT
!/::Send, {AltDown}{Delete} ; DELETE

; Shift 조합
+.::Send, {ShiftDown}{Insert} ; INSERT
+/::Send, {ShiftDown}{Delete} ; DELETE

; Ctrl 조합
^.::Send, {CtrlDown}{Insert} ; INSERT
^/::Send, {CtrlDown}{Delete} ; DELETE

; Ctrl + Alt 조합
^!.::Send, {CtrlDown}{AltDown}{Insert} ; INSERT
^!/::Send, {CtrlDown}{AltDown}{Delete} ; DELETE
```

### 멀티미디어 키

![멀티미디어](https://user-images.githubusercontent.com/44221447/178153014-815662c1-21f9-4cc0-b637-9f8cf65f96b3.png)

역시 유용하게 사용되는 키다.

```ahk
;***************************************
;   캡스락 설정
;***************************************
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   멀티미디어 설정
;***************************************
#if fn
[::Volume_Down
]::Volume_Up
m::Volume_Mute
```

### 기타

![기타](https://user-images.githubusercontent.com/44221447/178153096-d46a374e-e67d-40bd-be2f-37fe0a22de84.png)

해피해킹 키보드에서는 일반적인 CapsLock 자리에 Ctrl이 존재하고, 실제로 복사, 붙여넣기 단축키인 `Ctrl + c`, `Ctrl + v`를 할 때 `Ctrl` 위치가 `CapsLock` 자리에 있으면 훨씬 손목을 덜 꺾으면서 손도 벗어나지 않으면서 할 수 있다.

그래서 `CapsLock + c`, `CapsLock + v`로 복사, 붙여넣기 할 수 있도록 만들었다.

H 자리에는 `~`를 넣었는데, 앤프로2에서 `~`를 적으려면 굉장히 불편했기에 저 자리에 매핑했던 것이 손에 익어서 넣어봤다.

```ahk
;***************************************
;   캡스락 설정
;***************************************
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   단축키 설정
;***************************************
#if fn
c::Send, {CtrlDown}{c} ; COPY
v::Send, {CtrlDown}{v} ; PASTE

;***************************************
;   기타 설정
;***************************************
h::Send, ~ ; WAVE
```

## 완료된 모습

![total](https://user-images.githubusercontent.com/44221447/178153504-a99cdf6b-47f8-4cf6-b908-4abc444efd8f.png)

```ahk
;***************************************
;   캡스락 설정
;***************************************
*CapsLock:: fn:=1
*CapsLock Up:: fn:=0
LWin & Space:: CapsLock ; 캡스락 대체키

;***************************************
;   방향키 설정
;***************************************
; 기본 WASD, IJKL
#if fn
j::Send, {Left} ; LEFT
a::Send, {Left}
k::Send, {Down} ; DOWN
s::Send, {Down}
l::Send, {Right} ; RIGHT
d::Send, {Right}
i::Send, {Up} ; UP
w::Send, {Up}

; Shift 조합
+j::Send, {ShiftDown}{Left} ; LEFT
+a::Send, {ShiftDown}{Left}
+k::Send, {ShiftDown}{Down} ; DOWN
+s::Send, {ShiftDown}{Down}
+l::Send, {ShiftDown}{Right} ; RIGHT
+d::Send, {ShiftDown}{Right}
+w::Send, {ShiftDown}{Up}
+i::Send, {ShiftDown}{Up} ; UP

; Ctrl 조합
^j::Send, {CtrlDown}{Left} ; LEFT
^a::Send, {CtrlDown}{Left}
^k::Send, {CtrlDown}{Down} ; DOWN
^s::Send, {CtrlDown}{Down}
^l::Send, {CtrlDown}{Right} ; RIGHT
^d::Send, {CtrlDown}{Right}
^i::Send, {CtrlDown}{Up} ; UP
^w::Send, {CtrlDown}{Up}

; Alt 조합
!j::Send, {AltDown}{Left} ; LEFT
!a::Send, {AltDown}{Left}
!k::Send, {AltDown}{Down} ; DOWN
!s::Send, {AltDown}{Down}
!l::Send, {AltDown}{Right} ; RIGHT
!d::Send, {AltDown}{Right}
!i::Send, {AltDown}{Up} ; UP
!w::Send, {AltDown}{Up}

; Ctrl Shift 조합
^+j::Send, {CtrlDown}{ShiftDown}{Left} ; LEFT
^+a::Send, {CtrlDown}{ShiftDown}{Left}
^+k::Send, {CtrlDown}{ShiftDown}{Down} ; DOWN
^+s::Send, {CtrlDown}{ShiftDown}{Down}
^+l::Send, {CtrlDown}{ShiftDown}{Right} ; RIGHT
^+d::Send, {CtrlDown}{ShiftDown}{Right}
^+i::Send, {CtrlDown}{ShiftDown}{Up} ; UP
^+w::Send, {CtrlDown}{ShiftDown}{Up}

;***************************************
;   HOME, END, PGUP, PGDN 설정
;***************************************
`;::Send, {Home} ; HOME
'::Send, {End} ; END
q::Send, {PgUp} ; PageUp
e::Send, {PgDn} ; PageDown

; Shift 조합
+`;::Send, {ShiftDown}{Home} ; HOME
+'::Send, {ShiftDown}{End} ; END
+q::Send, {ShiftDown}{PgUp} ; PageUp
+e::Send, {ShiftDown}{PgDn} ; PageDown

; Alt 조합
!`;::Send, {AltDown}{Home} ; HOME
!'::Send, {AltDown}{End} ; END
!q::Send, {AltDown}{PgUp} ; PageUp
!e::Send, {AltDown}{PgDn} ; PageDown

;***************************************
;   Insert, Delete 설정
;***************************************
.::Send, {Insert} ; INSERT
/::Send, {Delete} ; DELETE

; Alt 조합
!.::Send, {AltDown}{Insert} ; INSERT
!/::Send, {AltDown}{Delete} ; DELETE

; Shift 조합
+.::Send, {ShiftDown}{Insert} ; INSERT
+/::Send, {ShiftDown}{Delete} ; DELETE

; Ctrl 조합
^.::Send, {CtrlDown}{Insert} ; INSERT
^/::Send, {CtrlDown}{Delete} ; DELETE

; Ctrl + Alt 조합
^!.::Send, {CtrlDown}{AltDown}{Insert} ; INSERT
^!/::Send, {CtrlDown}{AltDown}{Delete} ; DELETE

;***************************************
;   F키 설정
;***************************************
1::Send, {F1}
2::Send, {F2}
3::Send, {F3}
4::Send, {F4}
5::Send, {F5}
6::Send, {F6}
7::Send, {F7}
8::Send, {F8}
9::Send, {F9}
0::Send, {F10}
-::Send, {F11}
=::Send, {F12}

; Shift 조합
+1::Send, {ShiftDown}{F1}
+2::Send, {ShiftDown}{F2}
+3::Send, {ShiftDown}{F3}
+4::Send, {ShiftDown}{F4}
+5::Send, {ShiftDown}{F5}
+6::Send, {ShiftDown}{F6}
+7::Send, {ShiftDown}{F7}
+8::Send, {ShiftDown}{F8}
+9::Send, {ShiftDown}{F9}
+0::Send, {ShiftDown}{F10}
+-::Send, {ShiftDown}{F11}
+=::Send, {ShiftDown}{F12}

; Ctrl 조합
^1::Send, {CtrlDown}{F1}
^2::Send, {CtrlDown}{F2}
^3::Send, {CtrlDown}{F3}
^4::Send, {CtrlDown}{F4}
^5::Send, {CtrlDown}{F5}
^6::Send, {CtrlDown}{F6}
^7::Send, {CtrlDown}{F7}
^8::Send, {CtrlDown}{F8}
^9::Send, {CtrlDown}{F9}
^0::Send, {CtrlDown}{F10}
^-::Send, {CtrlDown}{F11}
^=::Send, {CtrlDown}{F12}

; Ctrl + Shift 조합
^+1::Send, {CtrlDown}{ShiftDown}{F1}
^+2::Send, {CtrlDown}{ShiftDown}{F2}
^+3::Send, {CtrlDown}{ShiftDown}{F3}
^+4::Send, {CtrlDown}{ShiftDown}{F4}
^+5::Send, {CtrlDown}{ShiftDown}{F5}
^+6::Send, {CtrlDown}{ShiftDown}{F6}
^+7::Send, {CtrlDown}{ShiftDown}{F7}
^+8::Send, {CtrlDown}{ShiftDown}{F8}
^+9::Send, {CtrlDown}{ShiftDown}{F9}
^+0::Send, {CtrlDown}{ShiftDown}{F10}
^+-::Send, {CtrlDown}{ShiftDown}{F11}
^+=::Send, {CtrlDown}{ShiftDown}{F12}

; Alt 조합
!1::Send, {AltDown}{F1}
!2::Send, {AltDown}{F2}
!3::Send, {AltDown}{F3}
!4::Send, {AltDown}{F4}
!5::Send, {AltDown}{F5}
!6::Send, {AltDown}{F6}
!7::Send, {AltDown}{F7}
!8::Send, {AltDown}{F8}
!9::Send, {AltDown}{F9}
!0::Send, {AltDown}{F10}
!-::Send, {AltDown}{F11}
!=::Send, {AltDown}{F12}

;***************************************
;   멀티미디어 설정
;***************************************
[::Volume_Down
]::Volume_Up
m::Volume_Mute

;***************************************
;   단축키 설정
;***************************************
c::Send, {CtrlDown}{c} ; COPY
v::Send, {CtrlDown}{v} ; PASTE

;***************************************
;   기타 설정
;***************************************
h::Send, ~ ; WAVE
```

## 시작 프로그램에 등록

---

스크립트를 만들었다면 `F5`를 눌러서 실행할 수 있다. 하지만 컴퓨터를 재부팅하면 다시 파일을 실행시켜야 하는 불편함이 있다.

이건 [시작 프로그램] 폴더에 스크립트를 넣어놓으면 간단히 해결된다.

### 바로가기 파일 생성

![바로가기](https://user-images.githubusercontent.com/44221447/178153645-b398185d-6fca-4be2-bdb6-2dc02b9bb4c5.png)

마우스 우클릭으로 바로가기 파일을 만들거나 `Alt`를 누른 채로 파일을 드래그하면 바로가기 파일이 만들어진다.

### 시작 프로그램에 넣기

![폴더](https://user-images.githubusercontent.com/44221447/178153746-c1cda12f-1c45-43db-833a-1d2809f9cadb.png)

`Win + R`을 눌러서 실행을 켜고 `Shell:startup` 을 타이핑하고 확인 버튼을 누르면 시작 프로그램 폴더가 열린다. 이 곳에 위에서 만든 바로가기 파일을 넣는다.

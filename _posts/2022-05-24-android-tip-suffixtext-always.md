---
layout: post
title: Android. How to always display Suffix Text on EditText
subtitle: Using TextInputLayout & TextInputEditText
categories: Android
tags: [tip, xml]
---

## π₯ν΄κ²°νκ³ μ νλ λ¬Έμ 

---

![design](https://user-images.githubusercontent.com/44221447/170051583-5eb36c81-f268-4b79-bbbf-8ba05242b684.png)

μμ κ°μ λμμΈμ λ°μλ€. νμ¬ μ§ν μ€μΈ νλ‘μ νΈμμλ νκ΅ μ΄λ©μΌλ‘λ§ κ°μνλ μ μ±μ΄λΌμ λ€μ ν­μ λμΌν λ©μΌ λλ©μΈμ νμνλ€.

μ λ κ² μ§μ νλ κ²μ Material λμμΈμ `TextInputLayout`κ³Ό `TextInputEditText`λ‘ κ°λ₯νλ€.

> Material μ»΄ν¬λνΈλ₯Ό μ¬μ©νκΈ° μν΄μλ λ€μμ μ€μ ν΄μΌ νλ€.
>
> λ°μ μ¬μ§μ λλ₯΄λ©΄ μ΄λν  μ μλ€. μ΅μ  λ²μ μ νμΈνμ.
>
> νμ¬(2022.05.25) μ΅μ  λ²μ μ `1.6.0`

[![implements](https://user-images.githubusercontent.com/44221447/170211886-6716ae09-f3ca-4943-b895-56c08bbc3755.png)](https://github.com/material-components/material-components-android/blob/master/docs/getting-started.md#2-maven-library-dependency)

![default_xml](https://user-images.githubusercontent.com/44221447/170088134-dd8e4497-b492-4951-90cd-ffdb02a356a5.png)

`TextInputLayout`μ `android:hint` μμ±μ ννΈλ‘ λ³΄μ¬μ€ νμ€νΈλ₯Ό μ§μ νκ³  `app:hintEnabled` μμ±μ trueλ₯Ό μ£Όλ©΄ λ€μκ³Ό κ°μ΄ ννΈκ° λ³΄μ¬μ§λ€.

![origin](https://user-images.githubusercontent.com/44221447/170208594-810878f2-4b19-4e2c-97bc-ee617bbd1aac.gif)

κ·Έλ°λ° μ°¨μ΄μ μ΄ μλ€λ©΄ `TextInputLayout`μ ν¬μ»€μ¦λ₯Ό λ°μ§ μμ μνμμλ **Suffix Text**κ° λ³΄μ΄μ§ μκ³ , ν¬μ»€μ¦λ₯Ό λ°μΌλ©΄ hintκ° μλ‘ μ¬λΌκ°λ©΄μ **Suffix Text**κ° λ³΄μ΄κ² λλ€.

μ²μμλ μ»€μ€ν λ·°λ₯Ό λ§λλ €κ³  νλλ° κ°λ¨ν ν΄κ²°ν  μ μλ λ°©λ²μ μ°Ύκ² λμ΄ κ³΅μ νλ €κ³  νλ€.

## π₯μ΄λ»κ² κ³ μ³μΌ ν κΉ

---

![layout_inspector](https://user-images.githubusercontent.com/44221447/170061800-66c0a697-82cc-481f-8f3c-4ee19da0c2d6.png)

layout inspectorλ‘ νμΈμ ν΄λ³΄λ ννΈ λΆλΆμ΄ μ²μμ μ μ²΄ κ³΅κ°μ μ°¨μ§νκ³  μλ κ²μ νμΈνλ€.

![focused_layout](https://user-images.githubusercontent.com/44221447/170062812-3c70ae8c-1ee4-4b5a-b9a8-a84962e7839b.png)

ν¬μ»€μ¦λ₯Ό λ°μ μνλ₯Ό νμΈν΄λ³΄λ μ μ΄μ μμλ **Suffix Text**κ° λ±μ₯νλ€. μ΄κ±Έ λ³΄μ hintμ κ°λ €μ Έμ λ³΄μ΄μ§ μμλ κ²μ΄ μλλΌ λ΄λΆμ μΌλ‘ ν¬μ»€μ¦λ₯Ό λ°μμ λλ§ λ³΄μ΄λλ‘ λ§λ€μ΄μ§ κ² κ°λ€.

μ°λ¦¬κ° ν΄κ²°ν΄μΌ νλ λΆλΆμ λ κ°μ§μ΄λ€.

1. ν¬μ»€μ¦λ₯Ό λ°μΌλ©΄ ννΈκ° μλ‘ μ¬λΌκ°λ κ²μ΄ μλλΌ **λ³΄μ΄μ§ μμμΌ νλ€.**
2. **ν¬μ»€μ¦λ₯Ό λ°μ§ μμ μνμμλ** **Suffix Text**κ° λ³΄μ¬μΌ νλ€.

## π₯ν΄κ²°ν΄λ³΄μ

---

### κΈμκ° μλ ₯λλ©΄ ννΈ μμ κΈ°

![solve_1](https://user-images.githubusercontent.com/44221447/170088259-aaf0ff95-07b6-4ccf-990d-c0a459e13d84.png)

μ΄ λΆλΆμ κ°λ¨νλ€. ννΈλ₯Ό `TextInputLayout`κ° μλλΌ `TextInputEditText`μμ νμνλ©΄ λλ€. `android:hint` μμ±μ `TextInputEditText`λ‘ μ?κΈ°κ³ , `TextInputLayout`μμ `app:hintEnabled` μμ±μ μ κ±°νλ©΄ λλ€. (κΈ°λ³Έ κ°μ΄ falseμ΄λ€)

![before](https://user-images.githubusercontent.com/44221447/170207633-b0d86662-409f-4c19-9f87-6c1454621171.gif)

μλ ₯ μ°½μ ν΄λ¦­νλ©΄ ννΈκ° μλ‘ μ¬λΌκ°μ§ μκ³ , κΈμλ₯Ό μλ ₯νλ©΄ ννΈκ° λ³΄μ΄μ§ μλλ€.

### ν­μ **Suffix Text** νμνκΈ°

![updateSuffixTextVisibility](https://user-images.githubusercontent.com/44221447/170070246-07ea1cbf-c147-4161-987e-a6b2347a753d.png)

`TextInputLayout.java` νμΌμ λ€μ Έλ΄€λ€. `updateSuffixTextVisibility()` ν¨μλ₯Ό μ°Ύμλλ° μ΄ ν¨μλ **Suffix Text**κ° μ‘΄μ¬νκ³ , ννΈκ° νμ₯λμ§ μλ μνμΌ λ **Suffix Text**λ₯Ό λ³΄μ΄κ² νλ λμμ νλ κ²μΌλ‘ μκ°λλ€.

ννΈκ° νμ₯λ μνλΌκ³  νλ©΄ ν¬μ»€μ¦λ₯Ό λ°μ§ μμ μνμμ ννΈκ° μλ ₯μ°½ μ μ²΄ λΆλΆμ λ?κ³  μλ μνλ₯Ό μλ―Ένλ κ² κ°λ€.

![description](https://user-images.githubusercontent.com/44221447/170020093-ceb93cf2-93bd-4226-9eb3-25bb742d75d6.png)

λ `TextInputLayout`μ `expandedHintEnabled` λΌλ μμ±μ΄ μλλ° μ΄ μμ±μ μ£Όμμ λ³΄λ©΄ λ€μκ³Ό κ°μ΄ μ νμλ€.

```text
νμ€νΈ νλκ° μ±μμ§μ§ μκ³  ν¬μ»€μ¦λ₯Ό λ°μ§ μμ κ²½μ° ννΈκ° μλ ₯ μμ­μ μ°¨μ§ν΄μΌ νλμ§ μ¬λΆ
```

μ¦ `expandedHintEnabled` μμ±μΌλ‘ κΈμκ° μλ ₯λμ§ μκ³ , ν¬μ»€μ¦λ₯Ό λ°μ§ μμ μνμμλ ννΈκ° μ μ²΄ μλ ₯ μμ­μ μ°¨μ§ν μ§ λ§μ§ μ΅μμ μ€ μ μλ€λ λ§μ΄λ€.

## π₯κ²°λ‘ 

---

![after](https://user-images.githubusercontent.com/44221447/170207931-611b789a-3f73-4c20-ace8-7ca61da26bb4.gif)

![complete_xml](https://user-images.githubusercontent.com/44221447/170086485-2277cab1-c41a-4900-9770-e90d6c3fe53f.png)

1. TextInputLayoutμ `app:expandedHintEnabled="false"` μμ± μ€μ 
2. TextInputEditTextμ `android:hint` μμ± μ€μ 

## π₯μΆκ° μ¬ν­

---

![comment](https://user-images.githubusercontent.com/44221447/170086891-751d4064-06e4-4514-9adc-6906dcd44e5c.png)

μ£Όμμλ μμ κ°μ μ€λͺμ΄ μ νμμλ€.

ννΈλ₯Ό TextInputLayoutμ΄ μλλΌ TextInputLayoutμ λ¬μΌλΌλ λ§μ΄λ€. λμ€μ ννΈλ₯Ό μμ ν  λ λ¬Έμ κ° μκΈΈ μ μλ€λ μ€λͺμ΄λ€.

μ°λ¦¬ νλ‘μ νΈμ μκ΅¬μ¬ν­μμλ ννΈκ° λ³κ²½λμ§ μκΈ° λλ¬Έμ λ¬Έμ κ° λμ§ μλ λΆλΆμ΄λΌκ³  μκ°νκ³ , μ½λλ₯Ό μ’ λ μ΄ν΄λ³΄λ EditTextμ ννΈκ° μ€μ λ κ²½μ° μ΄ ννΈλ₯Ό TextInputLayoutμΌλ‘ μ?κΈ°λ κ³Όμ μ΄ μλ κ²μ νμΈνλ€.

![reason_to_use_hint_at_layout](https://user-images.githubusercontent.com/44221447/170069189-482cdd52-6a78-4bfa-b6f2-ad77870be08f.png)

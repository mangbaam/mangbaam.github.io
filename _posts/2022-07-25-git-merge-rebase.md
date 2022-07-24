---
layout: post
title: Git merge와 rebase. 언제 어떻게 사용해야 할까
subtitle: 
categories: Git
tags: [git,github,rebase,merge]
---

## ⭐

git으로 branch를 만들어서 작업해보았다면 merge와 rebase에 대해서 들어본 적이 있을 것이다. 이 두 깃 명령은 비슷한 역할을 하는데 동작하는 방식을 아주 다르다.

여기서 오는 장단점과 언제 어떤 명령을 사용해야 적절한지에 대해서 포스팅하고자 한다.

본 글은 [Merging vs. Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)을 번역한 글이다.

---

## 개요

---

![overview](https://user-images.githubusercontent.com/44221447/180655576-28a0fc58-b212-4d42-82bf-d7f2c419ab59.png)

위 상황은 새로운 기능을 추가하기 위해 Feature 브랜치를 만들어 작업한 후 Main에 통합해야 하는 상황이다. 이때 사용할 수 있는 방법은 `rebase` 와 `merge` 가 있다.

git `rebase`와 git `merge`는 다른 브랜치의 변경 사항을 현재의 브랜치로 통합하는데 사용되지만 둘은 아주 다른 방식으로 이를 수행한다.

## merge

---

![merge](https://user-images.githubusercontent.com/44221447/180655600-489d571f-8ffc-465b-a74b-2c5ade490f55.png)

```git
git checkout feature
git merge main
```

또는

```git
git merge feature main
```

`merge`를 할 경우 위 그림과 같이 두 branch를 반영한 새로운 “**merge commit**”이 만들어진다.

새로운 commit이 만들어지는 것이기 때문에 기존의 브랜치는 변경되지 않기 때문에 안전하다고 할 수 있다.

반면 [업스트림](#upstream-이란)의 변경 사항을 반영할 때마다 외부 병합 커밋을 포함하게 된다. 그래서 main 브랜치가 아주 빈번히 변한다면 브랜치 히스토리를 번잡하게 만들 수 있다.

git log 옵션을 잘 사용하면 이런 문제점을 완화할 수는 있지만 다른 개발자가 히스토리를 이해하기 어렵게 만들 수 있다는 단점이 있다.

## rebase

---

![rebase](https://user-images.githubusercontent.com/44221447/180655650-d9c15d79-c437-4f4a-bb1b-0bd8de787d49.png)

```git
git checkout feature
git rebase main
```

`rebase`는 전체 Feature 브랜치에 main의 모든 새 커밋들을 브랜치가 뻗어나간 끝에서부터 추가한다. `merge` 커밋과 다르게 `rebase`는 원래 브랜치의 각각의 커밋들을 새로운 커밋으로 만들어 프로젝트 히스토리를 새로 써서 “**재배치**”한다

`rebase`의 가장 큰 장점은 프로젝트 히스토리를 훨씬 깔끔하게 만든다는 점이다. 우선, `merge`의 단점인 불필요한 병합 커밋을 제거하면서 위 그림과 같이 프로젝트 히스토리를 완벽하게 선형적으로 만들 수 있다.

그래서 `git log`, `git bisect`, `gitk`와 같은 명령을 사용해서 프로젝트를 더 쉽게 탐색할 수 있게 된다.

하지만 `rebase`에서는 안전성과 추적성에서 타협을 봐야한다. 그래서 [골든 룰](#rebase-골든-룰)을 따르지 않으면 프로젝트의 히스토리를 새로 쓰는 것은 협동 워크 플로우에서 잠재적으로 위험할 수 있다.

추가적으로 `rebase`는 병합 커밋에서 제공하는 컨텍스트를 잃기 때문에 업스트림의 변경 사항이 Feature에 통합된 시점을 알 수 없다.

## Interactive Rebase

---

![interactive](https://user-images.githubusercontent.com/44221447/180655672-ffdc617b-c7de-4692-ba93-d87ce8a1345b.png)

```git
git checkout feature
git rebase -i main
```

rebase에 -i 옵션을 주면 interactive rebasing을 할 수 있다. 위 명령으로 입력을 하면 다음과 같이 나타나는 텍스트 에디터가 나타난다.

```editor
rebase에 -i 옵션을 주면 interactive rebasing을 할 수 있다. 위 명령으로 입력을 하면 다음과 같이 나타나는 텍스트 에디터가 나타난다.
```

위 내용은 rebase가 수행된 후에 어떻게 만들어질 지를 나타낸다.

```editor
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

여기서 `pick`을 `fixup`으로 바꾸면 commit1과 commit2를 하나로 합칠 수 있게 된다. 만약 commit2가 commit1에서 스타일 변경이나 오타 수정 등 간단한 변경의 커밋일 때 하나로 합쳐서 브랜치 히스토리를 더욱 깔끔하게 볼 수 있다.

수정한 내용을 저장하고 닫으면 Git은 작성된 내용에 따라 rebase를 수행하게 된다. 이런 작업은 `git merge`로는 할 수 없는 작업이다.

## rebase 골든 룰

---

rebase에 대해 이해했다면, 언제 사용해도 될 지 말 지를 알아야 한다.

> rebase의 골든 룰은 절대 public에서 사용하지 말아야 한다는 것이다.

예를 들어 main 브랜치를 Featrue 브랜치에 rebase 한 상황을 생각해보자.

![goldenrule](https://user-images.githubusercontent.com/44221447/180655724-3c309240-8c8f-4ea3-b1f8-3df09b9bb64d.png)

rebase 명령은 main의 모든 커밋을 Feature 끝에 옮길 것이다. 문제는 이 작업이 우리의 레파지토리에서만 동작됐다는 점이다. 다른 모든 개발자들은 원래의 main에서 작업하고 있을 것이다.

rebase의 결과는 새로운 커밋으로 이어지기 때문에 Git은 우리의 main 브랜치 히스토리가 다른 모든 브랜치와 다르다고 생각할 것이다.

달라진 두 main 브랜치를 동기화하기 위한 유일한 방법은 다시 병합하는 것이다. 결과적으로는 두 개의 추가 병합 커밋이 생성되게 된다. 이런 상황이 되는 것은 좋지 않다.

그러니 `git rebase`를 수행하기 전에 항상 *“이 브랜치에서 누군가 작업하고 있는가?”*를 스스로에게 물어야 한다. 만약 *“그렇다”*는 대답이라면 당장 키보드에서 손을 떼고 비파괴적인 방법에 대해 생각해보아야 한다. (예를 들어 `git revert`)

그게 아니면 원하는 만큼 히스토리를 다시 작성하는 것이 안전하다.

## 결론

---

- 불필요한 병합 커밋 없이 깔끔하고 선형적인 히스토리를 선호한다면 다른 브랜치의 변경 사항을 통합할 때 `merge` 대신 `rebase`를 사용하면 된다.
- 프로젝트 전체 기록을 보존하고 public 커밋을 다시 작성해야 하는 위험을 방지하려면 `merge`를 사용하면 된다.

## 부록

---

### Upstream 이란?

`upstream`과 `downstream`은 두 레파지토리 간의 상대적인 개념이다. **A 레파지토리**에서 `pull`로 변경 사항을 **B 레파지토리**로 가져와서 작업 후 `push` 한다면 **A 레파지토리**가 `upstream`, **B 레파지토리**가 `downstream`이다.

그럼 이때 `remote origin`와 `remote upstream`의 차이가 뭐지? 라는 생각이 들 수 있다.

일반적으로는 크게 구분되지 않지만 가장 크게 구분될 때는 바로 `fork`를 해서 작업하는 경우이다. 주로 오픈소스 작업을 할 때 `fork` 해서 작업하는 경우가 많다.

**A 레파지토리**를 `fork` 해서 **B 레파지토리**를 만든 경우 **A 레파지토리**가 `upstream`, **B 레파지토리**가 `origin`이 된다.

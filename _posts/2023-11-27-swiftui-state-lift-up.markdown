---
layout: post
title:  "[SideProject] SwiftUI State 끌어올리기"
date:   2023-11-27 00:08:26 +0900
categories: jekyll update
---

## 목적
사이드 프로젝트를 진행하면서 생긴 문제점과 그 문제를 해결한 과정을 팀원에게 공유하기 위해 이 글을 작성합니다.

## 어떤 문제가 있나요?
앱을 처음 사용하는 사람들을 위한 가이드화면을 그리고 있었습니다. 가이드뷰은 반투명한 검은 배경에 처음 클릭해야하는 버튼을 지시하는 화면으로, ZStack을 이용해 뷰를 쌓아줄 생각을 했습니다. 하지만 TabBar는 ZStack으로 쌓은 가이드뷰가 적용되지 않은 모습을 확인할 수 있었습니다.
<p align="center">
    <img alt="fdas" src="https://velog.velcdn.com/images/mun9769/post/c2fd6338-8856-4f7d-b633-c7c5deaa8e89/image.png" width="30%">
    <em style="display: block;">TabBar는 검은배경이 적용되지 않았다</em>
</p>

## 왜 이런일이 발생했나요?

우리 앱의 **뷰 계층**은 다음과 같습니다.
~~~mermaid
graph TB
a[MainView] --> b[HomeView]
a --> TabBarView
k --> q
~~~

SwiftUI는 부모뷰가 가진 상태값을 자식뷰에게 전달하는 단방향 데이터 흐름 방식을 채택합니다.
자식뷰은 부모뷰의 상태를 공유받고 부모뷰의 상태 변경될 때마다 업데이트를 받을 수 있습니다.

a --> c[GuideView]
c --> LogoView
c --> LTCICardComponenet

b --> d[LogoView]
b --> e[LTCICardComponenet]
b -->|some views| SubmitView



## 시도
guide하는 뷰(이하 검은배경)를 만들기 위해 HomeViewState 값을 하나 더 추가해주었다.
```swift
class HomeViewModel: ObservableObject {
    enum HomeViewState {
        case guide // 추가된 enum 상태
        case before
        case after
    } ...
```
그리고 HomeViewState가 guide라면 `HomeView`에 검은배경을 추가하는 시도를 했다. 

하지만 결과는 아래와 같다. 
TabBar는 검은배경 적용이 안되는 것을 확인할 수 있었다.


## 왜 이런일이 있을까?
우리 앱의 뷰 계층은 아래 그림과 같습니다.

![](https://velog.velcdn.com/images/mun9769/post/3ea42293-9481-4e2a-9374-9d7659a90b49/image.png)

`MainView` 하위 계층에 `HomeView`와 `TabBar`가 존재하고, 둘은 ZStack으로 쌓여있다. 
코드로 보면 아래와 같다.
```swift
struct MainView: View {
    var body: some View {
        ZStack(alignment: .bottom){
            HomeView()
            ... // 생략
            TabBar()
        }
```

`HomeView`의 `background`를 주었기 때문에 `TabBar`의 배경은 적용이 안되는 것이였다.

## 어떻게 해결해야하나?


<img src="https://velog.velcdn.com/images/mun9769/post/55a821d8-c2b8-49c3-963f-dbe5cb34fc20/image.jpg" width="300">

검은 배경을 `HomeView`에 쌓지 말고 `MainView`에 쌓아주면 해결할 수 있다. 

## HomeViewModel에서 MainViewModel로 변경
앱을 처음 실행하는 상황이면 `MainView`는 검은 배경을 보여주어야 한다. 그래서 상태를 저장하는 HomeViewModel을 MainViewModel로 이전시켰다.


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

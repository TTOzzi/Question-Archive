# 야곰닷넷 질문모음 - 2



### Q.

> 움직이는 LaunchScreen 을 구현하고 싶어요.

LaunchScreen 에 이미지 슬라이드 애니메이션을 적용하고 싶은데 방법이 있을까요?

 [질문 바로가기](https://yagom.net/forums/topic/object-c에서-런치스크린-질문입니다/)

### A.

* 우선 LaunchScreen 은 동적인 화면이 아니라 정적인 화면 구성만 가능합니다. [Human Interface Guidelines: Launch Screen](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/launch-screen) 에 따르면 Launch Screen 은 앱의 첫 화면과 비슷한 형태여야 합니다. 그리고 앱이 실행되는 중에 잠깐만 보이는 화면이기에 애니메이션 등을 구현하기는 부적합합니다. 예외적으로 게임 앱의 경우, 게임 엔진을 시작하는 데 많은 시간이 소요되므로 로딩 화면을 표시할 수 있습니다.
* 시작 화면에 애니메이션 등이 보이는 것은 꼼수가 필요합니다. 시작화면이 보여진 후에 앱의 메인화면이 나오게 되는데, 그 화면이 시작화면과 동일한 상태로 시작하게 만들고, 거기서 애니메이션을 실행하여 다음 화면으로 이동하는 등의 처리를 합니다.
* Launch Screen 을 xib 로 만들고, 앱이 시작되고 정적인 Launch Screen 보여준 후 다시 불러와 애니메이션을 적용하고 불러온 뷰를 제거하는 방식도 있습니다. [Animate Your iOS Splash Screen](https://www.viget.com/articles/animated-ios-launch-screen/) 을 참고해보세요. 애니메이션 구현 때문에 글이 긴데 애니메이션 구현 부분을 제외하고 LaunchScreenManager 부분만 봐도 좋을듯합니다.
* 옆으로 슬라이드 되는 효과를 알고 싶다면 [UIScrollView](https://developer.apple.com/documentation/uikit/uiscrollview) 에 대해 공부하면 됩니다. [UIView](https://developer.apple.com/documentation/uikit/uiview) 의 애니메이션 관련 메서드와 UIScrollView 의 [setContentOffset(_:animated:)](https://developer.apple.com/documentation/uikit/uiscrollview/1619400-setcontentoffset) 정도만 공부해도 큰 힌트가 될 것 같습니다. [automatic UIScrollView with paging](https://stackoverflow.com/questions/17168741/automatic-uiscrollview-with-paging) 도 확인해보세요.

### 참고할 만한 비슷한 질문들

* [iOS Launch screen code not running](https://stackoverflow.com/questions/27398723/ios-launch-screen-code-not-running)
* [how to add animation to launch screen in iOS 9.3 using Objective c](https://stackoverflow.com/questions/37112950/how-to-add-animation-to-launch-screen-in-ios-9-3-using-objective-c)
* [Difference between launch image and splash screen](https://stackoverflow.com/questions/12140464/difference-between-launxch-image-and-splash-screen)
* [Animated Splash Screen](https://developer.apple.com/forums/thread/110295)
* [Splash screen in iOS games](https://stackoverflow.com/questions/29047522/splash-screen-in-ios-games)

----

### Q.

> override 할 때 super 의 메소드를 꼭 호출해야 하나요?

viewController 에서

```swift
override func viewDidLoad() {
  super.viewDidLoad()
}
```

super.viewDidLoad() 를 빼도 에러가 안 나던데 꼭 호출해야 하나요?

[질문 바로가기](https://yagom.net/forums/topic/override-할-때-super-꼭-호출해야-하나요/)

### A.

* super 는 부모 클래스를 의미하는데요. 질문 내용은 오버라이딩 할 때 부모의 작업을 실행할지 말지 선택하는 것이라 할 수 있겠죠. 보통 [viewDidLoad()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621495-viewdidload) 같이 뷰 라이프 사이클은 [템플릿 패턴](https://en.wikipedia.org/wiki/Template_method_pattern)으로 구현되어 있는데 뷰 컨트롤러의 뷰가 메모리에 로드될 때, OS 에 의해 호출되는 것이기 때문에 super 의 메소드를 호출하는 게 좋습니다.
* super 의 메소드를 호출하는 것이 선택인 경우도 있고 필수인 경우도 있습니다. 꼭 필요한지 아닌지는 문서에 보면 대부분 나와 있습니다. [AppKit viewDidLoad() 공식문서](https://developer.apple.com/documentation/appkit/nsviewcontroller/1434476-viewdidload) 처럼 discussion 부분에 설명이 있을 겁니다.
* override 할 때 super 를 호출하지 않아야 하는 경우도 있습니다. 대표적인 예시로 [loadView()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview) 가 있습니다. 
* super 메소드를 호출하는 것을 자꾸 깜빡하신다면 코딩 스타일을 교정해주는 [SwiftLint](https://realm.github.io/SwiftLint/overridden_super_call.html) 라이브러리를 이용해보셔도 좋습니다.
* 상속과 오버라이딩에 대해 더 알고 싶다면 [Swift: Inheritance](https://docs.swift.org/swift-book/LanguageGuide/Inheritance.html) 도 읽어보면 좋을것 같아요!

### 참고할 만한 비슷한 질문들

* [When to use super when overriding ios methods](https://stackoverflow.com/questions/38689059/when-to-use-super-when-overriding-ios-methods)
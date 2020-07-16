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
----

### Q.

> Designated initializer 와 Convenience initializer 의 차이가 무엇인가요?

Apple 문서에 보면 Convenience init 은 '같은 클래스에서 다른 이니셜라이저를 호출해야 한다' 라고 하는데 잘 이해가 안 됩니다. Convenience init 은 어떤 상황에 사용되고 Designated init 과의 차이는 무엇인가요?

[질문 바로가기](https://yagom.net/forums/topic/swift-초기화-이니셜라이져/)

### A.

* Designated init 은 클래스의 모든 프로퍼티가 초기화될 수 있도록 해줘야 하고, Convenience init 은 쉽게 생각하면 보조 이니셜라이저라고 할 수 있는데, Designated init 의 파라미터 일부를 기본값으로 설정해서 Convenience init 안에서 Designated init 을 호출해서 쓸 수 있는 거예요.

  ```swift
  class Beverage {
    var name: String
    var capacity: Int
    // 지정 이니셜라이저 - 모든 인스턴스의 저장 프로퍼티 값 초기화(할당)
    init(name: String, capacity: Int) {
      self.name = name
      self.capacity = capacity
    }
    // 편의 이니셜라이저 - 지정 이니셜라이저를 통해 인스턴스 초기화
    convenience init(name: String) {
      self.init(name: name, capacity: 330)
    }
    // 편의 이니셜라이저 - 다른 편의 이니셜라이저를 통해 인스턴스 초기화
    convenience init() {
      self.init(name: "Fanta")
    }
  }
  ```

  Convenience init 에는 속성 중 인스턴스 생성 시점에 지정해줘야 할 것만 파라미터로 넣어놓고, 그 안에서 기본값이 들어간 다른 이니셜라이저를 호출한다고 생각하시면 됩니다! 예시코드처럼 Convenience init 에는 Designated init 뿐만 아니라 다른 Convenience init 을 호출해도 됩니다.

  ![initializer chaining](https://docs.swift.org/swift-book/_images/initializerDelegation01_2x.png)

  하지만 위 그림처럼 같은 클래스 내부에서 init 체인이 연결되는 끝 지점은 항상 Designated init 이여야 합니다.

* 더 자세한 내용은 [Swift: Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html) 을 읽어보세요!

### 참고할 만한 비슷한 질문들

* [What the difference between designated and convenience init in this code below](https://stackoverflow.com/questions/29563147/what-the-difference-between-designated-and-convenience-init-in-this-code-below)
* [What is the difference between convenience init vs init in swift, explicit examples better](https://stackoverflow.com/questions/40093484/what-is-the-difference-between-convenience-init-vs-init-in-swift-explicit-examp)

----

### Q.

> loadView() 와 viewDidLoad() 의 차이가 무엇인가요?

loadView() 와 viewDidLoad() 의 차이점은 무엇이고 각각 어떤 경우에 사용하나요? 

그리고 코드만으로 UI 를 작성할 때는 view 를 그리는 코드를 loadView 에 작성해야 하나요?

[질문 바로가기](https://yagom.net/forums/topic/loadview와-viewdidload-차이에-대한-질문입니다/)

### A.

* 좋은 글이 있어 공유 드립니다. 뷰가 그려질 때와 뷰가 로드될 때를 나눠서 생각해보시면 좋을 것 같습니다. [[iOS] View 가 Load 될 경우](https://mrgamza.tistory.com/279)
* [loadView()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview) 메소드는 뷰 컨트롤러가 자신의 뷰, 그러니까 흔히 self.view 처럼 접근하는 그 뷰 컨트롤러의 메인 뷰를 로드할 때 호출되는 메소드입니다. 즉, 그 메인 뷰를 생성하려고 호출하는 메소드죠. 그래서 이 메소드 안에서 새로운 뷰를 만들어서 뷰 컨트롤러의 메인 뷰로 설정해줘도 됩니다. 뷰 컨트롤러의 기본 뷰를 커스텀 뷰로 사용하고자 할 때 유용합니다. 스토리보드를 쓰면 어차피 스토리보드에 있는 뷰를 가져와 쓸 테니 굳이 필요하지 않다고 볼 수 있겠네요.
* [viewDidLoad()](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621495-viewdidload) 는 이 뷰가 모두 생성되고 메모리에 생성된 후에 호출되는 메소드입니다. 즉, `뷰 컨트롤러의 메인 뷰가 생성되었으니 이제 이 위에 하고 싶은 걸 해라` 라는 뜻으로 보면 되겠습니다. 뷰가 생성된 이후에 해야 할 동작을 viewDidLoad() 내부에 작성합니다.
* [viewDidLoad() vs. loadView()(Swift3)](https://medium.com/yay-its-erica/viewdidload-vs-loadview-swift3-47f4ad195602), [viewDidLoad() Vs. loadView()](https://medium.com/swlh/viewdidload-vs-loadview-ddec6ac9bdd7) 이 글들도 참고해보면 좋고, 애플의 공식문서를 읽어봐도 도움이 될 것 같아요.

### 참고할 만한 비슷한 질문들

* [iPhone SDK: what is the difference between loadView and viewDidLoad?](https://stackoverflow.com/questions/573958/iphone-sdk-what-is-the-difference-between-loadview-and-viewdidload)
* [what is the difference between loadView and viewDidLoad?](https://stackoverflow.com/questions/3423785/what-is-the-difference-between-loadview-and-viewdidload)

----

### Q.

> iOS 13 미만에서도 애플 로그인을 구현해야 하나요? 구현해야 한다면 어떻게 구현하나요?

[App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/#sign-in-with-apple) 의 **Sign in with Apple** 에서는 다른 소셜 로그인 서비스를 사용하는 앱은 애플 로그인도 제공해야 한다고 합니다. 하지만 애플이 제공하는 [AuthenticationServices](https://developer.apple.com/documentation/authenticationservices) 프레임워크의 애플 로그인 기능은 iOS 13 이상만 지원합니다. iOS 13 미만에서도 애플 로그인을 구현해야 하나요?

[질문 바로가기](https://developer.apple.com/forums/thread/122755?answerId=417003022#417003022)

### A.

* iOS 가 아닌 다른 플랫폼이나 iOS 12 미만의 기기에서도 웹을 통해 애플 로그인을 구현할 수 있습니다. [available](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html) 속성을 활용해 iOS 버전으로 분기처리를 하여 13 이상의 기기에서 실행되었을 때는 AuthenticationServices 으로 로그인을 하고, 13 미만의 기기에서는 [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) 로 애플 로그인이 구현된 웹페이지를 띄워줍니다. 자세한 구현은 [Sign in with Apple JS](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js) 와 [Incorporating Sign in with Apple into Other Platforms](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_js/incorporating_sign_in_with_apple_into_other_platforms) 를 참고하세요.

### 참고할 만한 비슷한 질문들

* [How can I implement Sign-In-With-Apple in a webview?](https://stackoverflow.com/questions/60461480/how-can-i-implement-sign-in-with-apple-in-a-webview)
* [Will "Sign in With Apple" allow apps to be backward compatible with iOS 12 and lower?](https://stackoverflow.com/questions/57928420/will-sign-in-with-apple-allow-apps-to-be-backward-compatible-with-ios-12-and-l)
* [Sign In With Apple and Older Devices](https://developer.apple.com/forums/thread/649267)

----

### Q.

> 기기에서 애플 로그인을 할 때 사용자 정보(이름, 이메일)가 안 받아져요.

애플 로그인을 [ASAuthorizationAppleIDRequest](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidrequest?language=objc) 의 [requestedScopes](https://developer.apple.com/documentation/authenticationservices/asauthorizationopenidrequest/3153062-requestedscopes) 에 [fullName](https://developer.apple.com/documentation/authenticationservices/asauthorization/scope/3153028-fullname) 과 [email](https://developer.apple.com/documentation/authenticationservices/asauthorization/scope/3153027-email) 을 설정해서 이름과 이메일을 응답으로 받아오도록 구현하였습니다. 시뮬레이터에서 테스트할 때는 애플 로그인을 할 때마다 사용자 정보를 받아오지만, 실제 기기에서 테스트하면 첫 요청에서는 사용자 정보를 잘 받아오는데 그 후부턴 [user identifier](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidcredential/3153037-user) 만 받아오고 fullName 과 email 에는 nil 이 반환됩니다. 어떻게 해결해야 할까요?

[질문 바로가기](https://developer.apple.com/forums/thread/121496)

### A.

* 사용자가 해당 앱에 처음 로그인할 때만 사용자 정보를 [ASAuthorizationAppleIDCredential](https://developer.apple.com/documentation/authenticationservices/asauthorizationappleidcredential) 에 담아 보냅니다. 이후 동일한 계정으로 앱에 로그인하면 사용자 정보가 공유되지 않고 사용자 식별자만 ASAuthorizationAppleIDCredential 으로 반환합니다. 서버에 계정이 성공적으로 생성 되었는지 확인할 수 있을 때까지 처음에 받은 사용자 정보를 캐시 해두는 것을 권장합니다. [Authenticationg Users with Sign in with Apple](https://developer.apple.com/documentation/sign_in_with_apple/sign_in_with_apple_rest_api/authenticating_users_with_sign_in_with_apple) 의 **Send Information to App Servers and Verify Tokens** 부분을 참고하세요.
* 실제 기기에서 사용자 정보를 받아오는 동작을 여러 번 테스트해야 하는 상황이라면 앱의 Bundle Identifier 를 변경하거나, [Apple ID 계정 관리](https://appleid.apple.com/#!&page=signin) 의 **Apple ID를 사용하는 앱 및 웹 사이트** 에서 앱을 제거하여 첫 번째 로그인처럼 만들어주는 방법도 있습니다.


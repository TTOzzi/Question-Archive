# iOS 질문 모음 - 8

### Q.

> UIWindow 가 무엇인가요?

iOS 앱에서 UIWindow 가 무엇인가요??

[질문 바로가기](https://www.quora.com/What-is-UIWindow-in-Swift-Does-a-single-view-controller-contain-multiple-windows)

### A.

* 간단하게 말하자면, [UIWindow](https://developer.apple.com/documentation/uikit/uiwindow) 는 앱의 뷰 계층 구조에서 최상단에 고정되어 위치하며, 앱의 화면 콘텐츠에 대한 컨테이너 역할을 합니다. 앱의 뷰, 뷰 컨트롤러와 함께 작동하여 화면에 표시되는 뷰 계층 구조에 터치 이벤트를 전달하고, 화면 방향 변경과 같은 변경 사항을 관리합니다.

  <img width="40%" src="https://docs-assets.developer.apple.com/published/7f180a9ffc/c05af6a2-c616-482b-8f65-98013d40bb05.png"/><img width="60%" src="https://user-images.githubusercontent.com/50410213/92306401-d5a44900-efc9-11ea-97b8-c87a2dceb247.png"/>

  UIWindow 는 화면에 보이는 콘텐츠를 제공하지 않습니다. 화면에 보이는 모든 콘텐츠는 앱의 스토리보드에서 구성하는 [rootViewController](https://developer.apple.com/documentation/uikit/uiwindow/1621581-rootviewcontroller) 에서 제공합니다. UIWindow 의 역할은 UIKit 에서 이벤트를 수신하고, 관련된 이벤트를 루트 뷰 컨트롤러 및 관련 뷰에 전달하는 것입니다.

* 대부분의 iOS 앱에서는 하나의 윈도우만 만들고 사용합니다. 앱이 외부 디스플레이 사용을 지원한다면, 해당 외부 디스플레이에 콘텐츠를 표시하는 추가 윈도우를 만들 수 있습니다. 외부 디스플레이 지원 외에 앱 사용 중 전화 수신과 같은 다른 상황들은 일반적으로 시스템에 의해 윈도우가 생성됩니다.

* 윈도우는 [application(_:didFinishLaunchingWithOptions:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622921-application), [scene(_:willConnectTo:options)](https://developer.apple.com/documentation/uikit/uiscenedelegate/3197914-scene) 와 같은 앱의 수명주기 초기에 생성됩니다. 스토리보드를 사용하면 자동으로 [window](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate/3198093-window) 프로퍼티가 생성되어 keyWindow 로 설정되고 화면에 표시됩니다.

* 만약 스토리보드를 사용하지 않는다면, 스토리보드가 자동으로 해주었던 동작을 앱의 수명주기 초기에 다음과 같이 직접 코드로 작성해주어야 합니다.

  **iOS 13 이상**

  ```swift
  class SceneDelegate: UIResponder, UIWindowSceneDelegate {
      var window: UIWindow?
  
      func scene(_ scene: UIScene,
                 willConnectTo session: UISceneSession,
                 options connectionOptions: UIScene.ConnectionOptions) {
          guard let windowScene = (scene as? UIWindowScene) else { return }
  
          let window = UIWindow(windowScene: windowScene)
          window.rootViewController = ViewController()
          self.window = window
          window.makeKeyAndVisible()
      }
  }
  ```

  **iOS 13 미만**

  ```swift
  @UIApplicationMain
  class AppDelegate: UIResponder, UIApplicationDelegate {
      var window: UIWindow?
  
      func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
          window = UIWindow()
          window?.rootViewController = ViewController()
          window?.makeKeyAndVisible()
      
          return true
      }
  }
  ```

* 앱 수명주기 초기에 윈도우를 생성할 때를 제외하면 윈도우에 접근할 일이 거의 없지만, 몇몇 작업에서는 윈도우를 활용하기도 합니다. 현재 기기에 표시되는 윈도우를 기준으로 하는 좌푯값이 필요할 때나([Converting Coordinates in the View Hierarchy](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingViews/CreatingViews.html#//apple_ref/doc/uid/TP40009503-CH5-SW40)), 윈도우의 상태 변경에 따라 특정한 작업을 해주어야 할 때([Monitoring Window Changes](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingWindows/NaN#//apple_ref/doc/uid/TP40009503-CH4-SW13)) 와 같은 상황에 관련 메소드를 활용하여 윈도우와 관련된 작업을 수행합니다. 

### 참고할 만한 비슷한 질문, 자료

* [Windows and Screens](https://developer.apple.com/documentation/uikit/windows_and_screens)
* [Multiple Display Programming Guide for iOS: Understanding Windows and Screens](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/WindowAndScreenGuide/WindowScreenRolesinApp/WindowScreenRolesinApp.html#//apple_ref/doc/uid/TP40012555-CH4-SW8)
* [View Programming Guide for iOS: Windows](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingWindows/CreatingWindows.html#//apple_ref/doc/uid/TP40009503-CH4-SW13)
* [What is the purpose of UIWindow?](https://stackoverflow.com/questions/18282311/what-is-the-purpose-of-uiwindow)
* [How to Properly Remove Main.Storyboard(for iOS 13+)](https://ioscoachfrank.com/remove-main-storyboard.html)
* [Cocoa Application Competencies for iOS: Responder object](https://developer.apple.com/library/archive/documentation/General/Conceptual/Devpedia-CocoaApp/Responder.html)

-----

### Q.

> ScrollView 에서 contentOffset 과 contentInset 이 무엇인가요?

ScrollView 에서 contentOffset 과 contentInset 이 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/33286574/what-exactly-are-contentoffset-contentinset-of-scrollview)

### A.

* [contentOffset](https://developer.apple.com/documentation/uikit/uiscrollview/1619404-contentoffset) 은 전체 콘텐츠 뷰 영역을 기준으로 현재 화면에 보이는 스크롤 뷰 영역의 위치 좌표를 나타낸 값입니다.

  <img width="65%" src="https://i.stack.imgur.com/S8ZxB.jpg"/><img width="35%" src="https://user-images.githubusercontent.com/50410213/92323759-5a4ca100-f076-11ea-989d-5f92651f5275.gif"/>

  왼쪽 그림에서 파란 테두리가 스크롤 뷰의 영역, 검정 점선 테두리가 스크롤뷰의 콘텐츠 영역입니다. 사용자가 스크롤을 하면 화면에 보이는 콘텐츠의 영역이 변경되므로 contentOffset 의 값도 변경됩니다. contentOffset 의 값을 코드로 변경하면(슬라이더) 스크롤 뷰가 스크롤 되는 것과 같은 동작을 합니다.

* [contentInset](https://developer.apple.com/documentation/uikit/uiscrollview/1619406-contentinset) 은 스크롤 뷰 가장자리와 콘텐츠 영역의 가장자리 사이에 스크롤 가능한 공간(패딩)을 추가합니다.

  <img width="65%" src="https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/UIScrollView_pg/Art/contentSize_contentInset.jpg"/><img width="35%" src="https://user-images.githubusercontent.com/50410213/92323638-46546f80-f075-11ea-8e6c-c2e49089c746.gif"/>

  상단과 하단의 inset 값을 조절한 만큼 콘텐츠가 더 스크롤 되는 것(하늘색 영역)을 확인할 수 있습니다.

* 만약 글만으로는 잘 이해가 되지 않는다면, contentInset 과 contentOffset 값의 변화에 따른 UIScrollView 의 변화를 한눈에 보기 쉽게 만든 [샘플 앱](https://github.com/TTOzzi/InsetVsOffset)이 있으니 참고하세요!

### 참고할 만한 비슷한 질문, 자료

* [Scroll View Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/UIScrollView_pg/CreatingBasicScrollViews/CreatingBasicScrollViews.html)
* [What does contentOffset do in a UIScrollView?](https://stackoverflow.com/questions/3339798/what-does-contentoffset-do-in-a-uiscrollview)
* [What exactly are contentOffset, ContentInset and ContentSize of a UIScrollView?](https://levelup.gitconnected.com/what-exactly-are-contentoffset-contentinset-and-contentsize-of-a-uiscrollview-960207c75b88)
* [Understanding the contentOffset and contentInset properties of the UIScrollView class](https://fizzbuzzer.com/understanding-the-contentoffset-and-contentinset-properties-of-the-uiscrollview-class/)

-----


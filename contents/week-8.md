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


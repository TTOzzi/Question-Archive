# iOS 질문 모음 - 4

### Q.

> class 와 final class 의 차이가 무엇인가요?

그냥 선언한 클래스와 final 을 붙여 선언한 클래스의 차이가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/46049783/what-is-the-difference-between-final-class-and-class)

### A.

* 클래스를 선언할 때 final 을 사용해 클래스의 상속을 막을 수 있습니다. final 로 선언된 클래스는 다른 클래스에서 상속할 수 없습니다.

  ```swift
  final class A {
  }
  
  // Inheritance from a final class 'A' 에러 발생
  class B: A {
  }
  ```

  final 로 선언된 클래스를 상속받으려 하면 컴파일 에러가 발생합니다.

* 클래스뿐만 아니라 메소드, 프로퍼티, 서브스크립트에도 final 을 명시하여 재정의를 막을 수 있습니다.

  ```swift
  class A {
      final var name: String {
          return "A"
      }
      
      final subscript() -> Any? {
          get { nil }
          set(newValue) { }
      }
      
      final func hello() {
          print("hello A")
      }
  }
  
  class B: A {
      // Property overrides a 'final' property 에러 발생
      override var name: String {
          return "B"
      }
      
      // Subscript overrides a 'final' subscript 에러 발생
      override subscript() -> Any? {
          get { "override!" }
          set (newValue) { }
      }
      
      // Instance method overrides a 'final' instance method
      override func hello() {
          print("hello B")
      }
  }
  ```

  클래스의 상속은 허용하되 부분적으로 재정의를 막고 싶을 때 주로 사용합니다. [Swift: Inheritance - Preventing Overrides](https://docs.swift.org/swift-book/LanguageGuide/Inheritance.html#ID202) 도 읽어보세요!

* 만약 구현한 클래스를 상속할 일이 없다면 final 로 선언해주는 게 좋습니다. 상속하지 않을 클래스를 final 로 선언해주면 컴파일러에게 선언된 함수를 찾지 않고 직접 호출해야 한다고 알려주어 함수 호출에 의한 오버헤드를 줄여 성능을 향상시킵니다. 자세한 내용은 [Increasing Performance by Reducing Dynamic Dispatch](https://developer.apple.com/swift/blog/?id=27) 를 참고하세요.

### 참고할 만한 비슷한 질문들

* [Swift. Make all classes final?](https://stackoverflow.com/questions/47041883/swift-make-all-classes-final)
* [Does marking a Swift class final also make all contained vars, lets and functions gain Static Dispatch benefits automatically?](https://stackoverflow.com/questions/55308577/does-marking-a-swift-class-final-also-make-all-contained-vars-lets-and-function)

-----

### Q.

> UI 업데이트는 왜 메인 스레드에서만 해야 하나요?

애플은 모든 UI 관련 작업이 메인 스레드에서 수행되어야 한다고 합니다.

백그라운드 스레드에서 UI 업데이트 작업을 하면 왜 오류가 발생하나요?

[질문 바로가기](https://www.quora.com/Why-must-the-UI-always-be-updated-on-Main-Thread)

### A.

* 앱을 실행하면 [Cocoa Touch](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Cocoa.html) 에서 [UIApplication](https://developer.apple.com/documentation/uikit/uiapplication) 의 인스턴스가 메인 스레드에서 설정됩니다.

  ![responderchain](https://user-images.githubusercontent.com/50410213/88822217-8adf2680-d1fe-11ea-9c5e-1f9a74b3de5d.png)

  앱의 UI event 는 일반적으로 UIApplication -> [UIWindow](https://developer.apple.com/documentation/uikit/uiwindow) -> [UIViewController](https://developer.apple.com/documentation/uikit/uiviewcontroller) -> [UIView](https://developer.apple.com/documentation/uikit/uiview) -> subviews(UILabel, [UIButton](https://developer.apple.com/documentation/uikit/uibutton) 등) 와 같이 chain 으로 연결되는데, 이 [responder chain](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events) 을 따라 UIApplication 으로 전달됩니다. 이러한 event chain 이 메인 스레드에서 동작하므로 responder chain 에 포함된 모든 UI 관련 동작들은 메인 스레드에서 수행되어야 합니다. 

  간단하게 요약하자면 UI 와 관련된 모든 이벤트 처리를 메인 스레드에서 하므로 UI 업데이트는 반드시 메인 스레드에서 해야 합니다. [UIKit 공식문서](https://developer.apple.com/documentation/uikit)에서도 **Important** 로 메인 스레드에서의 UI 업데이트를 강조하고 있습니다.

* 또 다른 이유는 아이폰의 그래픽 렌더링 방식입니다. 아이폰의 [그래픽스 파이프라인](https://ko.wikipedia.org/wiki/그래픽스_파이프라인)은 동기식으로 동작합니다. 다음 자료는 WWDC 2014: Advanced Graphics and Animations 의 프레젠테이션 중 일부입니다.

  ![CoreAnimationPipeline1](https://user-images.githubusercontent.com/50410213/88821867-23c17200-d1fe-11ea-8576-f8dd32c11081.png)

  레이블의 텍스트를 업데이트하는 상황을 예로 들어보겠습니다. UIKit 은 화면에 텍스트를 렌더링합니다. 먼저 [벡터](https://ko.wikipedia.org/wiki/벡터_그래픽스)인 글꼴을 [래스터화](https://ko.wikipedia.org/wiki/래스터화)하고 텍스트를 픽셀로 변환하고 렌더 서버로 전송합니다. 텍스트를 자르거나, 투명도를 적용할 때와 같이 해당 텍스트가 혼합된 레이어의 일부분일 경우 그래픽 렌더러는 보여줘야 할 레이블의 픽셀을 계산합니다. 그런 다음 이 픽셀들을 화면으로 보내 표시하게 됩니다. 

  ![CoreAnimationPipeline2](https://user-images.githubusercontent.com/50410213/88821884-2623cc00-d1fe-11ea-9068-aa9b2078fd6b.png)

  Core Animation Pipeline 에서는 1/60 초만에 준비작업을 끝내고, 렌더링 서버로 데이터를 전송한 후 1/60 초만에 렌더링을 완료합니다. 이런 식으로 앱이 멈추지 않고 화면 표시가 가능합니다.

  하지만 백그라운드 스레드에서 UI 업데이트를 하게 된다면, 많은 백그라운드 스레드가 렌더링 정보를 변환하고 GPU 에 렌더링 요청을 보낼 것이고, 렌더링은 시스템 리소스가 많이 드는 작업이므로 GPU 가 이를 처리할 수 없어 심각한 문제가 발생합니다.

### 참고할 만한 비슷한 질문들

* [Why must UIKit operations be performed on the main thread?](https://stackoverflow.com/questions/18467114/why-must-uikit-operations-be-performed-on-the-main-thread)
* [iOS) 왜 main.sync 를 하면 안될까](https://zeddios.tistory.com/519)
* [iOS: Why the UI need to be updated on Main Thread](https://medium.com/@duwei199714/ios-why-the-ui-need-to-be-updated-on-main-thread-fd0fef070e7f)

----

### Q.

> 클래스 이름의 NS 접두사의 의미가 무엇인가요?

[Cocoa / Cocoa Touch](https://developer.apple.com/library/archive/documentation/General/Conceptual/DevPedia-CocoaCore/Cocoa.html) 의 많은 클래스의 이름에 NS 접두사가 붙어있는 것을 볼 수 있었는데요. 어떤 의미인가요?

[질문 바로가기](https://stackoverflow.com/questions/473758/what-does-the-ns-prefix-mean)

### A.

* 먼저 NS 접두사를 이해하려면 [Objective-C](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html) 의 특징을 알아야합니다. Objective-C 는 동적이고 객체 지향적으로 C 언어를 확장한 언어입니다. C 언어를 확장했기에 C++ 의 [namespace](https://docs.microsoft.com/ko-kr/cpp/cpp/namespaces-cpp?view=vs-2019) 같은 기능이 없습니다. 그래서 전역 namespace 에서 이름 충돌을 피하기위해 접두사가 필요합니다. 자기 자신만 사용할 목적으로 작성한 코드에서는 이 충돌을 신경쓰지 않아도 되지만, 다른 사람이 사용할 수 있도록 프레임워크 또는 라이브러리를 작성하는 경우 고유 접두사를 붙여야합니다. [Coding Guidelines for Cocoa](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingBasics.html#//apple_ref/doc/uid/20001281-1002226-BBCJECED) 과 [Google Objective-C Style Guide](https://google.github.io/styleguide/objcguide.html#prefixes) 에 접두사에 대한 설명이 있습니다. [CocoaDev: ChooseYourOwnPrefix](http://cocoadev.github.io/ChooseYourOwnPrefix/) 를 보면 Cocoa community 의 수많은 개발자가 자신이 선택한 고유한 접두사를 기록해놓은 것을 볼 수 있습니다.

  Cocoa 프레임워크는 [NeXTSTEP](https://en.wikipedia.org/wiki/NeXTSTEP) 운영체제용 앱을 만들기 위한 코드로부터 시작되었습니다. 1996년 애플이 다시 NeXT 를 인수했을 때 기존 클래스 이름을 포함하여 NeXTSTEP 의 라이브러리인 Foundation 과 AppKit 이 OS X 에 통합되었습니다(애플의 Cocoa 프레임워크에서는 현재까지도 같은 이름을 사용 중입니다). 이때 NeXTSTEP 의 개발자들은 앞서 설명한 충돌을 피하기 위한 접두사를 이름 앞에 추가하였는데, 이것이 바로 **NS** 입니다.

* NS 접두사가 붙어있는 타입들은 Swift 가 나오기 전 Objective-C 에서 사용되었던 타입들입니다. [Working with Foundation Types](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/working_with_foundation_types) 를 보면 알 수 있듯이 현재는 대부분의 NS 타입들이 Swift 의 타입들로 Bridge 되어 있어 Swift 의 타입들을 많이 사용하지만, NS 타입만의 고유한 특성을 활용하기 위해 NS 타입을 사용하기도 합니다. 

### 참고할 만한 비슷한 질문들

* [Swift - which types to use? NSString or String](https://stackoverflow.com/questions/24038629/swift-which-types-to-use-nsstring-or-string/24038680)
* [Error 와 NSError 의 차이가 무엇인가요?](https://yagom.net/forums/topic/기초적인-swift-문법-질문드립니다-ㅠㅠ/)
* [What is difference between NSDictionary vs Dictionary in Swift?](https://stackoverflow.com/questions/25554259/what-is-difference-between-nsdictionary-vs-dictionary-in-swift)
* [What are these NS prefixed types?](https://forums.swift.org/t/what-are-these-ns-prefixed-types/19045)

----

### Q.

> guard 와 if 의 차이점이 무엇인가요?

guard 를 사용해 조건문을 작성하면 if 를 사용할 때와 어떤 차이가 있나요?

[질문 바로가기](https://stackoverflow.com/questions/30791488/swifts-guard-keyword)

### A.

* 먼저 if 를 사용한 코드입니다.

  ```swift
  func foo(x: Int?) {
      if let x = x where x > 0 {
          // 조건을 만족했을 때 실행할 코드를 이곳에 작성합니다.
      } else {
          // 조건을 만족하지 못했을 때 실행할 코드를 이곳에 작성합니다.
      }
      // 이곳에 작성하는 코드는 위 조건과 관계없이 실행되게 됩니다.
  }
  ```

  if 안에 조건을 만족했을 때 실행할 코드들을 모두 작성하게 됩니다. if 블록을 벗어나면 옵셔널 바인딩 된 x 는 사용할 수 없습니다. 

  guard 와는 달리 if 밖에서 조건과 관계없이 실행되는 코드를 작성할 수 있습니다.

* guard 를 사용한 코드입니다.

  ```swift
  func foo(x: Int?) {
      guard let x = x where x > 0 else {
          // 조건을 만족하지 못했을 때 실행할 코드를 이곳에 작성합니다.
          return
      }
  
      // 조건을 만족했을 때 실행할 코드를 이곳에 작성합니다.
  }
  ```

  조건을 만족하지 않으면 else 문이 실행되어 else 문 안의 코드를 실행하고 함수를 종료합니다. guard 를 사용하면 조건을 만족하지 않을 땐 항상 continue, break, return 등을 사용하여 guard 가 선언된 스코프를 빠져나가야 합니다. 

  조건을 만족하면 현재 함수 내 guard 문이 실행된 이후의 모든 범위에서 옵셔널 바인딩 한 x 를 사용할 수 있습니다.

* 몇가지 팁을 드리자면, if 안에 스코프를 빠져나가는 코드가 있는 경우에는 guard 로 바꾸는 것이 좋습니다. 예외처리를 할 때도 guard 를 사용하면 좀 더 명시적인 예외처리가 가능합니다. if 가 여러개 중첩되어 코드 줄바꿈의 깊이가 깊어졌을 때도 guard 를 사용하면 깊이를 줄이고 가독성을 향상시킬 수 있습니다.

  ```swift
  func isZero(x: String) -> Bool? {
      if let x = Int(x) {
          if x == 0 {
              return true
          } else {
              return false
          }
      } else {
          return nil
      }
  }
  ```

  ```swift
  func isZero(x: String) -> Bool? {
      guard let x = Int(x) else { return nil }
      guard x == 0 else { return false }
      return true
  }
  ```

  String 을 인자로 받아 Int 로 변환하고, 그 값이 0 인지 아닌지 확인하는 함수입니다. 인자로 받은 문자열이 숫자가 아니면 nil 을 반환하고 0이면 true, 0이 아니면 false 를 반환합니다. if 를 사용해서 구현한 함수보다 guard 를 사용한 함수가 코드의 깊이도 얕고, 어떤 때에 어떤 값을 반환하는지 좀 더 명확하게 알 수 있는 것을 확인할 수 있습니다.

* if 보다 guard 를 사용했을 때 더 좋은 상황이 많지만, if 를 사용해야만 하는 상황도 있으므로 위의 특징들을 참고해서 본인이 작성한 코드의 상황에 맞게 사용하시면 됩니다.

### 참고할 만한 비슷한 질문들

* [Swift: guard let vs if let](https://stackoverflow.com/questions/32256834/swift-guard-let-vs-if-let)
* [What is basic difference between guard statement and if...else statement?](https://stackoverflow.com/questions/34703089/what-is-basic-difference-between-guard-statement-and-if-else-statement/34703255)
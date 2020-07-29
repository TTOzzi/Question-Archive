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

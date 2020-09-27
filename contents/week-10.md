# iOS 질문모음 - 10

### Q.

> 뷰와 레이어의 차이가 무엇인가요?

화면을 그리는 코드를 작성할 때 UIView 를 사용할 때가 있고 CALayer 를 사용할 때가 있는데, UIView 와 CALayer 의 차이가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/7826306/what-are-the-differences-between-a-uiview-and-a-calayer)

### A.

* 뷰와 레이어의 차이를 이해하려면 먼저 [CoreAnimation](https://developer.apple.com/documentation/quartzcore) 과 [UIKit](https://developer.apple.com/documentation/uikit) 의 차이부터 알아야 하는데요.

  ![UIKit](https://user-images.githubusercontent.com/50410213/94342513-68c22300-004c-11eb-9148-2d03b619178a.png)

  위 사진은 iOS 에서 화면을 그릴 때 사용하는 프레임워크의 계층구조입니다(AppKit 은 macOS). CoreAnimation, CoreGraphics 는 많은 기능을 제공하지만, 간단한 기능을 구현하는 데에도 많은 코드를 작성해야 한다는 번거로움이 있습니다. 일반적으로 앱을 구현할 때 복잡한 UI, 애니메이션이 필요하지 않기 때문에 자주 사용하는 인터페이스들이 구현된 더 높은 수준의 프레임워크를 제공해주는데 이것이 UIKit 입니다. 그리고 이 UIKit 에서 제공해주는 클래스가 UIView 이고, CALayer 는 한 단계 낮은 수준인 CoreAnimation 의 클래스입니다.

* [CALayer](https://developer.apple.com/documentation/quartzcore/calayer) 는 시각적 컨텐츠를 관리하고 컨텐츠를 화면에 표시하는 데 사용되는 위치, 크기, 변형과 같은 기하학적 정보를 가집니다. 레이어의 속성을 수정하여 콘텐츠에 애니메이션을 적용할 수도 있습니다. 레이어는 단순한 그래픽 표현만을 수행하며 별도의 스레드에서 GPU 를 사용해 그려집니다. 뷰에 비해 더 세부적인 설정이 가능하며 성능상 효율이 더 좋습니다.

* [UIView](https://developer.apple.com/documentation/uikit/uiview) 는 UI 의 기본 구성 요소입니다. 직사각형 영역에서 컨텐츠를 렌더링하고 컨텐츠에서 발생하는 모든 이벤트를 처리합니다.

  ![UIView](https://user-images.githubusercontent.com/50410213/94342273-fef54980-004a-11eb-95e3-14a563fdb0f9.png)

  iOS 에서 모든 뷰는 레이어를 포함하고 있습니다. 뷰는 레이어를 포함하고 있고, 그 레이어를 그려주는 인터페이스와 이벤트 처리를 포함한 별도의 클래스입니다(공식문서에서는 thin wrapper 라고 표현합니다). 뷰는 [UIResponder](https://developer.apple.com/documentation/uikit/uiresponder) 를 상속받아 터치, 모션, 프레스와 같은 이벤트를 처리합니다. 뷰와 관련된 작업은 레이어와 달리 이벤트 처리를 하므로 메인 스레드에서 발생하며 CPU 를 사용합니다. 뷰에 의해 레이어가 생성되면 뷰가 자기 자신을 레이어의 [delegate](https://developer.apple.com/documentation/quartzcore/calayer/1410984-delegate) 로 자동 할당합니다. 뷰를 통해 컨텐츠를 그리는 코드를 작성할 수 있지만, 이는 뷰가 처리하는 것이 아닌 뷰가 포함하는 레이어에 위임해서 컨텐츠를 그리도록 하는 것 입니다.

* 대부분의 경우 뷰를 활용해 화면을 그리겠지만, 레이어를 사용하는 것이 훨씬 좋거나 레이어를 사용해야만 하는 경우가 있습니다. 막대차트, 파이차트와 같은 이벤트 처리가 필요 없으면서 세부적인 시각적 표현이 필요한 경우나, 복잡한 애니메이션 설정이 필요한 경우 레이어를 사용해 구현합니다. CoreAnimation 은 [CATextLayer](https://developer.apple.com/documentation/quartzcore/catextlayer), [CAShapeLayer](https://developer.apple.com/documentation/quartzcore/cashapelayer) 등의 여러 가지 레이어 클래스를 제공하며, 이들을 활용해 정말 다양한 그래픽 표현이 가능합니다. 다양한 레이어 클래스를 활용한 예시는 [CALayer Tutorial for iOS: Gettinng Started](https://www.raywenderlich.com/10317653-calayer-tutorial-for-ios-getting-started) 를 참고하세요.

### 참고할 만한 비슷한 질문, 자료

* [UIView vs CALayer](https://medium.com/@fassko/uiview-vs-calayer-b55d932ff1f5)
* [iOS Brownbag: Views vs. Layers](https://dzone.com/articles/ios-brownbag-views-vs-layers)
* [A Beginner's Guide to CALayer](https://www.appcoda.com/calayer-introduction/)
* [Core Animation Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514-CH1-SW1)

-----

### Q.

> Swift 에서 Optional 은 어떻게 구현되어 있나요?

Optional 이 어떻게 구현되어 있는지 궁금해요.

[질문 바로가기](https://stackoverflow.com/questions/24548475/how-are-optional-values-implemented-in-swift)

### A.

* 해당 답변은 [Swift](https://github.com/apple/swift) 의 [Optional.swift](https://github.com/apple/swift/blob/main/stdlib/public/core/Optional.swift#L205) 코드를 설명하는 글입니다. 더 자세한 정보를 원한다면 해당 코드를 직접 보시는 것을 추천합니다.

* Swift 에서 [Optional](https://developer.apple.com/documentation/swift/optional) 은 제네릭을 활용한 열거형으로 구현되어있습니다.

  ```swift
  public enum Optional<Wrapped>: ExpressibleByNilLiteral {
      case none
      case some(Wrapped)
  }
  ```

  `none` 과 `some` 총 두 가지의 케이스를 가지며 `none` 은 값이 없는 경우를, `some` 은 값이 있는 경우를 나타냅니다. 옵셔널 타입을 선언할 때, `Optional<Int>` 와 같은 형태로 사용할 수도 있지만, 보통 `Int?` 와 같이 타입 뒤에 물음표 접미사를 붙여 타입이 옵셔널임을 나타냅니다.

* 옵셔널 타입은 총 두 개의 생성자가 구현되어있습니다.

  ```swift
  public init(_ some: Wrapped) { self = .some(some) }
  ```

  할당되는 값이 있을 때 사용되는 생성자입니다. 제네릭 파라미터로 전달되는 `Wrapped` 타입의 값을 받아 인스턴스에 `some` 케이스를 할당합니다.

  ```swift
  let number: Int? = Optional<Int>(3) // 이런 방법으로도 생성할 수 있지만,
  let number: Int? = 3 // 이렇게만 값을 할당해줘도 타입 추론을 통해 내부적으로 init(_ some: Wrapped) 생성자가 호출됩니다.
  ```

  다음 생성자는 할당되는 값이 nil 일때 사용되는 생성자입니다.

  ```swift
  // var number: Int? = nil 처럼 초기값이 nil 로 할당될 때 사용됩니다.
  public init(nilLiteral: ()) {
      self = .none
  }
  ```

  이 생성자는 [ExpressibleByNilLiteral](https://developer.apple.com/documentation/swift/expressiblebynilliteral) 프로토콜의 요구사항입니다. 인스턴스에 `none` 케이스를 할당합니다.

  ```swift
  let number: Int? = Optional<Int>(nilLiteral: ()) // 이번에도 이렇게 생성할 수 있지만,
  let number: Int? = nil // 이렇게 nil 을 할당해줘도 내부적으로 init(nilLiteral: ()) 가 호출됩니다.
  ```

  값이 있는 경우, 없는 경우 둘 다 직접 생성자를 호출할 일은 없습니다. 생성자를 생략하고 값을 할당해주어야 합니다.

* 옵셔널 인스턴스를 강제언래핑 할 때 붙이는 `!` 접미사는 다음과 같이 구현되어 있습니다.

  ```swift
  public var unsafelyUnwrapped: Wrapped {
      get {
          if let x = self {
              return x
          }
          _debugPreconditionFailure("unsafelyUnwrapped of nil optional")
      }
  }
  ```

  `if let` 으로 `self` 를 바인딩하여 값이 있다면 값을 반환하고, 없다면 런타임에러를 발생시킵니다.

  ```swift
  var number: Int? = 3
  number.unsafelyUnwrapped // 3
  number = nil
  number.unsafelyUnwrapped // 런타임 에러 발생
  number! // number.unsafelyUnwrapped 와 같습니다.
  ```

  직접 [unsafelyUnwrapped](https://developer.apple.com/documentation/swift/optional/1641793-unsafelyunwrapped) 를 호출할 수도 있지만, 보통은 접미사 `!` 를 붙여 `number!` 처럼 사용합니다.

* 옵셔널 값에 기본값을 설정해주는 `??` 연산자의 구현입니다.

  ```swift
  public func ?? <T>(optional: T?, defaultValue: @autoclosure () throws -> T)
          rethrows -> T {
      switch optional {
      case .some(let value):
          return value
      case .none:
          return try defaultValue()
      }
  }
  ```

  `Optional` 타입이 열거형으로 구현되어있어 `switch-case` 문으로 값이 있을 때와 없을 때를 나눠 구현된 것을 확인할 수 있습니다. 연산자의 왼쪽에 위치할 인자에 값이 존재한다면 그 값을, 존재하지 않는다면 `defaultValue` 클로저를 실행해 기본값을 반환합니다. 다음과 같이 사용됩니다.

  ```swift
  var number: Int? = 3
  print(number ?? 0) // 3 출력
  number = nil
  print(number ?? 0) // number 의 값이 없으므로 기본값 0 출력
  ```

* 옵셔널 값을 클로저를 통해 변환하기 위한 [map](https://developer.apple.com/documentation/swift/optional/1539476-map), [flatMap](https://developer.apple.com/documentation/swift/optional/1540500-flatmap) 이 구현되어 있습니다.

  ```swift
  public func map<U>(
      _ transform: (Wrapped) throws -> U // 클로저의 반환값이 옵셔널이 아님
  ) rethrows -> U? {
      switch self {
      case .some(let y):
          return .some(try transform(y))
      case .none:
          return .none
      }
  }
  
  public func flatMap<U>(
      _ transform: (Wrapped) throws -> U? // 클로저의 반환값이 옵셔널
  ) rethrows -> U? {
      switch self {
      case .some(let y):
          return try transform(y)
      case .none:
          return .none
      }
  }
  ```

  인자로 받은 클로저로 옵셔널 인스턴스의 값을 변환합니다. 두 메소드의 차이는 클로저 반환값의 옵셔널 여부입니다. `map` 은 클로저의 결과값으로 옵셔널이 아닌 값을 반환해야 하고, `flatMap` 은 옵셔널 값을 반환해도 됩니다.

  ```swift
  func map(_ string : String?) -> Int? {
      return string.map { Int($0) } // Value of optional type 'Int?' must be unwrapped to a value of type 'Int' 에러 발생
  }
  
  func flatMap(_ string: String?) -> Int? {
      return string.flatMap { Int($0) }
  }
  ```

  `map`, `flatMap` 을 사용하여 `String?` 을 `Int?` 로 변환하는 함수입니다. `map` 을 사용한 함수에선 클로저의 반환값이 `Int?` 가 아닌 `Int` 여야 한다는 컴파일 에러가 발생합니다.

* 디버깅을 위한 프로토콜도 채택하고 있습니다.

  ```swift
  extension Optional: CustomDebugStringConvertible {
      public var debugDescription: String {
          switch self {
          case .some(let value):
              var result = "Optional("
              debugPrint(value, terminator: "", to: &result)
              result += ")"
              return result
          case .none:
              return "nil"
          }
      }
  }
  
  extension Optional: CustomReflectable {
      public var customMirror: Mirror {
          switch self {
          case .some(let value):
              return Mirror(
                  self,
                  children: [ "some": value ],
                  displayStyle: .optional)
          case .none:
              return Mirror(self, children: [:], displayStyle: .optional)
          }
      }
  }
  ```

  디버깅 할 때 텍스트 출력을 위한 문자열을 정의하는 [CustomDebugStringConvertible](https://developer.apple.com/documentation/swift/customdebugstringconvertible) 과 인스턴스의 구조를 표현해주는 [Mirror](https://developer.apple.com/documentation/swift/mirror)  를 정의하는 [CustomReflectable](https://developer.apple.com/documentation/swift/customreflectable) 을 채택하고 있습니다.

  ```swift
  let nilNumber: Int? = nil
  let number: Int? = 3
  
  print("CustomDebugStringConvertible\n")
  debugPrint(nilNumber)
  debugPrint(number)
  print("\nCustomReflectable\n")
  dump(nilNumber)
  dump(number)
  ```

  두 프로토콜을 사용하는 대표적인 메소드인 [debugPrint](https://developer.apple.com/documentation/swift/1539920-debugprint) 와 [dump](https://developer.apple.com/documentation/swift/1539127-dump) 를 호출해보았습니다.

  <img width="321" alt="스크린샷 2020-09-27 오후 10 28 53" src="https://user-images.githubusercontent.com/50410213/94366128-e2234980-0110-11eb-9765-631528298a0c.png">

  내부 구현에 맞게 출력되고 있는 것을 확인할 수 있습니다.

* 옵셔널의 제네릭 파라미터로 전달되는 `Wrapped` 타입이 [Hashable](https://developer.apple.com/documentation/swift/hashable) 이나 [Equatable](https://developer.apple.com/documentation/swift/equatable) 을 만족할 때도 `Equatable` 의 `==` 연산자, `Hashable` 의 `hash(into:)` 메소드가 제대로 작동하도록 기본구현이 되어있습니다. [where 절을 활용한 extension](https://docs.swift.org/swift-book/LanguageGuide/Generics.html#ID553) 으로 메소드들의 기본구현을 해주고 있습니다.

  ```swift
  extension Optional: Equatable where Wrapped: Equatable {
      public static func ==(lhs: Wrapped?, rhs: Wrapped?) -> Bool {
          switch (lhs, rhs) {
          case let (l?, r?):
              return l == r
          case (nil, nil):
              return true
          default:
              return false
          }
      }
  }
  ```

  두 개의 인자가 모두 값이 존재할 때만 `Equatable` 을 채택한 `Wrapped` 타입의 `==` 연산자로 비교한 결과를 반환하고, `nil` 을 포함하는 경우는 따로 `case` 로 나눠 처리하는 것을 확인할 수 있습니다.

  ```swift
  extension Optional: Hashable where Wrapped: Hashable {
      public func hash(into hasher: inout Hasher) {
          switch self {
          case .none:
              hasher.combine(0 as UInt8)
          case .some(let wrapped):
              hasher.combine(1 as UInt8)
              hasher.combine(wrapped)
          }
      }
  }
  ```

  값의 유무를 `0`, `1` 로 구분하여 `nil` 일땐 0 을 해시하여 모든 `nil` 이 같은 해시값을 가지도록 하고, 값이 있을 땐 기존 해시값에 `1` 을 해시한 값을 추가하여 옵셔널로 선언된 인스턴스와 옵셔널이 아닌 인스턴스의 해시값에 차이를 두고 있습니다.

  ```swift
  let stringNil: String? = nil
  let intNil: Int? = nil
  
  print(stringNil.hashValue == intNil.hashValue) // true
  
  let number: Int = 3
  let optionalNumber: Int? = 3
  
  print(number.hashValue == optionalNumber.hashValue) // false
  ```

  `String?` 으로 선언된 `nil` 과 `Int?` 로 선언된 `nil` 의 해시값이 같고, `Int` 로 선언된 `3` 과 `Int?` 로 선언된 `3` 의 해시값이 다른 것을 확인할 수 있습니다.

* 비교할 타입이 `Equatable` 을 채택하지 않더라도 `nil` 과의 비교는 가능합니다.

  ```swift
  public struct _OptionalNilComparisonType: ExpressibleByNilLiteral {
      public init(nilLiteral: ()) {
      }
  }
  ```

  `nil` 과의 비교 연산자 코드를 작성하기 위해 `nil` 을 나타내줄 `_OptionalNilComparisonType` 구조체가 선언되어있습니다. `ExpressibleByNilLiteral` 를 채택하고 생성자만 가지고 있습니다.

  ```swift
  public static func ==(lhs: Wrapped?, rhs: _OptionalNilComparisonType) -> Bool {
      switch lhs {
      case .some:
          return false
      case .none:
          return true
      }
  }
  ```

  연산자의 좌측에 있는 인자가 `nil` 인지 아닌지에 따라 결과를 결정합니다. `==` 외에도 `!=`, `~=`  이 구현되어있습니다.

  ```swift
  struct A { }
  
  var a: A? = A()
  print(a == nil) // false
  a = nil
  print(a == nil) // true
  ```

  `A` 구조체가 `Equatable` 을 채택하지 않았음에도 nil 과의 비교에서는 `==` 연산자를 사용할 수 있는 것을 확인할 수 있습니다.

### 참고할 만한 비슷한 질문, 자료

* [Swift! Optionals?](https://medium.com/ios-os-x-development/swift-optionals-78dafaa53f3)
* [Map and flatMap difference in optional unwrapping in Swift 1.2](https://stackoverflow.com/questions/29556656/map-and-flatmap-difference-in-optional-unwrapping-in-swift-1-2)

# iOS 질문 모음 - 7

### Q.

> == 와 === 의 차이가 무엇인가요?

`==` 와 `===` 은 어떤 차이가 있나요?

[질문 바로가기](https://stackoverflow.com/questions/24002819/difference-between-and)

### A.

* `==` 연산자는 인스턴스의 **값**이 같은지 확인하고, `===` 연산자는 **참조하고 있는 인스턴스**가 같은지 확인합니다.

* 참조 타입인 클래스는 여러 상수와 변수가 동일한 인스턴스를 참조할 수 있습니다. 클래스의 인스턴스는 힙(Heap) 영역에 할당되고 그 인스턴스를 참조하는 상수, 변수는 스택(Stack) 영역에 할당됩니다. `==` 연산자는 인스턴스가 서로 같은지를 의미하며, 두 비교 대상의 참조가 같을 수도 있지만, 꼭 같은 참조를 가져야만 하지는 않습니다. `===` 연산자는 참조 타입만 비교할 수 있지만, `==` 연산자는 값 타입도 비교할 수 있습니다. String, Int 와 같은 대부분의 Swift 표준 라이브러리 기본 타입들은 [Equatable](https://developer.apple.com/documentation/swift/equatable) 프로토콜을 채택하고 있으나, 사용자 정의 클래스, 구조체는 Equatable 프로토콜을 채택하고 서로 같음을 판단하는 기준을 `static func == (lhs:, rhs:) -> Bool` 메소드를 구현함으로써 제공해주어야 합니다.

  ```swift
  class Person: Equatable {
      let id: Int
      let name: String
      
      init(id: Int, name: String) {
          self.id = id
          self.name = name
      }
      
      static func == (lhs: Person, rhs: Person) -> Bool {
          return lhs.id == rhs.id
      }
  }
  ```

  id 와 name 을 프로퍼티로 가지는 Person 클래스입니다. Equatable 프로토콜을 채택하여 id 프로퍼티의 값의 일치 여부로 서로 같음을 판단하도록 구현해주었습니다.

  ```swift
  let person1 = Person(id: 5, name: "Bob")
  let person2 = Person(id: 5, name: "TTOzzi")
  
  person1 == person2 // return true
  ```

  name 프로퍼티의 값이 다르더라도 id 값만으로 비교하므로 true 를 반환하는 것을 확인할 수 있습니다.

* `==` 와 달리 `===` 는 전역 함수로 이미 구현되어있습니다. 

  ```swift
  @inlinable
  public func === (lhs: AnyObject?, rhs: AnyObject?) -> Bool {
      switch (lhs, rhs) {
      case let (l?, r?):
          return ObjectIdentifier(l) == ObjectIdentifier(r)
      case (nil, nil):
          return true
      default:
          return false
      }
  }
  ```

   `===` 연산자의 내부 구현입니다. [AnyObject](https://developer.apple.com/documentation/swift/anyobject) 를 인자로 받도록 구현하여 비교 대상을 클래스 타입으로 제한합니다. 각 비교 대상으로 클래스 인스턴스의 고유한 ID 값인 [ObjectIdentifier](https://developer.apple.com/documentation/swift/objectidentifier) 를 생성하여 그 둘을 비교하도록 구현되어있습니다.

  ```swift
  let person1 = Person(id: 5, name: "Bob")
  let person2 = Person(id: 5, name: "Bob")
  let person3 = person1
  
  person1 === person2 // return false
  person1 === person3 // return true
  ```

  person1 과 person2 의 비교에서 프로퍼티가 완전히 일치함에도 false 를 반환하는 것을 확인할 수 있습니다. 그에 반해 person3 는 person1 과 같은 인스턴스를 참조하므로 true 를 반환합니다.

* Swift 에서 값의 일치 여부를 판단하는 연산자의 구현에 대해 좀 더 알고 싶다면 [Equatable.swift](https://github.com/apple/swift/blob/master/stdlib/public/core/Equatable.swift) 를 참고해보세요.

### 참고할 만한 비슷한 질문, 자료

* [Difference between using ObjectIdentifier() and '===' Operator](https://stackoverflow.com/questions/39587027/difference-between-using-objectidentifier-and-operator)

-----

### Q.

> mutating 이 무엇인가요?

메소드 앞의 mutating 키워드의 의미가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/51128666/what-does-the-swift-mutating-keyword-mean)

### A.

* 기본적으로 값 타입인 구조체와 열거형의 프로퍼티는 인스턴스 메소드 내에서 수정할 수 없습니다. 하지만 특정한 메소드 내에서 프로퍼티의 값을 변경해야 할 때가 있다면 메소드 앞에 mutating 키워드를 붙여 내부 프로퍼티를 수정하는 메소드를 만들 수 있습니다.

  ```swift
  struct Point {
      var x = 0.0
      var y = 0.0
    
      func moveBy(x deltaX: Double, y deltaY: Double) {
          // 'self' is immutable 에러 발생
          x += deltaX
          y += deltaY
      }
  }
  ```

  x, y 좌표를 가지는 Point 구조체를 만들어 주었습니다. moveBy(x:y:) 메소드에서 Point 구조체 내부의 x, y 프로퍼티의 값을 수정하게 되면 에러가 발생합니다.

  ```swift
  struct Point {
      var x = 0.0
      var y = 0.0
      
      mutating func moveBy(x deltaX: Double, y deltaY: Double) {
          x += deltaX
          y += deltaY
      }
  }
  ```

  이럴 때, mutating 키워드를 메소드 앞에 붙여 메소드가 구조체의 값을 변경할 것임을 명시해야 합니다. 

  ```swift
  struct Point {
      var x = 0.0
      var y = 0.0
      
      mutating func moveBy(x deltaX: Double, y deltaY: Double) {
          self = Point(x: x + deltaX, y: y + deltaY)
      }
  }
  ```

  self 에 새로운 인스턴스를 만들어 할당할 수도 있습니다. mutating 메소드로 내부 프로퍼티의 값을 변경하면 프로퍼티만 변경되는 것이 아니라, **프로퍼티가 변경된 완전히 새로운 인스턴스를 만들어 기존 인스턴스를 대체**하므로 각각의 프로퍼티를 수정하는 것과 self 에 새로운 인스턴스를 만들어 할당하는 것은 완벽하게 똑같이 작동합니다.

  ```swift
  let somePoint = Point()
  // Cannot use mutating member on immutable value 에러 발생
  somePoint.moveBy(x: 1, y: 2)
  ```

  같은 맥락으로 mutating 메소드를 호출한다는 것은 기존의 인스턴스를 새로운 인스턴스로 대체한다는 것을 의미하므로 구조체가 상수로 선언되어 있다면 mutating 메소드를 호출할 수 없습니다.

* 구조체와 같이 열거형도 똑같이 동작하며 다음과 같이 활용할 수 있습니다.

  ```swift
  enum TriStateSwitch {
      case off, low, high
      
      mutating func next() {
          switch self {
          case .off:
              self = .low
          case .low:
              self = .high
          case .high:
              self = .off
          }
      }
  }
  
  var ovenLight = TriStateSwitch.low
  ovenLight.next() // high
  ovenLight.next() // off
  ```

### 참고할 만한 비슷한 질문, 자료

* [Swift and mutating struct](https://stackoverflow.com/questions/24035648/swift-and-mutating-struct)
* [Mutating function inside class](https://stackoverflow.com/questions/38422781/mutating-function-inside-class)
* [Swift: Methods](https://docs.swift.org/swift-book/LanguageGuide/Methods.html)

-----

### Q.

> lazy 가 무엇인가요?

Swift 에서 lazy 키워드의 의미가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/44153817/what-is-lazy-meaning-in-swift)

### A.

* lazy 키워드로 변수를 선언하여 해당 변수가 처음 요청될 때 계산되는 변수를 만들 수 있습니다.

  ```swift
  class Properties {
      var property: Int = Properties.someExpensiveFunction("property")
      lazy var lazyProperty: Int = Properties.someExpensiveFunction("lazyProperty")
      
      static func someExpensiveFunction(_ name: String) -> Int {
          // 복잡한 로직...
          print("calculating \(name)...")
          return 0
      }
  }
  ```

  복잡한 계산 로직을 갖고, 많은 리소스를 소모하는 메소드 someExpensiveFunction 의 반환 값을 프로퍼티로 가지는 Properties 클래스입니다.

  ```swift
  let properties = Properties()
  ```

  <img width="272" alt="스크린샷 2020-08-31 오전 12 41 55" src="https://user-images.githubusercontent.com/50410213/91663273-d1d47a80-eb22-11ea-8b7b-c8f0d3de602b.png">

  생성자로 인스턴스를 생성하면, property 만 생성되는 것을 확인할 수 있습니다.

  ```swift
  let properties = Properties()
  print(properties.lazyProperty)
  ```

  <img width="311" alt="스크린샷 2020-08-31 오전 12 43 36" src="https://user-images.githubusercontent.com/50410213/91663313-02b4af80-eb23-11ea-9fb4-336c2303ff9e.png">

  lazy 로 선언한 프로퍼티는 프로퍼티에 접근할 때 생성됩니다. 이처럼 프로퍼티의 초깃값을 계산하는 데에 많은 리소스를 소모하는 경우, 인스턴스를 생성할 때 계산하지 않고 프로퍼티가 필요할 때 계산을 수행하도록 구현하기 위해 사용합니다.

* 추가로 인스턴스의 초기화가 완료될 때까지 값을 알 수 없는 외부 요인에 따라 프로퍼티의 초깃값이 달라질 때도 유용하게 사용합니다.

* lazy 프로퍼티는 처음 접근할 때만 계산되고, 그 후엔 값을 저장합니다. 하지만 프로퍼티가 초기화되지 않은 상태에서 여러 스레드에서 동시에 프로퍼티에 접근하는 경우, 프로퍼티를 초기화하기 위한 계산이 여러 번 이루어질 수 있습니다.

  ```swift
  let properties = Properties()
  for _ in 1...10 {
      DispatchQueue.global().async {
          print(properties.lazyProperty)
      }
  }
  ```

  여러 개의 백그라운드 스레드에서 동시에 lazyProperty 에 접근하는 코드입니다.

  <img width="326" alt="스크린샷 2020-08-31 오전 1 02 33" src="https://user-images.githubusercontent.com/50410213/91663730-aa32e180-eb25-11ea-821d-f46b884f7061.png">

  lazyProperty 의 초기화가 여러번 이루어진 것을 확인할 수 있습니다. 이런 경우, 중간에 값의 변경이 이루어져 예상치 못한 결과를 초래할 수 있으므로 주의해야 합니다.

* lazy 프로퍼티와 비슷하게 동작하는 [LazySequence](https://developer.apple.com/documentation/swift/lazysequence#overview) 도 존재합니다. 배열, 문자열 등 [Sequence](https://developer.apple.com/documentation/swift/sequence) 프로토콜을 채택한 타입이라면 기본적으로 [lazy](https://developer.apple.com/documentation/swift/sequence/1641562-lazy) 에 구현되어 있습니다. LazySequence 는 map, filter 와 같은 고차함수를 사용했을 때, 시퀀스 전체를 계산하지 않고, 접근한 인덱스의 값을 가져오는 데 필요한 계산만 수행합니다.

  ```swift
  func double(_ number: Int) -> Int {
      print("calculating \(number)...")
      return number * 2
  }
  ```

  인자로 받은 정수에 2를 곱해 반환하는 메소드입니다.

  ```swift
  let doubled = [1, 2, 3].map { double($0) }
  ```

  <img width="192" alt="스크린샷 2020-08-31 오전 1 34 22" src="https://user-images.githubusercontent.com/50410213/91664422-2d563680-eb2a-11ea-8448-e23bf8f46eb3.png">

  일반적인 배열에 map 을 사용하면 계산된 배열을 doubled 에 저장할 때 배열의 모든 값을 계산합니다.

  ```swift
  let lazyDoubled = [1, 2, 3].lazy.map { double($0) }
  print(lazyDoubled[1])
  ```

  <img width="196" alt="스크린샷 2020-08-31 오전 1 38 12" src="https://user-images.githubusercontent.com/50410213/91664505-a2c20700-eb2a-11ea-810e-02e5541f5bdc.png">

  배열에 구현되어있는 LazySequence 에 map 을 사용하였습니다. 접근한 값을 가져오는 데 필요한 계산만 수행하는 것을 확인할 수 있습니다. LazySequence 는 lazy 프로퍼티와는 달리 값을 저장하지 않고 매번 계산합니다. 이를 인지하고, 상황에 맞게 적절하게 사용하면 불필요한 계산을 피하고 성능을 향상시킬수 있을 것입니다. 좀 더 자세한 정보와 활용법은 [LazySequence.swift](https://github.com/apple/swift/blob/master/stdlib/public/core/LazySequence.swift) 를 참고하세요.

### 참고할 만한 비슷한 질문, 자료

* [Swift: Properties](https://docs.swift.org/swift-book/LanguageGuide/Properties.html#)
* [Lazy property's memory management](https://stackoverflow.com/questions/41478975/lazy-property-s-memory-management)
* [What are lazy variables?](https://www.hackingwithswift.com/example-code/language/what-are-lazy-variables)
* [Why and when to use lazy with Array in Swift?](https://stackoverflow.com/questions/51917054/why-and-when-to-use-lazy-with-array-in-swift)


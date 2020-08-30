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


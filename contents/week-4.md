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


# iOS 질문모음 - 9

### Q.

> @objc 의 의미가 무엇인가요?

@objc 키워드의 의미가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/30795117/when-to-use-objc-in-swift)

### A.

* @objc 는 Swift 로 작성된 코드에서 Objective-C 의 API 와의 호환을 위해 사용합니다. target-action 패턴(Selector)을 활용하는 [Timer](https://developer.apple.com/documentation/foundation/timer), [NotificationCenter](https://developer.apple.com/documentation/foundation/notificationcenter) 나, keyPath 를 활용하는 [KVO](https://developer.apple.com/documentation/swift/cocoa_design_patterns/using_key-value_observing_in_swift), [KVC](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/KeyValueCoding/index.html) 와 같은 Objc 기반 API 는 Objc 런타임을 통해 제공됩니다. Swift 4 이전엔 컴파일러가 Objc 런타임에 노출할 프로퍼티와 메소드를 자동으로 추론해줬지만, 추론하는 시기의 불분명함, 같은 이름(오버로딩)으로 인한 의도치 않은 충돌, 바이너리 크기 증가와 성능 저하 등의 이유([SE-0160: Limiting @objc inferece](https://github.com/apple/swift-evolution/blob/master/proposals/0160-objc-inference.md))로 Swift 4 부터는 @objc 를 명시하여 Objc 런타임에 노출할 프로퍼티와 메소드를 컴파일러에게 알려줘야 합니다.

* target-action 패턴의 #selector

  ```swift
  class ViewController: UIViewController {
      @IBOutlet weak var button: UIButton!
      
      override func viewDidLoad() {
          super.viewDidLoad()
          button.addTarget(self, action: #selector(tappedButton(_:)), for: .touchUpInside)
      }
      
      @objc func tappedButton(_ sender: UIButton?) {
          print("tapped button")
      }
  }
  ```

  #selector 에 전달하는 함수가 @objc 로 선언되어있지 않으면 컴파일 에러가 발생합니다.

* #keyPath

  ```swift
  class Person: NSObject {
      @objc var name: String
      @objc var friends: [Person] = []
      @objc var bestFriend: Person? = nil
      
      init(name: String) {
          self.name = name
      }
  }
   
  let gabrielle = Person(name: "Gabrielle")
  let jim = Person(name: "Jim")
  let yuanyuan = Person(name: "Yuanyuan")
  gabrielle.friends = [jim, yuanyuan]
  gabrielle.bestFriend = yuanyuan
   
  #keyPath(Person.name)
  // "name"
  gabrielle.value(forKey: #keyPath(Person.name))
  // "Gabrielle"
  #keyPath(Person.bestFriend.name)
  // "bestFriend.name"
  gabrielle.value(forKeyPath: #keyPath(Person.bestFriend.name))
  // "Yuanyuan"
  #keyPath(Person.friends.name)
  // "friends.name"
  gabrielle.value(forKeyPath: #keyPath(Person.friends.name))
  // ["Yuanyuan", "Jim"]
  ```

  #keyPath 로 접근하는 프로퍼티를 @objc 로 선언해주었습니다.

  ```swift
  @objcMembers class Person: NSObject {
      var name: String
      var friends: [Person] = []
      var bestFriend: Person? = nil
      
      init(name: String) {
          self.name = name
      }
  }
  ```

  클래스 전체를 Objc 런타임에 노출되어야 할 때 클래스를 @objcMembers 로 선언하여 내부의 모든 프로퍼티와 메소드를 @objc 로 선언한 것과 같은 효과를 줄 수도 있습니다. 하지만 클래스 전체를 포함하다 보니 Objc 런타임에 필요하지 않은 프로퍼티나 메소드까지 포함할 수 있어 바이너리 크기가 증가하고 성능이 저하될 수 있습니다. Objc 런타임의 기능을 많이 사용하는 라이브러리 등을 제외한 대부분 코드에서는 @objc 를 사용하는 것을 권장합니다.

* @IBAction, @IBOutlet 등의 Interface Builder 와 관련된 접두사들이나 @NSManaged, @GKInspectable 등의 접두사들은 내부적으로 @objc 로 구현되어 있어 따로 @objc 를 명시해주지 않아도 관련 기능들을 사용할 수 있습니다.

  ```swift
  class ViewController: UIViewController {
      @IBOutlet weak var button: UIButton!
      
      override func viewDidLoad() {
          super.viewDidLoad()
          button.addTarget(self, action: #selector(tappedButton(_:)), for: .touchUpInside)
      }
      
      @IBAction func tappedButton(_ sender: UIButton?) {
          print("tapped button")
      }
  }
  ```

  @IBAction 으로 선언된 메소드도 #selector 에 전달할 수 있는것을 확인할 수 있습니다.

### 참고할 만한 비슷한 질문, 자료

* [Using Objective-C Runtime Features in Swift](https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift)
* [Swift: Attributes](https://docs.swift.org/swift-book/ReferenceManual/Attributes.html#)
* [How can I deal with @objc inference deprecation with #selector() in Swift 4?](https://stackoverflow.com/questions/44390378/how-can-i-deal-with-objc-inference-deprecation-with-selector-in-swift-4)
* [@selector() in Swift?](https://stackoverflow.com/questions/24007650/selector-in-swift)

-----


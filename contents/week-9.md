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

### Q.

> 자기 자신의 프로퍼티나 메소드에 접근할 때 항상 self 를 붙여줘야 하나요?

클래스, 구조체, 열거형의 내부에서 자기 자신의 프로퍼티나 메소드에 접근할 때 self 를 명시해줄 수도, 생략할 수도 있는데요. self 를 계속 명시해주는 것과 self 를 꼭 필요할 때만 쓰는 것 중 어떤 방법이 더 나은가요?

[질문 바로가기](https://stackoverflow.com/questions/24215578/when-should-i-access-properties-with-self-in-swift)

### A.

* self 를 항상 명시한다면, 생략하는 것보다 명확하게 자기 자신의 프로퍼티나 메소드를 사용한다는 의도를 표현할 수 있습니다. 반대로, self 를 필요할 때만 명시하는 경우 코드가 간결해지고, 컴파일러에서 클로저 내부에서의 self 접근에 self 의 명시를 강제하기 때문에, 클로저 캡처로 인한 순환 참조가 발생할 가능성이 있는 부분이 눈에 띈다는 장점이 있습니다.

* 다음은 깃헙에서 star 가 많은 순서대로 나열한 회사들의 스타일 가이드입니다.

  <img src="https://user-images.githubusercontent.com/50410213/93017717-d4949c80-f605-11ea-9098-3f9f35db08fe.png"/>

  이 중 [raywenderlich](https://github.com/raywenderlich/swift-style-guide#use-of-self), [github](https://github.com/github/swift-style-guide#only-explicitly-refer-to-self-when-required), [linkedin](https://github.com/linkedin/swift-style-guide#3-coding-style), [airbnb](https://github.com/airbnb/swift#style) 에서는 컴파일러에서 필요로 할 때만 self 를 명시하고 [eure](https://github.com/eure/swift-style-guide#all-instance-properties-and-functions-should-be-fully-qualified-with-self-including-within-closures), [StyleShare](https://github.com/StyleShare/swift-style-guide#%ED%81%B4%EB%9E%98%EC%8A%A4%EC%99%80-%EA%B5%AC%EC%A1%B0%EC%B2%B4) 에서는 self 를 항상 명시한다고 합니다.

* 자료를 찾아보면서 self 를 명시해야 하느냐에 대한 논쟁이 굉장히 많았습니다. 실제로 (거절당했지만)[SE-0009: Require self for accessig istance members](https://github.com/apple/swift-evolution/blob/master/proposals/0009-require-self-for-accessing-instance-members.md) 와 같은 제안도 있었습니다. self 명시는 정해진 정답이 없습니다. 팀에 속해있다면 팀 내에서 컨벤션을 정하여 맞추면 되고, 혼자라면 스스로 기준을 두고 결정하여 통일성만 유지한다면 어떤 방법으로 작성하든 상관없다고 생각합니다.

### 참고할 만한 비슷한 질문, 자료

* [How to Use Correctly 'self' Keyword in Swift](https://dmitripavlutin.com/how-to-use-correctly-self-keyword-in-swift/)
* [Clean code: whe to use "self." in Swift, and when not to](http://thebugcode.github.io/when-to-self-in-swift-and-when-not-to-2/)

-----

### Q.

> 코드로 오토레이아웃을 설정하고 싶어요.

코드로 오토레이아웃을 어떻게 설정하나요?

[질문 바로가기](https://stackoverflow.com/questions/26180822/how-to-add-constraints-programmatically-using-swift)

### A.

* 코드로 오토레이아웃을 설정하는 방법에는 여러 가지가 있습니다. 아래 사진처럼 현재 화면에 보이는 뷰 중앙에 가로, 세로 길이가 100인 뷰를 추가하는 코드를 예시로 들어보겠습니다. 다음 예시들은 모두 같은 레이아웃을 설정하는 코드입니다.

  <img width="40%" alt="스크린샷 2020-09-13 오후 11 42 09" src="https://user-images.githubusercontent.com/50410213/93020902-cac96400-f61a-11ea-8ad4-271761e6627f.png">

* 코드로 설정하는 레이아웃은 크게 제약조건을 생성하는 부분과 제약조건을 활성화하는 부분으로 나뉩니다. 제약조건을 먼저 생성하고 생성한 제약조건을 활성화하는 방식으로 레이아웃을 설정합니다.

  ```swift
  override func viewDidLoad() {
          super.viewDidLoad()
          let squareView = UIView()
          squareView.backgroundColor = .systemIndigo
          view.addSubview(squareView)
          
          // autoResizingMask 를 제약조건으로 변환하지 않겠다는 것을 명시
          squareView.translatesAutoresizingMaskIntoConstraints = false
    
          // 레이아웃 제약조건 생성 코드 작성
          ...
          
          // 레이아웃 제약조건 활성화 코드 작성
          ...
      }
  ```

  코드로 오토레이아웃을 설정해 줄 땐 반드시 설정해줄 뷰의 [translatesAutoresizingMaskIntoConstraints](https://developer.apple.com/documentation/uikit/uiview/1622572-translatesautoresizingmaskintoco) 를 false 로 설정해주어야 합니다. 기본적으로 코드로 생성한 뷰는 true 를 기본값으로 가지며 인터페이스 빌더에 뷰를 추가하면 자동으로 false 로 설정되지만, 우리는 코드로 오토레이아웃을 설정해줄 것이기 때문에 직접 false 로 설정해주어야 합니다.

* 제약조건 생성

  * [NSLayoutConstraint 의 이니셜라이저](https://developer.apple.com/documentation/uikit/nslayoutconstraint/1526954-init)를 활용한 제약조건 생성

    ```swift
    let horizontalConstraint = NSLayoutConstraint(item: squareView, attribute: .centerX, relatedBy: .equal, toItem: view, attribute: .centerX, multiplier: 1, constant: 0)
    let verticalConstraint = NSLayoutConstraint(item: squareView, attribute: .centerY, relatedBy: .equal, toItem: view, attribute: .centerY, multiplier: 1, constant: 0)
    let widthConstraint = NSLayoutConstraint(item: squareView, attribute: .width, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: 100)
    let heightConstraint = NSLayoutConstraint(item: squareView, attribute: .height, relatedBy: .equal, toItem: nil, attribute: .notAnAttribute, multiplier: 1, constant: 100)
    ```

    NSLayoutConstraint 의 이니셜라이저는 제약조건 방정식을 그대로 코드로 표현한 형태입니다. 각각의 인자를 식으로 변환하면 `view1.attr1 <relation> multiplier × view2.attr2 + c` 와 같습니다.

  * [NSLayoutAnchor](https://developer.apple.com/documentation/uikit/nslayoutanchor) 를 활용한 제약조건 생성

    ```swift
    let horizontalConstraint = squareView.centerXAnchor.constraint(equalTo: view.centerXAnchor)
    let verticalConstraint = squareView.centerYAnchor.constraint(equalTo: view.centerYAnchor)
    let widthConstraint = squareView.widthAnchor.constraint(equalToConstant: 100)
    let heightConstraint = squareView.heightAnchor.constraint(equalToConstant: 100)
    ```
    
    NSLayoutConstraint 보다 가독성이 좋고 코드가 간결합니다. 공식문서에서는 코드로 오토레이아웃을 설정할 때 NSLayoutConstraint 의 이니셜라이저보단 NSLayoutAnchor 를 권장합니다. 이 둘의 가장 큰 차이점은 잘못된 제약조건을 설정하였을 때 나타납니다.
    
    ```swift
    let horizontalConstraint = NSLayoutConstraint(item: squareView, attribute: .centerX, relatedBy: .equal, toItem: view, attribute: .centerY, multiplier: 1, constant: 0)
    ```
    
    squareView 의 centerX 를 view 의 centerY 에 맞추라는 제약조건입니다. NSLayoutConstraint 를 사용했을 땐 위와 같이 잘못된 제약조건을 설정하면 런타임에 예외를 발생시킵니다.
    
    ```swift
    // Cannot convert value of type 'NSLayoutAnchor<NSLayoutYAxisAnchor>' to expected argument type 'NSLayoutAnchor<NSLayoutXAxisAnchor>' 컴파일 에러 발생
    let horizontalConstraint = squareView.centerXAnchor.constraint(equalTo: view.centerYAnchor)
    ```
    
    반면에, NSLayoutAnchor 를 사용했을 땐 제약조건의 대상이 되는 NSLayoutAnchor 의 타입체크가 이루어져 컴파일 에러를 발생시켜 개발자에게 잘못된 제약조건을 설정한 것을 인지시켜줍니다(하지만 완벽하게 방지해주진 않습니다. 런타임에 충돌이 발생할 수 있습니다.).

* 제약조건 활성화

  * UIView 의 [addConstraints(_:)](https://developer.apple.com/documentation/uikit/uiview/1622523-addconstraint) 를 활용한 제약조건 활성화

    ```swift
    view.addConstraints([horizontalConstraint, verticalConstraint, widthConstraint, heightConstraint])
    ```

    공식문서에서는 iOS 8 이상의 환경에선 addCostraints(_:) 를 직접 호출하는 것보다 isActive 를 활용하라고 합니다.

  * [NSLayoutConstraint 의 isActive](https://developer.apple.com/documentation/uikit/nslayoutconstraint/1527000-isactive) 를 활용한 제약조건 활성화

    ```swift
    horizontalConstraint.isActive = true
    verticalConstraint.isActive = true
    widthConstraint.isActive = true
    heightConstraint.isActive = true
    ```

    Bool 값을 할당하여 제약조건을 활성화, 비활성화할 수 있습니다.

  * [NSLayoutConstraint 의 active(_:)](https://developer.apple.com/documentation/uikit/nslayoutconstraint/1526955-activate) 를 활용한 제약조건 활성화

    ```swift
    NSLayoutConstraint.activate([horizontalConstraint, verticalConstraint, widthConstraint, heightConstraint])
    ```
    
    한 번의 호출로 여러 개의 제약조건을 활성화할 수 있습니다. 각 제약조건의 isActive 를 true 로 설정하는 것과 같습니다.

* Visual format 언어를 활용한 방법([constraints(withVisualFormat:options:metrics:views:)](https://developer.apple.com/documentation/uikit/nslayoutconstraint/1526944-constraints))

  ```swift
  class ViewController: UIViewController {
      
      override func viewDidLoad() {
          super.viewDidLoad()
          let squareView = UIView()
          squareView.backgroundColor = .systemIndigo
          view.addSubview(squareView)
          
          squareView.translatesAutoresizingMaskIntoConstraints = false
          // 레이아웃 제약조건 생성 코드
          let views = ["view": view!, "squareView": squareView]
          let horizontalConstraints = NSLayoutConstraint.constraints(withVisualFormat: "H:[view]-(<=0)-[squareView(100)]", options: .alignAllCenterY, metrics: nil, views: views)
          let verticalConstraints = NSLayoutConstraint.constraints(withVisualFormat: "V:[view]-(<=0)-[squareView(100)]", options: .alignAllCenterX, metrics: nil, views: views)
          // 레이아웃 활성화 코드
          // UIView 의 addConstrains(_:) 를 활용한 방법
          view.addConstraints(horizontalConstraints)
          view.addConstraints(verticalConstraints)
          // NSLayoutConstraint 의 active(_:) 를 활용한 방법
          NSLayoutConstraint.activate(horizontalConstraints)
          NSLayoutConstraint.activate(verticalConstraints)
      }    
  }
  
  ```

  애플에서 제공하는 Visual Format 언어를 활용해 문자열 형식으로 레이아웃을 표현합니다. 간단한 표현식만으로 한번에 여러 개의 제약조건을 생성할 수 있다는 장점이 있습니다. 하지만 표현의 완전성보다 시각화에 초점을 둔 기능으로 가로, 세로 비율 설정과 같이 설정하지 못하는 제약조건이 있다는 한계가 있고, 컴파일러가 Visual Format 언어의 유효성을 검사하지 않아 잘못된 제약조건을 설정하면 런타임에 예외가 발생합니다. 한 문장으로 여러 개의 제약조건을 표현할 수 있어 반환 값이 위의 예시들과는 달리 배열입니다. 그래서 활성화하는 코드도 조금 차이가 있습니다. 자세한 문법은 [Auto Layout Guide: Visual Format Language](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH27-SW1) 에서 확인하실 수 있습니다.

### 참고할 만한 비슷한 질문, 자료

* [Auto Layout Guide: Programmatically Creating Constraints](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/ProgrammaticallyCreatingConstraints.html#//apple_ref/doc/uid/TP40010853-CH16-SW1)

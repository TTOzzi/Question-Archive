# iOS 질문 모음 - 5

### Q.

> 왜 delegate 를 weak 으로 선언하나요?

delegation 패턴에서 delegate 가 weak 로 선언된 것을 확인할 수 있는데요. 왜 delegate 를 weak 로 선언하나요?

[질문 바로가기](https://stackoverflow.com/questions/8449040/why-use-weak-pointer-for-delegation)

### A.

* [retain cycle](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html#ID51) 을 피하기 위해 delegate 를 weak 로 선언합니다. retain cycle 은 두 클래스 인스턴스가 서로에 대한 강력한 참조를 가질 때 발생합니다.

  흔히 사용되는 UITableView 로 예를 들어보겠습니다. 

  ```swift
  class ViewController: UIViewController, UITableViewDelegate {
      // ViewController 가 tableView 를 소유
      var tableView: UITableView = UITableView()
      
      override func viewDidLoad() {
          super.viewDidLoad()
          view.addSubview(tableView)
          // tableView 의 delegate 로 자기 자신(ViewController)을 지정
          tableView.delegate = self
      }
  }
  
  ```

  편의를 위해 레이아웃 설정 코드는 생략하였습니다. ViewController 가 tableView 를 소유하고 있고, ViewController 를 tableView 의 delegate 로 지정합니다.

  위 코드를 그림으로 표현하면 다음과 같은데요.

  <img width="420" alt="스크린샷 2020-08-05 오후 10 02 26" src="https://user-images.githubusercontent.com/50410213/89417638-e31ba880-d769-11ea-8b63-42df3e364f91.png">

  ViewController 가 tableView 를 강력하게 참조하고, tableView 의 delegate 가 ViewController 를 약하게 참조합니다. 공식문서에서 UITableView 의 [delegate](https://developer.apple.com/documentation/uikit/uitableview/1614894-delegate) 를 찾아보면 weak 로 선언되어있는 것을 확인할 수 있습니다.

  ```swift
  weak var delegate: UITableViewDelegate? { get set }
  ```

  만약 delegate 가 strong 으로 선언되어있었다면 서로가 서로를 강하게 참조하는 retain cycle 이 발생하여 두 객체가 메모리에서 해제되지 않는 문제가 발생할 것입니다.

* delegate 를 누구로 지정 해주냐에 따라 strong 으로 선언해도 되는 경우도 존재합니다.

  ```swift
  protocol CustomViewDelegate: class {
      func userDidTap()
  }
  
  class CustomView: UIView {
      var delegate: CustomViewDelegate?
      
      func mockDelegatecall() {
          delegate?.userDidTap()
      }
  }
  
  // UserViewDelegate 프로토콜을 채택한 클래스
  class Delegate: CustomViewDelegate {
      func userDidTap() {
          print("tap!")
      }
  }
  
  class ViewController: UIViewController {
      // 별도의 delegate 객체를 생성
      let customView = CustomView()
      let customViewDelegate = Delegate()
      
      override func viewDidLoad() {
          super.viewDidLoad()
          view.addSubview(customView)
          customView.delegate = customViewDelegate
      }
  }
  ```

  ViewController 가 customView 와 customViewDelegate 를 소유하고 있고 customView 의 delegate 로 customViewDelegate 를 지정해주었습니다.

  위 코드를 그림으로 표현하면 다음과 같습니다.

  <img width="488" alt="스크린샷 2020-08-05 오후 10 44 09" src="https://user-images.githubusercontent.com/50410213/89420438-aea9eb80-d76d-11ea-917a-23051dfc29ab.png">

  delegate 를 담당하는 별도의 객체를 생성함으로써 customView 내부에 delegate 를 weak 으로 선언하지 않았지만, retain cycle 이 발생하지 않습니다. 

  추가로 delegate 를 구조체로 지정하는 경우에도 weak 를 사용하지 않아도 됩니다. 구조체는 값 유형이기 때문에 retain cycle 을 발생시키지 않습니다.

* weak 으로 선언하지 않아도 되는 경우도 있으나, 복잡한 구조의 코드에서 weak 로 선언하지 않은 delegate 는 예상치 못한 retain cycle 을 발생시킬 수 있으니 weak 으로 선언하는 것을 권장합니다.

### 참고할 만한 비슷한 질문들

* [Swift delegation - when to use weak pointer on delegate](https://stackoverflow.com/questions/30056526/swift-delegation-when-to-use-weak-pointer-on-delegate?rq=1)
* [How can I make a weak protocol reference in 'pure' Swift (without @objc)](https://stackoverflow.com/questions/24066304/how-can-i-make-a-weak-protocol-reference-in-pure-swift-without-objc)
* [swift delegation - when to use weak reference, why 'delegate' is nil?](https://stackoverflow.com/questions/34611873/swift-delegation-when-to-use-weak-reference-why-delegate-is-nil)

-----


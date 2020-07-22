# iOS 질문모음 - 3

### Q.

> RxSwift 에서 DisposeBag 을 쓰는 이유가 무엇인가요?

여러 예제코드에서 Observable 을 subscribe 한 후 항상 disposed(by: disposebag) 를 사용하던데 dispose 를 사용하는 이유가 무엇인가요? 메모리 누수를 줄이기 위해서인가요? 만약 dispose 를 해주지 않으면 어떻게 되나요?

[질문 바로가기](https://yagom.net/forums/topic/rxswift-disposebag을-써야하는-이유/)

### A.

* RxSwift 에서 [Observable](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Observable.swift) 을 subscribe 하면 항상 [Disposable](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposable.swift) 을 반환합니다. 이 Disposable 들을 dispose 해주지 않으면 메모리에 계속 남아 메모리 누수가 발생합니다. 그렇다면 일일이 Dispoable 프로토콜에 구현되어있는 dispose() 를 호출하여 없애주어야 하는데 [RxSwift 가이드 문서의 Disposing](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing) 에서는 dispose() 를 직접 호출하지 말고 DisposeBag 이나, takeUntil() 을 통해 dispose 하기를 권장합니다.

* [DisposeBag](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposables/DisposeBag.swift) 의 내부구현을 살펴보면

  ```swift
  private func dispose() {
      let oldDisposables = self._dispose()
  
      for disposable in oldDisposables {
          disposable.dispose()
      }
  }
  
  deinit {
      self.dispose()
  }
  ```

  이렇게 DisposeBag 에 담긴 모든 Disposable 들을 dispose 하는 메소드가 구현되어있고 이를 deinit 에서 호출합니다. 따라서 DisposeBag 을 가진 인스턴스가 메모리에서 해제될 때 DisposeBag 도 같이 해제되면서 내부의 Disposable 들을 dispose 해줍니다.

  추가로 특정 시점에 DisposeBag 의 Disposable 들을 초기화해야 할 경우 DisposeBag 의 dispose 메소드가 private 으로 선언되어 있어서

  ```swift
  self.disposeBag = DisposeBag()
  ```

  이런 방식으로 초기화할 수 있습니다. 

  같은 동작을 하면서 명시적으로 dispose 를 호출해주고 싶다면 [CompositeDisposable](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposables/CompositeDisposable.swift) 을 사용해보세요. dispose 뿐만 아니라 키값을 통해 특정 disposable 만 제거하는 등 다양한 기능이 있습니다.

* takeUntil() 연산자를 통해 특정 인스턴스가 deallocated 될 때까지만 살려두는 방법도 있습니다.

  ```swift
  sequence
      .takeUntil(self.rx.deallocated)
      .subscribe {
          print($0)
      }
  ```

* Dispose 를 하지 않아서 발생하는 문제도 있지만, subscribe 할 때 escaping 클로저에서의 강한 참조 때문에 발생하는 메모리 문제도 주의해야 합니다. RxSwift 에서의 메모리 누수 처리에 대해 더 자세히 알고 싶다면 [Disposing RxSwift's Memory Leaks](https://medium.com/gett-engineering/disposing-rxswifts-memory-leaks-6ceb73162170) 를 읽어보세요.

### 참고할 만한 비슷한 질문들

* [When should we call addDisposableTo(disposeBag) in RxSwift?](https://stackoverflow.com/questions/37724766/when-should-we-call-adddisposabletodisposebag-in-rxswift)
* [RxSwift DisposeBag usage in ViewController](https://stackoverflow.com/questions/55028020/rxswift-disposebag-usage-in-viewcontroller)
* [How do I unsubscribe from an observable?](https://stackoverflow.com/questions/39458195/how-do-i-unsubscribe-from-an-observable)
* [DisposeBag memory leak?](https://stackoverflow.com/questions/44296776/disposebag-memory-leak)

-----

### Q.

> leading, trailing 과 left, right 의 차이가 무엇인가요?

오토레이아웃을 공부하면서 left, right 보다는 leading, trailing 을 사용하는 것을 권장한다는 이야기를 많이 들었는데요.

leading, trailing 과 left, right 는 어떤 차이가 있나요?

[질문 바로가기](https://yagom.net/forums/topic/leading-trailing과-left-right/)

### A.

* leading, trailing 은 글의 시작방향과 끝방향을 뜻하는데요. 해당 기기의 언어 설정에 따라 유동적으로 변화합니다. 영어, 한국어 등의 LTR(left-to-right) 언어를 사용하는 기기에서는 leading 이 왼쪽, trailing 이 오른쪽이 되고 반대로 아랍어, 히브리어 등의 RTL(right-to-left) 언어를 사용하는 기기에서는 leading 이 오른쪽, trailing 이 왼쪽이 됩니다.
* left, right 는 항상 화면에서의 왼쪽, 오른쪽을 가리킵니다.
* [Auto Layout Guide: Rules of Thumb](https://developer.apple.com/library/archive/documentation/UserExperience/Conceptual/AutolayoutPG/WorkingwithConstraintsinInterfaceBuidler.html#//apple_ref/doc/uid/TP40010853-CH10-SW7) 에도 `Always use leading and trailing constraints instead of right and left.` 라는 문구가 있으니 UI 를 구성할 때 절대적으로 오른쪽, 왼쪽에 위치해야 하는 것이 아니라면 leading, trailing 을 사용하는 것을 권장합니다.

### 참고할 만한 비슷한 질문들

* [Difference between leftAnchor and leadingAnchor?](https://stackoverflow.com/questions/32981532/difference-between-leftanchor-and-leadinganchor/32981750)
* [Difference between NSLayoutAttributeLeft vs NSLayoutAttributeLeading](https://stackoverflow.com/questions/19971508/difference-between-nslayoutattributeleft-vs-nslayoutattributeleading)

-----

### Q.

> 왜 IBOutlet 을 weak var 로 선언하나요?

스토리보드를 공부하던 도중 IBOutlet 을 weak var 로 선언하는 것에 의문이 생겼습니다.

스토리보드에서 UI 를 생성했을 때, 스토리보드 내에서 해당 UI 의 인스턴스가 생성되는 건가요?

IBOutlet 을 상수가 아닌 변수로 선언하는 이유가 무엇인가요?

IBOutlet 을 strong 으로 선언해도 오류가 발생하지 않는데 필요에 따라 weak, strong 을 구분해서 써야한다고 합니다. 어떤 경우에 IBOutlet 을 strong 으로 선언하나요?

참고한 글

* [The difference between Weak & Strong in Swift](https://medium.com/@janakmshah/the-difference-between-weak-strong-in-swift-2c953cedd7c0)
* [Should Outlets Be Weak or Strong](https://cocoacasts.com/should-outlets-be-weak-or-strong)

[질문 바로가기](https://yagom.net/forums/topic/스토리보드와-인터페이스-빌더를-이용한-뷰-생성시/)

### A.

* 스토리보드에서 UI 를 생성하더라도 인스턴스가 스토리보드에서 생성되는 것은 아닙니다. 앱이 실행되어 뷰를 스토리보드에서 불러오는 순간에 스토리보드에 구성해 놓은 모양대로 뷰의 인스턴스가 생성되어 동작하게 됩니다. 즉, 스토리보드에는 어떤 뷰 클래스의 뷰가 어디에 위치할지 등의 정보만 담아두고, 실제로 뷰 클래스의 인스턴스가 생성되는 시점은 스토리보드로부터 뷰 구성 정보를 불러와서 화면에 보여주려 할 때(앱 동작 중) 입니다. 그래서 런타임 오류 발생 여지가 있어도 스토리보드에서는 오류가 발생하지 않고, 실행해서 화면이 보여지려고 하는 순간 오류로 앱이 죽죠. 스토리보드는 앱 동작 전에 이미 만들어져있으므로 스토리보드에 인스턴스가 생성된다고 볼 수는 없습니다. 그저 우리 눈에 그렇게 보이는 것뿐입니다.
* IBOutlet 은 변수로만 선언 가능합니다. 상수는 불가능합니다. 그 이유는 위의 설명과 연관이 있는데요, 이미 IBOutlet 프로퍼티가 메모리에 생성된 후에 동적으로 스토리보드 정보로부터 만들어진 인스턴스를 IBOutlet 프로퍼티에 할당해야 하기 때문입니다. 그런데 해당 프로퍼티가 상수면 변경(할당)이 불가능하므로 항상 변수여야 합니다.
* 통상 처음 배우는 분들은 뷰 컨트롤러의 뷰 위에 얹어지는 자식 뷰(subview)를 IBOutlet 프로퍼티로 지정하므로 그 기준으로 설명하겠습니다. 뷰 컨트롤러의 뷰 위에 얹어지는 자식 뷰를 IBOutlet 프로퍼티에 strong 으로 할당하면 자식 뷰의 retain count 가 1 증가합니다. IBOutlet 프로퍼티에 할당된 자식 뷰가 또 다른 뷰의 자식 뷰로 얹어지면 retain count 가 1 추가로 증가합니다. 그래서 뷰 컨트롤러의 뷰 위에 얹어진 자식 뷰의 retain count 는 IBOutlet 이 strong 으로 할당된 경우 기본적으로 2의 retain count 를 가집니다. 
* 위의 설명처럼 strong 으로 선언해도 오류는 발생하지 않습니다. 다만 자식 뷰가 부모 뷰(superview)에서 떨어져 나와도 retain count 가 1이 남아있으므로 뷰 컨트롤러가 메모리에서 해제되기 전까지 IBOutlet 변수에 할당된 뷰는 메모리에서 해제되지 않습니다(물론 부모 뷰에서 떨어져 나온 후 IBOutlet 변수에 nil 을 할당하면 메모리에서 해제됩니다. 이해가 안되면 ARC 에 대해 조금 더 공부하면 좋습니다.). 이를 의도하고 strong 으로 선언한다면 괜찮습니다. 경우에 따라서  해당 뷰를 부모 뷰에 붙였다 뗐다 해야하는 경우엔 유용합니다. 하지만 이것도 뷰를 매번 붙였다([addSubview(_:)](https://developer.apple.com/documentation/uikit/uiview/1622616-addsubview)) 뗐다([removeFromSuperview()](https://developer.apple.com/documentation/uikit/uiview/1622421-removefromsuperview))하기 보다는 뷰를 잠깐 안 보이게 [isHidden](https://developer.apple.com/documentation/uikit/uiview/1622585-ishidden) 으로 숨겨주는 방법이 있습니다. 이때는 부모 뷰에서 실질적으로 떨어져 나온 것이 아니므로 retain count 의 변화는 없습니다. 그래서 통상 의도적인 목적이 없는 경우에는 IBOutlet 변수를 weak 로 선언하여 IBOutlet 인스턴스의 retain count 를 1로 만들어줍니다. 부모 뷰에서 떨어져 나오면 retain count 가 0이 되므로 바로 메모리에서 해제되죠.

아래 영상을 보면 뷰를 스토리보드에 추가해주고 MyView 클래스를 뷰의 커스텀클래스로 지정해준 후 IBOutlet 을 strong 으로 선언했을 때, IBOutlet 변수에 할당된 뷰를 부모 뷰에서 제거해도 deinit 이 호출되지 않아요. 반대로 weak 로 선언한 후에 실행하면 뷰가 부모 뷰에서 제거된 이후에 deinit 이 호출되는 것을 확인할 수 있을 겁니다.

```swift
class MyView: UIView {
    deinit {
        print("뷰가 메모리에서 해제됨")
    }
}
```

![실행 영상](https://user-images.githubusercontent.com/50410213/88182765-8d280a80-cc6b-11ea-9d7c-ee550fe455eb.gif)

### 참고할 만한 비슷한 질문들

* [Why can't an @IBOutlet be assigned let instead of var in Swift](https://stackoverflow.com/questions/46893514/why-cant-an-iboutlet-be-assigned-let-instead-of-var-in-swift)
* [Why does Xcode create a weak reference for an IBOutlet?](https://stackoverflow.com/questions/21654113/why-does-xcode-create-a-weak-reference-for-an-iboutlet)
* [What is the point of using weak variables in IBOutlets?](https://www.quora.com/What-is-the-point-of-using-weak-variables-in-IBOutlets)

-----


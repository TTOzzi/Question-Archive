# iOS 질문모음 - 3

### Q.

> RxSwift 에서 DisposeBag 을 쓰는 이유가 무엇인가요?

여러 예제코드에서 Observable 을 subscribe 한 후 항상 disposed(by: disposebag) 를 사용하던데 dispose 를 사용하는 이유가 무엇인가요? 메모리 누수를 줄이기 위해서인가요? 만약 dispose 를 해주지 않으면 어떻게 되나요?

[원본 질문 링크](https://yagom.net/forums/topic/rxswift-disposebag을-써야하는-이유/)

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


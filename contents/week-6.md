# iOS 질문 모음 - 6

### Q.

> DispatchQueue 에서 main.sync 는 언제 사용하나요?

DispatchQueue 에 대해 공부하면서 생긴 의문인데, 많은 자료에서 DispatchQueue.main 에서 sync 를 호출하지 말라고 합니다. 왜 호출하지 말라고 하는 것이며, 그럼 왜 GCD 에서 DispatchQueue.main.sync 가 구현되어있는 건가요?

[질문 바로가기](https://stackoverflow.com/questions/42772907/what-does-main-sync-in-global-async-mean)

### A.

* 먼저 DispatchQueue.main 에서 sync 를 호출하면 안 되는 이유를 이해하려면 [Deadlock(교착 상태)](https://ko.wikipedia.org/wiki/교착_상태) 이란 개념을 알아야 하는데요. Deadlock 이란 `두 개 이상의 작업이 서로 상대방의 작업이 끝나기만을 기다리고 있기 때문에 결과적으로 아무것도 완료되지 못하는 상태` 라고 합니다. Deadlock 은 DispatchQueue.main 뿐만 아니라 백그라운드 스레드가 할당된 큐에서도 발생할 수 있습니다.

* 간단한 코드로 예를 들어보겠습니다. DispatchQueue 의 생성자로 myQueue 를 생성해주었습니다. 생성자의 attributes 를 concurrent 로 설정해주지 않으면 기본값으로 serial queue 를 생성하게 됩니다.

  ```swift
  let myQueue = DispatchQueue(label: "label")
  
  myQueue.async {
      myQueue.sync {
          // 외부 블록이 완료되기 전에 내부 블록은 시작되지 않습니다.
          // 외부 블록은 내부 블록이 완료되기를 기다립니다.
          // => deadlock
      }
      // 이 부분은 영원히 실행되지 않습니다.
  }
  ```

  myQueue 는 **serial queue** 이므로 한 번에 하나의 작업을 합니다. serial queue 에 추가되는 작업은 DispatchQueue 에서 관리하는 고유한 스레드에서 실행됩니다. 따라서 위 예시에서 sync 내부 블록은 외부 블록이 완료될 때까지 기다리고, sync 블록 이후의 동작은 sync 내부 블록이 완료될 때까지 기다리는 deadlock 이 발생하게 됩니다.

  이 문제는 myQueue 를 생성할 때 

  ```swift
  let myQueue = DispatchQueue(label: "com.ttozzi", attributes: .concurrent)
  ```

  와 같이 concurrent 속성을 설정하여 한번에 여러 작업을 실행할 수 있도록 해주어 해결할 수 있습니다.

* 다시 본론으로 돌아오면, DispatchQueue.main 은 프로그램의 메인 스레드에서 작업을 실행하는 전역적으로 사용이 가능한 **serial queue** 입니다. 앱의 run loop 와 함께 작동하며, 대기 중인 동작을 run loop 의 다른 이벤트 처리와 조율해줍니다. 이러한 DispatchQueue.main 에서 sync 를 호출하게 된다면 끊임없이 앱의 이벤트 처리를 하고 있던 메인 스레드가 sync 호출에 의해 멈추게 되고 위에서 예시로 들었던 코드와 같이 deadlock 이 발생하게 됩니다.

* 그럼 DispatchQueue.main 은 어떤 경우에 사용할까요? 백그라운드 스레드에서 이루어지는 작업들 사이에 순서에 맞게 메인 스레드에서 어떠한 작업이 이루어져야 할 때 사용합니다. 다음 코드를 보겠습니다.

  ```swift
  DispatchQueue.global().async {
      // UI 업데이트 전 실행되어야만 하는 코드
      DispatchQueue.main.sync {
          // UI 업데이트
      }
      // UI 업데이트 후에 실행되어야만 하는 코드
  }
  ```

  백그라운드 스레드에서 작업이 진행되는 중 UI 업데이트가 이루어져야 하는 상황입니다. 좀 더 구체적으로 예를 들자면 테이블 뷰나 콜렉션 뷰에 셀을 추가하거나 제거할 때, 셀의 추가, 제거 작업이 이루어지던 중 네트워크 통신과 같은 다른 비동기 작업으로 인해 DataSource 의 데이터가 변경된다면 DataSource 의 데이터와 뷰의 데이터가 일치하지 않아 에러가 발생하게 됩니다. 이를 방지하기 위해 사용합니다. 애플의 [PHPhotoLibraryChangeObserver](https://developer.apple.com/documentation/photokit/phphotolibrarychangeobserver) 문서 `Handling photo library changes in a collection view` 의 예시코드에서도 사진앨범의 데이터와 업데이트한 콜렉션 뷰의 데이터가 달라지는 상황을 방지하기위해 DispatchQueue.main.sync 를 호출하는 것을 확인할 수 있습니다.

### 참고할 만한 비슷한 질문, 자료

* [How do I create a deadlock in Grand Central Dispatch?](https://stackoverflow.com/questions/15381209/how-do-i-create-a-deadlock-in-grand-central-dispatch)
* [why concurrentQueue.sync DON'T cause deadlock](https://stackoverflow.com/questions/53293965/why-concurrentqueue-sync-dont-cause-deadlock)
* [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationQueues/OperationQueues.html#//apple_ref/doc/uid/TP40008091-CH102-SW2)
* [Difference between DispatchQueue.main.async and DispatchQueue.main.sync](https://stackoverflow.com/questions/44324595/difference-between-dispatchqueue-main-async-and-dispatchqueue-main-sync)
* [How to dispatch on main queue synchronously without a deadlock?](https://stackoverflow.com/questions/10330679/how-to-dispatch-on-main-queue-synchronously-without-a-deadlock)

-----


# iOS 질문 모음 - 6

### Q.

> DispatchQueue.global() 과 DispatchQueue.init() 의 차이가 무엇인가요?

백그라운드 스레드를 관리하는 큐를 생성하고 싶은데요. [DispatchQueue.global()](https://developer.apple.com/documentation/dispatch/dispatchqueue/2300077-global) 으로 생성하는 방법과 [DispatchQueue.init()](https://developer.apple.com/documentation/dispatch/dispatchqueue/2300059-init) 으로 label 을 설정하여 생성하는 방법에는 어떤 차이가 있나요?

[질문 바로가기](https://stackoverflow.com/questions/53489764/understanding-dispatch-queue-threading/53489813#53489813)

### A.

* DispatchQueue.global() 은 인자로 받은 우선순위에 해당하는 전역 시스템 큐를 반환합니다. 이 전역 시스템 큐는 여러 작업을 동시에 수행하는 concurrent queue 입니다.

* DispatchQueue 의 생성자로 만든 큐는 인자로 받은 여러 설정을 적용한 큐를 생성합니다.

  ```swift
  let myQueue = DispatchQueue(label: "myQueue")
  ```

  위와 같이 label 만으로 만들게 되면, unspecified 의 우선순위를 가진 serial queue 를 생성합니다.

  ```swift
  let myQueue = DispatchQueue(label: "myQueue", qos: .userInitiated, attributes: .concurrent)
  ```

  우선순위와 serial, concurrent 구분을 직접 설정해줄 수도 있습니다.

* 지금까지의 예시들을 봤을 때 두 방식에 별다른 차이가 없는데요. 그럼 왜 global() 을 사용하지 않고 생성자를 사용할까요?

  첫 번째는 디버깅에서의 이점이 있습니다.

  ```swift
  func debugQueue(_ queue: DispatchQueue) {
      for _ in 1...5 {
          queue.async {
              print("break point")
          }
      }
      
      for _ in 1...5 {
          queue.async {
              print("break point")
          }
      }
  }
  ```

  디버깅 상황을 만들기 위해 큐를 인자로 받아 그 큐에서 간단한 동작을 하는 함수를 만들었습니다. 위에서 만든 myQueue 를 인자로 넘겨주고 print 가 실행되는 부분에 브레이크포인트를 설정해보겠습니다.

  <img width="456" alt="스크린샷 2020-08-16 오전 12 44 22" src="https://user-images.githubusercontent.com/50410213/90315863-b598e100-df59-11ea-9c0a-433f4f912daf.png">

  Xcode 의 Debug navigator 를 보면 브레이크포인트가 설정된 코드가 실행된 큐와 스레드, 그리고 해당 큐가 관리 중인 스레드들도 확인할 수 있습니다. 지금은 큐가 하나뿐이지만 사용하는 큐가 더 많아지고, 해야 하는 작업량도 많아진다면 label 을 통해 큐를 구분할 수 있으니 디버깅하기 편해지겠죠?

  <img width="458" alt="스크린샷 2020-08-16 오전 1 16 24" src="https://user-images.githubusercontent.com/50410213/90316593-22ae7580-df5e-11ea-925c-889c446a2d58.png">

  반면에 DispatchQueue.global() 을 넘겨주면 시스템에 구현되어있는 전역 큐를 사용하게 되어 디버깅이 어려워지게 됩니다.

  두 번째는 [barrier](https://developer.apple.com/documentation/dispatch/dispatchworkitemflags/1780674-barrier) 와 같은 DispatchQueue 의 몇 가지 세부적인 설정을 사용하기 위함입니다. barrier 란 단어 뜻 그대로 동시성을 지원하는 비동기 큐에서 장벽 같은 역할을 해주는 기능인데요.

  ![barrier](https://koenig-media.raywenderlich.com/uploads/2014/09/Dispatch-Barrier-Swift.png)

  위 그림처럼 비동기 작업들이 실행되다가 flag 가 barrier 로 설정된 작업이 실행되면 그 작업이 끝날 때 까지 serial queue 처럼 동작하게 됩니다. 간단한 예시코드를 보겠습니다.

  ```swift
  let myQueue = DispatchQueue(label: "myQueue", attributes: .concurrent)
  
  for i in 1...5 {
      myQueue.async {
          print("\(i)")
      }
  }
  
  myQueue.async(flags: .barrier) {
      print("barrier!!")
      sleep(5)
  }
  
  for i in 6...10 {
      myQueue.async {
          print("\(i)")
      }
  }
  ```

  숫자들을 비동기로 출력하는 코드 사이에 flags 를 barrier 로 설정한 코드를 추가해주었습니다.

  ![barrier](https://user-images.githubusercontent.com/50410213/90316847-f98ee480-df5f-11ea-9de9-2bc991e8bf83.gif)

  숫자들이 출력되다가 barrier 블록이 실행되자 다음 작업들이 barrier 블록이 끝날 때까지 기다리는 것을 확인할 수 있습니다. 만약 barrier 가 없다면 concurrent queue 에서 비동기로 숫자들을 출력하기 때문에 한번에 모든 숫자가 출력될 것입니다. [barrier](https://developer.apple.com/documentation/dispatch/1452917-dispatch_barrier_sync?language=occ) 공식문서를 보면 barrier 를 지정하는 큐는 직접 만든 concurrent queue 여야만 한다고 합니다. barrier 를 포함한 몇몇 세부적인 설정들은 글로벌 큐에서는 사용할 수 없습니다. 

### 참고할 만한 비슷한 질문, 자료

* [Grand Central Dispatch Tutorial for Swift 4](https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2)
* [DispatchQueue sync vs sync barrier in concurrent queue](https://stackoverflow.com/questions/58236153/dispatchqueue-sync-vs-sync-barrier-in-concurrent-queue)

----

### Q.

> DispatchQueue 에서 main.sync 는 언제 사용하나요?

DispatchQueue 에 대해 공부하면서 생긴 의문인데, 많은 자료에서 DispatchQueue.main 에서 sync 를 호출하지 말라고 합니다. 왜 호출하지 말라고 하는 것이며, 그럼 왜 GCD 에서 DispatchQueue.main.sync 가 구현되어있는 건가요?

[질문 바로가기](https://stackoverflow.com/questions/42772907/what-does-main-sync-in-global-async-mean)

### A.

* 먼저 DispatchQueue.main 에서 sync 를 호출하면 안 되는 이유를 이해하려면 [Deadlock(교착 상태)](https://ko.wikipedia.org/wiki/교착_상태) 이란 개념을 알아야 하는데요. Deadlock 이란 `두 개 이상의 작업이 서로 상대방의 작업이 끝나기만을 기다리고 있기 때문에 결과적으로 아무것도 완료되지 못하는 상태` 라고 합니다. Deadlock 은 DispatchQueue.main 뿐만 아니라 백그라운드 스레드를 관리하는 큐에서도 발생할 수 있습니다.

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
  let myQueue = DispatchQueue(label: "label", attributes: .concurrent)
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

### Q.

> 특정한 여러 개의 비동기 작업이 완료된 후에 다른 작업을 실행해주고 싶어요.

반복문을 사용하여 네트워크 요청을 여러 번 보내는데, 모든 요청이 완료된 후 다른 작업을 실행하는 방법이 있나요?

[질문 바로가기](https://stackoverflow.com/questions/35906568/wait-until-swift-for-loop-with-asynchronous-network-requests-finishes-executing)

### A.

* [DispatchGroup](https://developer.apple.com/documentation/dispatch/dispatchgroup) 을 활용해 그룹에 연결된 작업들이 완료되었을 때 핸들러를 실행해줄 수 있습니다. 5개의 이미지를 다운받은 후 UI 를 업데이트해줘야 하는 상황을 예시로 들어보겠습니다.

  ```swift
  override func viewDidLoad() {
      super.viewDidLoad()
          
      for _ in 1...5 {
          DispatchQueue.global().async {
              print("이미지 다운로드 요청!")
              sleep(3)
              print("이미지 다운로드 완료!")
          }
      }
          
      print("UI 업데이트!")
  }
  ```

  이미지 데이터를 다운받는데 3초가 걸린다고 가정하고 작성한 예시코드입니다. 코드를 실행해보면

  ![async](https://user-images.githubusercontent.com/50410213/90325097-6f23a080-dfb2-11ea-87b8-1ca882419fde.gif)

  이미지 다운로드를 비동기로 요청하기 때문에 UI 업데이트와 이미지 다운로드 간의 순서를 보장하지 않는 것을 확인할 수 있습니다.

  ```swift
  override func viewDidLoad() {
      super.viewDidLoad()
      
      let downloadGroup = DispatchGroup()
      
      for _ in 1...5 {
          DispatchQueue.global().async(group: downloadGroup) {
              print("이미지 다운로드 요청!")
              sleep(3)
              print("이미지 다운로드 완료!")
          }
      }
      
      downloadGroup.notify(queue: .main) {
          print("UI 업데이트!")
      }
  }
  ```

  downloadGroup 이라는 DispatchGroup 을 만들어 다운로드 요청 작업들을 downloadGroup 에 연결해주고 downloadGroup 의 작업이 끝나면 main 큐에서 UI 업데이트를 하도록 해주었습니다.

  ![dispatchGroup](https://user-images.githubusercontent.com/50410213/90325212-ba8a7e80-dfb3-11ea-8654-07a9dafd7d81.gif)

  이미지 다운로드가 모두 완료된 후 UI 업데이트가 출력되는 것을 확인할 수 있습니다.

  ```swift
  downloadGroup.wait()
  ```

  [wait()](https://developer.apple.com/documentation/dispatch/dispatchgroup/2016090-wait) 을 활용해 wait() 을 호출한 큐에서 그룹의 작업이 완료될 때까지 동기적으로 기다리도록 할 수도 있습니다. 그러나 이 방법은 wait() 을 호출한 큐를 차단하므로 deadlock 이 발생할 수 있어 주의해야 합니다. deadlock 발생을 피하려고 [wait(timeout:)](https://developer.apple.com/documentation/dispatch/dispatchgroup/1780590-wait) 으로 최대 대기 시간을 설정해주기도 합니다.

* 비동기 작업관련 API 가 내부적으로 비동기로 구현되어 있어 async 블록에서 group 지정을 해줄 수 없는 경우 다음과 같이 [enter()](https://developer.apple.com/documentation/dispatch/dispatchgroup/1452803-enter), [leave()](https://developer.apple.com/documentation/dispatch/dispatchgroup/1452872-leave) 를 활용하는 방법도 있습니다.

  ```swift
  override func viewDidLoad() {
      super.viewDidLoad()
      
      let downloadGroup = DispatchGroup()
      
      for _ in 1...5 {
          guard let imageURL = URL(string: "https://example.com") else { return }
          downloadGroup.enter()
          URLSession.shared.dataTask(with: imageURL) { (data, response, error) in
              // 이미지 다운로드 요청 완료 후 동작
              downloadGroup.leave()
          }.resume()
      }
      
      downloadGroup.notify(queue: .main) {
          print("UI 업데이트!")
      }
  }
  ```

### 참고할 만한 비슷한 질문, 자료

* [Waiting until the task finishes](https://stackoverflow.com/questions/42484281/waiting-until-the-task-finishes)
* [Waiting until two async blocks are executed before stating another block](https://stackoverflow.com/questions/11909629/waiting-until-two-async-blocks-are-executed-before-starting-another-block)
* [Swift DispatchGroup notify before task finish](https://stackoverflow.com/questions/11909629/waiting-until-two-async-blocks-are-executed-before-starting-another-block)

----

### Q.

> 동시에 실행되는 비동기 작업의 개수를 제한할 수 있나요?

비동기적으로 데이터베이스에서 데이터를 수집하고 있습니다. 동시에 실행되는 작업의 개수를 제한할 수 있는 방법이 있나요?

[질문 바로가기]()

### A.

* [OperationQueue](https://developer.apple.com/documentation/foundation/operationqueue) 의 [maxConcurrentOperationCount](https://developer.apple.com/documentation/foundation/operationqueue/1414982-maxconcurrentoperationcount) 나 [DispatchSemaphore](https://developer.apple.com/documentation/dispatch/dispatchsemaphore) 를 활용해 제한할 수 있습니다. 데이터를 수집하는데에 3초의 시간이 소모되며, 동시에 3개의 작업만 해야 한다고 가정하고 예시코드를 작성해 보겠습니다.

* OperationQueue 를 활용한 방법

  ```swift
  func acquire(data: Int) {
      print("데이터 \(data) 수집 시작!")
      sleep(3)
      print("데이터 \(data) 수집 완료!")
  }
  
  let acquiringQueue = OperationQueue()
  acquiringQueue.maxConcurrentOperationCount = 3
  
  for i in 1...10 {
      acquiringQueue.addOperation {
          acquire(data: i)
      }
  }
  ```

  데이터 수집 작업을 관리할 acquiringQueue 라는 OperationQueue 를 만들어 주었습니다. OperationQueue 의 maxConcurrentOperationCount 를 3으로 설정해 동시에 실행할 수 있는 작업의 수를 3개로 제한해주었습니다. 다음은 총 10개의 데이터를 수집하는 코드를 실행한 모습입니다.

  ![operationqueue](https://user-images.githubusercontent.com/50410213/90326614-5c669700-dfc5-11ea-83bd-3bc7f5f62e18.gif)

  작업을 3개씩 실행하는 것을 확인할 수 있습니다.

* DispatchSemaphore 를 활용한 방법

  ```swift
  let acquiringQueue = DispatchQueue(label: "acquiringQueue", attributes: .concurrent)
  let acquiringSemaphore = DispatchSemaphore(value: 3)
  
  DispatchQueue.global().async {
      for i in 1...10 {
          // count -= 1
          acquiringSemaphore.wait()
          acquiringQueue.async {
              acquire(data: i)
              // count += 1
              acquiringSemaphore.signal()
          }
      }
  }
  ```

  이번에는 DispatchQueue 의 생성자로 동시성을 지원하는 사용자 지정 큐와 DispatchSemaphore 를 생성해주었습니다. 세마포어를 만들 때 사용 가능한 리소스의 수를 지정해줍니다. 이 값은 세마포어가 제한할 작업의 개수가 됩니다. 비동기 호출 직전에 [wait()](https://developer.apple.com/documentation/dispatch/dispatchsemaphore/2016071-wait) 을 호출하여 세마포어의 값을 1 감소시킵니다. 세마포어의 값이 음수가 되면 함수는 커널에 스레드를 차단하도록 지시합니다. wait() 을 메인 스레드에서 호출하게 되면 메인 스레드를 차단하기 때문에 주의해야합니다. 비동기 작업이 완료된 후 [signal()](https://developer.apple.com/documentation/dispatch/dispatchsemaphore/1452919-signal) 을 호출하여 세마포어의 값을 1 증가시키면 차단된 스레드 중 하나가 차단 해제되고 남은 작업을 수행하게 됩니다.

* 동시에 실행되는 작업의 개수를 제한하는 경우 외에도 일반적으로 DispatchSemaphore 는 공유 자원에 접근하는 스레드의 개수를 제한해야 할 때 유용하게 사용합니다.

### 참고할 만한 비슷한 질문, 자료

* [When to use Semaphore instead of Dispatch Group?](https://stackoverflow.com/questions/49923810/when-to-use-semaphore-instead-of-dispatch-group)
* [Does DispatchSemaphore wait for specific thread objects?](https://stackoverflow.com/questions/61309944/does-dispatchsemaphore-wait-for-specific-thread-objects)
* [can I call on semaphore.wait() main thread?](https://stackoverflow.com/questions/50791315/can-i-call-on-semaphore-wait-main-thread)

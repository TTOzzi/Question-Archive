# iOS 질문모음 - 9

### Q.

> @objc 의 의미가 무엇인가요?

@objc 키워드의 의미가 무엇인가요?

[질문 바로가기](https://stackoverflow.com/questions/30795117/when-to-use-objc-in-swift)

### A.

* @objc 는 Swift 로 작성된 코드에서 Objective-C 의 API 와의 호환을 위해 사용합니다. [Timer](https://developer.apple.com/documentation/foundation/timer), [NotificationCenter](https://developer.apple.com/documentation/foundation/notificationcenter) 등의 target-action 패턴이나 KVO, KVC 와 같은 Objc 기반 API 는 Objc 런타임을 통해 제공되기 때문에 `@objc` 를 명시하여 Objc 런타임에 노출할 프로퍼티와 메소드를 컴파일러에게 알려줘야 합니다.
* 답변2
* ...

### 참고할 만한 비슷한 질문, 자료

* [Using Objective-C Runtime Features in Swift](https://developer.apple.com/documentation/swift/using_objective-c_runtime_features_in_swift)
* [링크2]()
* [@selector() in Swift?](https://stackoverflow.com/questions/24007650/selector-in-swift)

-----


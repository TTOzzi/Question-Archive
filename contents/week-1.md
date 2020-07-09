# 야곰닷넷 질문모음 - 1



### Q. alpha, opaque, opacity 의 차이를 모르겠어요.

> alpha, opaque, opacity 의 차이가 무엇인가요?
>
> [질문 바로가기](https://yagom.net/forums/topic/alpha-opaque-opacity의-차이를-명확히-모르겠어요-ㅠㅠ/)

### A.

> - [alpha](https://developer.apple.com/documentation/uikit/uiview/1622417-alpha) : 0.0 ~ 1.0 범위의 값으로, 0.0 은 완전한 투명, 1.0 은 완전한 불투명입니다.
>
>   ++ 이 속성값을 변경하면 변경된 뷰 뿐만 아니라 변경된 뷰의 하위 뷰를 포함 한 모든 콘텐츠에도 영향을 줍니다.
>
> - [isOpaque](https://developer.apple.com/documentation/uikit/uiview/1622622-isopaque) : 뷰가 투명한지 불투명한지 결정하는 Bool 값, true 로 설정하면 drawing 시스템이 뷰를 완전히 불투명하게 처리합니다. -> 성능을 향상시킬 수 있음.
>
>   ++ 뷰의 투명도에 직접적인 영향을 주진 않고 drawing 시스템에게 뷰를 어떻게 처리해야 할지 알려줍니다. true 로 설정해주면 iOS 에서 뷰를 렌더링할 때 뷰가 불투명하다는 것을 가정하고 최적화하여 더 빠르게 렌더링합니다. 뷰에 조금이라도 불투명한 부분이 있다면 false 로 설정해주어야 합니다.
>
> - [opacity](https://developer.apple.com/documentation/quartzcore/calayer/1410933-opacity): 불투명도. 0.0(투명) ~ 1.0(불투명) 범위내에 있어야 하며, 해당 범위를 벗어나는 값은 최소값 또는 최대값으로 고정됩니다. 이 속성의 기본값은 1.0 이며, alpha 와 차이는 alpha 는 UIView 에, opacity 는 CALayer 의 프로퍼티라는 것입니다.

### 참고할 만한 비슷한 질문들
  * [UIView: opaque vs. alpha vs. opacity](https://stackoverflow.com/questions/8520434/uiview-opaque-vs-alpha-vs-opacity)
  * [Is the opacity and alpha the same thing for UIView](https://stackoverflow.com/questions/15381436/is-the-opacity-and-alpha-the-same-thing-for-uiview/15381634)

----

### Q. 특정 앱의 설치 여부를 확인하는 방법이 있나요?

> 페이스북이나 인스타그램 같은 sns 앱들이 기기에 설치되어 있는지 확인할 수 있는 방법이 있을까요?
>
> [질문 바로가기](https://yagom.net/forums/topic/특정-앱이-설치되어-있는지-여부를-확인하는-방법이/)

### A.

> * [canOpenURL(_:)](https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl#discussion) 을 활용한 방법
>
>   ```swift
>   func isInstagramInstalled() -> Bool {
>       return UIApplication.shared.canOpenURL("instagram://app".url())
>   }
>   func isFacebookInstalled() -> Bool {
>       return UIApplication.shared.canOpenURL("fb://".url())
>   }
>   ```
>
>   ++ 매개 변수로 URL 을 받아 해당 URL 을 처리할 수 있는 앱이 있는지를 반환해주는 메소드입니다. **iOS 9.0 이상에서 사용할 때는 앱의 Info.plist 파일에 [LSApplicationQueriesSchemes](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/LaunchServicesKeys.html#//apple_ref/doc/plist/info/LSApplicationQueriesSchemes) 를 추가해주어야 합니다. URLScheme 을 추가하지 않으면 앱의 설치 여부와 관계없이 항상 false 를 반환합니다.**
>
> * SDK 를 활용한 방법 ([Kakao iOS SDK](https://developers.kakao.com/sdk/reference/ios-legacy/release/Classes/KLKTalkLinkCenter.html#//api/name/isAvailableWithError:))
>
>   ```swift
>   func isKakaoInstalled() -> Bool {
>       return KLKTalkLinkCenter.shared().isAvailableWithError()
>   }
>   ```
>
>   ++ 카카오톡뿐만 아니라 [페이스북](https://developers.facebook.com/docs/ios/), [트위터](https://developer.twitter.com/en/docs/developer-utilities/twitter-libraries) 등 다른 sns 앱들도 SDK 나 library 를 지원하고 있으니 찾아보면 다른 방법들도 있을 것 같습니다.

### 참고할 만한 비슷한 질문들

* [How to check app is installed or not in phone](https://stackoverflow.com/questions/41545283/how-to-check-app-is-installed-or-not-in-phone)

----

### Q. 화면 캡쳐를 막는 방법이 있나요?

> 캡쳐를 감지하는 Notification 은 찾았는데 막는 방법을 못 찾겠어요.
>
> ++ 사용자가 캡쳐를 할 때 보내지는 [userDidTakeScreenshotNotification](https://developer.apple.com/documentation/uikit/uiapplication/1622966-userdidtakescreenshotnotificatio)
>
> [질문 바로가기](https://yagom.net/forums/topic/화면-캡쳐를-막는-방법을-알아보고-있는데요-어떻게/)

### A.

> * 어떤 원리를 통해 가능한지는 잘 모르겠지만 [ScreenShieldKit](https://screenshieldkit.com/) 이라는 상용 라이브러리도 있다고 들었습니다. 기본적으로 완전히 막지는 못하고 막아야 할 콘텐츠만 안 보이게 되는 것 같아요. 또, 저 라이브러리로 화면이 영상 녹화되는지도 파악하고 콘텐츠를 안 보이게 할 수 있는 것 같습니다.
> * 영상 녹화 중 안 보이게 하려면 이런 방법을 통해서도 가능할 듯합니다. [Detecting screen capturing in iOS 11](https://medium.com/@abhimuralidharan/detecting-screen-capturing-in-ios-11-cca15881c785)
>
> * ++ 다른 사이트들에서 [Photos 프레임워크](https://developer.apple.com/documentation/photokit)와 userDidTakeScreenshotNotification 을 활용해 유저가 화면을 캡쳐했을 때 사진 앱에 접근해 캡쳐된 사진을 지우는 방법이 있다는 답변이 많아 직접 구현해 보았습니다.
>
>   ```swift
>   NotificationCenter.default.addObserver(forName: UIApplication.userDidTakeScreenshotNotification, object: nil, queue: .main) { _ in
>               let fetchOptions = PHFetchOptions()
>               // 생성 날짜 순으로 정렬
>               fetchOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: false)]
>               
>               // 지정한 옵션으로 정렬된 모든 이미지를 가져옴
>               let fetchResult = PHAsset.fetchAssets(with: .image, options: fetchOptions)
>                                                                                                                                
>               // 정렬된 이미지 중 가장 최근 이미지를 가져옴
>               guard let capturedImage = fetchResult.firstObject else { return }
>               
>               // 삭제 동작 수행
>               PHPhotoLibrary.shared().performChanges({
>                   PHAssetChangeRequest.deleteAssets([capturedImage] as NSFastEnumeration)
>               }) { isSuccess, error in
>                   // 성공, 실패 후 동작 처리
>               }
>           }
>   ```
>
>   그러나 생각했던 것과 달리 notification 을 받은 시점이 아직 사진 앱에 스크린샷이 추가되기 전이고, 사진 앱에 추가된 후라고 하더라도
>
>   <img src="https://user-images.githubusercontent.com/50410213/86616352-acc6fe00-bff0-11ea-8f6e-ebdbeb84bff7.png" width="40%">
>
>   위 사진처럼 매번 사용자에게 권한을 요청하여 캡쳐를 막는다고 보긴 어려웠습니다.

### 참고할 만한 비슷한 질문들

* [Prevent user from taking screenshot of my app like Netflix](https://developer.apple.com/forums/thread/123725)
* [Prevent screen capture in an iOS app](https://stackoverflow.com/questions/18680028/prevent-screen-capture-in-an-ios-app)
* [iOS 11 Screen Recording - Copyright Risks](https://developer.apple.com/forums/thread/86521)

----

### Q. 이미지 애니메이션은 어떻게 구현하나요?

>gif 로드, 애니메이션 프레임 수 만큼 이미지 파일 리스트를 반복으로 보여주기, 코드로 개별 이미지 애니메이션 구현 등 여러 방식을 적용해보고 있는데 여러 개의 이미지를 활용한 애니메이션을 구현할 때의 효율적인 방법이 있나요?
>
>[질문 바로가기](https://yagom.net/forums/topic/이미지-애니메이션-구현하실때-어떤-방식으로-하시/)

### A.

>* iOS 에서 gif 는 [UIImageView](https://developer.apple.com/documentation/uikit/uiimageview) 등의 클래스에서 직접적으로 지원하지 않습니다.
>
>  [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) 등을 통한 간접적인 방법을 사용해야 하므로 효율이 좋지 못하고 성능상의 이점도 없습니다.
>
>* 가장 간단한 방법으로는 UIImageView 클래스의 [animationImages](https://developer.apple.com/documentation/uikit/uiimageview/1621068-animationimages) 프로퍼티를 사용하여 애니메이션에 사용할 이미지를 넣는 방법입니다. [How To Make iOS Animations With UIImageView in Swift](https://marcosantadev.com/make-uiimageview-animations/) 를 참고해보세요. 만약 좀 더 상세하게 제어하고 싶거나 복잡한 애니메이션이 필요하다고 생각하면 라이브러리를 활용하는 것도 좋은 방법입니다.
>
>* 효율을 위해서는 내장되어있는 기능을 사용하는 것이 가장 좋으나 디자이너가 복잡한 애니메이션을 요구하는 경우도 많습니다. 그럴 때 사용할 만한 [lottie](https://lottiefiles.com/) 라는 라이브러리도 있습니다. JSON 형식의 애니메이션을 사용하는데 꽤 직관적이어서 좋습니다. [lottie-ios github 저장소](https://github.com/airbnb/lottie-ios)
>
>* ++ [UIImage](https://developer.apple.com/documentation/uikit/uiimage) 클래스의 [animatedImage(with:duration:)](https://developer.apple.com/documentation/uikit/uiimage/1624149-animatedimage) 메소드를 활용하는 방법도 있습니다.
>
>* ++ 이미지 애니메이션에 한정된 것은 아니지만 좀 더 복잡하고 부드러운 애니메이션을 원한다면 [animateKeyframes(withDuration:delay:options:animations:completion:)](https://developer.apple.com/documentation/uikit/uiview/1622552-animatekeyframes) 나 [Core Animation](https://developer.apple.com/documentation/quartzcore) 을 활용한 여러 가지 방법들이 있습니다.

### 참고할 만한 비슷한 질문들

* [How to animate the change of image in an UIImageView](https://stackoverflow.com/questions/2834573/how-to-animate-the-change-of-image-in-an-uiimageview)
* [Swift - How to animate Images?](https://stackoverflow.com/questions/24364504/swift-how-to-animate-images)
* [Animation using array of images in sequence](https://stackoverflow.com/questions/6040528/animation-using-array-of-images-in-sequence)

----

### Q. 클로저에서 weak self 는 언제 사용하면 되나요?

>```swift
>class Thing {
>  var disposable: Disposable?
>  var total: Int = 0
>  deinit {
>    disposable?.dispose()
>  }
>  init(producer: SignalProducer<Int, NoError>) {
>    disposable = producer.startWithNext { number in
>      self.total += number
>      print(self.total)
>    }
>  }
>}
>```
>
>위 코드에서 클로저는 self 가 해제될 때 까지 기다리고 self 는 클로저가 해제될 때까지 기다리는 strong reference  cycle 상황을 만든다고 하는데 이 부분이 이해가 안됩니다.
>
>```swift
>disposable = producer.startWithNext { [weak self] number in
>  self?.total += number
>  print(self?.total)
>}
>```
>
>위와 같이 클로저 내부에서 self 를 사용하게 되는 모든 경우에서 [weak self] 를 사용하면 되는 것인지도 궁금합니다.
>
>코드 출처 [[Swift] Closure에서 weak self 의 사용](https://greenchobo.tistory.com/3)
>
>[질문 바로가기](https://yagom.net/forums/topic/closure의-weak-self-사용에-대한-질문입니다/)

### A.

>* 우선 위 내용을 이해하려면 스위프트의 ARC 라는 개념과 클로저의 Capture 라는 개념을 이해해야 합니다. <br/>예제 코드의 클로저는 클로저가 생성되는 시점의 인스턴스 상태를 유지하기 위해 클로저 내부의 인스턴스를 모두 획득(capture)해둡니다. 그러기 위해서 **self** 즉, 클로저는 프로퍼티로 갖게 되는 **Thing** 클래스의 인스턴스가 클로저에 의해 획득되어 reference count 가 1 증가합니다. 또, 해당 클로저가 **disposable** 이라는 인스턴스 프로퍼티에 할당되므로 클로저의 reference count 도 1 증가합니다. 서로 reference count 를 올려주기 때문에 순환참조 문제가 발생합니다. <br/>이에 대해 더 알아보고 싶다면 스위프트 언어 가이드 문서 중 [ARC 문서](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html) 를 참고하면 훨씬 도움이 됩니다. 영어가 너무 어렵다면 ARC 및 클로저 값 획득 등의 내용으로 검색해보면 많은 내용을 찾아볼 수 있습니다. [[Swift] Automatic Reference Counting 정리](http://minsone.github.io/mac/ios/swift-automatic-reference-counting-summary)

### 참고할 만한 비슷한 질문들

* [Is it the right way using '[weak self]' in swift closure?](https://stackoverflow.com/questions/54719191/is-it-the-right-way-using-weak-self-in-swift-closure)
* [Where does the weak self go?](https://stackoverflow.com/questions/41991467/where-does-the-weak-self-go)
* [Shall we always use [unowned self] inside closure in Swift](https://stackoverflow.com/questions/24320347/shall-we-always-use-unowned-self-inside-closure-in-swift)


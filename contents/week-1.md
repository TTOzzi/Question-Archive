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


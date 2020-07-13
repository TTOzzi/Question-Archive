# 야곰닷넷 질문모음 - 2



### Q.

> 움직이는 LaunchScreen 을 구현하고 싶어요.

LaunchScreen 에 이미지 슬라이드 애니메이션을 적용하고 싶은데 방법이 있을까요?

 [질문 바로가기](https://yagom.net/forums/topic/object-c에서-런치스크린-질문입니다/)

### A.

* 우선 LaunchScreen 은 동적인 화면이 아니라 정적인 화면 구성만 가능합니다. [Human Interface Guidelines: Launch Screen](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/launch-screen) 에 따르면 Launch Screen 은 앱의 첫 화면과 비슷한 형태여야 합니다. 그리고 앱이 실행되는 중에 잠깐만 보이는 화면이기에 애니메이션 등을 구현하기는 부적합합니다. 예외적으로 게임 앱의 경우, 게임 엔진을 시작하는 데 많은 시간이 소요되므로 로딩 화면을 표시할 수 있습니다.
* 시작 화면에 애니메이션 등이 보이는 것은 꼼수가 필요합니다. 시작화면이 보여진 후에 앱의 메인화면이 나오게 되는데, 그 화면이 시작화면과 동일한 상태로 시작하게 만들고, 거기서 애니메이션을 실행하여 다음 화면으로 이동하는 등의 처리를 합니다.
* Launch Screen 을 xib 로 만들고, 앱이 시작되고 정적인 Launch Screen 보여준 후 다시 불러와 애니메이션을 적용하고 불러온 뷰를 제거하는 방식도 있습니다. [Animate Your iOS Splash Screen](https://www.viget.com/articles/animated-ios-launch-screen/) 을 참고해보세요. 애니메이션 구현 때문에 글이 긴데 애니메이션 구현 부분을 제외하고 LaunchScreenManager 부분만 봐도 좋을듯합니다.
* 옆으로 슬라이드 되는 효과를 알고 싶다면 [UIScrollView](https://developer.apple.com/documentation/uikit/uiscrollview) 에 대해 공부하면 됩니다. [UIView](https://developer.apple.com/documentation/uikit/uiview) 의 애니메이션 관련 메서드와 UIScrollView 의 [setContentOffset(_:animated:)](https://developer.apple.com/documentation/uikit/uiscrollview/1619400-setcontentoffset) 정도만 공부해도 큰 힌트가 될 것 같습니다. [automatic UIScrollView with paging](https://stackoverflow.com/questions/17168741/automatic-uiscrollview-with-paging) 도 확인해보세요.

### 참고할 만한 비슷한 질문들

* [iOS Launch screen code not running](https://stackoverflow.com/questions/27398723/ios-launch-screen-code-not-running)
* [how to add animation to launch screen in iOS 9.3 using Objective c](https://stackoverflow.com/questions/37112950/how-to-add-animation-to-launch-screen-in-ios-9-3-using-objective-c)
* [Difference between launch image and splash screen](https://stackoverflow.com/questions/12140464/difference-between-launxch-image-and-splash-screen)
* [Animated Splash Screen](https://developer.apple.com/forums/thread/110295)
* [Splash screen in iOS games](https://stackoverflow.com/questions/29047522/splash-screen-in-ios-games)


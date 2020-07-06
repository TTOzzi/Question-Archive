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
> - [opacity](https://developer.apple.com/documentation/quartzcore/calayer/1410933-opacity): 불투명도. 0.0(투명)~1.0(불투명) 범위내에 있어야 하며, 해당 범위를 벗어나는 값은 최소값 또는 최대값으로 고정됩니다. 이 속성의 기본값은 1.0이며, alpha와 차이는 alpha는 UIView에, opacity는 CALayer의 프로퍼티라는 것입니다.

### 참고할 만한 비슷한 질문들
  * [UIView: opaque vs. alpha vs. opacity](https://stackoverflow.com/questions/8520434/uiview-opaque-vs-alpha-vs-opacity)
  * [Is the opacity and alpha the same thing for UIView](https://stackoverflow.com/questions/15381436/is-the-opacity-and-alpha-the-same-thing-for-uiview/15381634)


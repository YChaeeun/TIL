# Android Weekly #558 - 안드로이드 개발의 미래

## A Glimpse Into the Future of Android Development

{% embed url="https://molidevwrites.com/a-glimpse-into-the-future-of-android-development/" %}

### Jetpack Compose

\
코틀린 언어 하나로 모든 것을 할 수 있도록 변화해왔음

* 그동안 코틀린이 대체해 온 것
*
  * 자바
  * Groovy (Gradle KTS)

이제 XML 차례

처음에는 러닝커브가 있지만 공부하다보면 익숙해질 것!



### Foldable Devices

아직은 사용자가 많지는 않지만, 나중에는 폴더블 형태의 디바이스들이 늘어날 것

구글에서도 새로운 태블릿을 출시할 텐데, 아마 큰 스크린을 위한 툴과 가이드도 함께 나오게 될 것

폴더블 화면, 화면 분할, 롤러블 기기 화면 대응을 해야 할 수도



[https://dev.to/tkuenneth/foldable-aware-app-layout-fei](https://dev.to/tkuenneth/foldable-aware-app-layout-fei)



### Kotlin Multiplatform Mobile

플러터나 자마린은 한계가 느껴졌다

* 성능
* 센서 접근
* 새로운 언어와 새로운 프레임워크를 새롭게 공부해야 함\


반면 KMM 은 여러 장점이 있다

* 코틀린으로 새로 언어를 공부할 필요가 없음
* iOS 에서도 사용할 수 있어서 중복을 많이 줄일 수 있음
* 성능 이슈 없음
* 네이티브 접근 용이 (ex. 센서)



지금은 공식적으로 stable 하지 않아서 사용하기 두려울 수도 있지만, 꽤 큰 규모의 사용자가 있는 앱에서도 KMM으로 잘 서비스하고 있다



### Managing over-engineering and architecture

모듈화, 클린 아키텍쳐, 싱글 액티비티 패턴 등 여러 아키텍쳐들이 등장하고, 이를 적용하려는 프로젝트들이 많다

물론 장점도 있고, 발전하는 것도 좋지만 작은 앱의 경우에는 너무 과한 경우도 있다

이 흐름이 지나고 나면 오히려 더 단순한 아키텍쳐를 적용하려는 움직임이 있을 것이라고 생각한다


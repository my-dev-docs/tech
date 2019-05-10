# Architecture Components

Android Architecture 컴포넌트는 견고하고 테스트가 가능하며 유지 관리가 쉬운 앱을 설계하는데 도움이 되는 라이브러리 모음입니다. 

UI 컴포넌트 라이프 사이클을 관리하고 데이터 지속성을 처리하는 클래스부터 시작합니다.

* 앱의 라이프 사이클을 쉽게 관리 할 수 있습니다. 새로운 [라이프 사이클 인식 컴포넌트](https://developer.android.com/topic/libraries/architecture/lifecycle)로 액티비티 및 프래그먼트 라이프 사이클을 관리 할 수 있습니다. 생존 구성이 변경되어 메모리 누수가 방지되고 UI에 데이터를 쉽게 로드 할 수 있습니다.
* [LiveData](https://developer.android.com/topic/libraries/architecture/livedata)를 사용하여 기본 데이터베이스가 변경 될 때 뷰에 알리도록 데이터 객체를 작성합니다.
* [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)은 앱 방향 전환 중에 소멸되지 않은 UI 관련 데이터를 저장합니다.
* [Room](https://developer.android.com/topic/libraries/architecture/room)은 SQLite 오브젝트 매핑 라이브러리입니다. 상용구 코드를 피하고 SQLite 테이블 데이터를 Java 오브젝트로 쉽게 변환하는데 사용합니다. Room은 컴파일 타임에 SQLite 문을 검사할 수 있고 RxJava, Flowable 및 LiveData를 반환 할 수도 있습니다.

### 최신 뉴스 및 비디오 <a id="toc_0"></a>

* [Android Jetpack: Improve Your App's Architecture](https://www.youtube.com/watch?v=7p22cSzniBM)
* [Android Jetpack: what's new in Architecture Components \(Google I/O '18\)](https://www.youtube.com/watch?v=pErTyQpA390)
* [Build an App with Architecture Components](https://www.youtube.com/watch?v=BofWWZE1wts)
* [Announcing Architecture Components 1.0 Stable](https://android-developers.googleblog.com/2017/11/announcing-architecture-components-10.html)
* [7 Pro-Tips for Room](https://medium.com/androiddevelopers/7-pro-tips-for-room-fbadea4bfbd1)
* [ViewModels and LiveData: Patterns + AntiPatterns](https://medium.com/google-developers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54) 

### 기타 리소스 <a id="toc_1"></a>

Android 아키텍처 컴포넌트에 대한 자세한 내용은 아래의 다른 리소스를 참고 합니다.

#### 예제 <a id="toc_2"></a>

* [Sunflower](https://github.com/googlesamples/android-sunflower)
  * Android Jetpack의 Android 개발 우수 case를 보여주는 원예 앱입니다.
* [Android Architecture Components basic sample](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)
* [\(more...\)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#samples)

#### 코드랩 <a id="toc_3"></a>

* Android Room with a View [\(Java\)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view) [\(Kotlin\)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin)
* [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)
* [\(more...\)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#codelabs)

#### 트레이닝 <a id="toc_4"></a>

* [Udacity: Developing Android Apps](https://www.udacity.com/course/new-android-fundamentals--ud851)

#### 블로그 <a id="toc_5"></a>

* [Announcing Architecture Components 1.0 Stable](https://android-developers.googleblog.com/2017/11/announcing-architecture-components-10.html)
* [Android and Architecture](https://android-developers.googleblog.com/2017/05/android-and-architecture.html)
* [\(more...\)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#blogs)

#### 비디오 <a id="toc_6"></a>

* [Android Jetpack: what's new in Architecture Components \(Google I/O '18\)](https://www.youtube.com/watch?v=pErTyQpA390)
* [Android Jetpack: easy background processing with WorkManager \(Google I/O '18\)](https://www.youtube.com/watch?v=IrKoBFLwTN0)
* [\(more...\)](https://developer.android.com/topic/libraries/architecture/additional-resources.html#videos)


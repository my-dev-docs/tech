# 프로젝트에 추가하기

시작하기 전에 [Architecture Component Guide](https://developer.android.com/jetpack/docs/getting-started)를 읽어 보는 것이 좋습니다. 이 가이드에는 모든 Android 앱에 대한 몇 가지 유용한 원칙이 포함되어 있으며 아키텍처 컴포넌트를 함께 사용하는 방법이 나와 있습니다.

아키텍처 컴포넌트는 Google의 Maven 저장소에서 사용할 수 있습니다. 이를 사용하려면 저장소를 프로젝트에 추가해야 합니다.

프로젝트의 build.gradle 파일\(앱 또는 모듈의 build.gradle파일이 아님\)을 열고 google\(\) 저장소를 다음과 같이 추가합니다.

```groovy
allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

### 디펜던시 선언 <a id="toc_1"></a>

앱 또는 모듈의 build.gradle 파일을 열고 필요한 아티팩트를 디펜던시로 추가합니다. 모든 아키텍처 컴포넌트를 디펜던시로 추가하거나 필요한 것만 선택해 추가할 수 있습니다. 

각 Architecture Component에 대한 디펜던시 선언에 대한 지침은 릴리스 정보를 참고 합니다.

* [Futures \(found in androidx.concurrent\)](https://developer.android.com/jetpack/androidx/releases/concurrent)
* [Lifecycle components \(including ViewModel\)](https://developer.android.com/jetpack/androidx/releases/lifecycle)
* [Navigation \(including SafeArgs\)](https://developer.android.com/jetpack/androidx/releases/navigation)
* [Paging](https://developer.android.com/jetpack/androidx/releases/paging)
* [Room](https://developer.android.com/jetpack/androidx/releases/room)
* [WorkManager](https://developer.android.com/jetpack/androidx/releases/work)

AndroidX 리팩토링 내용과 AndroidX 리팩토링이 이러한 클래스 패키지 및 모듈 ID에 미치는 영향에 대한 자세한 내용은 [AndroidX 리팩토링 설명서](https://developer.android.com/topic/libraries/support-library/refactor)를 참고 합니다.

### 코틀린 <a id="toc_2"></a>

여러 AndroidX 디펜던시는 Kotlin 확장 모듈을 지원합니다. 이러한 모듈의 이름에는 접미사 "-ktx"가 붙습니다. 예:

```groovy
implementation "androidx.lifecycle:lifecycle-viewmodel:$lifecycle_version"
```

아래처럼 -ktx를 붙입니다.

```groovy
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
```

[ktx 문서](https://developer.android.com/kotlin/ktx)에서 Kotlin 확장에 대한 문서를 포함하여 자세한 정보를 찾을 수 있습니다.

`참고` Kotlin 으로 작성하는 앱의 경우 annotationProcessor 대신 kapt를 사용해야 합니다. kotlin-kapt 플러그인도 추가해야 합니다.


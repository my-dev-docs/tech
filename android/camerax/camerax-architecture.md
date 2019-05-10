---
description: 'https://developer.android.com/training/camerax/architecture 를 번역한 문서입니다.'
---

# CameraX 아키텍처

CameraX는 [Camera2 API](https://developer.android.com/reference/android/hardware/camera2/package-summary) 기능을 보다 쉽게 활용할 수 있도록 Jetpack에 추가 되었습니다. 이 문서에서는 CameraX 아키텍처, API 구조, Lifecycle 사용 방법 및 유즈케이스 결합 방법을 설명합니다.

## CameraX 구조 <a id="toc_1"></a>

개발자는 CameraX를 사용하여 유즈케이스라는 추상화를 통해 디바이스의 카메라와 인터렉션합니다. 현재는 다음 유즈케이스를 사용할 수 있습니다.

* 미리보기: SurfaceTexture 미리보기 준비.
* 이미지 분석: 머신 러닝과 같은 분석을 위해 CPU가 접근할 수 있는 버퍼를 제공.
* 이미지 캡처: 사진 캡처 및 저장.

유즈케이스를 결합하여 동시에 활성화 할 수 있습니다. 예를 들어, 앱은 미리보기 유즈케이스를 사용하여 카메라에서 본 이미지를 사용자가 볼 수 있게 해주고, 이미지 분석 유즈케이스를 사용하여 사진 속 사람이 웃고 있는지 알 수 있으며, 이미지 캡처 유즈케이스를 사용하여 촬형한 사진을 저장할 수 있습니다.

## API 모델 <a id="toc_2"></a>

라이브러리를 사용하려면 다음 내용을 정해줘야 합니다.

* 구성 옵션에 필요한 유즈케이스를 설정.
* 리스너를 연결하여 출력 데이터를 처리.
* [Android 아키텍처 컴포넌트 중 Lifecycle](https://developer.android.com/topic/libraries/architecture)에 유즈케이스를 바인딩하여 카메라 활성 시점 및 데이터 생성 시점과 같은 플로우 설계.

유즈케이스 구성 오브젝트는 set\(\) 메서드를 사용하여 유즈케이스를 설정하고 build\(\) 메서드를 호출하여 구성을 완료합니다. 각 유즈케이스 오브젝트는 유즈케이스 별 API 세트를 제공합니다. 예를 들어, 이미지 캡처 유즈케이스는 tackPicture\(\) 메서드를 제공합니다.

onResume\(\)과 onPause\(\)에서 시작 및 중지 메서드를 호출하는 대신 CameraX.bindToLifecycle\(\)을 사용하여 카메라와 연관된 라이프 사이클을 지정합니다. Android 아키텍처 컴포넌트인 Lifecycle이 데이터의 시작, 종료 그리고 생성을 관리합니다. Lifecycle 컴포넌트는 CameraX에게 카메라 캡처 세션을 구성할 시점을 알려주고 라이프 사이클 전환에 맞게 카메라 상태를 적절하게 변경합니다.

각 유즈케이스의 구현 과정은 [미리보기 구현](https://developer.android.com/training/camerax/preview), [이미지 분석](https://developer.android.com/training/camerax/analyze) 그리고 [사진 촬영](https://developer.android.com/training/camerax/take-photo)을 참고합니다.

## API 모델 예제 <a id="toc_3"></a>

미리보기 유즈케이스는 화면에 표시할 SurfaceTexture를 제공합니다. 앱은 다음 코드처럼 구성 옵션을 사용하여 유즈케이스를 만듭니다.

```kotlin
val previewConfig = PreviewConfig.Builder().build()
val preview = Preview(previewConfig)

val textureView: TextureView = findViewById(R.id.textureView)

// 리스너에서 출력 데이터를 처리합니다.
preview.setOnPreviewOutputUpdateListener { previewOutput ->
    textureView.surfaceTexture = previewOutput.surfaceTexture
}

// 다음 코드를 사용하여 Android 라이프 사이클에 유즈케이스를 바인딩합니다.
CameraX.bindToLifecycle(this as LifecycleOwner, preview)
```

더 많은 예제 코드는 [CameraX 공식 예제 앱](https://github.com/android/camera/CameraXBasic)을 참고합니다.

## CameraX 라이프사이클 <a id="toc_4"></a>

CameraX는 라이프 사이클을 관찰하여 카메라는 켜는 시점, 캡처 세션을 생성할 시점 그리고 중지 및 종료 시점을 결정합니다. 유즈케이스 API는 진행 상황을 모니터 할 수 있도록 메소드와 콜백을 제공합니다.

[유즈케이스 결합](https://developer.android.com/training/camerax/architecture#combine-use-cases)에서 설명한 것처럼 결합된 유즈케이스를 단일 라이프 사이클에 바인딩 할 수 있습니다. 결합할 수 없는 유즈케이스를 앱에서 지원해야 하는 경우 다음 중 하나를 실행할 수 있습니다.

* 호환 가능한 유즈케이스를 여러 [프래그먼트](https://developer.android.com/reference/androidx/fragment/app/Fragment.html)로 나누어 결합한 다음 프래그먼트 간에 전환하기.
* 커스텀 라이프 사이클 컴포넌트를 생성하고 이를 사용하여 카메라 라이프 사이클을 개발자가 직접 제어하기.

카메라 유즈케이스의 라이프 사이클 소유자와 뷰를 분리한 경우\(예를 들어, 커스텀 라이프 사이클 사용 또는 [리테인 프레그먼트](https://developer.android.com/reference/android/app/Fragment.html#setRetainInstance%28boolean%29)를 사용하는 경우\), 모든 유즈케이스가 CameraX.unbindAll\(\)을 사용하여 바인딩을 해제하거나 각각의 유즈케이스를 개별적으로 바인딩 해제해야 합니다. 또는 표준 라이프 사이클을 사용하여 onCreate 메서드에서 유즈케이스를 바인딩하면 CameraX가 캡처 세션을 열고 닫는 것을 관리 할 수 있고 유즈케이스의 바인딩을 해제 할 수 있습니다.

모든 카메라 기능이 AppCompatActivity와 AppCompat Fragment 같은 단일 라이프 사이클 인식 컴포넌트에 해당하는 경우, 해당 컴포넌트의 라이프 사이클을 사용하여 필요한 모든 유즈케이스를 바인딩하면 라이프 사이클 인식 컴포넌트가 활성화 되었을 때 카메라를 사용할 수 있는지 확인할 수 있습니다. 또한 안전하게 해제\(dispose\)되어 리소스를 낭비하지 않을 수 있습니다.

## 커스텀 LifecycleOwners <a id="toc_5"></a>

특별한 경우, 표준 Android LifecycleOwner에 앱을 바인딩 하지 않고 커스텀 LifecycleOwner을 만들어 CameraX 세션 라이프 사이클을 명시적으로 제어할 수 있도록 앱을 만들 수 있습니다.

다음 예제 코드는 간단한 커스텀 LifecycleOwner를 만드는 방법을 보여줍니다.

```kotlin
class CustomLifecycle : LifecycleOwner {
    private val lifecycleRegistry: LifecycleRegistry

    init {
        lifecycleRegistry = LifecycleRegistry(this);
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }
    ...
    fun doOnResume() {
        lifecycleRegistry.markState(State.RESUMED)
    }
    ...
    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }
}
```

이 LifecycleOwner를 사용하면 앱에서 코드의 원하는 지점에서 상태 전환을 배치할 수 있습니다. 앱에서 이 기능을 구현하는 방법에 대한 자세한 내용은 [커스텀 LifecycleOwner 구현하기](https://developer.android.com/topic/libraries/architecture/lifecycle#implementing-lco)를 참고합니다.

## 유즈케이스 결합하기 <a id="toc_6"></a>

여러 유즈케이스를 동시에 실행할 수 있습니다. 유즈케이스를 라이프 사이클에 순서대로 바인딩할 수 있지만 CameraX.bindToLifecycle\(\)을 호출하여 모든 유즈케이스를 한 번에 바인딩하는 것이 좋습니다. 구성 변경에 대한 자세한 내용은 [구성 변경 처리하기](https://developer.android.com/guide/topics/resources/runtime-changes)를 참고합니다.

다음 예제 코드는 두개의 유즈케이스를 만들어 동시에 실행하도록 설정합니다. 그리고 두 개의 유즈케이스에 라이프 사이클을 바인딩하였기 때문에 유즈케이스는 라이프 사이클에 따라 시작하고 중지합니다.

```kotlin
val imageCapture: ImageCapture

override fun onCreate() {
    val previewConfig = PreviewConfig.Builder().build()
    val imageCaptureConfig = ImageCaptureConfiguration.Builder().build()

    val imageCapture = ImageCapture(imageCaptureConfig)
    val preview = Preview(previewConfig)

    val textureView = findViewById(R.id.textureView)

    preview.setOnPreviewOutputUpdateListener { previewOutput ->
        textureView.surfaceTexture = previewOutput.surfaceTexture
    }

    CameraX.bindToLifecycle(this as LifecycleOwner, preview, imageCapture)
}
```

현재 지원하는 구성 조합은 다음과 같습니다. 



| 미리보기 | 이미지분석 | 이미지캡처 | 유즈케이스 결합 |
| :---: | :---: | :---: | :--- |
| O | O | O | 미리보기, 사진 촬영 그리고 이미지 스트림 분석을 사용자에게 제공합니다. |
|  | O | O | 사진 촬영하고 이미지 스트림을 분석합니다. |
| O | O |  | 스트리밍 되는 이미지 분석을 기반으로 만들어진 시각 효과가 적용된 미리보기를 제공합니다. |
| O |  | O | 카메라가 보는 것을 화면에 표시하고 사용자가 사진을 촬영할 수 있습니다. |

`참고` 호환되지 않는 유즈케이스 결합을 작성하면 createCaptureSession\(\)이 처음 호출 될 때 런타임 에러가 발생합니다. 실행중인 세션에 다른 유즈케이스를 추가하는 경우 재구성해야 할 수 도 있습니다. 이로 인해 눈에 보이는 결함이 발생할 수 있습니다.

## 권한 <a id="toc_7"></a>

앱은 `CAMERA` 권한이 필요합니다. 이미지를 파일에 저장한다면 Android Q 이상을 실행하는 디바이스를 제외하고 `WRITE_EXTERNAL_STORATE` 권한이 필요합니다. 

앱의 권한 구성에 대한 자세한 내용은 [앱 권한 요청](https://developer.android.com/training/permissions/requesting)을 참고합니다.

## 요구 사항 <a id="toc_8"></a>

CameraX의 최소 버전 요구 사항은 다음과 같습니다.

* Android API레벨 21
* Android 아키텍처 컴포넌트 1.1.1

라이프 사이클을 인식하는 액티비티의 경우 FragmentActivity 또는 AppCompatActivity를 사용합니다.

## 디펜던시 선언 <a id="toc_9"></a>

CameraX에 디펜던시을 추가하려면 프로젝트에 [Google Maven 저장소](https://developer.android.com/studio/build/dependencies#google-maven)를 추가해야 합니다. 

프로젝트의 build.gradle 파일을 열고 다음과 같이 google\(\) 저장소를 추가합니다.

```text
allprojects {
  repositories {
    google()
    jcenter()
  }
}
```

각 모듈의 build.gradle 파일에 다음을 추가합니다.

```groovy
dependencies {
    // CameraX core library.
    def camerax_version = "1.0.0-alpha01"
    implementation "androidx.camera.core:core:$camerax_version"
    // If you want to use CameraView.
    implementation "androidx.camera.core:view:$camerax_version"
    // If you want to use Camera2 extensions.
    implementation "androidx.camera.core:camera2:$camerax_version"
}
```

이러한 요구 사항을 충족하도록 앱을 구성하는 방법에 대한 자세한 내용은 [디펜던시 선언](https://developer.android.com/jetpack/androidx/releases/lifecycle#declaring_dependencies)을 참고 합니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


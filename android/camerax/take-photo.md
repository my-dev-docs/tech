---
description: 'https://developer.android.com/training/camerax/take-photo 를 번역한 문서입니다.'
---

# 이미지 캡처

이미지 캡처 유즈케이스는 고해상도, 고화질의 사진을 캡처할 수 있게 설계되어 있습니다. 그리고 자동 화이트 밸런스, 자동 노출 및 자동 초점\(3A, auto-white-balance, auto-exposure, auto-focus\) 기능을 제공합니다. 또한 간다한 수동 카메라 제어도 제공합니다.

호출자는 다음 옵션을 포함하여 캡처 한 이미지를 사용하는 방법을 결정해야 합니다.

* `takePicture(OnImageCapturedListener)`
  * 이 메서드는 캡처 된 이미지의 메모리 버퍼를 제공합니다.
* `takePicture(File, OnImageSavedListener)`
  * 이 메서드는 캡처 된 이미지를 제공된 파일 위치에 저장합니다.
* `takePicture(File, OnImageSavedListener, Metadata)`
  * 이 메서드를 사용하면 저장된 파일의 Exif에 포함될 메타데이터를 지정할 수 있습니다.

## 구현하기 <a id="toc_1"></a>

사진을 촬영하기 위한 기본적인 제어 기능을 제공합니다. 플래시 옵션과 연속 자동 초점을 사용하여 사진을 촬영할 수 있습니다.

사진 캡처 지연을 최적화하려면 ImageCapture.CaptureMode를 `MIN_LATENCY`로 설정합니다. 품질을 최적화하려면 `MAX_QUALITY`로 설정합니다.

다음 예제 코드는 사진을 촬영할 수 있도록 앱을 구성하는 방법을 보여줍니다.

```kotlin
val imageCaptureConfig = ImageCaptureConfig.Builder()
  .setTargetRotation(windowManager.defaultDisplay.rotation)
  .build()

val imageCapture = ImageCapture(imageCaptureConfig)

CameraX.bindToLifecycle(this as LifecycleOwner, imageCapture, imageAnalysis, preview)
```

카메라를 구성한 후, 이어지는 코드에서 사용자 액션에 따라 사진을 촬영합니다.

```kotlin
fun onClick() {
    val file = File(...)
    imageCapture.takePicture(file,
        object : ImageCapture.OnImageSavedListener {
            override fun onError(error: ImageCapture.UseCaseError,
                                 message: String, exc: Throwable?) {
                // insert your code here.
            }
            override fun onImageSaved(file: File) {
                // insert your code here.
            }
        })
}
```

이미지 캡처 메서드는 JPEG 형식을 완벽하게 지원합니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


---
description: 'https://developer.android.com/training/camerax/analyze를 번역한 문서입니다.'
---

# 이미지 분석

이미지 분석 유즈케이스는 이미지 프로세싱, 컴퓨터 비전 또는 기계 학습 추론을 실행하기 위해 CPU가 접근 할 수 이미지를 앱에 제공합니다. 앱은 각 프레임에 실행되는 Analyzer 메서드를 구현합니다.

## 구현하기 <a id="toc_0"></a>

ImageProxy 매개변수와 ratation 매개변수를 받는 메서드를 setAnalyzer\(\) 메서드에 전달하여 이미지를 처리합니다. 

다음 예제 코드는 이를 수행하는 방법과 이미지 분석 유즈케이스와 미리보기를 LifecycleOwner에 바인딩하는 방법을 보여줍니다. 미리보기 유즈케이스를 만드는 방법에 대한 자세한 내용은 [미리보기 구현](https://developer.android.com/training/camerax/preview.md)을 참고 합니다.

이 메서드에서 반환할 때 이미지 레퍼런스가 해제됩니다. 따라서 메서드 내에서 분석을 완료하거나 분석 메서드에서 이미지 레퍼런스를 외부로 전달하는 대신 복사를 해야합니다.

```kotlin
val imageAnalysisConfig = ImageAnalysisConfig.Builder()
    .setTargetResolution(Size(1280, 720))
    .build()
val imageAnalysis = ImageAnalysis(imageAnalysisConfig)

imageAnalysis.setAnalyzer({ image: ImageProxy, rotationDegrees: Int ->
    // 여기에 코드를 작성합니다.
})

CameraX.bindToLifecycle(this as LifecycleOwner, imageAnalysis, preview)
```

CameraX는 [`YUV_422_888`](https://developer.android.com/reference/android/graphics/ImageFormat.html#YUV_420_888) 형식으로 이미지를 생성합니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


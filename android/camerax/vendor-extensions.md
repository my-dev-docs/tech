---
description: 'https://developer.android.com/training/camerax/vendor-extensions를 번역한 문서입니다.'
---

# 공급 업체 확장 기능

CameraX는 보케, HDR 및 추가 기능과 같은 디바이스 공급 업체 별 효과에 접근하기 위한 API를 제공합니다. API를 사용하여 현재 디바이스에서 특정 확장 기능을 사용할 수 있는지 여부를 조회하고 그 확장 기능을 우선적으로 사용할 수 있습니다. 즉, 디바이스의 확장 기능을 사용할 수 있는 경우, 해당 확장 기능이 활성화되고 그렇지 않은 경우에는 다운 그레이드됩니다.

공급 업체는 모든 효과와 기능을 구현할 책임이 없습니다. 공급 업체가 구현하지 않은 기능은 CameraX 구현을 사용합니다. 기본 구현에서는 기능을 사용할 수 없다고 보고 이를 사용하지 않도록 설정합니다.

## 이미지 캡처시 효과 사용하기 <a id="toc_1"></a>

공급 업체 확장 기능을 CameraX 유즈케이스에 적용하려면 Extender 오브젝트를 만듭니다. Extender 오브젝트를 통해 효과나 기능 설정을 담도록 Builder를 구성할 수 있습니다. 확장 기능을 사용할 수 있는지 조회하는 것은 중요합니다. 확장 기능을 사용할 수 없는 경우, enableExtension\(\) 호출은 아무 것도 하지 않기 때문입니다.

이미지 캡처 유즈케이스를 위한 확장 기능을 구현하려면 다음 예제 코드처럼 적절한 이미지 캡처 확장을 구현합니다.

```kotlin
import androidx.camera.extensions.BokehExtender

fun onCreate() {
    // 일반 워크플로우와 동일한 빌더를 만듭니다.
    val builder = ImageCaptureConfig.Builder()

    // 확장 구성을 적용하기 위해 Extender 오브젝트를 만듭니다.
    val bokehImageCapture = BokehImageCaptureExtender.create(builder)

    // 확장 기능을 사용할 수 있는지 조회합니다.(선택적으로)
    if (bokehImageCapture.isExtensionAvailable()) {
        // 사용가능하면 확장 기능을 활성화합니다.
        bokehImageCapture.enableExtension()
    }

    // 확장 기능을 사용하지 않을 때와 같은 흐름으로 구성을 완료합니다.
    val config = builder.build()
    val useCase = ImageCapture(config)
    CameraX.bindToLifecycle(this as LifecycleOwner, useCase)
}
```

## 효과 사용하지 않기 <a id="toc_2"></a>

공급 업체 확장 기능을 사용하지 않으려면 이미지 캡처 유즈케이스 또는 미리보기 유즈케이스의 새로운 인스턴스를 만들면 됩니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


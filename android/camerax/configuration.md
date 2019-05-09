---
description: 'https://developer.android.com/training/camerax/configuration 를 번역한 문서입니다.'
---

# 구성

구성 인터페이스 클래스를 사용하여 각각의 CameraX 유즈케이스를 구성합니다. CameraX 유즈케이스 구성을 통해 다양한 측면으로 유즈케이스를 제어할 수 있습니다.

예를 들어, 이미지 캡처 유즈케이스는 사용할 렌즈, 타켓 종횡비 및 스레딩 방법을 설정할 수 있습니다. 다음은 예제 코드입니다.

```kotlin
val config = ImageCaptureConfig.Builder()
    .setLensFacing(...)
    .setFlashMode(...)
    .setTargetAspectRatio(...)
    .build()
```

각 유즈케이스 별 구성에 대한 자세한 내용은 미리보기 구현, 이미지 분석 그리고 사진 촬영을 참고합니다.

### 자동 구성 선택 <a id="toc_1"></a>

CameraX는 앱을 실행하는 디바이스의 기능을 자동으로 구성할 수 있습니다. 예를 들어, 해상도를 지정하지 않거나 지정한 해상도를 디바이스가 지원하지 않는 경우 CameraX는 사용할 최적의 해상도를 자동으로 결정합니다. 이 모든 작업은 뒷단에서 처리되므로 디바이스 별 코드를 작성할 필요가 없습니다.

### 회전 <a id="toc_2"></a>

카메라 회전은 유즈케이스 생성 도중에 디스플레이의 회전과 일치하도록 설정됩니다. 즉, CameraX는 미리보기에서 보는 것과 일치하도록 출력을 생성합니다. 멀티 디스플레이 디바이스를 위해서 유즈케이스 오브젝트를 구성할 때 현재 디스플레이 방향을 전달하는 매개변수에 커스텀 값을 전달하여 회전을 변경할 수 있습니다. 또는 유즈케이스 오브젝트를 생성하고 난 후 동적으로 회전값을 전달하여 회전을 변경할 수도 있습니다.

앱에서 구성 빌더를 사용하여 타켓 회전을 구성할 수 있습니다. 또한 라이프 사이클이 실생 중인 경우에도 유즈케이스 API\(예: ImageAnalysis.setTargetRotation\(\)\)의 메서드를 사용하여 회전 설정을 업데이트 할 수 있습니다. 

앱이 세로 모드로 잠겨 있을 때 이 메서드를 사용할 수 있습니다. 즉, 디바이스 회전시 다시 구성하지 않지만 사진 및 분석 유즈케이스는 디바이스의 현재 회전을 알아야할 경우에 사용할 수 있습니다. 예를 들어, 얼굴 인식을 위해 얼굴의 방향을 바르게 하거나 사진을 가로 또는 세로로 설정하기 위해서 회전 인식이 필요할 수 있습니다.

캡처 된 이미지 데이터는 회전하지 않고 저장되지만 Exif 데이터에는 회전 정보가 포함되어 있습니다. 그래서 갤러리 앱은 저장 후 올바른 방향으로 이미지를 표시 할 수 있습니다. 올바른 방향으로 미리보기 데이터를 표시하려면 Preview.PreviewOutput\(\)의 메타 데이터 출력을 사용할 수 있습니다. 메타 데이터를 사용해 GLSurfaceView 디스플레이를 변형\(transform\) 할 수 있습니다. 

다음 예제 코드는 구성 API를 통해 회전을 설정하는 방법을 보여줍니다.

```kotlin
val previewConfig = PreviewConfig.Builder()
        .setTargetRotation(windowManager.defaultDisplay.rotation)
        .build()
```

설정된 회전에 따라 각 유즈케이스는 이미지 데이터를 직접 회전하거나 회전 되지 않은 이미지 데이터를 사용하는 컨슈머에게 회전 메타 데이터를 제공합니다.

* **미리보기**: GLSurfaceView 디스플레이를 올바르게 변환하기 위해서 Preview.PreviewOutput.getRotationDegrees\(\)를 사용하여 메타 데이터 출력을 제공합니다.
* **이미지 분석**: 디스플레이 좌표를 기준으로 이미지 버퍼 좌표를 알 수 있도록 메타데이터 출력을 제공합니다. analyze\(\) 메서드는 ratationDegrees 매개변수를 제공합니다. 이 것은 이미지 분석 시 이미지를 뷰 파인더와 일치시키기 위한 회전값입니다.
* **이미지 캡처**: 이미지 Exif 메타데이터가 회전 설정을 기록하도록 변경됩니다.

## 카메라 해상도 <a id="toc_3"></a>

디바이스 기능, 디바이스에서 지원하는 [하드웨어 수준](https://developer.android.com/reference/android/hardware/camera2/CameraCharacteristics#INFO_SUPPORTED_HARDWARE_LEVEL), 유즈케이스 그리고 사용 가능한 종횡비의 조합에 따라 이미지 해상도를 설정할 수 있도록 CameraX를 구성할 수 있습니다. 또는 해당 구성을 지원하는 유즈케이스에서 해당 종회비로 특정 타켓 해상도를 설정할 수 있습니다. 타켓 종횡비와 일차하는 해상도가 특정 타켓 해상도보다 우선 순위가 높습니다.

### 자동 해상도 <a id="toc_4"></a>

CameraX는 CameraX.bindToLifecycle\(\)에 지정한 유즈케이스를 바탕으로 최적의 해성도 설정을 자동으로 결정할 수 있습니다. 가능하다면 단일 세션에서 동시에 실행하는 모든 유즈케이스를 CameraX.bindToLifecycle\(\) 를 한 번만 호출하여 모두 바인딩하는 것이 좋습니다. CameraX는 디바이스가 지원하는 하드웨어 수준을 고려하고 디바이스 별 차이점을 고려하여 바인딩 기반 유즈케이스의 해상도를 결정합니다.\(디바이스가 [사용 가능한 스트림 구성](https://developer.android.com/reference/android/hardware/camera2/CameraDevice#createCaptureSession%28android.hardware.camera2.params.SessionConfiguration%29)을 충족시킬 수도 있고 충족하지 못할 수도 있음\) 디바이스 별 코드를 최소화하면서 다양한 디바이스에서 앱을 실행할 수 있도록 하는 것이 목표입니다.

이미지 캡처 및 이미지 분석 유즈케이스의 기본 종횡비는 4: 3입니다.

유즈케이스는 구성 가능한 종횡비를 가지므로 앱은 UI 다지인을 바탕으로 원하는 종횡비를 지정할 수 있습니다. 되도록이면 요청 된 종횡비와 일치하도록 디바이스가 지원하는 종횡비를 선택하여 CameraX 출력을 생성합니다. 정확하게 일치하는 해상도를 디바이스가 지원하지 않으면 가장 많은 조건을 충족하는 해상도가 선택됩니다. 따라서 앱은 앱에서 카메라가 표시되는 방법을 결정하고 CameraX는 서로 다른 장치의 설정을 충족시키는 최적의 카메라 해상도 설정을 결정합니다.

예를 들어, 앱은 다음 중 하나를 수행 할 수 있습니다.

* 이미지 캡처를 위해서 4:3을 사용하도록 레이아웃을 구성합니다.
* 전체 화면으로 레이아웃을 구성합니다. 시스템 UI 막대를 고려하면 편차가 있기에 이 비율은 장치마다 다릅니다.
* 정사각형 레이아웃을 구성합니다.

CameraX는 내부의 Camera2 서페이스 해상도를 자동으로 선택합니다. 다음 표에서는 해상도를 보여줍니다.

| 유즈케이스 | **내부 서페이스 해상도** | **출력 데이터 해상도** |
| :--- | :--- | :--- |
| 미리보기 | **종횡비**: 타겟에 가장 적합한 해상도. | 내부 서페이스 해상도. 메타 데이터를 제공하여 뷰를 타겟 종횡비로 자르고 크기를 조정하고 회전 할 수 있습니다. |
| \*\*\*\* | **기본 해상도**: 가장 높은 미리보기 해상도 또는 위의 종횡비와 일치하는 가장 높은 디바이스 해상도. |  |
| \*\*\*\* | **최대 해상도**: 미리보기 크기\(디바이스 화면 해상도와 일치하는 최적의 크기\) 또는 1080p\(1920x1080\) 중 더 작은 크기 |  |
| 이미지 분석 | **종횡비**: 타겟에 가장 적합한 해상도. | 내부 서페이스 해상도 |
| \*\*\*\* | **기본 해상도**: 기본 타겟 해상도는 640x480으로 설정됨. 타겟 해상도와 해당 종횡비를 조정하면 1080p이하 해상도 중 최상의 해상도를 선택함. |  |
| \*\*\*\* | **최대 해상도**: CameraX에서 1080p로 제한됩니다. 타겟 해상도는 기본 640x480으로 설정되므로 640x480보다 큰 해상도를 원할 경우 [setTargetResolution](https://developer.android.com/reference/kotlin/androidx/camera/core/ImageAnalysisConfig.Builder#settargetresolution) 및 [setTargetAspectRatio](https://developer.android.com/reference/kotlin/androidx/camera/core/ImageAnalysisConfig.Builder#settargetaspectratio)를 사용하여 지원되는 해상도에서 가장 가까운 해상도를 설정해야 합니다. |  |
| 이미지 캡처 | **종횡비**: 설정에 가장 적합한 화면 비율. | 내부 서페이스 해상도 |
| \*\*\*\* | **기본 해상도**: 사용 가능한 가장 높은 해상도 또는 위의 종횡비와 일치하는 가장 높은 디바이스 해상도. |  |
| \*\*\*\* | **최대 해상도**: [StreamConfigurationMap\#getOutputSizes](https://developer.android.com/reference/android/hardware/camera2/params/StreamConfigurationMap#getOutputSizes%28int%29)에서 얻은 값 중 JPEG 타입의 카메라 디바이스 최대 출력 해상도 |  |

### 해상도 지정하기 <a id="toc_5"></a>

다음 예제 코드처럼 유즈케이스 구성을 만들 때 setTargetResolution\(Size resolution\) 메서드를 사용하여 특정 해상도를 설정할 수 있습니다.

```kotlin
val imageAnalysisConfig = ImageAnalysisConfig.Builder()
    .setTargetResolution(Size(1280, 720))
    .build()
```

지정한 해상도에 따라 특정 타켓 종횡비를 설정할 수 있습니다. 타켓 종횡비는 선택한 해상도에 영향을 줍니다. 선택한 해상도를 얻으려면 타켓 종횡비를 타켓 해상도와 일치하도록 설정합니다. 결과 해상도는 디바이스 기능과 연결된 다른 유즈케이스를 고려하여 결정됩니다. 

요청한 해상도와 종횡비를 충족하는 것이 없다면 가장 가까운 해상도가 선택되고 이마저도 없다면 640x480이 선택됩니다.

CameraX는 요청을 바탕으로 가장 적합한 해상도를 적용합니다. 종횡비를 맞추는 것이 가장 큰 요구사항이라면 setTargetAspectRatio만 지정합니다. 그러면 CameraX가 디바이스를 바탕으로 적절한 해상도를 결정합니다. 효율적인 이미지 처리를 위해 해상도 설정이 가장 큰 요구사항이라면 setTargetResolution\(Size resolution\)을 사용합니다.\(디바이스의 처리 능력에 따라 이미지를 작은 크기나 중간 크기로 만드는 일이 필요할 때가 있습니다.\)

`참고` setTargetResolution을 사용하면 종횡비가 다른 유즈케이스와 다른 버퍼를 받을 수 있습니다. 종횡비가 일치해야 하는 경우 두 유즈케이스에서 반환 된 버퍼의 크기를 확인하고 둘 중 하나를 다른 것과 일치시키기 위해서 잘라내거나 크기를 조절해야 합니다.

정확한 해상도가 필요한 경우 createCaptureSession의 표를 참고하여 각 하드웨어 레벨에서 지원하는 최대 해상도를 결정합니다. 현재 디바이스에서 지원하는 특정 해상도를 확인하려면 StreamConfigurationMap.getOutputSize\(int\)를 참고 합니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


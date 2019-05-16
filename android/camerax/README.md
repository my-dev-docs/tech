---
description: 'https://developer.android.com/training/camerax 를 번역한 문서입니다.'
---

# CameraX

`소개영상` [https://www.youtube.com/watch?v=QYkTXJ2TuiA](https://www.youtube.com/watch?v=QYkTXJ2TuiA)

CameraX는 Jetpack 서포트 라이브러리로 카메라 앱을 보다 쉽게 개발할 수 있도록 도와줍니다. Android 5.0\(API레벨 21\)과 호환되는 대부분의 Android 기기에서 작동하는 일관되고 사용하기 쉬운 API를 제공합니다.

CameraX는 camera2의 기능을 활용하지만 라이프 사이클을 식별하는 보다 간단한 유즈케이스 기반 접근 방식을 사용합니다. 또한 디바이스 호환성 문제를 해결하므로 디바이스 별 코드를 작성할 필요가 없습니다. 이러한 점은 앱에 카메라 기능을 추가할 때 작성해야 하는 코드의 양을 줄여줍니다.

마지막으로, CameraX를 사용하면 개발자는 단 두 줄의 코드로 디바이스에 설치된 기본 카메라 앱과 동일한 카메라 경험과 기능을 사용할 수 있습니다. [CameraX Extensions](https://developer.android.com/training/camerax/vendor-extensions)는 옵셔널한 애드온으로 Portrait, HDR, Night, Beauty와 같은 효과를 개발하는 앱에 추가할 수 있습니다. 단, [디바이스](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/camera/extensions/ExtensionsSupportedDevices.md)가 해당 기능을 지원을 해야 합니다.

`코드랩` [https://codelabs.developers.google.com/codelabs/camerax-getting-started](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

`참고` CameraX 라이브러리는 알파 단계로 API가 아직 완성되지 않았습니다. 프로덕션에서는 사용하지 않기를 권장합니다. API가 정식 릴리즈 되기까지 소스 단계에서든 바이너리 단계에서든 호환되지 않을 수 있기 때문에 프로덕션에서는 알파 단계인 라이브러리 사용을 피해야 합니다.

## 주요 이점 <a id="toc_1"></a>

CameraX는 다음과 같은 방법으로 개발자 경험을 향상시킵니다.

### 쉬운 사용성 <a id="toc_2"></a>

CameraX는 디바이스마다 다른 점을 관리하는데 시간을 낭비하지 않습니다. 대신 유즈케이스를 도입하여 필요한 작업에만 집중합니다. 몇 가지 기본적인 유즈케이스가 있습니다.

* [미리보기](https://tech.burt.pe.kr/android/camerax/preview): 디스플레이에서 이미지 얻기
* [이미지 분석](https://tech.burt.pe.kr/android/camerax/analyze-image): MLKit과 같은 알고리즘에 데이터를 전달하기 위한 버퍼에 대한 완전한 접근.
* [이미지 캡처](https://tech.burt.pe.kr/android/camerax/take-photo): 고품질 이미지 저장

이러한 유즈케이스는 Android 5.0\(API레벨 21\) 이상을 실행하는 모든 디바이스에서 작동하므로 대부분의 디바이스에서 동일한 코드를 사용할 수 있습니다.

![&#xADF8;&#xB9BC;1. CameraX&#xB294; Android 5.0\(API&#xB808;&#xBCA8; 21\) &#xC774;&#xC0C1;&#xC744; &#xD0C0;&#xCF13;&#xC73C;&#xB85C; &#xD558;&#xC5EC; &#xB300;&#xBD80;&#xBD84;&#xC758; Android &#xB514;&#xBC14;&#xC774;&#xC2A4;&#xB97C; &#xC9C0;&#xC6D0;&#xD569;&#xB2C8;&#xB2E4;.](../../.gitbook/assets/cx_compatibility.png)

### 여러 디바이스에서 일관성 유지 <a id="toc_3"></a>

여러 앱에서 카메라 동작을 일관되게 관리하는 것은 어렵습니다. 종횡비, 방향, 회전, 미리보기 크기 및 고해상도 이미지 크기 등 많은 점을 고려해야 합니다. CameraX는 이러한 고려사항을 해결하여 쉽게 사용할 수 있습니다.

Google은 Android 5.0\(API레벨 21\) 이후 여러 디바이스 및 운영체제에서 다양한 카메라 동작을 테스트하는 자동화된 CameraX 테스트 연구소에 투자하고 있습니다. 이러한 테스트는 지속적으로 실행되어 광범위한 문제를 확인하고 해결합니다.

우리의 목표는 시간이 지남에 따라 테스트 부담을 크게 줄이는 것입니다.

![&#xADF8;&#xB9BC;2. &#xC790;&#xB3D9;&#xD654; &#xB41C; CameraX &#xD14C;&#xC2A4;&#xD2B8; &#xB7A9;&#xC740; &#xB9CE;&#xC740; &#xB514;&#xBC14;&#xC774;&#xC2A4; &#xD0C0;&#xC785; &#xBC0F; &#xC81C;&#xC870;&#xC5C5;&#xCCB4;&#xC5D0;&#xC11C; &#xC77C;&#xAD00;&#xB41C; API &#xACBD;&#xD5D8;&#xC744; &#xBCF4;&#xC7A5;&#xD569;&#xB2C8;&#xB2E4;.](../../.gitbook/assets/cx_testing-lab.png)

### 새로운 카메라 경험 <a id="toc_4"></a>

CameraX에는 [확장](https://developer.android.com/training/camerax/vendor-extensions)이라고 하는 애드온이 있습니다. 애드온을 사용하면 디바이스와 함께 제공되는 기본 카메라 앱과 동일한 기능을 코드 두 줄로 사용할 수 있습니다.

사용 가능한 첫 번째 기능 세트에는 Portrait, HDR, Night 그리고 Beauty가 있습니다. 이러한 기능을 [지원하는 디바이스](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-master-dev/camera/extensions/ExtensionsSupportedDevices.md)에서만 사용할 수 있습니다.

![ &#xADF8;&#xB9BC;3. CameraX&#xB294; &#xC778;&#xBB3C; &#xC0AC;&#xC9C4;&#xC744; &#xCC0D;&#xC744; &#xB54C; &#xC0AC;&#xC6A9;&#xD558;&#xBA74; &#xC88B;&#xC740; Portrait &#xAC19;&#xC740; &#xC0C8;&#xB85C;&#xC6B4; &#xC778;&#xC571; &#xACBD;&#xD5D8;&#xC744; &#xC81C;&#xACF5;&#xD569;&#xB2C8;&#xB2E4;. &#xADF8;&#xB9BC;3&#xC740; &#xD654;&#xC6E8;&#xC774; Mate 20 Pro&#xC5D0;&#xC11C; CameraX&#xB97C; &#xC0AC;&#xC6A9;&#xD558;&#xC5EC; &#xCC0D;&#xC740; &#xC0AC;&#xC9C4;&#xC785;&#xB2C8;&#xB2E4;.](../../.gitbook/assets/cx_portrait-mode.png)

## 문서 <a id="toc_5"></a>

* [CameraX 아키텍처](https://tech.burt.pe.kr/android/camerax/camerax-architecture)
* [구성 옵션](https://tech.burt.pe.kr/android/camerax/configuration)
* [미리보기 구현](https://tech.burt.pe.kr/android/camerax/preview)
* [이미지 분석](https://tech.burt.pe.kr/android/camerax/analyze-image)
* [이미지 캡처](https://tech.burt.pe.kr/android/camerax/take-photo)
* [제조 업체 확장 기능](https://tech.burt.pe.kr/android/camerax/vendor-extensions)

## 추가 자료 <a id="toc_6"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_7"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_8"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


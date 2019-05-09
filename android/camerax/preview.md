---
description: 'https://developer.android.com/training/camerax/preview 를 번역한 문서입니다.'
---

# 미리보기

미리보기 유즈케이스는 카메라 입력을 스트리밍하는 SurfaceTexture를 생성합니다. 또한 미리보기를 알맞게 표시하기 위해 뷰를 자르고, 크기를 조정하고 회전을 하기 위한 추가 정보도 제공합니다.

카메라가 활성화되면 미리보기 이미지가 SurfaceTexture로 스트리밍됩니다. SurfaceTexture는 TextureView 또는 GLSurfaceView에 연결할 수 있습니다.

## 구현하기 <a id="toc_1"></a>

다음 예제 코드는 PreviewOutput 사용 방법을 보여줍니다.

```kotlin
val previewConfig = PreviewConfig.Builder().build()
val preview = Preview(previewConfig)

preview.setOnPreviewOutputUpdateListener {
    previewOutput: Preview.PreviewOutput? ->
        // 여기에 코드를 작성합니다. 
        // 예를 들어, previewOutput?.getSurfaceTexture() 사용하고 
        // GL 렌더러로 포스트합니다.
}

CameraX.bindToLifecycle(this as LifecycleOwner, preview)
```

## CameraView 사용하기 <a id="toc_2"></a>

CameraX는 미리보기를 사용하여 구현한 CameraView 클래스를 제공합니다. 이 클래스는 보다 편리한 View API 구현을 제공합니다. View API는 이미지 데이터를 자동으로 자르고 크기를 조절하고 회전을 시킵니다.

미리보기에는 토치 모드, 포커스 그리고 줌 컨트롤이 구현되어 있습니다.

## 추가 자료 <a id="toc_10"></a>

CameraX에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

### 코드랩 <a id="toc_11"></a>

* [CameraX 시작하기](https://codelabs.developers.google.com/codelabs/camerax-getting-started)

### 코드 예제 <a id="toc_12"></a>

* [CameraX 공식 예제 앱](https://github.com/android/camera/tree/master/CameraXBasic)


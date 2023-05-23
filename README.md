# FaceFlow - 얼굴 인식 서비스

Module, CameraX Preview, Permission, Google Vision, CustumView(Paint), Bezier Curves, PathMeasure

## CameraX

Android Jetpack 라이브러리 중 하나로, 카메라 개발을 위한 라이브러리.
카메라 기능을 쉽고 간편하게 구현할 수 있도록 돕는 도구로, 다양한 하드웨어 구성에 대해 일관성 있는 인터페이스를 제공한다. (기기별 분기코드가 감소)

### 특징

- **Lifecycle-aware**: CameraX는 lifecycle과 연결되어 있어, 앱의 생명주기 상태에 따라 카메라를 자동으로 시작하거나 종료합니다. 이로 인해 개발자는
  카메라의 상태 관리에 대해 걱정하지 않아도 됩니다.
- **Device Compatibility**: CameraX는 안드로이드 5.0(API 레벨 21) 이상의 90% 이상의 디바이스에서 잘 동작하도록 설계되었습니다. 이
  라이브러리는 다양한 디바이스 사양과 카메라 하드웨어에 대한 일관된 API를 제공합니다.
- **Easy-to-use APIs**: CameraX는 프리뷰, 분석, 사진 촬영 등의 주요 유스케이스에 대한 간단한 API를 제공합니다. 이 API들을 이용하면 개발자는 쉽게
  카메라 기능을 추가하고 커스텀할 수 있습니다.
- **Extensions**: CameraX는 Portrait, HDR, Night 등과 같은 특수한 기능을 활성화하기 위한 확장 API를 제공합니다. 이 기능은 지원하는
  디바이스에서만 사용할 수 있습니다.

### face_recognition 모듈: Camera.kt

```kotlin
class Camera(private val context: Context) {
    private val preview by lazy {
        Preview.Builder()
            .build()
            .also {
                it.setSurfaceProvider(previewView.surfaceProvider)
            }
    }

    private val cameraSelector by lazy {
        CameraSelector.Builder()
            .requireLensFacing(CameraSelector.LENS_FACING_FRONT)
            .build()
    }

    private lateinit var cameraProviderFuture: ListenableFuture<ProcessCameraProvider>
    private lateinit var previewView: PreviewView

    private var cameraExecutor = Executors.newSingleThreadExecutor()

    fun initCamera(layout: ViewGroup) {
        previewView = PreviewView(context)
        layout.addView(previewView)
    }

    private fun openPreview() {
        cameraProviderFuture = ProcessCameraProvider.getInstance(context)
            .also { providerFuture ->
                providerFuture.addListener({

                }, ContextCompat.getMainExecutor(context))
            }
    }

    private fun startPreview(context: Context) {
        val cameraProvider = cameraProviderFuture.get()
        try {
            cameraProvider.unbindAll()
            cameraProvider.bindToLifecycle(
                context as LifecycleOwner,
                cameraSelector,
                preview
            )
        } catch (e: Exception) {
            e.stackTrace
        }
    }
}
```

#### 설명

- `preview` 객체: `Preview.Builder()`를 사용하여 생성. 이는 카메라의 미리보기를 설정하는데 사용된다. `preview` 객체는 생성과
  동시에 `setSurfaceProvider()` 함수를 호출하여 뷰에서 영상을 받아올 수 있도록 설정할 수 있다.
- `cameraSelector` 객체: `CameraSelector.Builder()`를 사용하여 생성되며, 사용할 카메라를 선택하는 데 사용된다. 이 프로젝트에서는 전면카메라를
  사용하므로 빌더 패턴에 `.requireLensFacing(CameraSelector.LENS_FACING_FRONT)`를 추가한다.
- `cameraProviderFuture` 객체: `ProcessCameraProvider`의 인스턴스를 가져올 때 사용된다. 이 객체를 사용하여 카메라의 라이프 사이클을
  관리하고, 미리보기 및 다른 카메라 유스케이스를 바인딩할 수 있다.
    - `ProcessCameraProvider`는 CameraX 라이브러리에서 가장 중요한 클래스 중 하나로, 카메라 기능에 대한 생명주기를 관리하고 카메라 유스케이스(
      미리보기, 사진 촬영, 비디오 촬영 등)를 카메라와 바인딩하는 역할을 담당한다.
    - 또한 사용 가능한 카메라 장치에 대한 정보를 캡슐화하고, 해당 카메라 장치에 유스케이스를 바인딩하는 기능을 제공한다. 시스템 서비스로 작동하며, 동일한 프로세스 내의
      모든 앱 간에 카메라 사용을 조정한다. 이로 인해 여러 앱이 동시에 카메라를 점유하려고 할 때 충돌이 발생하는 것을 방지한다.
- `initCamera()`: 이 함수는 레이아웃에 `PreviewView`를 추가하는데 사용된다. 이 함수를 호출하여 카메라의 미리보기를 화면에 표시할 수 있다.
- `openPreview()`: 이 함수는 `ProcessCameraProvider`의 인스턴스를 가져오는데 사용된다. 이 인스턴스를 사용하여 카메라의 라이프 사이클을 관리하고,
  미리보기 및 다른 카메라 유스케이스를 바인딩하게 된다.
- `startPreview()`: 이 함수는 미리보기를 시작하는 데 사용된다. 이 함수를 호출하여 미리 정의된 `cameraProvider`, `cameraSelector`
  , `preview` 객체를 사용하여 미리보기가 시작된다. 이 함수는 또한 예외를 처리하여 카메라의 바인딩 과정에서 발생할 수 있는 오류를 처리한다.

## Bezier Curves

부드러운 곡선을 그리기 위해 사용.

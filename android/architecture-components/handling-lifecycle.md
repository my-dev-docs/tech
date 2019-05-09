---
description: >-
  https://developer.android.com/topic/libraries/architecture/lifecycle 를 번역한
  문서입니다.
---

# 라이프 사이클 처리

라이프 사이클 인식 컴포넌트는 액티비티 및 프래그먼트와 같은 다른 컴포넌트의 라이프 사이클 상태 변경에 대한 응답으로 작업을 수행합니다. 이러한 컴포넌트를 사용하면 구성이 쉽고 일반적으로 가볍고 유지 관리가 쉬운 코드를 생성 할 수 있습니다.

일반적인 패턴은 액티비티 및 프래그먼트의 라이프 사이클 메소드에 따라 컴포넌트 동작을 구현하는 것입니다. 그러나 이 패턴은 체계적이지 않은 코드 및 에러 확산을 초래합니다. 라이프 사이클 인식 컴포넌트를 사용하면 컴포넌트에 의존하는 코드를 라이프 사이클 메서드에서 컴포넌트 자체로 옮길 수 있습니다.

android.arch.lifecycle 패키지는 라이프 사이클 인식 컴포넌트를 빌드 할 수 있는 클래스와 인터페이스를 제공합니다. 이 컴포넌트는 액티비티 또는 프래그먼트의 현재 라이프 사이클 상태에 따라 자동으로 동작을 조정할 수 있습니다.

`참고` Android 프로젝트에 [android.arch.lifecycle](https://developer.android.com/reference/android/arch/lifecycle/package-summary.html)을 임포트하려면 [라이프 사이클 릴리스 노트](https://developer.android.com/jetpack/androidx/releases/lifecycle#declaring_dependencies)에서 디펜던시를 선언하는 방법을 참고 합니다.

Android 프레임워크에 정의 된 대부분의 앱 컴포넌트에는 라이프 사이클이 첨부되어 있습니다. 라이프 사이클은 운영체제 또는 프로세스에서 실행되는 프레임워크 코드에 의해 관리됩니다. 그들은 안드로이드가 작동하는 방식의 중심에 있으며 앱은 이를 존중해야 합니다. 그렇게 하지 않으면 메모리 누수가 발생하거나 앱이 중단 될 수 있습니다.

디바이스 위치를 화면에 보여주는 액티비티가 있다고 가정 해봅시다. 일반적인 구현은 다음과 같습니다.

```kotlin
internal class MyLocationListener(
        private val context: Context,
        private val callback: (Location) -> Unit
) {

    fun start() {
        // 위치 서비스 시스템에 연결
    }

    fun stop() {
        // 위치 서비스 시스템과 연결 해제
    }
}

class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this) { location ->
            // UI 업데이트
        }
    }

    public override fun onStart() {
        super.onStart()
        myLocationListener.start()
        // 액티비티 라이프 사이클에 응답해야 하는 다른 컴포넌트 관리
    }

    public override fun onStop() {
        super.onStop()
        myLocationListener.stop()
        // 액티비티 라이프 사이클에 응답해야 하는 다른 컴포넌트 관리
    }
}
```

이 예제는 괜찮아 보이지만 실제 앱에서는 라이프 사이클의 현재 상태에 대한 응답으로 UI 및 기타 컴포넌트를 관리하기 위한 호출이 너무 많습니다. 여러 컴포넌트를 관리하면 onStart\(\) 및 onStop\(\)과 같은 라이프 사이클 메서드에 많은 코드가 저장되므로 유지 관리가 어려워집니다.

또한 액티비티 또는 프래그먼트가 정지하기 전에 컴포넌트를 시작한다는 보장은 없습니다. onStart\(\)에서 설정 확인 같은 장시간의 작업을 수행 할 필요가 있는 경우 특히 그렇습니다. onStart\(\) 전에 onStop\(\) 메소드가 종료하고 컴포넌트를 필요 이상으로 유지하는 경쟁 상태가 발생할 수 있습니다.

```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this) { location ->
            // UI 업데이트
        }
    }

    public override fun onStart() {
        super.onStart()
        Util.checkUserStatus { result ->
            // 액티비티가 중단 후 이 콜백이 호출 된다면 어떻게 될까요?
            if (result) {
                myLocationListener.start()
            }
        }
    }

    public override fun onStop() {
        super.onStop()
        myLocationListener.stop()
    }

}
```

android.arch.lifecycle 패키지는 이러한 문제를 유연하고 분리 된 방식으로 해결하는데 도움을 주는 클래스와 인터페이스를 제공합니다.

## Lifecycle <a id="toc_1"></a>

Lifecycle은 액티비티 또는 프래그먼트와 같은 컴포넌트의 라이프 사이클 상태 정보를 유지하면서 다른 오브젝트가 이 상태를 관찰 할 수 있게 하는 클래스입니다.

Lifecycle은 연관련 컴포넌트의 라이프 사이클 상태를 추적하기 위해 두 가지 주요 열거형을 사용합니다.

**이벤트\(Event\)**

* 프레임워크 및 라이프 사이클 클래스에서 전달 된 라이프 사이클 이벤트. 이러한 이벤트는 액티비티 및 프래그먼트의 콜백 이벤트에 매핑됩니다.

**상태\(State\)**

* 라이프 사이클 오브젝트가 추적하는 컴포넌트의 현재 상태입니다.

![](../../.gitbook/assets/lifecycle-states.png)

상태를 그래프의 노드로, 이벤트를 이 노드 사이의 엣지로 생각합니다. 

클래스는 메서드에 어노테이션을 추가하여 컴포넌트의 라이프 사이클 상태를 모니터링 할 수 있습니다. 다음 예제와 같이 Lifecycle 클래스의 addObserver\(\) 메소드를 호출하고 이 때 옵저버의 인스턴스를 전달하여 옵저버를 추가 할 수 있습니다.

```kotlin
class MyObserver : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun disconnectListener() {
        ...
    }
}

myLifecycleOwner.getLifecycle().addObserver(MyObserver())
```

위의 예제에서 myLifecycleOwner 오브젝트는 LifecycleOwner 인터페이스를 구현하고 있습니다. 이 인터페이스는 다음 절에서 설명합니다.

## LifecycleOwner <a id="toc_2"></a>

LifecycleOwner는 클래스에 라이프 사이클이 있음을 나타내는 단일 메서드 인터페이스입니다. 클래스가 구현해야하는 메서드는 getLifecycle\(\) 입니다. 앱 프로세스의 라이프 사이클을 관리하려면 ProcessLifecycleOwner를 참고 합니다.

이 인터페이스는 다양한 클래스\(예: Fragment 및 AppCompatActivity\)에서 라이프 사이클의 소유권을 추상화하고 함께 사용되는 컴포넌트를 작성할 수 있게합니다. LifecycleOwner 인터페이스는 모든 커스텀 앱 클래스에서 구현할 수 있습니다.

LifecycleObserver를 구현하는 컴포넌트는 LifecycleOwner를 구현하는 컴포넌트와 원활하게 작동합니다. 소유자가 라이프 사이클을 제공하고 옵저버가 관찰하기 위해 등록 할 수 있기 때문입니다.

위치 추적 예제의 경우 MyLocationListener 클래스에서 LifecycleObserver를 구현 한 다음 onCreate\(\) 메서드에서 액티비티의 라이프 사이클을 사용하여 초기화 할 수 있습니다. 이렇게 하면 MyLocationListener 클래스가 자급 자족 할 수 있습니다. 즉, 라이프 사이클 상태 변경에 응답하는 로직은 액티비티 대신 MyLocationListener에 선언됩니다. 각 컴포넌트가 자체 로직를 저장하도록하면 액티비티 및 프래그먼트 로직을 보다 쉽게 관리 할 수 있습니다.

```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this, lifecycle) { location ->
            // UI 업데이트
        }
        Util.checkUserStatus { result ->
            if (result) {
                myLocationListener.enable()
            }
        }
    }
}
```

일반적인 유즈케이스는 라이프 사이클이 양호하지 않은 경우 특정 콜백을 호출하지 않도록 하는 것입니다. 예를 들어 액티비티 상태를 저장 한 후 콜백이 프래그먼트 트랜잭션을 실행하면 크래시가 발생하므로 콜백을 호출하지 않습니다.

이 유즈케이스를 쉽게 하기 위해 라이프 사이클 클래스는 다른 오브젝트가 현재 상태를 쿼리 할 수 있게합니다.

```kotlin
internal class MyLocationListener(
        private val context: Context,
        private val lifecycle: Lifecycle,
        private val callback: (Location) -> Unit
) {

    private var enabled = false

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun start() {
        if (enabled) {
            // 연결한다.
        }
    }

    fun enable() {
        enabled = true
        if (lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
            // 연결되지 않은 경우 연결한다.

        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun stop() {
        // 연결되어 있다면 연결을 끊는다.

    }
}
```

이 구현을 통해 LocationListener 클래스는 라이프 사이클을 완전히 인식합니다. 다른 액티비티나 프래그먼트에서 LocationListener를 사용해야 할 경우에는 이를 초기화해야 합니다. 모든 설정 및 해체 작업은 클래스 자체에서 관리합니다.

라이브러리에서 Android 라이프 사이클과 연계 할 필요가 있는 클래스를 제공하는 경우 라이프 사이클 인식 컴포넌트를 사용하는 것이 좋습니다. 라이브러리 클라이언트는 클라이언트 측에서 수동 라이프 사이클 관리를 하지 않고 이러한 컴포넌트를 쉽게 통합 할 수 있습니다.

### 커스텀 LifecycleOwner 구현

Support Library 26.1.0 이후 프래그먼트 및 액티비티는 이미 LifecycleOwner 인터페이스는를 구현하고 있습니다.

LifecycleOwner를 만들려는 커스텀 클래스가 있는 경우 LifecycleRegistry 클래스를 사용할 수 있습니다. 다만 다음 예제 코드와 같이 해당 클래스에 이벤트를 전달해야 합니다.

```kotlin
class MyActivity : Activity(), LifecycleOwner {

    private lateinit var lifecycleRegistry: LifecycleRegistry

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleRegistry = LifecycleRegistry(this)
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }

    public override fun onStart() {
        super.onStart()
        lifecycleRegistry.markState(Lifecycle.State.STARTED)
    }

    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }
}
```

## 라이프 사이클 인식 컴포넌트의 모범 사례 <a id="toc_4"></a>

* UI 컨트롤러\(액티비티 및 프래그먼트\)는 최대한 가볍게 유지합니다. 그들은 자신의 데이터를 얻으려고 시도해서는 안되며 대신 ViewModel을 사용하여 이를 수행하고 LiveData 오브젝트를 관찰하여 변경 사항을 뷰에 반영합니다.
* 데이터가 변경 될 때 뷰를 업데이트하거나 ViewModel에 사용자 액션을 알리는 것이 UI 컨트롤러의 역할로 하는 데이터 기반 UI를 작성합니다.
* 데이터 로직을 ViewModel 클래스에 작성합니다. ViewModel은 UI 컨트롤러와 앱의 다른 부분 간의 커넥터 역할을 합니다. 그러나 ViewModel은 데이터 임포트\(예: 네트워크에서 임포트\)에 대한 책임이 없습니다. 대신 ViewModel은 적절한 컴포넌트를 호출하여 데이터를 가져온 다음 결과를 UI 컨트롤러에 다시 제공해야 합니다.
* 데이터 바인딩을 사용하여 뷰와 UI 컨트롤러 간에 깔끔한 인터페이스를 유지합니다. 이를 통해 뷰를 보다 이해하기 쉽게 만들 수 있습니다. 그리고 액티비티 및 프래그먼트 구현시 필요한 업데이트 코드를 최소화 할 수 있습니다. Java 프로그래밍 언어에서 이를 수행하려면 [Butter Knife](http://jakewharton.github.io/butterknife/)와 같은 라이브러리를 사용하여 상용구 코드를 피하고 더 나은 추상화를 할 수 있습니다.
* UI가 복잡한 경우 UI 변경 사항을 처리 할 수 있는 [프리젠터 ](http://www.gwtproject.org/articles/mvp-architecture.html#presenter)클래스를 만드는 것이 좋습니다. 이것은 어려운 작업 일 수 있지만 UI 컴포넌트를 더 쉽게 테스트 할 수 있습니다.
* ViewModel에서 뷰 또는 액티비티 컨텍스트를 참조하지 마십시오. ViewModel이 액티비티를 오래 유지하면\(구성 변경의 경우\) 액티비티의 메모리가 누수되고 가비지 컬렉터에 의해 제대로 삭제되지 않습니다.
* [Kotlin 코루틴](https://developer.android.com/topic/libraries/architecture/coroutines)을 사용하여 장시간 실행되는 작업 및 비동기로 실행할 수 있는 작업을 관리합니다.

## 라이프 사이클 인식 컴포넌트에 대한 유즈케이스 <a id="toc_5"></a>

라이프 사이클 인식 컴포넌트를 사용하면 다양한 상황에서 라이프 사이클을 보다 쉽게 관리 할 수 있습니다. 다음은 몇 가지 예입니다.

* 큰단위 및 세분화 된 위치 업데이트 사이를 전환합니다. 라이프 사이클 인식 컴포넌트를 사용하면 위치 애플리케이션이 표시 될 때 세분화 된 위치 업데이트를 활성화하고 애플리케이션이 백그라운드에 있을 때 큰단위 업데이트로 전환 할 수 있습니다. LiveData는 사용자가 위치를 변경할 때 앱이 자동으로 UI를 업데이트 할 수 있도록 하는 라이프 사이클 인식 컴포넌트입니다.
* 비디오 버퍼링을 중지하고 시작합니다. 라이프 사이클 인식 컴포넌트를 사용하여 가능한 빨리 비디오 버퍼를 시작하지만 앱이 완전히 부팅 될 때까지 재생을 연기합니다. 또한 앱이 종료 될 때 라이프 사이클 인식 컴포넌트를 사용하여 버퍼링을 종료 할 수 있습니다.
* 네트워크 연결을 시작하고 중지합니다. 라이프 사이클 인식 컴포넌트를 사용하면 앱이 포그라운드에 있을 때 네트워크 데이터를 실시간으로 업데이트\(스트리밍\) 할 수 있으며 앱이 백그라운드로 들어갈 때 자동으로 일시 중지되도록 할 수 있습니다.
* 애니메이션 drawable을 일시 중지했다가 재시작합니다. 라이프 사이클 인식 컴포넌트를 사용하면 앱이 백그라운드에 있는 동안 애니메이션 drawable을 일시 중지 하고 앱이 포그라운드에 있는 경우 drawable을 재시작 할 수 있습니다.

## 정지 이벤트 처리 <a id="toc_6"></a>

라이프 사이클이 AppCompatActivity 또는 Fragment에 속하면 Lifecycle의 상태가 CREATED로 변경되고 AppCompatActivity 또는 Fragment의 onSaveInstanceState\(\)가 호출 될 때 ON\_STOP 이벤트가 전달됩니다.

onSaveInstanceState\(\)를 통해 Fragment 또는 AppCompatActivity의 상태를 저장할 때 해당 UI는 ON\_START를 호출하기 전까지 변경 불가능한 것으로 간주됩니다. 상태를 저장 한 후에 UI를 변경하려고하면 앱의 탐색 상태가 일치하지 않을 수 있습니다. 따라서 상태 저장 후 앱이 프래그먼트 트랜잭션을 실행하면 FragmentManager가 예외를 throw합니다. 자세한 정보는 commit\(\)을 참고 합니다.

옵저버와 연관된 라이프 사이클이 아직 STARTED가 아닌 경우 LiveData는 옵저버를 비활성화하여 이 엣지 조건이 발생하지 않게 합니다. 뒷단에서는 옵저버 호출을 결정하기 전에 isAtLeast\(\)를 호출합니다.

안타깝게도 AppCompatActivity의 onStop\(\) 메서드는 onSaveInstanceState\(\) 이후에 호출됩니다. 따라서 UI 상태 변경은 허용되지 않지만 Lifecycle은 CREATED 상태로 이동하지 않는 갭이 생깁니다.

이 문제를 방지하기 위해 베타2 이전 버전의 라이프 사이클 클래스는 이벤트를 디스패치하지 않고 상태를 CREATED로 표시하여 시스템이 onStop\(\)을 호출 할 때까지 이벤트가 전달되지 않더라도 현재 상태를 확인하는 모든 코드가 실제 값을 가져 오도록합니다.

불행히도,이 솔루션에는 두 가지 큰 문제점이 있습니다.

* API 레벨 23 이하에서는 Android 시스템이 실제로 다른 액티비티에 의해 부분적으로 덮여 있더라도 액티비티 상태를 저장합니다. 즉, Android 시스템은 onSaveInstanceState\(\)를 호출하지만 반드시 onStop\(\)을 호출하지는 않습니다. 이로 인해 잠재적으로 긴 간격이 만들어지며 UI 상태를 수정할 수 없더라도 옵저버는 라이프 사이클을 계속 활성 상태로 간주합니다.
* 비슷한 동작을 LiveData 클래스에 표시하려는 클래스는 Lifecycle 버전 베타 2 이하에서 제공하는 해결 방법을 구현해야 합니다.

`참고` 이 흐름을 더 간단하게 만들고 이전 버전과의 호환성을 향상시키기 위해 lifecycle 오브젝트는 버전 1.0.0-rc1부터 CREATED로 표시되고 onStop\(\)을 기다리지 않고 onSaveInstanceState\(\)가 호출 될 때 ON\_STOP이 전달됩니다. 이렇게 하면 코드에 영향을 미치는 경우가 거의 없지만, API 레벨 26 이하의 액티비티 클래스에 있는 호출 순서와 일치하지 않으므로 이를 주의해야 합니다.

## 추가자료 <a id="toc_7"></a>

라이프 사이클 지원 컴포넌트를 사용하여 라이프 사이클을 처리하는 방법에 대한 자세한 내용은 다음 추가 자료를 참고 합니다.

#### 예제

* [Android Architecture Components Basic Sample](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)
* [Sunflower](https://github.com/googlesamples/android-architecture-components)
  * 안드로이드 아키텍처 컴포넌트 모범 사례를 보여주는 데모 앱

#### 코드랩

* [Android Lifecycle-aware components](https://codelabs.developers.google.com/codelabs/android-lifecycles/index.html?index=..%2F..%2Findex#0)

#### 블로그

* [Introducing Android Sunflower](https://medium.com/androiddevelopers/introducing-android-sunflower-e421b43fe0c2)


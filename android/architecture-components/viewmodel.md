---
description: >-
  https://developer.android.com/topic/libraries/architecture/viewmodel 을 번역한
  문서입니다.
---

# ViewModel

ViewModel 클래스는 라이프 사이클 인식 하여  UI 관련 데이터를 저장 및 관리하도록 설계되었습니다. ViewModel 클래스를 사용하면 화면 회전과 같은 구성 변경 후에도 데이터를 유지할 수 있습니다.

`참고` [ViewModel](https://developer.android.com/reference/androidx/lifecycle/ViewModel.html)을 Android 프로젝트로 임포트하려면 [라이프 사이클 릴리스 노트](https://developer.android.com/jetpack/androidx/releases/lifecycle#declaring_dependencies)에서 디펜던시 선언 방법을 참고 합니다.

Android 프레임워크는 액티비티 및 프래그먼트와 같은 UI 컨트롤러의 라이프 사이클을 관리합니다. 안드로이드 프레임워크는 특정 사용자 동작 또는 제어권 밖에 있는 디바이스 이벤트에 대한 응답으로 UI 컨트롤러를 파괴하거나 다시 만들 것인지 결정할 수 있습니다.

시스템이 UI 컨트롤러를 파괴하거나 다시 만들면 UI 컨트롤러에 저장된 일시적인 UI 관련 데이터가 손실됩니다. 예를 들어 앱의 액티비티 중 하나에 사용자 목록이 포함될 수 있습니다. 구성이 변경될 때 액티비티를 재생성 하면 새로운 액티비티는 사용자 목록을 다시 확보해야 합니다.

간단한 데이터의 경우 onSaveInstanceState\(\) 메서드를 사용하여 onCreate\(\)에서 번들의 데이터를 복원 할 수 있지만 이 메서드는 대량의 데이터가 아니라 직렬화 및 역직렬화 될 수 있는 소량의 데이터에 대해서만 작동합니다. 사용자 목록 또는비트맵 같은 큰 데이터를 저장하기에는 적절치 않습니다.

또 다른 문제는 UI 컨트롤러가 종종 비동기 호출을 하는데 값을 반환하는데 시간이 걸릴 수 있다는 것입니다. UI 컨트롤러는 이러한 호출을 관리하고 잠재적인 메모리 누수를 방지하기 위해 삭제 된 후 시스템이 그들을 확실하게 정리해야 합니다. 이 일에는 많은 유지 관리가 필요하며 구성 변경을 위해 오브젝트를 다시 만드는 경우 오브젝트가 이미 만들어진 호출을 다시 진행해야 하기 때문에 리소스가 낭비됩니다.

액티비티 및 프래그먼트와 같은 UI 컨트롤러는 주로 UI 데이터를 표시하거나 사용자 작업에 반응하거나 권한 요청처럼 OS와의 커뮤니케이션을 처리하는데 사용됩니다. UI 컨트롤러는 데이터베이스 또는 네트워크에서 데이터를 로드해야 하므로 클래스가 점점 뚱뚱해 집니다. UI 컨트롤러에 너무 많은 책임을 할당하면 다른 클래스에게 작업을 위임하지 않고 모든 작업을 단일 클래스에서 자체적으로 처리하려고 할 수 있습니다. 이런 식으로 UI 컨트롤러에 너무 많은 책임을 할당하면 테스트를 만들기가 더 어려워 집니다.

UI 컨트롤러 로직에서 뷰 데이터 소유권을 분리하는 것이 더 쉽고 효율적입니다.

## ViewModel 구현 <a id="toc_1"></a>

아키텍처 컴포넌트는 UI 컨트롤러가 사용할 수 있는 ViewModel 클래스를 제공하여 UI를 위한 데이터 준비를 돕도록 합니다. ViewModel 오브젝트는 구성 변경 중에도 자동으로 유지되므로 저장된 데이터를 다음 액티비티 또는 프래그먼트 인스턴스에서 즉시 사용할 수 있습니다. 예를 들어, 앱에 사용자 목록을 표시해야 하는 경우 아래 예제 코드와 같이 사용자 목록을 가져 와서 액티비티 또는 프래그먼트 대신 ViewModel이 데이터를 유지하도록 책임을 할당할 수 있습니다.

```kotlin
class MyViewModel : ViewModel() {
    private val users: MutableLiveData<List<User>> by lazy {
        MutableLiveData().also {
            loadUsers()
        }
    }

    fun getUsers(): LiveData<List<User>> {
        return users
    }

    private fun loadUsers() {
        // 비동기 작업을 수행하여 사용자 목록을 확보합니다.
    }
}
```

그러면 다음과 같이 액티비티에서 목록에 접근 할 수 있습니다

```kotlin
class MyActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        // 시스템이 액티비티의 onCreate() 메서드를 처음 호출하면 ViewModel이 만들어집니다.
        // 다시 생성된 액티비티는 첫 번째 액티비티에서 생성된 동일한 MyViewModel 인스턴스를 수신합니다.
        val model = ViewModelProviders.of(this).get(MyViewModel::class.java)
        model.getUsers().observe(this, Observer<List<User>>{ users ->
            // UI 업데이트
        })
    }
}
```

액티비티가 재생성되면, 첫 번째 액티비티에 의해 생성 된 동일한 MyViewModel 인스턴스를 받게됩니다. 소유자 액티비티가 완료되면 프레임워크는 ViewModel 오브젝트의 onCleared\(\) 메소드를 호출하여 리소스를 정리합니다.

`주의` ViewModel은 뷰, 라이프 사이클 또는 액티비티 컨텍스트에 대한 레퍼런스를 포함 할 수 있는 어떤 클래스든지 절대로 참조하면 안됩니다.

ViewModel 오브젝트는 뷰 또는 LifecycleOwners의 특정 인스턴스보다 오래 유지되도록 설계되었습니다. 이러한 설계 사용하면 ViewModel이 뷰 및 Lifecycle 오브젝트를 모르기 때문에 ViewModel을 위한 테스트를 보다 쉽게 작성할 수 있습니다. ViewModel 오브젝트에는 LiveData 오브젝트와 같은 LifecycleObservers가 포함될 수 있습니다. 그러나 ViewModel 오브젝트는 LiveData 오브젝트와 같은 라이프 사이클 인식 오브젝트의 변경 사항을 관찰해서는 안됩니다. ViewModel이 시스템 서비스를 찾는 경우처럼 앱 컨텍스트가 필요한 경우 ViewModel 클래스는 AndroidViewModel 클래스를 확장한 것이기 때문에 생성자에서 Application을 받는 생성자를 가질 수 있습니다.\(Application 클래스가 Context를 확장하므로\)

## ViewModel 라이프 사이클 <a id="toc_2"></a>

ViewModel을 얻을 때 ViewModel 오브젝트의 스코프는 ViewModelProvider에 전달 된 라이프 사이클로 제한됩니다. ViewModel은 라이프 사이클의 스코프가 영구적으로 사라질 때까지 메모리에 남아 있습니다: 액티비티의 경우에는 finished 일 때 메모리에서 정리되고 프래그먼트의 경우에는 detached 일 때 메모리에서 정리됩니다.

그림 1은 액티비티가 회전 한 후 종료하였을 때 발생하는 라이프 사이클 상태를 보여줍니다. 액티비티 라이프 사이클 옆에 ViewModel의 수명이 그려져 있습니다.이 특정 다이어그램은 액티비티의 상태를 나타냅니다. 동일한 기본 상태가 프래그먼트의 라이프 사이클에도 적용됩니다.

![](../../.gitbook/assets/viewmodel-lifecycle.png)

시스템은 먼저 액티비티 오브젝트의 onCreate\(\) 메소드를 호출 할 때 ViewModel을 요청합니다. 시스템은 디바이스 화면을 회전 할 때처럼 액티비티의 라이프 사이클 동안 onCreate\(\)를 여러 번 호출 할 수 있습니다. ViewModel은 처음 ViewModel이 요청된 시점부터 액티비티가 완료되고 소멸되는 시점까지 유지됩니다.

## 프래그먼트 간 데이터 공유 <a id="toc_3"></a>

단일 액티비티 내에서 두 개 이상의 프래그먼트가 서로 통신하는 것은 일반적입니다. 마스터/디테일 프래그먼트의 일반적인 경우를 생각해 봅시다.. 거기에서는 사용자가 목록에서 아이템을 선택하는 프래그먼트와 선택된 아이템의 내용을 표시하는 다른 프래그먼트가 있습니다. 두 프래그먼트는 인터페이스를 정의해야하고 소유자 액티비티느 인터페이스를 통해 두 프래그먼트를 함께 바인딩해야하기 때문에 결코 쉬운 일이 아닙니다. 또한 두 프래그먼트는 아직 생성되지 않았거나 표시되지 않은 다른 프래그먼트도 처리해야 합니다.

ViewModel 오브젝트를 사용하여 이 일반적인 고통을 해결할 수 있습니다. 이러한 프래그먼트는 아래 예제 코드와 같이 액비티비의 스코프를 사용하는 ViewModel을 공유하여 프래그먼트간에 커뮤니케이션을 처리할 수 있습니다. 

```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()

    fun select(item: Item) {
        selected.value = item
    }
}

class MasterFragment : Fragment() {

    private lateinit var itemSelector: Selector

    private lateinit var model: SharedViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        model = activity?.run {
            ViewModelProviders.of(this).get(SharedViewModel::class.java)
        } ?: throw Exception("Invalid Activity")
        itemSelector.setOnClickListener { item ->
            // UI 업데이트
        }
    }
}

class DetailFragment : Fragment() {

    private lateinit var model: SharedViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        model = activity?.run {
            ViewModelProviders.of(this).get(SharedViewModel::class.java)
        } ?: throw Exception("Invalid Activity")
        model.selected.observe(this, Observer<Item> { item ->
            // UI 업데이트
        })
    }
}
```

두 프래그먼트 모두 자신을 포함하고 있는 액티비티를 검색합니다. 따라서 각 프래그먼트가 ViewModelProvider를 가져 오면 동일한 SharedViewModel 인스턴스를 받으며 인스턴스의 스코프는 이 액티비티로 제한됩니다. 

이 방법에는 다음과 같은 장점이 있습니다.

* 액티비티는 아무 것도 할 필요가 없으며 이 커뮤니케이션을 이해할 필요가 없습니다.
* 프래그먼트는 서로를 알 필요가 없습니다. SharedViewModel 만 알면 됩니다. 프래그먼트 중 하나가 사라져도 다른 프래그먼트는 평소와 같이 계속 작동합니다.
* 각 프래그먼트는 다른 프래그먼트의 라이프 사이클에 관계 없이 자체 라이프 사이클을 가지고 있습니다. 프래그먼트 하나가 다른 프래그먼트로 바뀌어도 UI는 아무런 문제 없이 계속 작동합니다.

## ViewModel로 로더 교체하기 <a id="toc_4"></a>

CursorLoader와 같은 로더 클래스는 종종 앱의 UI 데이터를 데이터베이스와 동기화하는데 사용됩니다. 로더를 ViewModel로 바꿀 수 있습니다. ViewModel을 사용하면 UI 컨트롤러와 데이터로드 작업이 분리되므로 클래스 간의 참조가 줄어 듭니다.

로더를 사용하는 일반적인 방법으로 앱은 CursorLoader를 사용하여 데이터베이스의 내용을 관찰 할 수 있습니다. 데이터베이스의 값이 변경되면 로더가 자동으로 데이터를 다시 로드하고 UI를 업데이트합니다.

![&#xADF8;&#xB9BC; 2. &#xB85C;&#xB354;&#xB97C; &#xC0AC;&#xC6A9;&#xD558;&#xC5EC; &#xB370;&#xC774;&#xD130; &#xB85C;&#xB529;&#xD558;&#xAE30;](../../.gitbook/assets/viewmodel-loader.png)

ViewModel은 Room과 LiveData를 함께 사용하여 로더를 대체합니다. ViewModel은 디바이스 구성이 변경된 후에도 데이터가 유지 되도록 합니다. 데이터베이스가 변경되면 Room은 LiveData에게 알리고 LiveData는 수정 된 데이터로 UI를 업데이트합니다.

![&#xADF8;&#xB9BC; 3. ViewModel&#xC744; &#xC0AC;&#xC6A9;&#xD558;&#xC5EC; &#xB370;&#xC774;&#xD130; &#xB85C;&#xB4DC; &#xD558;&#xAE30;](../../.gitbook/assets/viewmodel-replace-loader.png)

## ViewModel과 함께 코루틴 사용하기 <a id="toc_5"></a>

ViewModel은 Kotlin 코루틴을 지원합니다. 자세한 내용은 [Android 아키텍처 컴포넌트에 Kotlin 코루틴 사용하기](https://developer.android.com/topic/libraries/architecture/coroutines)를 참고 합니다.

## 추가 정보 <a id="toc_6"></a>

데이터가 더 복잡 해짐에 따라 데이터를 로드하기 위해 별도의 클래스를 선택할 수 있습니다. ViewModel의 목적은 UI 컨트롤러의 데이터를 캡슐화하여 데이터가 구성 변경에 영향을 받지 않도록하는 것입니다. 구성 변경 사항을 통해 데이터를 로드, 유지 및 관리하는 방법에 대한 자세한 내용은 [UI 상태 저장](https://developer.android.com/topic/libraries/architecture/saving-states.html)을 참고 합니다.

[Android 앱 아키텍처 가이드](https://developer.android.com/topic/libraries/architecture/guide.html#fetching_data)에서는 이러한 기능을 처리 할 수 있는 저장소\(repository\) 클래스를 만들 것을 제안합니다.

## 추가 자료 <a id="toc_7"></a>

ViewModel 클래스에 대한 자세한 내용은 다음 자료를 참고 합니다.

### 예제 <a id="toc_8"></a>

* [Android Architecture Components basic sample](https://github.com/googlesamples/android-architecture-components/tree/master/BasicSample)
* [Sunflower](https://github.com/googlesamples/android-sunflower), a gardening app illustrating Android development best practices with Android Jetpack.

### 코드랩 <a id="toc_9"></a>

* Android Room with a View [\(Java\)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view) [\(Kotlin\)](https://codelabs.developers.google.com/codelabs/android-room-with-a-view-kotlin)
* [Android lifecycle-aware components codelab](https://codelabs.developers.google.com/codelabs/android-lifecycles/#0)

### 블로그 <a id="toc_10"></a>

* [ViewModels : A Simple Example](https://medium.com/androiddevelopers/viewmodels-a-simple-example-ed5ac416317e)
* [ViewModels: Persistence, onSaveInstanceState\(\), Restoring UI State and Loaders](https://medium.com/androiddevelopers/viewmodels-persistence-onsaveinstancestate-restoring-ui-state-and-loaders-fc7cc4a6c090)
* [ViewModels and LiveData: Patterns + AntiPatterns](https://medium.com/androiddevelopers/viewmodels-and-livedata-patterns-antipatterns-21efaef74a54)
* [Kotlin Demystified: Understanding Shorthand Lambda Syntax](https://medium.com/androiddevelopers/kotlin-demystified-understanding-shorthand-lamba-syntax-74724028dcc5)
* [Kotlin Demystified: Scope functions](https://medium.com/androiddevelopers/kotlin-demystified-scope-functions-57ca522895b1)
* [Kotlin Demystified: When to use custom accessors](https://medium.com/androiddevelopers/kotlin-demystified-when-to-use-custom-accessors-939a6e998899)
* [Lifecycle Aware Data Loading with Architecture Components](https://medium.com/google-developers/lifecycle-aware-data-loading-with-android-architecture-components-f95484159de4)

### 비디오 <a id="toc_11"></a>

* [Android Jetpack: ViewModel](https://www.youtube.com/watch?v=5qlIPTDE274&t=30s)


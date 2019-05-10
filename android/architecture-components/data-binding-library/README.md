# 데이터 바인딩 라이브러리

데이터 바인딩 라이브러리는 프로그래밍 방식이 아닌 선언 방식으로 레이아웃의 UI 컴포넌트를 앱의 데이터 소스에 바인딩 할 수 있는 지원 라이브러리입니다.

레이아웃은 대개 UI 프레임워크 메소드 호출 코드가 있는 액티비티에서 정의됩니다. 예를 들어, 다음 코드는 findViewById\(\)를 호출하여 TextView 위젯을 찾고 이를 viewModel 변수인 userName 속성에 바인딩합니다.

```kotlin
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}
```

다음 예제는 데이터 바인딩 라이브러리를 사용하여 텍스트를 레이아웃 파일의 위젯에 직접 할당하는 방법을 보여줍니다. 따라서 위에 표시된 Kotlin 코드를 호출 할 필요가 없습니다. 대입 표현식에 @{} 문법을 사용합니다.

```markup
<TextView
    android:text="@{viewmodel.userName}" />
```

레이아웃 파일 바인딩 컴포넌트를 사용하면 액티비티에서 많은 UI 프레임워크 호출 코드을 제거 할 수 있으므로 유지 관리가 더 쉽고 간단 해집니다. 또한 앱의 성능을 향상시키고 메모리 누수 및 null 포인터 예외를 방지 할 수 있습니다.

### 데이터 바인딩 라이브러리 사용하기 <a id="toc_0"></a>

다음 페이지에서 Android 앱에서 데이터 바인딩 라이브러리를 사용하는 방법에 대해 알아보십시오.

#### [시작하기](https://developer.android.com/topic/libraries/data-binding/start.html) <a id="toc_1"></a>

Android Studio에서 데이터 바인딩 코드 지원을 비롯하여 개발 환경에서 데이터 바인딩 라이브러리를 사용할 수 있도록 만드는 방법에 대해 알아보십시오.

#### [레이아웃 및 바인딩 표현식](https://developer.android.com/topic/libraries/data-binding/expressions.html) <a id="toc_2"></a>

표현식 언어를 사용하면 레이아웃의 뷰에 변수를 연결하는 표현식을 작성할 수 있습니다. 데이터 바인딩 라이브러리는 레이아웃의 뷰를 데이터 오브젝트에 바인딩하는데 필요한 클래스를 자동으로 생성합니다. 이 라이브러리는 레이아웃에서 사용할 수 있는 임포트, 변수 및 포함\(include\)과 같은 기능을 제공합니다.

라이브러리의 이러한 기능은 기존 레이아웃과 완벽하게 공존합니다. 예를 들어 표현식에서 사용할 수 있는 바인드 변수는 UI 레이아웃의 루트 엘리먼트의 형제인 data 엘리먼트 내에 정의됩니다. 다음 엘리먼트와 같이 이 두 엘리먼트가 모두 layout 태그에 포함됩니다.

```markup
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!-- UI 레이아웃 루트 엘리먼트 -->
</layout>
```

#### [Observable 데이터 오브젝트 사용](https://developer.android.com/topic/libraries/data-binding/observability.html) <a id="toc_3"></a>

데이터 바인딩 라이브러리는 데이터의 변경 사항에 대한 데이터를 쉽게 관찰 할 수 있는 클래스와 메서드를 제공합니다. 기본 데이터 소스가 변경 될 때 UI 새로 고침에 대해 걱정할 필요가 없습니다. 변수 또는 속성을 관찰할 수 있습니다. 이 라이브러리를 사용하면 오브젝트, 필드 또는 콜렉션을 Observable하게 만들 수 있습니다.

#### [생성 된 바인딩 클래스](https://developer.android.com/topic/libraries/data-binding/generated-binding.html) <a id="toc_4"></a>

데이터 바인딩 라이브러리는 레이아웃의 변수 및 뷰에 접근하기 위한 바인딩 클래스를 생성합니다. 이 페이지는 생성 된 바인딩 클래스를 사용하고 커스텀하는 방법을 보여줍니다.

#### [어댑터 바인딩](https://developer.android.com/topic/libraries/data-binding/binding-adapters.html) <a id="toc_5"></a>

각 레이아웃 표현식에는 적절한 속성이나 리스너를 설정하는데 필요한 프레임워크 호출을 만드는 바인딩 어댑터가 있습니다. 예를 들어, 바인딩 어댑터는 setText\(\) 메서드를 호출하여 text 속성을 설정하거나 setOnClickListener\(\) 메서드를 호출하여 click 이벤트에 대한 리스너를 추가 할 수 있습니다. 이 페이지의 예제에 사용 된 android:text 속성 어댑터와 같은 가장 일반적인 바인딩 어댑터는 android.databinding.adapters 패키지에서 사용할 수 있습니다. 일반적으로 사용되는 바인딩 어댑터 목록은 [어댑터](https://android.googlesource.com/platform/frameworks/data-binding/+/studio-master-dev/extensions/baseAdapters/src/main/java/androidx/databinding/adapters)를 참고 합니다. 다음 예제와 같이 커스텀 어댑터를 작성할 수도 있습니다.

```kotlin
@BindingAdapter("app:goneUnless")
fun goneUnless(view: View, visible: Boolean) {
    view.visibility = if (visible) View.VISIBLE else View.GONE
}
```

#### [아키텍처 컴포넌트에 레이아웃 뷰 바인딩](https://developer.android.com/topic/libraries/data-binding/architecture.html) <a id="toc_6"></a>

Android 지원 라이브러리에는 강력하고 테스트 가능하며 유지 관리가 쉬운 애플리케이션을 설계하는데 사용할 수 있는 [아키텍처 컴포넌트](https://developer.android.com/topic/libraries/architecture/index.html)가 포함되어 있습니다. 데이터 바인딩 라이브러리와 함께 아키텍처 컴포넌트를 사용하여 UI 개발을 더욱 단순화 할 수 있습니다.

#### [양방향 데이터 바인딩](https://developer.android.com/topic/libraries/data-binding/two-way) <a id="toc_7"></a>

데이터 바인딩 라이브러리는 양방향 데이터 바인딩을 지원합니다. 이러한 바인딩에 사용되는 표기법은 속성에 대한 데이터 변경 사항을 수신하고 해당 속성에 대한 사용자 업데이트를 동시에 수신하는 기능을 지원합니다.


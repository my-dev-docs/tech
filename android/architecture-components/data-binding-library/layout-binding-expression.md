# 레이아웃 및 바인딩 표현식

표현식 언어를 사용하면 뷰에서 전달한 이벤트를 처리하는 표현식을 작성할 수 있습니다. 데이터 바인딩 라이브러리는 레이아웃의 뷰를 데이터 오브젝트에 바인딩하는데 필요한 클래스를 자동으로 생성합니다.

데이터 바인딩 레이아웃 파일은 약간 다릅니다. layout 루트 태그로 시작하고 data 엘리먼트와 뷰 루트 엘리먼트가 계속됩니다. 이 뷰 엘리먼트는 루트가 아닌 바인딩 레이아웃 파일에 있는 것입니다. 다음 코드는 예제 레이아웃 파일을 보여줍니다.

```markup
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

data의 user 변수를 레이아웃에서 사용할 수 있는 속성으로 선언합니다.

```markup
<variable name="user" type="com.example.User" />
```

레이아웃의 표현식은 "@{}"문법을 사용하여 어트리뷰트 속성에 기록됩니다. 여기에서 TextView의 text는 user 변수의 firstName 속성으로 설정됩니다.

```markup
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}" />
```

`참고` 레이아웃 표현식은 유닛 테스트가 불가능하고 IDE 지원이 제한되어 작고 단순해야 합니다. 레이아웃 표현식을 단순화하기 위해 커스텀 바인딩 어댑터를 사용할 수 있습니다.

### 데이터 오브젝트 <a id="toc_1"></a>

이제 User 엔티티를 설명하는 오브젝트가 있다고 가정합니다.

```kotlin
data class User(val firstName: String, val lastName: String)
```

이 타입의 오브젝트에는 변경되지 않는 데이터가 있습니다. 앱에서 일반적으로 읽히기만 하고 변경되지 않는 데이터입니다. 다음 예제와 같이 Java의 접근자 메서드 사용과 같은 일련의 규칙에 따라 오브젝트를 사용할 수도 있습니다.

```java
public class User {
  private final String firstName;
  private final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
  public String getFirstName() {
      return this.firstName;
  }
  public String getLastName() {
      return this.lastName;
  }
}
```

데이터 바인딩 관점에서 이 두 클래스는 동일합니다. android:text 속성에 대한 @{user.firstName} 표현식은 이전 클래스의 firstName 필드와 후자 클래스의 getFirstName\(\) 메소드에 접근합니다. 또는 메서드가 있다면 firstName\(\)도 확인됩니다.

### 데이터 바인딩 <a id="toc_2"></a>

각 레이아웃 파일에 대한 바인딩 클래스를 생성합니다. 클래스 이름은 파스칼 대소문자로 변환 된 레이아웃 파일의 이름을 기반으로하며 Binding 접미사가 추가됩니다. 위의 레이아웃 파일 이름은 activity_main.xml이므로 해당 빌드 클래스는 ActivityMainBinding입니다. 위의 레이아웃 파일 이름은 activity_main.xml이므로 해당 빌드 클래스는 ActivityMainBinding입니다. 이 클래스에는 레이아웃 속성\(예: user 변수\)에 대한 모든 바인딩이 포함되어 있으며 바인딩 표현식의 값을 지정하는 방법을 알고 있습니다. 바인딩을 만드는 데 권장되는 방법은 다음 예제와 같이 레이아웃을 인플래이션 할 때 이 작업을 수행하는 것입니다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil.setContentView(
            this, R.layout.activity_main)

    binding.user = User("Test", "User")
}
```

런타임에 앱은 UI에 Test user를 표시합니다. 또는 다음 예제와 같이 LayoutInflater를 사용하여 뷰를 가져올 수 있습니다. 

```kotlin
val binding: ActivityMainBinding = ActivityMainBinding.inflate(getLayoutInflater())
```

Fragment, ListView 또는 RecyclerView 어댑터에서 데이터 바인딩을 사용하는 경우 다음 코드 예제와 같이 바인딩 클래스 또는 DataBindingUtil 클래스의 inflate\(\) 메서드를 사용하는 것이 좋습니다.

```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

### 표현식 언어 <a id="toc_3"></a>

표현식 언어는 관리되는 코드의 표현식과 매우 비슷합니다. 표현식 언어에서 다음 연산자와 키워드를 사용할 수 있습니다.

* Mathematical + - / \* %
* String concatenation +
* Logical && \|\|
* Binary & \| ^
* Unary + - ! ~
* Shift &gt;&gt; &gt;&gt;&gt; &lt;&lt;
* Comparison == &gt; &lt; &gt;= &lt;= \(Note that &lt; needs to be escaped as `&lt;`\)
* instanceof
* Grouping \(\)
* Literals - character, String, numeric, null
* Cast
* Method calls
* Field access
* Array access \[\]
* Ternary operator ?:

예제:

```markup
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```

#### 미지원 연산자 <a id="toc_4"></a>

관리 코드에서 사용할 수 있는 표현식 문법 중 다음 연산은 지원하지 않습니다.

* this
* super
* new
* 명시적인 제네릭 호출

#### null 병합 연산자 <a id="toc_5"></a>

null 병합 연산자\(??\)는 피연산자가 null이 아니면 왼쪽 피연산자를 선택하고 이전 피연산자가 null 이면 오른쪽 피연산자가 선택됩니다. 

```markup
android:text="@{user.displayName ?? user.lastName}"
```

이 기능은 다음과 같습니다.

```text
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

#### 속성 레퍼런스 <a id="toc_6"></a>

표현식은 필드, getter 및 ObservableField 오브젝트에 대해 다음 포맷을 사용하여 클래스의 속성을 레퍼런스 할 수 있습니다.

```markup
android:text="@{user.lastName}"
```

#### null 포인터 예외 회피하기 <a id="toc_7"></a>

생성 된 데이터 바인딩 코드는 자동으로 null 값을 검사하여 null 포인터 예외를 피합니다. 예를 들어, @{user.name} 표현식에서 user가 null 인 경우 user.name에 기본값 null이 지정됩니다. age 타입이 int 인 user.age를 레퍼런스하는 경우 데이터 바인딩은 기본값 0을 사용합니다.

#### 컬렉션 <a id="toc_8"></a>

\[\] 연산자를 사용하여 배열, 리스트, sparse 리스트 및 map과 같은 공용 컬렉션에 접근 할 수 있습니다.

```markup
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```

`참고` XML을 문법적으로 올바르게 만들려면 `<` 문자를 이스케이프 처리해야 합니다. 예를 들어 List `<String>` 대신 List `&lt;String>`으로 작성해야 합니다.

object.key 표기법을 사용하여 map의 값을 레퍼런스 할 수도 있습니다. 예를 들어 위의 예제에서 @{map\[key\]}는 @{map.key}로 바꿀 수 있습니다.

#### 문자열 리터럴 <a id="toc_9"></a>

다음 예제와 같이 속성 값을 작은 따옴표로 묶어 표현식에 큰 따옴표를 사용할 수 있습니다. 

```markup
android:text='@{map["firstName"]}'
```

속성 값을 둘러싸기 위해 큰 따옴표를 사용할 수도 있습니다. 이렇게 하면 문자열 리터럴은 백틱\(\`\)으로 묶어야 합니다.

```markup
android:text="@{map[`firstName`]}"
```

#### 리소스 <a id="toc_10"></a>

다음 문법을 사용하여 표현식에서 리소스에 접근 할 수 있습니다.

```markup
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

포멧 문자열 및 복수 매개변수를 제공할 수 있습니다. 

```markup
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

복수가 여러 매개변수를 받는 경우 모든 매개변수를 전달해야 합니다.

```markup

  Have an orange
  Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

일부 리소스는 다음 표와 같이 명시적 타입 평가가 필요합니다.

| Type | Normal reference | Expression reference |
| :--- | :--- | :--- |
| String\[\] | @array | @stringArray |
| int\[\] | @array | @intArray |
| TypedArray | @array | @typedArray |
| Animator | @animator | @animator |
| StateListAnimator | @animator | @stateListAnimator |
| color int | @color | @color |
| ColorStateList | @color | @colorStateList |

### 이벤트 처리 <a id="toc_11"></a>

데이터 바인딩을 사용하면 이벤트를 처리하기 위해 뷰에서 전달되는 표현식을 작성할 수 있습니다\(예: onClick\(\) 메소드\). 이벤트 등록 정보 이름은 몇 가지 예외를 제외하고는 리스너 메소드의 이름에 의해 결정됩니다. 예를 들어 View.OnClickListener에는 onClick\(\) 메서드가 있으므로 이 이벤트의 속성은 android:onClick입니다.

click 이벤트에는 충돌을 피하기 위해 android:onClick 이외의 속성이 필요한 특수 이벤트 핸들러가 있습니다. 이러한 속성을 사용하여 충돌을 피할 수 있습니다.

| Class | Listener setter | Attribute |
| :--- | :--- | :--- |
| [`SearchView`](https://developer.android.com/reference/android/widget/SearchView.html) | [`setOnSearchClickListener(View.OnClickListener)`](https://developer.android.com/reference/android/widget/SearchView.html#setOnSearchClickListener%28android.view.View.OnClickListener%29) | `android:onSearchClick` |
| [`ZoomControls`](https://developer.android.com/reference/android/widget/ZoomControls.html) | [`setOnZoomInClickListener(View.OnClickListener)`](https://developer.android.com/reference/android/widget/ZoomControls.html#setOnZoomInClickListener%28android.view.View.OnClickListener%29) | `android:onZoomIn` |
| [`ZoomControls`](https://developer.android.com/reference/android/widget/ZoomControls.html) | [`setOnZoomOutClickListener(View.OnClickListener)`](https://developer.android.com/reference/android/widget/ZoomControls.html#setOnZoomOutClickListener%28android.view.View.OnClickListener%29) | `android:onZoomOut` |

다음 메커니즘을 사용하여 이벤트를 처리 할 수 있습니다.

* [메소드 ](https://developer.android.com/topic/libraries/data-binding/expressions#method_references): 표현식에서 리스너 메소드의 서명과 일치하는 메소드를 레퍼런스 할 수 있습니다. 표현식이 메소드 참조로 평가 되면, 데이터 바인딩은 메소드 참 및 소유자 오브젝트를 리스너에 랩핑하고 리스너를 타겟 뷰에 설정합니다. 표현식이 null 로 평가 되면 데이터 바인딩은 리스너를 작성하지 않고 대신 null 리스너를 설정합니다.
* [리스너 바인딩](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings): 이벤트가 발생할 때 평가 되는 람다식입니다. 데이터 바인딩은 항상 뷰에 설정된 리스너를 만듭니다. 이벤트가 전달되면 리스너는 람다식을 계산합니다.

#### 메서드 참조 <a id="toc_12"></a>

이벤트는 android:onClick과 비슷한 방식으로 핸들러 메소드에 직접 바인딩 될 수 있습니다. View onClick 속성의 주요 장점 중 하나는 식이 컴파일 타임에 처리되므로 메서드가 없거나 서명이 올바르지 않으면 컴파일 타임 에러가 발생합니다. 

메소드 참조와 리스너 바인딩 간의 주요 차이점은 이벤트가 발행했을 때가 아니라 데이터가 바인딩 될 때 실제 리스너 구현이 작성된다는 것입니다. 이벤트가 발생했을 때 표현식을 평가하려면 [리스너 바인딩](https://developer.android.com/topic/libraries/data-binding/expressions#listener_bindings)을 사용해야 합니다.

핸들러에 이벤트를 지정하려면 호출 할 메소드의 이름을 값으로 갖는 일반 바인딩 표현식을 사용합니다. 예를 들어 다음 샘플 레이아웃 데이터 오브젝트를 생각해보십시오.

```kotlin
class MyHandlers {
    fun onClickFriend(view: View) { ... }
}
```

바인딩 표현식은 다음과 같이 뷰의 클릭 리스너를 onClickFriend\(\) 메서드에 할당합니다.

```markup
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

`참고` 표현식에 있는 메소드의 시그니처는 리스너 오브젝트에 있는 메소드의 시그니처와 정확히 일치해야 합니다.

#### 리스너 바인딩 <a id="toc_13"></a>

리스너 바인딩은 이벤트가 발생할 때 실행되는 바인딩 표현식입니다. 메서드 참와 비슷하지만 임의의 데이터 바인딩 식을 실행할 수 있습니다. 이 기능은 Gradle 버전 2.0 이상인 Android Gradle 플러그인에서 사용할 수 있습니다.

메서드 참조에서는 메서드의 매개변수 이벤트 리스너의 매개변수와 일치해야 했습니다. 리스너 바인딩은 반환값만이 리스너의 예상 반환값과 일치해야 합니다\(유효하지 않은 경우는 제외\). 예를 들어 onSaveClick\(\) 메서드가 있는 다음 Presenter 클래스를 생각해보십시오.

```kotlin
class Presenter {
    fun onSaveClick(task: Task){}
}
```

그런 다음 click 이벤트를 onSaveClick\(\) 메서드에 다음과 같이 바인딩 할 수 있습니다.

```markup
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable name="task" type="com.android.example.Task" />
        <variable name="presenter" type="com.android.example.Presenter" />
    </data>
    <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
        <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
        android:onClick="@{() -> presenter.onSaveClick(task)}" />
    </LinearLayout>
</layout>
```

표현식에서 콜백을 사용하면 데이터 바인딩이 자동으로 필요한 리스너를 만들고 이를 이벤트에 등록합니다. 뷰가 이벤트를 트리거하면 데이터 바인딩은 지정된 표현식을 평가합니다. 정규 바인딩 표현식과 마찬가지로 이러한 리스너 표현식을 평가할 때도 데이터 바인딩의 null 및 스레드 안전성을 얻을 수 있습니다.

위의 예에서는 onClick\(View\)에 전달 된 뷰 매개변수를 정의하지 않았습니다. 리스너 바인딩은 리스너 매개변수에 대해 두 가지 옵션을 제공합니다. 메소드의 모든 매개변수를 무시하거나 모든 매개변수의 이름을 지정할 수 있습니다. 매개변수의 이름을 지정하면 표현식에서 매개변수를 사용할 수 있습니다. 예를 들어, 위의 표현식은 다음과 같이 쓸 수 있습니다.

```markup
android:onClick="@{(view) -> presenter.onSaveClick(task)}"
```

또는 표현식에서 이 매개변수를 사용하려면 다음과 같이 작업합니다.

```kotlin
class Presenter {
    fun onSaveClick(view: View, task: Task){}
}
```

```markup
android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

매개변수가 여러 개 있는 람다 표현식을 사용할 수 있습니다.

```kotlin
class Presenter {
    fun onCompletedChanged(task: Task, completed: Boolean){}
}
```

```markup
<CheckBox android:layout_width="wrap_content" android:layout_height="wrap_content"
      android:onCheckedChanged="@{(cb, isChecked) -> presenter.completeChanged(task, isChecked)}" />
```

리스닝 중인 이벤트가 void 타입이 아닌 값을 반환하면 표현식도 동일한 타입의 값을 반환해야 합니다. 예를 들어 롱 프레스 이벤트를 수신하려는 경우 표현식은 Bool 값을 반환해야 합니다.

```kotlin
class Presenter {
    fun onLongClick(view: View, task: Task): Boolean { }
}
```

```markup
android:onLongClick="@{(theView) -> presenter.onLongClick(theView, task)}"
```

표현식이 null 오브젝트로 인해 평가 될 수 없는 경우, 데이터 바인딩은 해당 타입의 기본값을 반환합니다. 예를 들어, 레퍼런스 타입의 경우 null, int의 경우 0, Bool의 경우 false 를 반환합니다.

Predicate\(예: 삼항연산자\)가 있는 표현식을 사용해야 하는 경우 void를 기호로 사용할 수 있습니다.

```markup
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

#### 복잡한 리스너 사용하지 않기 <a id="toc_14"></a>

리스너 표현식은 매우 강력하며 코드를 매우 쉽게 읽을 수 있습니다. 반면에 복잡한 표현식을 포함하는 리스너는 레이아웃을 읽고 유지하기 어렵게 만듭니다. 표현식은 UI에서 콜백 메소드로 사용 가능한 데이터를 전달하는 것처럼 간단해야 합니다. 리스너 표현식에서 호출 한 콜백 메소드에서 비즈니스 로직을 구현해야 합니다.

### 임포트, 변수 및 포함\(include\) <a id="toc_15"></a>

데이터 바인딩 라이브러리는 임포트, 변수 및 포함과 같은 기능을 제공합니다. 임포트를 사용하면 레이아웃 파일에서 쉽게 레퍼런스 할 수 있습니다. 변수를 사용하면 바인딩 표현식에 사용할 수 있는 속성을 설명 할 수 있습니다. 포함을 사용하면 앱 전체에서 복잡한 레이아웃을 재사용 할 수 있습니다.

#### 임포트 <a id="toc_16"></a>

임포트를 사용하면 관리 코드처럼 레이아웃 파일에서 클래스를 쉽게 레퍼런스 할 수 있습니다. 0개 이상의 import 엘리먼트를 데이터 엘리먼트 내부에서 사용될 수 있습니다. 다음 코드 예제는 View 클래스를 레이아웃 파일로 가져옵니다.

```markup
<data>
    <import type="android.view.View"/>
</data>
```

View 클래스를 임포트하면 바인딩 표현식에서 뷰 클래스를 레퍼런스 할 수 있습니다. 다음 예제는 View 클래스의 VISIBLE 및 GONE 상수를 레퍼런스하는 방법을 보여줍니다.

```markup
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

**타입별칭**

클래스 이름 충돌이 있으면 클래스 중 하나를 별칭\(alias\)으로 바꿀 수 있습니다. 다음 예제에서는 com.example.real.estate 패키지의 View 클래스의 이름을 Vista로 변경합니다.

```markup
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

Vista를 사용하여 com.example.real.estate를 레퍼런스 할 수 있습니다. View는 레이아웃 파일 내에서 android.view.View를 레퍼런스하는데 사용될 수 있습니다.

**다른 클래스 임포트하기**

임포트한 타입은 변수 및 표현식에서 타입 레퍼런스로 사용할 수 있습니다. 다음 예제에서는 변수의 타입으로 사용되는 사용자 및 목록을 보여줍니다.

```markup
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List<User>"/>
</data>
```

`주의` Android Studio는 아직 임포트를 처리하지 않으므로 임포트한 변수에 대한 자동 완성이 IDE에서 작동하지 않을 수 있습니다. 앱을 컴파일한 후 변수 정의에서 완전한 이름을 사용하여 IDE 문제를 해결할 수 있습니다.

임포트한 타입을 사용하여 표현식의 일부를 캐스팅할 수도 있습니다. 다음 예제는 connectionproperty를 User 타입으로 캐스팅합니다.

```markup
<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

임포트한 타입은 표현식에서 정적 필드 및 메소드를 레퍼런스 할 때도 사용할 수 있습니다. 다음 코드는 MyStringUtils 클래스를 가져 와서 capitalize 메서드를 참고 합니다.

```markup
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

관리되는 코드와 마찬가지로 java.lang.\*이 자동으로 임포트됩니다.

#### 변수 <a id="toc_19"></a>

데이터 엘리먼트 안에서 여러 변수 엘리먼트를 사용할 수 있습니다. 각 변수 엘리먼트는 레이아웃 파일에서 바인딩 표현식에 사용될 속성을 설명합니다. 다음 예제에서는 user, image 및 note 변수를 선언합니다. 

```markup
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
```

변수 타입은 컴파일 타임에 검사되므로 변수가 Observable을 구현하거나 Observable한 콜렉션 인 경우 해당 타입으로 리플렉션 되어야 합니다. 변수가 Observable 인터페이스를 구현하지 않는 기본 클래스 또는 인터페이스이면 변수는 관찰되지 않습니다.

다양한 구성\(예: 가로 또는 세로\)에 다른 레이아웃 파일이 있는 경우 변수가 결합됩니다. 이러한 레이아웃 파일 간에는 변수 정의가 충돌해서는 안됩니다.

생성 된 바인딩 클래스에는 기술 된 각 변수에 대한 setter 및 getter가 있습니다. 변수는 setter가 호출 될 때까지 기본 관리 코드 값을 갖습니다.. 레퍼런스 타입의 경우는 null, int 타입의 경우는 0, Bool 타입의 경우는 false 입니다.

필요한 경우 표현식을 바인딩하기 위해 context라는 특수 변수를 생성합니다. context의 값은, 루트 뷰의 getContext\(\) 메소드의 Context 오브젝트입니다. context란 이름으로 변수를 명시적으로 선언하면컨텍스트 변수를 대체합니다.

#### 포함\(include\) <a id="toc_20"></a>

app 네임스페이스 및 속성의 변수 이름을 사용하여 레이아웃 바인딩에서 포함한 레이아웃으로 변수를 전달할 수 있습니다. 다음 예제에서는 name.xml 및 conttact.xml 레이아웃 파일에 포함 된 user 변수를 보여줍니다.

```markup
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

데이터 바인딩은 merge 엘리먼트의 차일드를 포함하는 것을 지원하지 않습니다. 예를 들어 다음 레이아웃은 지원되지 않습니다.

### 추가 자료 <a id="toc_21"></a>

데이터 바인딩에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

#### 예제 <a id="toc_22"></a>

* [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

#### 코드랩 <a id="toc_23"></a>

* [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

#### 블로그 <a id="toc_24"></a>

* [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)


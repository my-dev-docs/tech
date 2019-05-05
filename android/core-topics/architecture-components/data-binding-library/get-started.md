# 시작하기

Android Studio에서 데이터 바인딩 코드 지원을 비롯하여 개발 환경에서 데이터 바인딩 라이브러리를 사용할 수 있도록 만드는 방법에 대해 알아봅니다.

데이터 바인딩 라이브러리는 유연성과 광범위한 호환성을 제공합니다. 이 라이브러리는 지원 라이브러리이므로 Android 4.0\(API 레벨 14\) 이상을 실행하는 장치에서 사용할 수 있습니다.

프로젝트에서 최신 Gradle용 Android 플러그인을 사용하는 것이 좋습니다. 데이터 바인딩은 1.5.0 이상에서 지원됩니다. 자세한 내용은 [Gradle용 Android 플러그인을 업데이트하는 방법](https://developer.android.com/studio/releases/gradle-plugin.html#updating-plugin)을 참고 합니다.

### 빌드 환경 <a id="toc_0"></a>

데이터 바인딩을 시작하려면 Android SDK 관리자의 지원 리포지토리에서 라이브러리를 다운로드합니다. 자세한 내용은 [IDE 및 SDK 도구 업데이트](https://developer.android.com/studio/intro/update.html)를 참고 합니다.

데이터 바인딩을 사용하도록 애플리케이션을 구성하려면 다음 예와 같이 dataBinding 엘리먼트를 app 모듈의 build.gradle 파일에 추가합니다.

```text
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

`참고` 앱 모듈이 데이터 바인딩을 직접 사용하지 않더라도 데이터 바인딩을 사용하는 라이브러리에 의존하는 다른 모듈을 위해 앱 모듈에서 데이터 바인딩을 구성해야 합니다.

### Android Studio 데이터 바인딩 지원 <a id="toc_1"></a>

Android Studio는 데이터 바인딩 코드를 위한 다양한 편집 기능을 지원합니다. 예를 들어, 데이터 바인딩 표현식에 대해 다음 기능을 지원합니다.

* 문법 강조
* 표현식 언어의 문법 에러 표시
* XML 코드 완성
* [탐색](https://www.jetbrains.com/help/idea/2017.1/navigation-in-source-code.html)\(예: 선언문 탐색\) 및 [빠른 문서](https://www.jetbrains.com/help/idea/2017.1/viewing-inline-documentation.html) 찾기

`주의` 배열 및 제네릭 타입\(예: Observable 클래스\)은 에러를 잘못 표시 할 수 있습니다.

레이아웃 편집기의 미리보기 화면에는 데이터 바인딩 표현식에 대한 기본값이 표시됩니다\(제공된 경우\). 예를 들어, 미리보기 화면은 다음 예제에 선언 된 TextView 위젯에 my\_default 값을 표시합니다.

```text
<TextView android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName, default=my_default}"/>
```

기본값을 프로젝트의 설계 단계에서만 표시해야 하는 경우 [도구 속성 참고서](https://developer.android.com/studio/write/tool-attributes.html)에서 설명한대로 기본 표현식 값 대신 도구 속성을 사용할 수 있습니다.

### 클래스 바인딩을 위한 새로운 데이터 바인딩 컴파일러 <a id="toc_2"></a>

Android Gradle 플러그인 버전 3.1.0-alpha06에는 바인딩 클래스를 생성하는 새로운 데이터 바인딩 컴파일러가 포함되어 있습니다. 새로운 컴파일러는 점차적으로 빌드 프로세스의 속도를 높이는 바인딩 클래스를 만듭니다. 바인딩 클래스에 대한 자세한 내용은 [생성 된 바인딩 클래스](https://developer.android.com/topic/libraries/data-binding/generated-binding)를 참고 합니다.

이전 버전의 데이터 바인딩 컴파일러는 관리 코드를 컴파일하는 단계에서 바인딩 클래스를 생성했습니다. 이 경우 관리 코드가 컴파일되지 않으면 바인딩 클래스를 찾을 수 없다는 여러 에러가 나타날 수 있습니다. 새로운 데이터 바인딩 컴파일러는 관리 코드 컴파일러에서 앱을 빌드하기 전에 바인딩 클래스를 생성하여 이러한 에러를 방지합니다.

새로운 데이터 바인딩 컴파일러를 사용하려면 gradle.properties 파일에 다음 옵션을 추가합니다.

```text
android.databinding.enableV2=true
```

다음 매개변수를 추가하여 gradle 명령에서 새 컴파일러를 활성화 할 수도 있습니다.

```text
-Pandroid.databinding.enableV2=true
```

`참고` Android 플러그인 버전 3.1의 새 데이터 바인딩 컴파일러는 이전 버전과 호환되지 않습니다. 증분 컴파일러의 이점을 사용하려면 이 기능을 사용하여 모든 바인딩 클래스를 생성해야 합니다. 그러나 Android 플러그인 버전 3.2의 새 컴파일러는 이전 버전에서 생성 된 바인딩 클래스와 호환됩니다. 버전 3.2에서는 새 컴파일러가 기본으로 사용됩니다.

새 데이터 바인딩 컴파일러가 활성화되면 다음과 같은 변경 사항이 적용됩니다.

* Android Gradle 플러그인은 관리 코드를 컴파일하기 전에 레이아웃 용 바인딩 클래스를 생성합니다.
* 레이아웃이 여러 타겟 리소스 구성에 포함되어 있으면 데이터 바인딩 라이브러리는 동일한 리소스 ID를 공유하지만 뷰 타입이 아닌 모든 뷰에 대해 android.view.View를 기본 뷰 타입으로 사용합니다.
* 라이브러리 모듈 용 바인딩 클래스는 컴파일되어 해당 Android 아카이브 파일\(AAR\)에 패키징됩니다. 이러한 라이브러리 모듈에 의존하는 앱 모듈은 더 이상 바인딩 클래스를 다시 생성 할 필요가 없습니다. AAR 파일에 대한 자세한 내용은 [Android 라이브러리 만들기](https://developer.android.com/studio/projects/android-library)를 참고 합니다.
* 모듈의 바인딩 어댑터는 더 이상 모듈의 디펜던시 어댑터의 동작을 변경할 수 없습니다. 바인딩 어댑터는 자체 모듈의 코드와 모듈 사용자에게만 영향을 미칩니다.

### 추가 자료 <a id="toc_3"></a>

데이터 바인딩에 대한 자세한 내용은 아래의 추가 자료를 참고 합니다.

#### 예제 <a id="toc_4"></a>

* [Android Data Binding Library samples](https://github.com/googlesamples/android-databinding)

#### 코드랩 <a id="toc_5"></a>

* [Android Data Binding codelab](https://codelabs.developers.google.com/codelabs/android-databinding)

#### 블로그 <a id="toc_6"></a>

* [Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)




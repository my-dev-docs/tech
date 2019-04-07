# 3장. 메서드 등록

네이티브 라이브러리를 만들기 위해서 항상 javah로 자바의 클래스 파일을 사용해 헤더 파일을 뽑아 내야 하는 것은 아니다. JNI 네이티브 코드에서 Java 클래스 파일을 찾아 해당 클래스에 있는 네이티브 메서드 선언에 메서드 정의를 바인딩 할 수 있다. 이 방법을 사용하면 지저분한 패키지명+함수명을 담은 헤더 파일과 함수 이름이 필요없게 된다. 코드를 좀 더 깔끔하게 유지할 수 있다.

메서드 정의를 바인딩 하려면 JNI\_OnLoad 함수에서 JavaVM을 사용해야 한다.

기존에 작업순서는 아래와 같았다.

1. Java파일 작성
2. Java파일 컴파일
3. class 파일에서 javah로 C/C++용 헤더파일 추출
4. C/C++ 코드 작성
5. 네이티브 라이브러리 작성
6. 실행

하지만 이번에는 아래의 순서대로 진행해 보겠다.

1. C/C++ 코드 작성
2. 네이티브 라이브러리 작성
3. Java파일 작성
4. 실행

우선 JNI 코드 작성은 C++을 사용한다. C언어를 사용하는가? 아니면 C++언어를 사용하는가에 따라 JNI의 함수 인자들이 달라지기 때문이다.

이번 튜토리얼에서는 더하기와 곱셈을 수행하는 계산기를 만들 것이다. 우선 아래처럼 C++ 코드를 작성하자

```bash
$ vi calculator.cpp
```

```cpp
#include <jni.h>
#include <iostream>

jint add
(
    JNIEnv *env,
    jobject thiz,
    jint    a,
    jint    b
)
{
    return a + b;
}

jint mul
(
    JNIEnv *env,
    jobject thiz,
    jint    a,
    jint    b
)
{
    return a * b;
}
```

이제 위의 두 함수를 네이티브 라이브러리로 선언한 Java클래스를 검색하여 해당 네이티브 메서드의 구현체로 등록해 보자. 모든 jni 라이브러리를 로드할 때, JNI\_OnLoad 함수가 실행된다. 따라서 JNI\_OnLoad에서 해당 작업을 수행하면 좋다. JNI\_OnLoad는 모든 JNI 프로그램의 시작점이다.

```cpp
JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *pVM,
    void        *reserved
)
{
    JNIEnv *env;
    if(pVM->GetEnv(reinterpret_cast<void **>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    // 메서드 시그니쳐를 생성한다.
    JNINativeMethod nm[2];
    nm[0].name = const_cast<char *>("add");
    nm[0].signature = const_cast<char *>("(II)I");
    nm[0].fnPtr = reinterpret_cast<void *>(add);
    nm[1].name = const_cast<char *>("mul");
    nm[1].signature = const_cast<char *>("(II)I");
    nm[1].fnPtr = reinterpret_cast<void *>(mul);

    // 클래스를 찾고 해당 클래스에 네이티브 메서드로 등록한다.
    jclass cls = env->FindClass("Calculator");
    env->RegisterNatives(cls, nm, 2);
    return JNI_VERSION_1_6;
}
```

전체 코드는 아래와 같다.

```cpp
#include <jni.h>
#include <iostream>

jint add
(
    JNIEnv *env,
    jobject thiz,
    jint    a,
    jint    b
)
{
    return a + b;
}

jint mul
(
    JNIEnv *env,
    jobject thiz,
    jint    a,
    jint    b
)
{
    return a * b;
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *pVM,
    void            *reserved
)
{
    JNIEnv *env;
    if(pVM->GetEnv(reinterpret_cast<void **>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm[2];
    nm[0].name = const_cast<char *>("add");
    nm[0].signature = const_cast<char *>("(II)I");
    nm[0].fnPtr = reinterpret_cast<void *>(add);
    nm[1].name = const_cast<char *>("mul");
    nm[1].signature = const_cast<char *>("(II)I");
    nm[1].fnPtr = reinterpret_cast<void *>(mul);

    jclass cls = env->FindClass("Calculator");
    env->RegisterNatives(cls, nm, 2);
    return JNI_VERSION_1_6;
}
```

위 코드를 컴파일해서 네이티브 라이브러리를 만들자

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -c calculator.cpp
$ g++ -dynamiclib -o libcalculator.jnilib calculator.o
```

이제 이 네이티브 라이브러리를 사용하는 Java클래스를 만들자.

```java
class Calculator {

    private native int add(int a, int b);
    private native int mul(int a, int b);

    public static void main(String[] args) {

        Calculator calc = new Calculator();

        System.out.println("1+1 = " + calc.add(1, 1));
        System.out.println("10*10 = " + calc.mul(10, 10));    
    }
    static {
        System.loadLibrary("calculator");
    }
}
```

위의 Java코드를 컴파일하고 실행해 보자

```bash
$ javac Calculator.java
$ java Calculator
1+1 = 2
10*10 = 100
```

참고로 자바 타입에 대한 시그니쳐는 아래와 같다.

**Java VM Type Signatures**

| Signature | Java Type |
| :--- | :--- |
| Z | boolean |
| B | byte |
| C | char |
| S | short |
| I | int |
| J | long |
| F | float |
| D | double |
| L fully-qualified-class ; | fully-qualified-class |
| \[type | type\[\] |
| \( arg-types \) ret-type | method type |

**예제**

| Method | Signature |
| :--- | :--- |
| void f1\(\) | \(\)V |
| int f2\(int, long\) | \(IJ\)I |
| boolean f3\(int\[\]\) | \(\[I\)B |
| double f4\(String, int\) | \(Ljava/lang/String;I\)D |
| void f5\(int, String \[\], char\) | \(I\[Ljava/lang/String;C\)V |


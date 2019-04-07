# 12장. JNI에서 문자열 다루기 1/3

자바의 String은 유니코드문자열이다. JNI는 유니코드 인코딩 종류에 따라 자바 String을 처리하는 API를 제공한다.

* UTF-8 API
* UTF-16 API

이 튜토리얼에서는 UTF-8 API만을 다룰 것이다. JNI의 API를 통해 자바 String을 C 문자열\(null 포인터로 종료되는 문자 집합\)로 변경하는 단계는 아래와 같다.

1. 자바 String 인스턴스를 jstring으로 전달받는다.
2. GetStringUTFChars\(\) 함수로 널 종료 문자에 대한 포인터를 얻는다. 이 문자열은 UTF-8로 인코딩되고 힙에 동적으로 할당된다.
3. 2 단계에서 얻은 널 종료 문자열로 원하는 작업을 한다
4. ReleaseStringUTFChars\(\) 함수로 힙에 할당한 널 종료 문자열 메모리 공간을 해제한다.

유니코드 인코딩에 따른 JNI문자열 함수는 아래와 같다.

* 문자열 포인터 얻기
  * GetStringUTFChars\(\) \[UTF-8\]
  * GetStringChars\(\) \[UTF-16\]
* 문자열에 할당한 메모리 해제
  * ReleaseStringUTFChars\(\) \[UTF-8\]
  * ReleaseStringChars\(\) \[UTF-16\]
* 문자열 길이
  * GetStringUTFLength\(\) \[UTF-8\]
  * GetStringLength\(\) \[UTF-16\]
* 자바 문자열 생성
  * NewStringUTF\(\) \[UTF-8\]
  * NewString \[UTF-16\]

함수명을 보면 알 수 있듯이 UTF-8 API에는 UTF라는 단어가 함수 이름에 들어간다.

문자열 API를 활용하는 예제를 만들어 보자. 우선 자바 문자열을 전달받아서 콘솔에 출력해 보자.

```bash
$ vi PrintString.cpp
```

```cpp
#include <jni.h>
#include <iostream>

void printString
(
    JNIEnv      *env,
    jobject     thiz,
    jstring     str
)
{
    //convert to c-string from java-string
    //In other words, copy java-string to heap memory on native
    const char *cString = env->GetStringUTFChars(str, nullptr);

    std::cout << "c-string is " << cString << std::endl;

    //release heap memory
    env->ReleaseStringUTFChars(str, cString);

}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM      *vm,
    void        *reserved
)
{
    JNIEnv      *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm {
        const_cast<char*>("printString"),
        const_cast<char*>("(Ljava/lang/String;)V"),
        reinterpret_cast<void*>(printString)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

코드를 컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c PrintString.cpp
$ g++ -dynamiclib -o libprintstring.jnilib printstring.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native void printString(String str);

    public static void main(String[] args) {

        Client client = new Client();
        client.printString("Hello, World!");
    }

    static {
        System.loadLibrary("printstring");
    }
}
```

컴파일하고 사용해 본다.

```bash
$ javac Client.java
$ java Client
c-string is Hello, World!
```


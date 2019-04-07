# 21장. 자바 클래스 찾기 1/2

네이티브 레이어에서 자바 레이어에 존재하는 클래스를 찾아 보자. 클래스를 찾는 이유는 자바 레이어에 존재하는 클래스의 인스턴스를 만들어 사용하거나 네이티브 레이어에서 자바 레이어로 콜백을 보내기 위함이다.

클래스를 찾은 후 할 수 있는 일은 정적 메서드를 호출하거나 정적 필드를 사용하는 것이다. 그리고 인스턴스 메서드나 인스턴스 필드를 사용하고 싶다면 클래스를 찾은 후 먼저 클래스의 인스턴스를 생성해야만 인스턴스 메서드나 인스턴스 필드를 사용할 수 있다. 아래와 같이 정리할 수 있다.

* 자바 클래스의 정적 메서드를 사용할 때
  1. 클래스를 찾는다
  2. 정적 메서드 ID를 구한다
  3. 정적 메서드를 호출한다.
* 자바 클래스의 정적 필드를 사용할 때
  1. 클래스를 찾는다
  2. 정적 필드 ID를 구한다
  3. 정적 필드를 사용한다.
* 자바 클래스의 인스턴스 메서드를 사용할 때
  1. 클래스를 찾는다
  2. 생성자 메서드 ID를 구한다
  3. NewObject로 인스턴스를 만든다.
  4. 인스턴스 메서드 ID를 구한다
  5. 인스턴스 메서드를 호출한다.
* 자바 클래스의 인스턴스 필드를 사용할 때
  1. 클래스를 찾는다
  2. 생성자 메서드 ID를 구한다
  3. NewObject로 인스턴스를 만든다.
  4. 인스턴스 필드 ID를 구한다
  5. 인스턴스 필드를 사용한다.

먼저 자바 레이어의 클래스를 찾는 예제를 작성해 보자.

```bash
$ vi FindClass.cpp
```

```cpp
#include <jni.h>
#include <iostream>

JNIEXPORT void JNICALL findClass
(
    JNIEnv        *env,
    jobject        thiz
)
{
    //문자열 클래스를 찾는다.
    jclass stringClass = env->FindClass("java/lang/String");
    if(stringClass == nullptr) {
        std::cout << "Failed to find the String class" << std::endl;
    } else {
        std::cout << "Succss to find the String class" << std::endl;
    }
}


JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM    *vm,
    void        *reserved
)
{
    JNIEnv    *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod    nm[1] = {
        {
            const_cast<char *>("findClass"),
            const_cast<char *>("()V"),
            reinterpret_cast<void*>(findClass)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

이 예에서는 java.lang.String 클래스를 찾아보고 있다. 이 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindClass.cpp
$ g++ -dynamiclib -o libfindclass.jnilib findclass.o
```

자바 코드에서 이 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native void findClass();

    public static void main(String[] args) {

        Client client = new Client();

        client.findClass();
    }

    static {
        System.loadLibrary("findclass");
    }

}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
Succss to find the String class
```


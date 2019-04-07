# 6장. double 데이터형 값 주고 받기

double 데이터형 값을 전달하면 해당 값의 제곱근을 반환하는 예제를 만들어 보자.

```cpp
#include <jni.h>
#include <cmath>

jdouble square
(
    JNIEnv      *env,
    jobject     thiz,
    jdouble     value
)
{
    return std::sqrt(value);
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM      *vm,
    void        *reserved
)
{
    JNIEnv  *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    //c++11
    //Brace-Initialization
    JNINativeMethod nm {
                const_cast<char *>("square"),
                const_cast<char *>("(D)D"),
                reinterpret_cast<void*>(square) };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

이번 예제에서는 JNINativeMethod 구조체 초기화 방법으로 C++11 에서 소개된 브레이스 초기화 방법을 사용했다. 자세한 내용은 C++11의 문법책을 참고하길 바란다.

이제 코드를 컴파일해서 라이브러리를 만들어보자.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c PassAndReturnDouble.cpp
$ g++ -dynamiclib -o libpardouble.jnilib passandreturndouble.o
```

c++11 기능을 사용하려면 -std 옵션에 c++11 을 주어야 한다. 이제 자바 클래스에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

        private native double square(double value);

        public static void main(String[] args) {

                Client client = new Client();

                System.out.println("The square of 2 is " + client.square(2));
                System.out.println("The square of 9 is " + client.square(9));
        }

        static {
                System.loadLibrary("pardouble");
        }
}
```

자바 코드를 컴파일하고 실행해 보자.

```bash
$ javac Client.java
$ java Client
The square of 2 is 1.4142135623730951
The square of 9 is 3.0
```


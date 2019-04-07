# 7장. float 데이터형 값 주고 받기

반지름을 전달하면 원의 넓이를 반환하는 예제를 만들어 보자.

```bash
$ vi PassAndReturnFloat.cpp
```

```cpp
#include <jni.h>

jfloat circleArea
(
    JNIEnv      *env,
    jobject     thiz,
    jfloat      radius
)
{
    return 3.141592 * radius * radius;
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

    JNINativeMethod nm {
        const_cast<char *>("circleArea"),
        const_cast<char *>("(F)F"),
        reinterpret_cast<void *>(circleArea)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

컴파일하고 라이브러리로 만든다

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c PassAndReturnFloat.cpp
$ g++ -dynamiclib -o libpardouble.jnilib passandreturndouble.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native float circleArea(float radius);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("circle radius is 1. The area is " + client.circleArea(1));
        System.out.println("circle radius is 5. The area is " + client.circleArea(5));

    }

    static {
        System.loadLibrary("parfloat");
    }
}
```

자바 코드를 컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
circle radius is 1. The area is 3.141592
circle radius is 5. The area is 78.5398
```


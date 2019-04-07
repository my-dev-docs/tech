# 9장. short 데이터형 값 주고 받기

두 개의 값을 전달해 최대값을 반환하는 예제를 작성해 보자.

```bash
$ vi PassAndReturnShort.cpp
```

```cpp
#include <jni.h>

jshort max
(
    JNIEnv      *env,
    jobject     thiz,
    jshort      a,
    jshort      b
)
{
    if( a > b )
        return a;
    return b;
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM      *vm,
    void        *reserved
)
{
    JNIEnv      *env;
    if(vm->GetEnv(reinterpret_cast<void **>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm {
        const_cast<char *>("max"),
        const_cast<char *>("(SS)S"),
        reinterpret_cast<void *>(max)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c PassAndReturnShort.cpp
$ g++ -dynamiclib -o libparshort.jnilib passandreturnshort.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native short max(short a, short b);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("choose the max value between 10, 20 : " + client.max((short)10, (short)20));
        System.out.println("choose the max value between 50, 10 : " + client.max((short)50, (short)10));
    }

    static {
        System.loadLibrary("parshort");
    }
}
```

자바 코드를 컴파일하고 실행한다.

```bash
$ javac Client.java
$ java Client
choose the max value between 10, 20 : 20
choose the max value between 50, 10 : 50
```


# 10장. char 데이터형 값 주고 받기

문자를 전달하면 그 다음 문자를 반환하는 예제를 작성해 보자.

```bash
$ vi PassAndReturnChar.cpp
```

```cpp
#include <jni.h>

jchar nextChar
(
    JNIEnv      *env,
    jobject     thiz,
    jchar       ch
)
{
    return ch + 1;
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
        const_cast<char*>("nextChar"),
        const_cast<char*>("(C)C"),
        reinterpret_cast<void*>(nextChar)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c PassAndReturnChar.cpp
$ g++ -dynamiclib -o libparshort.jnilib passandreturnshort.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native char nextChar(char ch);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("The next character of 'a' is " + client.nextChar('a'));
        System.out.println("The next character of 'B' is " + client.nextChar('B'));
    }

    static {
        System.loadLibrary("parchar");
    }
}
```

자바 코드를 컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
The next character of 'a' is b
The next character of 'B' is C
```


# 5장. boolean 데이터형 값 주고 받기

boolean 값을 전달하면 그 값을 토글 시켜서 반환하는 예제를 작성해 보자.

```bash
$ vi PassAndReturnBool.cpp
```

```cpp
#include <jni.h>

jboolean toggle
(
    JNIEnv        *env,
    jobject        thiz,
    jboolean        value
)
{
    return !value;
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *vm,
    void        *reserved
)
{
    JNIEnv *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod    nm[1];
    nm[0].name = const_cast<char *>("toggle");
    nm[0].signature = const_cast<char *>("(Z)Z"); // boolean을 받고 boolean을 반환하는 함수 시그니쳐
    nm[0].fnPtr = reinterpret_cast<void *>(toggle);

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

컴파일해서 libparbool.jnilib 파일을 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -c PassAndReturnBool.cpp
$ g++ -dynamiclib -o libparbool.jnilib passandreturnbool.o
```

위에서 만든 라이브러리를 Java클래스에서 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    private native boolean toggle(boolean value);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("toggle(true) is " + client.toggle(true));
        System.out.println("toggle(false) is " + client.toggle(false));
    }

    static {
        System.loadLibrary("parbool");
    }
}
```

Java코드를 컴파일하고 실행한다.

```bash
$ javac Client.java
$ java Client
toggle(true) is false
toggle(false) is true
```


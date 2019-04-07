# 4장. int 데이터형 값 주고 받기

원시 데이터형의 값을 자바 레이어에서 네이티브 레이어로 전달하고 어떤 연산을 거친 후 되돌려 받는 예제를 작성해 보자. 우선 int 데이터형 값을 주고 받는 것으로 시작하자.

PassAndReturnInt.cpp 파일을 생성하고 작성한다.

```bash
$ vi PassAndReturnInt.cpp
```

```cpp
#include <jni.h>

jint PassAndReturnInt
(
    JNIEnv        *env,
    jobject         thiz,
    jint             data
)
{
    return data * data;
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *vm,
    void        *reserved
)
{
    JNIEnv *env;
    if(vm->GetEnv(reinterpret_cast<void **>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm[1];
    nm[0].name = const_cast<char *>("PassAndReturnInt");
    nm[0].signature = const_cast<char *>("(I)I");
    nm[0].fnPtr = reinterpret_cast<void *>(PassAndReturnInt);

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

작성한 코드를 libpassandreturnint.jnilib으로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -c PassAndReturnInt.cpp
$ g++ -dynamiclib -o libpassandreturnint.jnilib passandreturnint.o
```

위에서 만든 네이티브 라이브러리를 사용하는 Java클래스를 작성한다.

```bash
$ vi Client.java
```

```java
public class Client {
    private native int PassAndReturnInt(int data);

    public static void main(String[] args) {

        Client c = new Client();
        System.out.println("pass 10 and return = " + c.PassAndReturnInt(10));
    }

    static {
        System.loadLibrary("passandreturnint");
    }
}
```

위의 Java코드를 컴파일하고 실행해 보자

```bash
$ javac Client.java
$ java Client
pass 10 and return = 100
```


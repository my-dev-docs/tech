# 11장. byte 데이터형 값 주고 받기

숫자를 전달하면 2의 보수를 반환하는 예제를 작성해 보자.

```bash
$ vi PassAndReturnByte.cpp
```

```cpp
#include <jni.h>

jbyte complement2
(
    JNIEnv      *env,
    jobject     thiz,
    jbyte       value
)
{
    return (~value) + 1;
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
        const_cast<char*>("complement2"),
        const_cast<char*>("(B)B"),
        reinterpret_cast<void*>(complement2)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c PassAndReturnByte.cpp
$ g++ -dynamiclib -o libparbyte.jnilib passandreturnbyte.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native byte complement2(byte value);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("The two's complement of 2 is " + client.complement2((byte)2));
        System.out.println("The two's complement of 3 is " + client.complement2((byte)3));
        System.out.println("The two's complement of -1 is " + client.complement2((byte)-1));
    }

    static {
        System.loadLibrary("parbyte");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
The two's complement of 2 is -2
The two's complement of 3 is -3
The two's complement of -1 is 1
```


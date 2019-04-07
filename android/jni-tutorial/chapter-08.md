# 8장. long 데이터형 값 주고 받기

숫자를 전달하면 전달한 숫자의 팩토리얼을 반환한는 예제를 작성해 보자.

```bash
$ vi PassAndReturnLong.cpp
```

```cpp
#include <jni.h>

jlong factorial
(
    JNIEnv      *env,
    jobject     thiz,
    jlong       num
)
{
    if(num <= 1) return 1L;

    return num * factorial(env, thiz, num-1);
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
        const_cast<char *>("factorial"),
        const_cast<char *>("(J)J"),
        reinterpret_cast<void *>(factorial)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

재귀함수로 팩토리얼을 구현했다. 컴파일하고 라이브러리를 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c PassAndReturnLong.cpp
$ g++ -dynamiclib -o libparlong.jnilib passandreturnlong.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native long factorial(long num);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("The factorial of 10 is " + client.factorial(10));
        System.out.println("The factorial of 15 is " + client.factorial(15));
    }

    static {
        System.loadLibrary("parlong");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
The factorial of 10 is 3628800
The factorial of 15 is 1307674368000
```


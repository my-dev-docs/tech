# 14장. JNI에서 문자열 다루기 3/3

자바 문자열에 새로운 문자열을 붙여서 반환하는 예제를 작성해 보자.

```bash
$ vi Greeting.cpp
```

```cpp
#include <jni.h>
#include <string>

jstring greeting
(
    JNIEnv      *env,
    jobject     thiz,
    jstring     name
)
{
    //convert java-string to c-string
    const char *cName = env->GetStringUTFChars(name, nullptr);

    std::string msg = "Hello, " + std::string(cName);

    env->ReleaseStringUTFChars(name, cName);

    //convert c-string to java-string
    jstring ret = env->NewStringUTF(msg.c_str());
    return ret;
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
        const_cast<char*>("greeting"),
        const_cast<char*>("(Ljava/lang/String;)Ljava/lang/String;"),
        reinterpret_cast<void*>(greeting)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

이름을 담은 자바 문자열 앞에 Hello 문자열을 붙인 후 자바 문자열로 반환한다. 문자열 연산을 쉽게 하기 위해서 STL string 클래스를 사용했다. 코드를 컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c Greeting.cpp
$ g++ -dynamiclib -o libgreeting.jnilib greeting.o
```

자바코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native String greeting(String name);

    public static void main(String[] args) {


        Client client = new Client();


        System.out.println(client.greeting("Robin"));
        System.out.println(client.greeting("Batman"));
        System.out.println(client.greeting("토르"));

    }

    static {
        System.loadLibrary("greeting");
    }
}
```

자바 코드를 컴파일하고 실행한다.

```bash
$ javac Client.java
$ java Client
Hello, Robin
Hello, Batman
Hello, 토르
```


# 25장. 자바 클래스 인스턴스 생성하기

네이티브 레이어에서 자바 클래스를 찾은 후 클래스의 인스턴스를 생성해 보자. String 클래스의 인스턴스를 생성하는 예제를 작성해 보자.

```bash
$ vi CreateInstance.cpp
```

```cpp
#include <jni.h>
#include <iostream>

jobject newStringInstance
(
    JNIEnv        *env,
    jobject        thiz
)
{
    //Find the class
    jclass stringClass = env->FindClass("java/lang/String");
    if(stringClass == nullptr) {
        std::cout << "Failed to find the String class" << std::endl;
        return nullptr;
    }

    //Find the Constructor Id
    jmethodID constructorID = env->GetMethodID(stringClass, "<init>", "()V");
    if(constructorID == nullptr) {
        std::cout << "Failed to find the constructor of String class" << std::endl;
        return nullptr;
    }

    return env->NewObject(stringClass, constructorID);
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *vm,
    void            *reserved
)
{
    JNIEnv      *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm[1] = {
        {
            const_cast<char*>("newStringInstance"),
            const_cast<char*>("()Ljava/lang/String;"),
            reinterpret_cast<void*>(newStringInstance)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

자바 클래스의 인스턴스를 생성하려면 자바 클래스를 찾은 후 생성자 메서드의 ID를 구해야 한다. 생성자 메서드 ID를 구한 후 NewObject 메서드를 호출하면 자바 클래스의 인스턴스를 생성할 수 있다.

위 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c CreateInstance.cpp
$ g++ -dynamiclib -o libcreateinstance.jnilib createinstance.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native String newStringInstance();

    public static void main(String[] args) {

        Client client = new Client();
        String str = client.newStringInstance();
        if(str instanceof String) {
            System.out.println("Success to create an instance of String class");
        } else {
            System.out.println("Failed to create an instance of String class");
        }
    }

    static {
        System.loadLibrary("createinstance");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
Success to create an instance of String class
```


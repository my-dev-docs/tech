# 24장. 정적필드 찾기

자바 클래스에서 정적필드를 찾는 예제를 작성해 보자. Integer클래스에서 Integer형의 최대값을 가지고 있는 MAX\_VALUE 정적필드를 네이티브 레이어에서 찾아보자.

```bash
$ vi FindStaticField.cpp
```

```cpp
#include <jni.h>
#include <iostream>

jint getMaxValueOfInteger
(
    JNIEnv        *env,
    jobject        thiz
)
{

    //Find the Integer class
    jint maxValue = -1;
    jclass integerClass = env->FindClass("java/lang/Integer");
    if(integerClass == nullptr) {

        std::cout << "Failed to find the Integer class" << std::endl;

    } else {
        //Find the static field
        jfieldID maxIntegerID = env->GetStaticFieldID(integerClass, "MAX_VALUE", "I");
        if(maxIntegerID == nullptr) {
            std::cout << "Failed to find the MAX_VALUE field in the Integer class" << std::endl;
        } else {
            //Get the MAX_VALUE value
            maxValue = env->GetStaticIntField(integerClass, maxIntegerID);
        }
    }

    return maxValue;
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

    JNINativeMethod nm[1] = {
        {
            const_cast<char*>("getMaxValueOfInteger"),
            const_cast<char*>("()I"),
            reinterpret_cast<void*>(getMaxValueOfInteger)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

위의 코드를 보면 아래의 순서대로 정적필드를 찾았다.

1. 클래스를 찾는다
   * FindClass\(\)
2. 정적 필드 ID를 구한다
   * GetStaticFieldID\(\)
3. 정적 필드의 값을 구한다.
   * GetStaticIntField\(\)

위 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindStaticField.cpp
$ g++ -dynamiclib -o libfindstaticfield.jnilib findstaticfield.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native int getMaxValueOfInteger();

    public static void main(String[] args) {

        Client client = new Client();
        System.out.println("Integer's MAX_VALUE is " + client.getMaxValueOfInteger());

    }


    static {
        System.loadLibrary("findstaticfield");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
Integer's MAX_VALUE is 2147483647
```


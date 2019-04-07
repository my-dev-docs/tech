# 23장. 정적 메서드 찾기

자바 클래스에서 정적메서드를 찾아 호출하는 예제를 작성해 보자. Integer 클래스에서 문자열을 정수형으로 변환해 주는 parseInt 메서드는 Integer 클래스의 정적메서드다. 이 정적메서드를 네이티브 레이어에서 찾아 호출해 보자.

```bash
$ vi FindStaticMethod.cpp
```

```cpp
#include <jni.h>
#include <iostream>


jint stringToInt
(
    JNIEnv        *env,
    jobject        thiz,
    jstring        str
)
{
    //find the java.lang.Integer class
    jint ret = -1;
    jclass integerClass = env->FindClass("java/lang/Integer");
    if(integerClass == nullptr) {
        std::cout << "Failed to find the Integer class" << std::endl;
    } else {
        //find the parseInt static method of the Integer class
        jmethodID mid = env->GetStaticMethodID(integerClass, "parseInt", "(Ljava/lang/String;)I");
        if(mid == nullptr) {
            std::cout << "Failed to find the parseInt static method of Integer" << std::endl;
        } else {
            //call the method to get a result
            ret = env->CallStaticIntMethod(integerClass, mid, str);
        }
    }
    return ret;
}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *vm,
    void        *reserved
)
{
    JNIEnv    *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod    nm[1] = {
        {
            const_cast<char*>("stringToInt"),
            const_cast<char*>("(Ljava/lang/String;)I"),
            reinterpret_cast<void*>(stringToInt)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

위의 코드를 보면 아래의 순서대로 자바 클래스의 정적메서드를 찾아 호출했다.

1. 클래스를 찾는다
   * FindClass\(\)
2. 정적메서드 ID를 구한다.
   * GetStaticMethodID\(\)
3. 정적메서드를 호출한다.
   * CallStaticIntMethod\(\)

정적메서드를 호출하는 CallStatic...로 시작하는 함수는 여러가지 종류가 있다. 반환 타입과 인자 전달 방법에 따라 나뉜다. 여기서는 하나의 값만 반환하고 하나의 값만 인자로 전달하는 함수만 살펴본다. 더 자세한 목록은 [오라클 공식문서](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/functions.html#wp20949) 를 참고한다. CallStaticMethod\(\) 형식으로 이름을 가지며 목록은 아래와 같다.

* void CallStaticVoidMethod\(\)
* jobject CallStaticObjectMethod\(\)
* jboolean CallStaticBooleanMethod\(\)
* jbyte CallStaticByteMethod\(\)
* jchar CallStaticCharMethod\(\)
* jshort CallStaticShortMethod\(\)
* jint CallStaticIntMethod\(\)
* jlong CallStaticLongMethod\(\)
* jfloat CallStaticFloatMethod\(\)
* jdouble CallStaticDoubleMethod\(\)

위 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindStaticMethod.cpp
$ g++ -dynamiclib -o libfindstaticmethod.jnilib findstaticmethod.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native int stringToInt(String str);

    public static void main(String[] args) {
        Client client = new Client();

        System.out.println("15 is " + client.stringToInt("15"));
        System.out.println("-15 is " + client.stringToInt("-15"));
        System.out.println("1a is " + client.stringToInt("1a"));
    }

    static {
        System.loadLibrary("findstaticmethod");
    }
}
```

컴파일하고 실행한다.

```bash
$ javac Client.java
$ java Client
15 is 15
-15 is -15
```

마지막 실행은 아래처럼 예외가 발생한다. 1a 문자열을 정수 숫자로 변환할 수 없기 때문이다. 네이티브 레이어에서 예외를 처리할 수 있는데 후에 살펴볼 것이다.

```java
Exception in thread "main" java.lang.NumberFormatException: For input string: "1a"
    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)
    at java.lang.Integer.parseInt(Integer.java:492)
    at java.lang.Integer.parseInt(Integer.java:527)
    at Client.stringToInt(Native Method)
    at Client.main(Client.java:10)
```


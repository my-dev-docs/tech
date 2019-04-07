# 13장. JNI에서 문자열 다루기 2/3

네이티브 레이어에서 C/C++의 널포인터 종료 문자열을 자바 문자열로 변환해 반환하는 예제를 작성해 보자.

```bash
$ vi GetString.cpp
```

```cpp
#include <jni.h>

jstring getString
(
    JNIEnv      *env,
    jobject     thiz
)
{
    jstring msg = env->NewStringUTF("This is our string!");
    return msg;
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
        const_cast<char*>("getString"),
        const_cast<char*>("()Ljava/lang/String;"),
        reinterpret_cast<void*>(getString)
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, &nm, 1);
    return JNI_VERSION_1_6;
}
```

위의 getString\(\) 함수를 보면 자바 문자열을 새로 만들어서 반환한다. 하지만, 메모리를 해제하지는 않는다. NewStringUTF로 생성한 자바문자열은 로컬레퍼런스가 되며 로컬레퍼런스의 메모리는 함수가 리턴할 때 자동으로 해제되기 때문이다. 자세한 내용은 아래 블로그의 글을 참고한다.

* [http://aroundck.tistory.com/622](http://aroundck.tistory.com/622) - \[Java\] JNI Tutorial - Local and Global References

이제 위의 코드를 컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c GetString.cpp
$ g++ -dynamiclib -o libgetstring.jnilib getstring.o
```

위 라이브러리를 사용하는 자바 코드를 작성한다.

```java
public class Client {

    public native String getString();

    public static void main(String[] args) {


        Client client = new Client();

        String str = client.getString();
        System.out.println("str is " + str);
        System.out.println("The length of str is " + str.length());

    }

    static {
        System.loadLibrary("getstring");
    }
}
```

자바 코드를 컴파일하고 실행한다

```bash
$ javac Client.java
$ java Client
str is This is our string!
The length of str is 19
```


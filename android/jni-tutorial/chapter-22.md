# 22장. 자바 클래스 찾기 2/2

패키지 명을 포함한 자바 클래스 이름을 전달해 해당 클래스를 찾는 예제를 작성해 보자.

```bash
$ vi FindClass2.cpp
```

```cpp
#include <jni.h>
#include <iostream>
#include <algorithm>
#include <string>

/**
 * 클래스 이름으로 클래스를 찾는다.
 */
JNIEXPORT void JNICALL findClass2
(
    JNIEnv        *env,
    jobject        thiz,
    jstring        className
)
{
    //convert java-string to c-string
    const char *cName = env->GetStringUTFChars(className, nullptr);

    std::string name = std::string(cName);
    env->ReleaseStringUTFChars(className, cName);

    //convert com.package.someclass to com/package/someclass
    std::replace(name.begin(), name.end(), '.', '/');

    //find some class
    jclass someClass = env->FindClass(name.c_str());
    if(someClass == nullptr) {
        std::cout << "Failed to find the " << name << " class" << std::endl;
    } else {
        std::cout << "Success to find the " << name << " class" << std::endl;
    }

}

JNIEXPORT jint JNICALL JNI_OnLoad
(
    JavaVM        *vm,
    void            *reserved
)
{
    JNIEnv *env;
    if(vm->GetEnv(reinterpret_cast<void**>(&env), JNI_VERSION_1_6)) {
        return -1;
    }

    JNINativeMethod nm[1] = {
        {
            const_cast<char*>("findClass2"),
            const_cast<char*>("(Ljava/lang/String;)V"),
            reinterpret_cast<void*>(findClass2)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

네이티브 레이어에서 자바 레이어의 클래스를 찾기 위해서 점을 슬래시로 변경해야 한다. 관련 내용이 findClass2 함수에 있다. 이제 이 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindClass2.cpp
$ g++ -dynamiclib -o libfindclass2.jnilib findclass2.o
```

자바 코드에서 라이브러리를 사용해 보자.

```java
public class Client {

    public native void findClass2(String fullClassName);

    public static void main(String[] args) {

        Client client = new Client();

        if(args.length == 0) {
            client.findClass2("java.lang.String");
            client.findClass2("com.earth.Person");
        } else {
            client.findClass2(args[0]);
        }

    }

    static {
        System.loadLibrary("findclass2");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
Success to find the java/lang/String class
Failed to find the com/earth/Person class
```


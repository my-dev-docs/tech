# 27장. 자바 클래스의 인스턴스 필드 찾기

자바 클래스의 인스턴스 필드를 찾는 예제를 작성해 보자. 인스턴스 필드를 찾을 경우 인스턴스 필드의 가시성은 고려되지 않는다. 즉, 자바 클래스의 private 인스턴스 필드도 찾을 수 있다.

직접 작성한 클래스이 private 인스턴스 필드를 찾은 후 값을 출력하고 해당 필드에 값을 재설정해 보자.

```bash
$ vi FindInstanceField.cpp
```

```cpp
#include <jni.h>
#include <iostream>


void printPrivateFieldOfThiz
(
    JNIEnv        *env,
    jobject        thiz
)
{
    //find the class of this
    jclass thizClass = env->GetObjectClass(thiz);

    if(thizClass == nullptr) {
        std::cout << "Failed to find the class of thiz" << std::endl;
        return;
    }

    //find the private name field
    jfieldID nameID = env->GetFieldID(thizClass, "name", "Ljava/lang/String;");
    if(nameID == nullptr) {
        std::cout << "Failed to find the name field in the thiz" << std::endl;
        return;
    }

    jstring name = static_cast<jstring>( env->GetObjectField(thiz, nameID) );
    if(name == nullptr) {
        std::cout << "Failed to get the reference of the name field" << std::endl;
        return;
    }

    //convert java-string to c-string
    const char *cName = env->GetStringUTFChars(name, nullptr);
    if(cName == nullptr)
        return;

    std::cout << "I found [ " << cName << " ] !!!" << std::endl;

    env->ReleaseStringUTFChars(name, cName);


    //change the name value
    jstring newName = env->NewStringUTF("Jerry Boy");
    if(newName == nullptr)
        return;

    env->SetObjectField(thiz, nameID, newName);
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

    JNINativeMethod nm[1] ={

        {
            const_cast<char*>("printPrivateFieldOfThiz"),
            const_cast<char*>("()V"),
            reinterpret_cast<void*>(printPrivateFieldOfThiz)
        }

    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

직접 만든 클래스에서 name 이라는 private 인스턴스 필드를 찾아서 걊을 출력하고 값을 다시 재설정한다.

위 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindInstanceField.cpp
$ g++ -dynamiclib -o libfindinstancefield.jnilib findinstancefield.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    private String name = "Burt";

    public native void printPrivateFieldOfThiz();

    public static void main(String[] args) {

        Client client = new Client();
        client.printPrivateFieldOfThiz();    

        System.out.println("After finding the name : " + client.name );
    }

    static {
        System.loadLibrary("findinstancefield");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
I found [ Burt ] !!!
After finding the name : Jerry Boy
```


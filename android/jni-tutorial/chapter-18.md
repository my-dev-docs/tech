# 18장. 배열 다루기 3/4

자바 레이어에서 전달 받은 배열의 모든 요소를 출력하는 예제를 만들어 보자. 자바 배열의 원소에 어떻게 접근하는지 이해하는 것이 중요하다.

```bash
$ vi PrintArray.cpp
```

```cpp
#include <jni.h>
#include <iostream>

void printIntArray
(
    JNIEnv      *env,
    jobject     thiz,
    jintArray   intArray
)
{
    //get c-pointer to intArray
    jint *cintArray = env->GetIntArrayElements(intArray, nullptr);

    //get the length of intArray
    jsize length = env->GetArrayLength(intArray);

    //print array element
    for(int i=0; i<length; i++) {
        std::cout << cintArray[i] << " ";
    }
    std::cout << std::endl;

    env->ReleaseIntArrayElements(intArray, cintArray, 0);
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
            const_cast<char*>("printIntArray"),
            const_cast<char*>("([I)V"),
            reinterpret_cast<void*>(printIntArray)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

자바 배열을 네이티브에서 다루기 위해서는 자바 배열을 가리키는 C언어 포인터가 필요하다. JNIEnv의 GetIntArrayElements\(\) 함수로 얻을 수 있다. 또한 배열 길이를 알을 때에는 GetArrayLength\(\) 함수를 사용해 배열 길이를 알 수 있다.

위 코드를 컴파일하고 라이브러리로 만들어 보자.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c PrintArray.cpp
$ g++ -dynamiclib -o libprintarray.jnilib printarray.o
```

컴파일한 라이브러리를 사용하는 자바 코드를 작성한다.

```bash
$ vi Client.java
```

```java
public class Client {

    public native void printIntArray(int[] intArray);

    public static void main(String[] args) {

        Client client = new Client();
        client.printIntArray(new int[]{1, 2, 3, 4, 5});
    }

    static {
        System.loadLibrary("printarray");
    }
}
```

코드를 컴파일하고 실행해 보자.

```bash
$ javac Client.java
$ java Client
1 2 3 4 5
```


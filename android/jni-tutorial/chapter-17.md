# 17장. 배열 다루기 2/4

네이티브 레이어에서 자바 배열을 생성해서 자바 레이어로 반환하는 예제를 작성해 보자.

```bash
$ vi GenIntArray.cpp
```

```cpp
#include <jni.h>

// 배열 초기화 방식1
jintArray genIntArray
(
    JNIEnv      *env,
    jobject     thiz,
    jint        length,
    jint        initValue
)
{
    // make a new int array
    jintArray intArray = env->NewIntArray(length);

    // make c-array to set the int array
    jint *cintArray = new jint[length];
    for(int i=0; i<length; i++) {
        cintArray[i] = initValue;
    }

    // copy c-array to java-array
    env->SetIntArrayRegion(intArray, 0, length, cintArray);

    delete [] cintArray;

    return intArray;
}

// 배열 초기화 방식2
jintArray genIntArray2
(
    JNIEnv      *env,
    jobject     thiz,
    jint        length,
    jint        initValue
)
{
    // make a new int array
    jintArray intArray = env->NewIntArray(length);

    // get c-array pointer to set intArray
    jint *ptrArray = env->GetIntArrayElements(intArray, nullptr);
    for(int i=0; i<length; i++) {
        ptrArray[i] = initValue;
    }
    env->ReleaseIntArrayElements(intArray, ptrArray, 0);
    return intArray;
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

    JNINativeMethod nm[2] = {
        {
            const_cast<char*>("genIntArray"),
            const_cast<char*>("(II)[I"),
            reinterpret_cast<void*>(genIntArray)
        }
        ,
        {
            const_cast<char*>("genIntArray2"),
            const_cast<char*>("(II)[I"),
            reinterpret_cast<void*>(genIntArray2)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 2);
    return JNI_VERSION_1_6;
}
```

네이티브에서 자바 배열을 생성하는 단계는 두 단계로 나뉜다.

1. NewArray\(\) 함수로 자바 배열을 선언한다.
2. 만든 배열을 초기화 한다.
   1. C언어로 배열을 만들어서 영역 복사를 하거나
   2. 자바 배열을 가리키는 C언어 포인터를 얻어 자바 배열을 초기화 하거나

이제 위의 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c GenIntArray.cpp
$ g++ -dynamiclib -o libgenintarray.jnilib genintarray.o
```

자바코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native int[] genIntArray(int length, int initValue);
    public native int[] genIntArray2(int length, int initValue);

    public static void main(String[] args) {


        Client client = new Client();

        {
            int[] array = client.genIntArray(10, 1);
            for(int e : array) {
                System.out.println(e);
            }
        }

        {
            int[] array = client.genIntArray2(20, 2);
            for(int e : array) {
                System.out.println(e);
            }
        }
    }

    static {
        System.loadLibrary("genintarray");
    }
}
```

자바 코드를 컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
1
1
1
1
1
1
1
1
1
1
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
2
```


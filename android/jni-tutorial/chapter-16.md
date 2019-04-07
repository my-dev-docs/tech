# 16장. 배열 다루기 1/4

자바 배열을 표현하는 JNI 배열 데이터형은 jarray이고 배열이 담는 데이터형에 따라 jarray를 상속받은 여러 배열 데이터형이 존재한다.

* jarray
  * jbooleanArray
  * jbyteArray
  * jcharArray
  * jshortArray
  * jintArray
  * jlongArray
  * jfloatArray
  * jdoubleArray
  * jobjectArray

자바 배열을 전달 받아서 길이를 반환하는 예제를 작성해 보자.

```bash
$ vi GetArrayLength.cpp
```

```cpp
#include <jni.h>

jint getArrayLength
(
    JNIEnv          *env,
    jobject         thiz,
    jarray          elements
)
{
    return env->GetArrayLength(elements);
}

jint getIntArrayLength
(
    JNIEnv          *env,
    jobject         thiz,
    jintArray       elements
)
{
    return env->GetArrayLength(elements);
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

    JNINativeMethod nm[2] ={

        {
            const_cast<char*>("getArrayLength"),
            const_cast<char*>("([Ljava/lang/Object;)I"),
            reinterpret_cast<void*>(getArrayLength)
        }
        ,
        {
            const_cast<char*>("getIntArrayLength"),
            const_cast<char*>("([I)I"),
            reinterpret_cast<void*>(getIntArrayLength)
        }

    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 2);
    return JNI_VERSION_1_6;
}
```

위의 예제에서 jarray를 java.lang.Object 의 배열로 매핑했는데 사실 jobjectArray로 매핑하는 것이 더 정확하다. 일부러 jarray를 쓴 이유는 jarray 를 시그니쳐로 표현할 수 없기 때문에 하나의 함수로 모든 종류의 배열 길이를 얻을 수 없음을 보여주기 위함이다. 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c GetArrayLength.cpp
$ g++ -dynamiclib -o libgetarraylength.jnilib getarraylength.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    private native int getArrayLength(Object[] array);
    private native int getIntArrayLength(int[] array);

    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("The length of array " + client.getArrayLength(new Integer[]{1, 2, 3, 4, 5}));

        //캐스팅 오류가 발생한다.
        //System.out.println("The length of array " + client.getArrayLength(new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9}));

        System.out.println("The length of array " + client.getIntArrayLength(new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9}));
    }

    static {
        System.loadLibrary("getarraylength");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
The length of array 5
The length of array 9
```


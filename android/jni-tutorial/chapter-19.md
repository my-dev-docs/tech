# 19장. 배열 다루기 4/4

자바 레이어에서 전달 받은 배열을 C/C++ 배열로 변환한 후, 특정 작업을 수행한 다음 다시 자바 배열로 반환하는 예제를 작성해 보자. 배열의 합과 평균을 배열로 반환해 보자.

```bash
$ vi AverageArray.cpp
```

```cpp
#include <jni.h>

void addValue
(
    JNIEnv          *env,
    jobject         thiz,
    jdoubleArray    doubleArray,
    jdouble         value
)
{
    //get the length of doubleArray
    jsize length = env->GetArrayLength(doubleArray);
    if(length == 0)
        return ;

    //convert java-array to c-array jdouble[]
    jdouble *cdoubleArray = env->GetDoubleArrayElements(doubleArray, nullptr);

    //add value
    for(int i=0; i<length; i++) {
        cdoubleArray[i] += value;
    }

    env->ReleaseDoubleArrayElements(doubleArray, cdoubleArray, 0);
}

jdoubleArray getSumAndAverage
(
    JNIEnv          *env,
    jobject         thiz,
    jdoubleArray    doubleArray
)
{
    //get the length of doubleArray
    jsize length = env->GetArrayLength(doubleArray);
    if(length == 0)
        return nullptr;

    //convert java-array to c-array jdouble[]
    jdouble *cdoubleArray = env->GetDoubleArrayElements(doubleArray, nullptr);

    jdouble sum = 0.0;
    //add value
    for(int i=0; i<length; i++) {
        sum += cdoubleArray[i];
    }

    jdouble average = sum / length;

    env->ReleaseDoubleArrayElements(doubleArray, cdoubleArray, 0);

    //make a new jdoubleArray to return
    jdoubleArray newDoubleArray = env->NewDoubleArray(2);

    //make a c array
    jdouble *newCDoubleArray = new jdouble[2];
    newCDoubleArray[0] = sum;
    newCDoubleArray[1] = average;

    //copy c-array to java-array
    env->SetDoubleArrayRegion(newDoubleArray, 0, 2, newCDoubleArray);

    delete [] newCDoubleArray;


    return newDoubleArray;
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
            const_cast<char*>("addValue"),
            const_cast<char*>("([DD)V"),
            reinterpret_cast<void*>(addValue)
        }
        ,
        {
            const_cast<char*>("getSumAndAverage"),
            const_cast<char*>("([D)[D"),
            reinterpret_cast<void*>(getSumAndAverage)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 2);
    return JNI_VERSION_1_6;
}
```

위 코드에는 배열의 모든 원소에 특정 값을 더하는 함수와 배열의 총합과 평균을 2개의 배열 원소에 담아 반환하는 함수가 있다. 이 두 함수의 코드는 지금까지 배열을 다룬 내용을 종합한 것이니 따로 설명은 하지 않는다.

코드를 컴파일하고 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Java/JavaVirtualMachines/Current/" -std=c++11 -c AverageArray.cpp
$ g++ -dynamiclib -o libaveragearray.jnilib averagearray.o
```

이제 이 라이브러리를 사용하는 자바 코드를 작성한다.

```bash
$ vi Client.java
```

```java
public class Client {

    private native void addValue(double[] array, double value);
    private native double[] getSumAndAverage(double[] array);

    public static void main(String[] args) {

        Client client = new Client();

        double [] array = new double[] { 1, 2, 3, 4, 5 };

        client.addValue(array, 10);

        for(double e : array) {
            System.out.print(e + " ");
        }
        System.out.println("");

        double [] ret = client.getSumAndAverage(array);

        System.out.println("sum = " + ret[0] + " average = " + ret[1]);

    }

    static {
        System.loadLibrary("averagearray");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
11.0 12.0 13.0 14.0 15.0
sum = 65.0 average = 13.0
```


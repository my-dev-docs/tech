# 28장. 네이티브 쓰레드 실행하기

네이티브 레이어에서 c++11 에 추가된 std::thread를 실행해 보자.

```bash
$ vi StartThread.cpp
```

```cpp
#include <jni.h>
#include <iostream>
#include <thread>

void startThreads
(
    JNIEnv      *env,
    jobject     thiz,
    jint        count
)
{
    //start thread1
    std::thread th1 = std::thread([&] {
        for(int i=1; i<=count; ++i) {
            std::cout << "[thread1] counting : " << i << std::endl;
        }
    });


    //start thread2
    std::thread th2 = std::thread([&] {
        for(int i=count; i>0; --i) {
            std::cout << "[thread2] counting : " << i << std::endl;
        }
    });

    //waiting for finish of the thread1
    if(th1.joinable()) {
        th1.join();
    }

    //waiting for finish of the thread2
    if(th2.joinable()) {
        th2.join();
    }
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
            const_cast<char*>("startThreads"),
            const_cast<char*>("(I)V"),
            reinterpret_cast<void*>(startThreads)
        }
    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 1);
    return JNI_VERSION_1_6;
}
```

쓰레드 두 개를 만들어 수를 카운팅한다.

코드를 컴파일해 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c StartThread.cpp
$ g++ -dynamiclib -o libstartthread.jnilib startthread.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$ vi Client.java
```

```java
public class Client {

    public native void startThreads(int count);

    public static void main(String[] args) {

        Client client = new Client();
        System.out.println("=============== START ===================");
        client.startThreads(10);
        System.out.println("=============== FINISH ========");

    }

    static {
        System.loadLibrary("startthread");
    }
}
```

컴파일하고 실행해 본다.

```bash
=============== START ===================
[thread1] counting : 1
[thread1] counting : 2
[thread1] counting : 3
[thread1] counting : 4
[thread1] counting : 5
[thread1] counting : 6
[thread1] counting : 7
[thread1] counting : 8
[thread1] counting : 9
[thread1] counting : 10
[thread2] counting : 10
[thread2] counting : 9
[thread2] counting : 8
[thread2] counting : 7
[thread2] counting : 6
[thread2] counting : 5
[thread2] counting : 4
[thread2] counting : 3
[thread2] counting : 2
[thread2] counting : 1
=============== FINISH ========
```


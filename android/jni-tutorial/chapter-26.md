# 26장. 자바 클래스 인스턴스 메서드 찾기

자바 클래스의 인스턴스 메서드를 찾아 호출해 보는 예제를 작성해 보자. 자바 패키지에 존재하는 클래스의 인스턴스 메서드와 우리가 직접 만든 자바 클래스의 인스턴스 메서드를 호출해 볼 것이다.

인스턴스 메서드 호출은 네이티브 레이어에서 자바 레이어로 콜백하기 위해서 많이 쓰이기도 한다.

```bash
$ vi FindInstanceMethod.cpp
```

```cpp
jstring toUpperString
(
    JNIEnv        *env,
    jobject        thiz,
    jstring        str
)
{
    //create String instance
    //find the string class
    jclass stringClass = env->FindClass("java/lang/String");
    if(stringClass == nullptr) {
        std::cout << "Failed to find the String class" << std::endl;
        return nullptr;
    }

    //find the contructor id
    jmethodID constructorID = env->GetMethodID(stringClass, "<init>", "(Ljava/lang/String;)V");
    if(constructorID == nullptr) {
        std::cout << "Failed to find the constructor of String class" << std::endl;
        return nullptr;
    }

    jstring stringObject = static_cast<jstring>( env->NewObject(stringClass, constructorID, str) );

    //find the toUpperCase Method
    jmethodID toUpperCaseID = env->GetMethodID(stringClass, "toUpperCase", "()Ljava/lang/String;");
    if(toUpperCaseID == nullptr) {
        std::cout << "Failed to find the toUpperCase method" << std::endl;
        return nullptr;
    }

    //call method
    return static_cast<jstring>(env->CallObjectMethod(stringObject, toUpperCaseID));
}

jstring toLowerString
(
    JNIEnv        *env,
    jobject        thiz,
    jstring        str
)
{
    //find the string class
    jclass stringClass = env->FindClass("java/lang/String");
    if(stringClass == nullptr) {
        std::cout << "Failed to find the String class" << std::endl;
        return nullptr;
    }

    //find the toLowerCase Method
    jmethodID toLowerCaseID = env->GetMethodID(stringClass, "toLowerCase", "()Ljava/lang/String;");
    if(toLowerCaseID == nullptr) {
        std::cout << "Failed to find the toLowerCase method" << std::endl;
        return nullptr;
    }

    //call method
    return static_cast<jstring>(env->CallObjectMethod(str, toLowerCaseID));
}


void doJobAndThenCallback
(
    JNIEnv        *env,
    jobject        thiz,
    jint            value
)
{
    //find the class of thiz object
    jclass thizClass = env->GetObjectClass(thiz);

    //find the onStart Method
    jmethodID onStartID = env->GetMethodID(thizClass, "onStart", "()V");
    if(onStartID == nullptr) {
        std::cout << "Failed to find the onStart method" << std::endl;
        return;
    }

    //call the onStart callback Method
    env->CallVoidMethod(thiz, onStartID);

    //do some jobs
    for(auto i=0; i<value; ++i) {
        std::cout << i << " ";
    }
    std::cout << std::endl;

    //find the onFinish Method
    jmethodID onFinishID = env->GetMethodID(thizClass, "onFinish", "()V");
    if(onFinishID == nullptr) {
        std::cout << "Failed to find the onFinish method" << std::endl;
        return ;
    }

    //call the onFinish callback Method
    env->CallVoidMethod(thiz, onFinishID);
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

    JNINativeMethod nm[3] ={

        {
            const_cast<char*>("toUpperString"),
            const_cast<char*>("(Ljava/lang/String;)Ljava/lang/String;"),
            reinterpret_cast<void*>(toUpperString)
        }
        ,
        {
            const_cast<char*>("toLowerString"),
            const_cast<char*>("(Ljava/lang/String;)Ljava/lang/String;"),
            reinterpret_cast<void*>(toLowerString)
        }
        ,
        {
            const_cast<char*>("doJobAndThenCallback"),
            const_cast<char*>("(I)V"),
            reinterpret_cast<void*>(doJobAndThenCallback)
        }

    };

    jclass cls = env->FindClass("Client");
    env->RegisterNatives(cls, nm, 3);
    return JNI_VERSION_1_6;
}
```

java.lang.String 클래스의 toUpperCase, toLowerCase 인스턴스 메서드를 찾아 호출하고 직접 만든 Client 클래스에서는 onStart, onFinish 인스턴스 메서드를 호출한다.

위 코드를 컴파일해서 라이브러리로 만든다.

```bash
$ g++ "-I/System/Library/Frameworks/JavaVM.framework/Versions/Current/Headers/" -std=c++11 -c FindInstanceMethod.cpp
$ g++ -dynamiclib -o libfindinstancemethod.jnilib findinstancemethod.o
```

자바 코드에서 라이브러리를 사용해 보자.

```bash
$vi Client.java
```

```java
public class Client {
    public native String toUpperString(String str);
    public native String toLowerString(String str);
    public native void doJobAndThenCallback(int value);


    public static void main(String[] args) {

        Client client = new Client();

        System.out.println("hello, world - " + client.toUpperString("hello, world"));
        System.out.println("HELLO, WORLD - " + client.toLowerString("HELLO, WORLD"));

        client.doJobAndThenCallback(5);
    }

    void onStart() {
        System.out.println("Starting the job...");
    }

    void onFinish() {
        System.out.println("Finished");
    }

    static {
        System.loadLibrary("findinstancemethod");
    }
}
```

컴파일하고 실행해 본다.

```bash
$ javac Client.java
$ java Client
hello, world - HELLO, WORLD
HELLO, WORLD - hello, world
Starting the job...
0 1 2 3 4
Finished
```


# javap

chromium을 빌드할 때, 아래와 같은 오류가 발생하면서 빌드 실패가 발생한다.

```bash
$ autoninja -C out/AndroidDebug chrome_public_apk
```

```python
ninja: Entering directory `out/AndroidDebug'
[54/39252] ACTION //base:android_runtime_jni_headers(//build/toolchain/android:android_clang_arm64)
FAILED: gen/base/android_runtime_jni_headers/Runnable_jni.h gen/base/android_runtime_jni_headers/Runtime_jni.h 
python ../../base/android/jni_generator/jni_generator.py --ptr_type=long --includes ../../../../../base/android/jni_generator/jni_generator_helper.h --jar_file ../../third_party/android_sdk/public/platforms/android-28/android.jar --output_file gen/base/android_runtime_jni_headers/Runnable_jni.h --output_file gen/base/android_runtime_jni_headers/Runtime_jni.h --input_file=java/lang/Runnable.class --input_file=java/lang/Runtime.class
Traceback (most recent call last):
  File "../../base/android/jni_generator/jni_generator.py", line 1639, in <module>
    sys.exit(main())
  File "../../base/android/jni_generator/jni_generator.py", line 1633, in main
    GenerateJNIHeader(java_path, header_path, args)
  File "../../base/android/jni_generator/jni_generator.py", line 1523, in GenerateJNIHeader
    jni_from_javap = JNIFromJavaP.CreateFromClass(input_file, options)
  File "../../base/android/jni_generator/jni_generator.py", line 858, in CreateFromClass
    universal_newlines=True)
  File "/usr/lib/python2.7/subprocess.py", line 394, in __init__
    errread, errwrite)
  File "/usr/lib/python2.7/subprocess.py", line 1047, in _execute_child
    raise child_exception
OSError: [Errno 2] No such file or directory
[63/39251] ACTION //build/android:prepare_android_lint_cache(//build/toolchain/android:android_clang_arm64)
ninja: build stopped: subcommand failed.

```

이 문제는 oracle이 JDK 배포 방식을 변경하면서 개발자가 수동으로 JDK를 설치했을 때 발생할 수 있다. 오류 원인을 보면 JNI 관련 작업을 수행할 때 javap를 찾지 못해서 발생하는 것이다.

JDK가 설치된 폴더를 찾아 PATH에 등록해 주면 문제를 해결할 수 있다. JDK 설치 및 PATH 설정은 아래 문서를 참고한다.

{% embed url="https://tech.burt.pe.kr/ubuntu/install-oracle-jdk8" %}




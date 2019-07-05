# install oracle-jdk8

오라클이 라이센스 정책을 변경하면서 회원가입을 해야 JDK를 받을 수 있다. 따라서 기존 ppa 레포를 추가하여 설치하는 방법을 더 이상 사용할 수 없다.

직접 JDK 8 을 다운로드하여 설치하는 것이 가장 정확한 방법이다.

{% embed url="https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html" %}

위 링크에서 Linux64용 JDK를 받는다. 받은 곳에 임의 이름의 쉘스크립틀를 만든다.

```bash
#!/bin/sh

tar -xvf jdk-8*
sudo rm -rf /usr/lib/jvm
sudo mkdir /usr/lib/jvm
sudo mv ./jdk1.8* /usr/lib/jvm/jdk1.8.0
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.8.0/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.8.0/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.8.0/bin/javaws" 1
sudo chmod a+x /usr/bin/java
sudo chmod a+x /usr/bin/javac
sudo chmod a+x /usr/bin/javaws
```

위 파일 이름이 install-java8.sh 이라면 아래처럼 실행 권한을 주고 실행한다.

```text
$ chmod +x install-java8.h
$ ./install-java8.sh
```

설치가 마무리 되면 java의 버전을 확인한다.

```text
$ java -version
java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
```




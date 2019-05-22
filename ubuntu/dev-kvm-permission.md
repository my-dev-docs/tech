# /dev/kvm on Android Studio

## /dev/kvm <a id="devkvm"></a>

안드로이드 에뮬레이터 /dev/kvm 퍼미션 요청시 아래 과정을 거쳐 해결할 수 있다.

`참고` [https://stackoverflow.com/questions/37300811/android-studio-dev-kvm-device-permission-denied](https://stackoverflow.com/questions/37300811/android-studio-dev-kvm-device-permission-denied)

### Install qemu-kvm <a id="install-qemu-kvm"></a>

```text
sudo apt install qemu-kvm
```

### Add user to kvm group <a id="add-user-to-kvm-group"></a>

```text
sudo adduser <Replace with username> kvm
```

### change owner <a id="change-owner"></a>

```text
sudo chown <Replace with username> /dev/kvm
```


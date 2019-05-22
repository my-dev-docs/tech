# UEFI USB

## Unetbootin

Mac에서 UEFI USB를 만들 때 우분투 공식 가이드에 있는 Etcher 보다 Unetbootin 이 더 낫다. Etcher는 UEFI Bootable USB를 만들지 못하는 것 같다.

{% embed url="https://github.com/unetbootin/unetbootin" %}

## 다운로드

{% embed url="https://unetbootin.github.io/" %}

## USB 포멧

```text
diskutil eraseDisk FAT32 [diskname] [/dev/disk#]
```

### 디스크 목록 얻기

```text
diskutil list
```

### 예제

```text
diskutil eraseDisk FAT32 USB /dev/disk3
```


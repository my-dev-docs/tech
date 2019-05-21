# 특정 git 폴더에서 zsh 이 느려질 때

## git\_prompt

* 특정 git 폴더에서 oh-my-zsh이 너무 느려지는 경우가 있음.
  * chromium 레포지토리를 받아 `cd src` 로 들어가려 하면 많은 시간을 무응답으로 대기해야 함
* 속도가 느려지는 원인은 oh-my-zsh의 git\_prompt 기능 때문에 발생
* 따라서 해당 폴더에서만 git\_prompt를 사용하지 않게 하면 됨

## 해결법 <a id="toc_1"></a>

`.git`이 있는 위치로 이동하여 아래 명령어를 수행한 한다.

```text
$ git config --add oh-my-zsh.hide-status 1
$ git config --add oh-my-zsh.hide-dirty 1
```

* 해당 폴더에서 git\_prompt를 사용하지 않게 함.
* 현재 브랜치 및 git 상태가 나타나지 않는 불편함은 감수해야 함 \*


---
description: Swift 5.0에서 더욱 강력해진 Raw 문자열을 소개합니다.
---

# Raw 문자열

{% embed url="https://www.youtube.com/watch?v=e6tuUzmxyOU" %}

[SE-0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md)은 백슬래시와 따옴표\(“\)가 이스케이프 문자나 문자열 종결자로 해석하지 않고 순수 문자 리터럴로 해석하게끔 Raw 문자열을 만드는 기능을 추가했습니다. 이렇게 하면 많은 경우에 도움이 되지만 특히 정규표현식을 사용할 때 도움이 됩니다.

Raw 문자열을 사용하려면 다음과 같이 문자열 앞에 `#` 기호를 한 개 이상 배치합니다.

```swift
let rain = #"The "rain" in "Spain" falls mainly on the Spaniards."#
```

문자열의 시작과 끝 부분에 있는 `#` 기호는 문자열의 시작과 끝을 구분하는 기호가 되므로 Swift는 "rain"및 "Spain"을 둘러싼 따옴표를 문자열 종결자로 해석하지 않고 문자 따옴표로 이해합니다. Raw 문자열을 사용하면 백슬래시도 사용할 수 있습니다.

```swift
let keypaths = #"Swift keypaths such as \Person.name hold uninvoked references to properties."#
```

백슬래시는 문자열에서 이스케이프 문자가 아닌 문자로 취급됩니다. 이 경우 문자열 보간이 다르게 작동합니다.

```swift
let answer = 42
let dontpanic = #"The answer to life, the universe, and everything is #(answer)."#
```

문자열 \(answer\)는 문자 그대로 해석되기 때문에, Raw 문자열에서 문자열 보간을 사용하기 위해서는 여분의 `#`을추가해야 합니다. 즉, 문자열 보간을 사용하기 위해 \#\(answer\)을 사용합니다.

Swift 5.0의 Raw 문자열은 해시 기호를 처음과 끝에 사용합니다. 필요한 경우 해시 기호를 두 개 이상 사용할 수 있습니다. 당장 좋은 예를 들기는 어렵습니다. 해시 기호를 두 개 이상 사용하는 경우가 정말 드물기 때문입니다. 그러나 다음 문자열을 생각해 봅시다. `My dog sail "woof"#gooddog.`

해시 기호 앞에 공백이 없으므로 Swift는 `#`을 문자열 종결자로 해석합니다. 이 경우에는 구분 기호를 `#` 에서 `##` 으로 변경해야 합니다.

```swift
let str = ##"My dog said "woof"#gooddog"##
```

시작과 끝에 있는 해시 기호는 서로 개수가 같아야 합니다. Raw 문자열은 Swift의 멀티 라인 문자열 시스템과 완벽하게 호환됩니다. `#"""` 로 시작하고 `"""#` 끝내면 됩니다.

```swift
let multiline = #"""
The answer to life,
the universe,
and everything is #(answer).
"""#
```

Raw 문자열은 백슬래시를 많이 사용하는 정규 표현식에서 특히 유용합니다. 예를 들어, \Person.name 같은 keypath를 찾는 간단한 정규식을 작성해 봅시다.

```swift
let regex1 = "\\\\[A-Z]+[A-Za-z]+\\.[a-z]+"
```

Raw 문자열을 사용하면 백슬래시를 절반으로 줄일 수 있습니다.

```swift
let regex2 = #"\\[A-Z]+[A-Za-z]+\.[a-z]+"#
```

백슬래시를 완전히 제거할 수는 없습니다. 정규 표현식이 백슬래시를 정규 표현식의 일부로 사용하기 때문입니다.

이 글은 [https://www.hackingwithswift.com/articles/126/whats-new-in-swift-5-0](
https://www.hackingwithswift.com/articles/126/whats-new-in-swift-5-0) 을 편역한 것입니다.


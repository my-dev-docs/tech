---
description: '번역 중입니다 :)'
---

# 커스텀 문자열 보간

[SE-0228](https://github.com/apple/swift-evolution/blob/master/proposals/0228-fix-expressiblebystringinterpolation.md)은 Swift의 문자열 보간 시스템을 획기적으로 개선하였습니다. 더 효율적이고 유연한 시스템으로 이전에는 불가능했던 새로운 기능을 제공합니다. 새로운 문자열 보간 시스템을 사용하면 객체가 문자열에 나타나는 방식을 제어 할 수 있습니다.

Swift는 문자열 보간식 내에 구조체를 사용하면 구조체 이름과 모든 속성을 출력합니다. 디버깅을 돕고자 하는 스위프트의 기본 동작입니다. 그러나 클래스는 그렇지 않습니다. 문자열 보간식에 클래스를 지원하거나 문자열 보간식으로 출력되는 내용에 특정 형식을 적용하고자 할 때 Swift 5.0의 새로운 문자열 보간 시스템을 사용할 수 있습니다.

User 구조체를 정의해 봅시다.

```swift
struct User {
    var name: String
    var age: Int
}
```

User 구조체 인스턴스의 내용을 깔끔하게 출력하는 특별한 문자열 보간을 User 구조체에 추가해 봅시다. String.StringInterpolation을 확장하고 appendInterpolation\(\) 메서드를 정의합니다.​

Swift는 이미 여러가지 기본으로 제공되는 메서드들을 가지고 있어서 User 구조체 인스턴스가 문자열 보간에 쓰일 때, 호출할 메서드를 찾게 됩니다.

이 예제의 경우 기본으로 제공되는 메서드 중 하나인 appendInterpolation\(\)를 구현하여 사용자 이름과 나이를 문자열 하나로 출력합니다.

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: User) {
        appendInterpolation("My name is \(value.name) and I'm \(value.age)")
    }
}
```

이제 user를 생성하고 데이터를 출력 할 수 있습니다.

```swift
let user = User(name: "Guybrush Threepwood", age: 33)
print("User details: \(user)")
```

위 코드는 User details: My name is Guybrush Threepwood and I’m 33 라고 출력합니다. 커스텀 문자열 보간을 사용하여 출력된 것 입니다.

이 기능은 CustomStringConvertible 프로토콜을 구현하는 것과 다르지 않습니다. 그러나 커스텀 문자열 보간은 더 많은 고급 기능을 제공합니다.

커스텀 보간 메서드는 레이블이 있거나 또는 레이블이 없는 인자를 필요한만큼 받을 수 있습니다. 예를 들어, 다음과 같이 다양한 스타일을 사용하여 숫자를 출력하는 문자열 보간을 구현할 수 있습니다.

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ number: Int, style: NumberFormatter.Style) {
        let formatter = NumberFormatter()
        formatter.numberStyle = style

        if let result = formatter.string(from: number as NSNumber) {
            appendLiteral(result)
        }
    }
}
```

NumberFormatter 클래스는 통화\(currency\)\($ 72.83\), 서수\(ordinal\)\(1st, 12th, ordinal\) 및 발음\(spell out\)\(five, forty-three\)을 비롯한 많은 스타일을 갖고 있습니다. 그래서 다음처럼 난수를 발음으로 변경하여 문자열로 만들 수 있습니다.

```swift
let number = Int.random(in: 0...100)
let lucky = "The lucky number this week is \(number, style: .spellOut)."
print(lucky)
```

appendLiteral\(\)을 필요에 따라 여러 번 호출하거나 아예 호출하지 않아도 됩니다. 다음처럼 문자열을 여러 번 반복하는 커스텀 문자열 보간을 구현할 수 있습니다.

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(repeat str: String, _ count: Int) {
        for _ in 0 ..< count {
            appendLiteral(str)
        }
    }
}

print("Baby shark \(repeat: "doo ", 6)")
```

appendInterpolation\(\)은 일반 메서드이므로 Swift의 모든 기능을 사용할 수 있습니다. 예를 들어, 문자열 배열을 결합하는 커스텀 보간을 구현할 수 있습니다. 만약 배열이 비어 있으면 문자열을 반환하지 않고 클로저를 실행합니다.

```swift
extension String.StringInterpolation {
    mutating func appendInterpolation(_ values: [String], empty defaultValue: @autoclosure () -> String) {
        if values.count == 0 {
            appendLiteral(defaultValue())
        } else {
            appendLiteral(values.joined(separator: ", "))
        }
    }
}

let names = ["Harry", "Ron", "Hermione"]
print("List of students: \(names, empty: "No one").")
```

value.count 가 0일 경우에는 @autoclosure를 적용하여 defaultValue로 단순한 값을 사용하거나 복잡한 함수를 호출할 수 있습니다.

ExpressibleByStringLiteral과 ExpressibleByStringInterpolation 프로토콜을 결합하여 모든 타입이 문자열 보간을 사용하도록 만들 수 있습니다. 여기에 CustomStringConvertible 프로토콜을 추가하면 원하는 타입을 문자열로 출력 할 수 있습니다.

이를 위해 몇 가지 규칙을 지켜야 합니다.

* 

---
description: JSON과 같은 포맷과 호환되도록 데이터 타입을 인코딩 및 디코딩할 수 있습니다.
---

# 커스텀 타입 인코딩 및 디코딩

### 개요

네트워크 연결을 통해 데이터를 보내거나 디스크에 데이터를 저장하거나 API 및 서비스에 데이터를 보내는 일은 대부분 프로그래밍 작업에 포함됩니다. 이러한 작업에서는 데이터를 전송하는 동안 중간 타입으로 데이터를 인코딩 하거나 중간 타입에서 데이터로 디코딩해야 합니다.​

Swift 표준 라이브러리는 데이터 인코딩 및 디코딩을 위한 표준 접근법을 가지고 있습니다. Encodable 및 Decodable 프로토콜을 커스텀 타입이 구현하여 표준 접근법을 사용할 수 있습니다.

이 프로토콜들을 채택하면 Encoder 및 Decoder 프로토콜의 구현을 통해 데이터를 가져와 JSON 또는 프로퍼티 리스트\(plist\)와 같은 포맷으로 데이터를 인코딩하거나 디코딩 할 수 있습니다. 인코딩 및 디코딩을 모두 지원하려면 Encodable 및 Decodable 프로토콜을 결합한 Codable 프로토콜을 사용합니다. 이를 통해 여러분의 타입을 Codable 타입으로 작성할 수 있습니다.

### 자동으로 인코딩 및 디코딩하기​

Codable을 만드는 가장 쉬운 방법은 이미 Codable인 타입을 사용하여 속성을 선언하는 것입니다. Codable을 지원하는 타입은 String, Int 및 Double과 같은 표준 라이브러리 타입과 Date, Data 및 URL과 같은 Foundation 타입입니다. Codable 속성만을 사용하는 모든 타입은 Codable 프로토콜 준수를 선언만 해도 Codable 프로토콜을 자동으로 따르게 됩니다.

랜드 마크 이름과 창립 연도를 저장하는 Landmark 구조체를 정의해 봅시다.

```swift
struct Landmark {
    var name: String
    var foundingYear: Int
}
```

Landmark 구조체가 Codable 프로토콜을 준수하면 Encodable 및 Decodable의 모든 프로토콜 요구 사항을 자동으로 충족하게 됩니다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    
    // 구조체 안에서 메서드가 선언되진 않았지만
    // Landmark는 이제 Codable 프로토콜 메서드인 init(from:)과 encode(to:)를 지원합니다.
}
```

커스텀 타입이 Codable 프로토콜을 채택하면 내장 데이터 포맷으로 타입을 직렬화거나 커스텀 인코더 및 디코더가 제공하는 포맷으로 직렬화할 수 있습니다. Landmark 자체에는 Property List 또는 JSON을 처리하는 코드가 없지만 Landmark 를 PropertyListEncoder 및 JSONEncoder 클래스를 사용하여 인코딩 할 수 있습니다.

Codable한 커스텀 타입으로 구성된 또 다른 커스텀 타입에도 동일한 원칙이 적용됩니다. 해당 속성이 모두 Codable이면 커스텀 타입 전체 또한 Codable입니다. 다음은 Landmark 구조체에 location 속성을 추가 할 때 자동으로 Codable이 적용되는 사례를 보여줍니다.

```swift
struct Coordinate: Codable {
    var latitude: Double
    var longitude: Double
}

struct Landmark: Codable {
    // Double, String 그리고 Int 모두는 Codable 타입입니다.
    var name: String
    var foundingYear: Int
    
    // Codable인 커스텀 타입을 추가하면 Landmark 전체는 계속 Codable한 타입이 됩니다.
    var location: Coordinate
}
```

Array, Dictionary 그리고 Optional과 같은 타입은 Codable한 타입을 원소로 담게 되면 Codable 타입이 됩니다. Landmark에 Coordinate 인스턴스 배열을 추가 할 수 있으며 전체 Landmark 구조체는 여전히 Codable한 타입이 됩니다. 다음은 Landmark 구조체에 Codable타입을 사용하여 여러 속성을 추가 할 때 자동으로 Codable을 준수하는 사례입니다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    
    // 아래 속성을 추가해도 Landmark는 여전히 Codable 타입입니다.
    var vantagePoints: [Coordinate]
    var metadata: [String: String]
    var website: URL?
}
```

### 인코딩과 디코딩을 나눠서 사용하기

때로는 인코딩과 디코딩 모두 가능한 Codable이 필요하지 않을 수 있습니다. 예를 들어 앱에서 원격 네트워크 API를 호출만 하고 응답을 받지 않는다면 응답을 디코딩 할 필요가 없습니다. 데이터 인코딩만 사용하는 경우 Encodable 프로토콜을 채택합니다. 반대로 주어진 데이터 타입으로 디코딩만 하는 경우 Decodable 프로토콜을 채택합니다.

다음은 데이터를 인코딩만 하거나 디코딩만하는 Landmark 구조체 선언 사례입니다.

```swift
struct Landmark: Encodable {
    var name: String
    var foundingYear: Int
}

struct Landmark: Decodable {
    var name: String
    var foundingYear: Int
}
```

### CodingKey를 사용하여 인코딩과 디코딩 할 속성 선택하기

Codable 타입은 CodingKey 프로토콜을 준수하는 CodingKeys라는 이름의 중첩된 enum을 선언 할 수 있습니다. 이 enum이 있는 경우, enum의 case 들은 Codable 타입의 인스턴스를 인코딩 하거나 디코딩 할 때 포함할 속성 목록을 제공합니다. enum의 각 case이름은 타입의 해당 속성 이름과 일치해야 합니다.

인스턴스를 디코딩하거나 인코딩할 때 특정 속성을 제외하려는 경우 CodingKeys enum에서 속성을 생략합니다. CodingKeys에서 생략 된 속성은 기본값이 필요합니다. 자동으로 Decodable 또는 Codable한 타입이 되기 위해서 입니다.

직렬화 된 데이터 타입에서 사용되는 키가 데이터 타입의 속성 이름과 일치하지 않으면 CodingKeys enum의 rawValue 타입으로 String을 지정하여 대체하는 키 이름을 제공해야 합니다. enum의 각 case의 rawValue로 사용하는 문자열은 인코딩 및 디코딩 중에 사용되는 키 이름입니다. case 이름과 rawValue 간의 연결을 통해 Swift API 디자인 지침에 따라 데이터 이름을 지정할 수 있습니다. 모델링하는 타입의 이름, 구두점 및 대소문자를 일치시키지 않아도 됩니다.

다음은 Landmark 구조체를 인코딩 및 디코딩할 때 foundingYear라는 대체 속성 키 이름을 사용하는 사례입니다.

```swift
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    var vantagePoints: [Coordinate]
    
    enum CodingKeys: String, CodingKey {
        case name = "title"
        case foundingYear = "founding_date"
        
        case location
        case vantagePoints
    }
}
```

### 직접 인코딩 및 디코딩 하기

사용자가 정의한 구조체가 인코딩 된 데이터와 구조가 다를 경우 인코딩 및 디코딩 과정을 구현하여 자신만의 인코딩 및 디코딩 로직을 작성할 수 있습니다. 다음은 Coordinate 구조체가 additionalInfo 컨테이너 내부 elevation 속성을 지원하도록 Coordinate 구조체를 확장한 사례입니다.

```swift
struct Coordinate {
    var latitude: Double
    var longitude: Double
    var elevation: Double

    enum CodingKeys: String, CodingKey {
        case latitude
        case longitude
        case additionalInfo
    }
    
    enum AdditionalInfoKeys: String, CodingKey {
        case elevation
    }
}
```

Coordinate 타입의 인코딩 데이터는 중첩 데이터를 포함하고 있습니다. Coordinate 타입이 Encodable 및 Decodable 프로토콜을 사용하면 두 개의 CodingKey enum을 사용하여 특정 레벨의 Codable 속성 목록을 제공해야 합니다.

다음은 Decodable 프로토콜을 채택하여 init\(from:\)을 구현한 확장 코드입니다.

```swift
extension Coordinate: Decodable {
    init(from decoder: Decoder) throws {
        let values = try decoder.container(keyedBy: CodingKeys.self)
        latitude = try values.decode(Double.self, forKey: .latitude)
        longitude = try values.decode(Double.self, forKey: .longitude)
        
        let additionalInfo = try values.nestedContainer(keyedBy: AdditionalInfoKeys.self, forKey: .additionalInfo)
        elevation = try additionalInfo.decode(Double.self, forKey: .elevation)
    }
}
```

초기자는 매개변수인 Decoder 인스턴스를 사용하여 Coordinate 인스턴스의 속성을 초기화합니다. Swift 표준 라이브러리에서 제공하는 키-컨테이너 API를 사용하여 Coordinate 인스턴스의 두 속성을 초기화합니다.

다음은 Encodable 프로토콜을 채택하여 encode\(to:\)를 구현한 확장 코드입니다.

```swift
extension Coordinate: Encodable {
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        try container.encode(latitude, forKey: .latitude)
        try container.encode(longitude, forKey: .longitude)
        
        var additionalInfo = container.nestedContainer(keyedBy: AdditionalInfoKeys.self, forKey: .additionalInfo)
        try additionalInfo.encode(elevation, forKey: .elevation)
    }
}
```

encode\(to:\) 메소드는 이전 Decodable 확장에서 수행한 디코딩 작업과 반대로 작업을 수행합니다. 인코딩 및 디코딩 과정을 직접 작성 할 때 사용되는 키-컨테이너 타입에 대한 자세한 내용은 [KeyedEncodingContainerProtocol](https://developer.apple.com/documentation/swift/keyedencodingcontainerprotocol) 및 [UnkeyedEncodingContainer](https://developer.apple.com/documentation/swift/unkeyedencodingcontainer)를 참고합니다.

### 이 글은 다음 문서를 편역한 것입니다.

{% embed url="https://developer.apple.com/documentation/foundation/archives\_and\_serialization/encoding\_and\_decoding\_custom\_types" %}


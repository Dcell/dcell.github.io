---
layout: post
comments: true
---
## 前言
Swift 4.0版本引入了一种新的对象序列化的方式`Codeable`，用于代替原先OC语法的`NSCode`协议。
在程序执行过程中，我们经常需要通过网络发送数据，保存数据到磁盘，这往往是一个对象序列化的过程；在Swift4.0开始，系统提供一套对象编解码的协议，可以自动或者自定义的实现对象的序列化。

```
typealias Codable = Decodable & Encodable
```
## 自动解码和编码
想要对象可编码，最简单的方式就是用可编码的类型去声明属性；
> 为了描述简单，结构体和对象都描述为对象

这些可编码的属性包括：`Int` `String` `Double` `Date` `Data` `URL`等

```
struct Landmark {
    var name: String
    var foundingYear: Int
}
```
接下来，我们只需要让对象实现`Codeable`协议，该对象就自动实现了编码和解码。

```
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    
    // Landmark now supports the Codable methods init(from:) and encode(to:), 
    // even though they aren't written as part of its declaration.
}
```
但是我们平常开发中，属性往往也是自定义对象，在`Codeable`协议中，只需要所有的属性都支持编解码，那么该对象也能编解码。

```
struct Coordinate: Codable {
    var latitude: Double
    var longitude: Double
}

struct Landmark: Codable {
    // Double, String, and Int all conform to Codable.
    var name: String
    var foundingYear: Int
    
    // Adding a property of a custom Codable type maintains overall Codable conformance.
    var location: Coordinate
}
```
内置的类型，比如`Array` `Dictionary`,只需要元素实现了`Codeable`，那么也支持编解码。

```
struct Landmark: Codable {
    var name: String
    var foundingYear: Int
    var location: Coordinate
    
    // Landmark is still codable after adding these properties.
    var vantagePoints: [Coordinate]
    var metadata: [String: String]
    var website: URL?
}
```

## 手动解码和编码
如果你对象和编码结构不同，你可以自定义编码和解码协议的实现，完成对象的编码和解码。

```
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
解码实现`init(from decoder: Decoder)`

```
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

编码实现 `encode(to encoder: Encoder)` 

```
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

## JSONEncoder & JSONDecoder
Swift提供了系统的JSON数据的编解码方式，可以将实现`Codeable`对象和JSON对象相互转化，非常的简单和方便。

```
struct GroceryProduct: Codable {
    var name: String
    var points: Int
    var description: String?
}

let pear = GroceryProduct(name: "Pear", points: 250, description: "A ripe pear.")

let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

let data = try encoder.encode(pear)
print(String(data: data, encoding: .utf8)!)

/* Prints:
 {
   "name" : "Pear",
   "points" : 250,
   "description" : "A ripe pear."
 }
*/
```


```
struct GroceryProduct: Codable {
    var name: String
    var points: Int
    var description: String?
}

let json = """
{
    "name": "Durian",
    "points": 600,
    "description": "A fruit with a distinctive scent."
}
""".data(using: .utf8)!

let decoder = JSONDecoder()
let product = try decoder.decode(GroceryProduct.self, from: json)

print(product.name) // Prints "Durian"
```
## PS
我自己写的一个KV存储库[SwiftLvDB](https://github.com/Dcell/SwiftLvDB)，也将支持`Codeable`对象存储，敬请关注。
## 参考
https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types
https://developer.apple.com/documentation/foundation/jsonencoder
https://developer.apple.com/documentation/foundation/jsondecoder
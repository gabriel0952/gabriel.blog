---
layout: post
title: 擴展(Extensions)翻譯與筆記
tags: [note, iOS, Swift, Extensions]
categories: [iOS]
---

> 本文根據 [The swift programming language](https://docs.swift.org/swift-book/LanguageGuide/Extensions.html) ，進行翻譯以及筆記整理。

Swift 中的 extensions 可以：
- 添加 計算型 instance properties 和計算型 type properties
- 定義 instance 方法和 type 方法
- 提供新的 initializers
- 定義下標(subscripts)
- 定義和使用新的 nested types
- 使一個已存在的的類型符合某個協議

## Extension 的語法
### 宣告方式:
``` swift
extension SomeType {
    // new functionality to add to SomeType goes here
}
```

對想要擴展的 type 可以使其採用一種或多種 protocols
``` swift
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```

### Computed Properties (計算屬性)
Extension 可以將計算的實例屬性和計算的類型屬性添加到現有類型。下面例子向 Swift 的內置 Double 類型添加了五個計算實例屬性，為使用距離單位提供基本支持：

``` swift
extension Double {
    var km: Double { return self * 1_000.0 }
    var  m: Double { return self }
    var cm: Double { return self / 100.0 }
    var mm: Double { return self / 1_000.0 }
    var ft: Double { return self / 3.28084 }
}
let oneInch = 25.4.mm
print("One inch is \(oneInch) meters")
// Prints "One inch is 0.0254 meters"
let threeFeet = 3.ft
print("Three feet is \(threeFeet) meters")
// Prints "Three feet is 0.914399970739201 meters"
```

這些計算的屬性表示Double值應視為某個長度單位。儘管將它們實現為計算屬性，但是可以使用點語法將這些屬性的名稱附加到浮點文字值上，以作為使用該文字來執行距離轉換的一種方式。

這些屬性是 read-only 的計算屬性，因此為了簡潔起見，它們不使用get關鍵字表示。它們的返回值是Double類型的，並且可以在接受Double的任何地方的數學計算中使用：

``` swift
let aMarathon = 42.km + 195.m
print("A marathon is \(aMarathon) meters long")
// Prints "A marathon is 42195.0 meters long"
```

### Initializers (初始化器)
擴展可以向現有的類型添加新的初始化器。這樣一來就可以擴展其他類型，以接受自己自定義的類型作為初始化程序參數，或者提供未包含在該類型的原始實現中的其他初始化選項。

下面例子定義了一個自定義的Rect結構來表示一個幾何矩形。該示例還定義了兩個支持的結構，稱為“Size”和“Point”，這兩個結構的所有屬性的默認值都是0.0：

``` swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}
struct Rect {
    var origin = Point()
    var size = Size()
}
```

因為Rect結構為其所有屬性提供默認值，所以它會自動接收默認初始化程序和成員初始化程序，如默認初始化程序中所述。這些初始化程序可用於創建新的Rect實例：

``` swift
let defaultRect = Rect()
let memberwiseRect = Rect(
    origin: Point(x: 2.0, y: 2.0), 
    size: Size(width: 5.0, height: 5.0)
)
```

您可以擴展Rect結構以提供採用特定中心點和大小的附加初始化程序：

``` swift
extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

這個新的 initializer 開始於根據提供的 `center` 和 `size` 計算適當的原點。然後，初始化程序會調用結構的 automatic memberwise initializer init（origin：size :)，它將新的 `origin` 和 `size` 存儲在適當的屬性中：

``` swift
let centerRect = Rect(
    center: Point(x: 4.0, y: 4.0), 
    size: Size(width: 3.0, height: 3.0)
)
// centerRect's origin is (2.5, 2.5) and its size is (3.0, 3.0)
```

### Methods (方法相關)

Extensions 可以向現有類型添加新的實例方法和類型方法。下面例子向 Int 類型添加一個稱為 `repetitions` 的新實例方法：

``` swift
extension Int {
    func repetitions(task: () -> Void) {
        for _ in 0..<self {
            task()
        }
    }
}
```

`repetitions（task :)` 方法採用 `（）-> Void` 類型的單個 argument，這個 function 是沒有參數且不返回值的函數。

定義此 extension 後，可以在任何整數上調用 repetitions（task :) method 來執行多次任務：

``` swift
3.repetitions {
    print("Hello!")
}
// Hello!
// Hello!
// Hello!
```

#### 特化的 instance 方法

可以以 extension 的方式來修改 instance 本身。
當 Structure and enumeration methods 要修改時，需將 instance 標註為 `mutating`

下面這個例子，對 Swift's Int 的值做平方的運算。

``` swift
extension Int {
    mutating func square() {
        self = self * self
    }
}
var someInt = 3
someInt.square()
// someInt is now 9
```

### Subscripts (下標)

Extensions 可以對已存在的類型新增新的 subscripts。 
下面這個例子是幫 Swift’s built-in Int type 新增一個 subscript。 這個 subscript [n] 會回傳十進制中由右數起的第 n 位數值：


``` swift
123456789[0] returns 9
123456789[1] returns 8
```

…and so on:

``` swift
extension Int {
    subscript(digitIndex: Int) -> Int {
        var decimalBase = 1
        for _ in 0..<digitIndex {
            decimalBase *= 10
        }
        return (self / decimalBase) % 10
    }
}
746381295[0]
// returns 5
746381295[1]
// returns 9
746381295[2]
// returns 2
746381295[8]
// returns 7
```

如果輸入的數值超出擁有的位數，則會回傳 0 ，可視為在其位數前補 0。

``` swift
746381295[9]
// returns 0, as if you had requested:
0746381295[9]
```

### Nested Types (巢狀)

Extensions 可以幫已存在的 classes, structures, and enumerations 加入新的 nested types：

``` swift
extension Int {
    enum Kind {
        case negative, zero, positive
    }
    var kind: Kind {
        switch self {
        case 0:
            return .zero
        case let x where x > 0:
            return .positive
        default:
            return .negative
        }
    }
}
```
這個例子為 Int 新增了一個叫做 Kind 的 nested enumeration，用來表示數字的類型具體來說，它表示數字是負數，零數還是正數。

並寫在例子中也新增了一個名為 kind 的 computed instance property，他會為整數回傳適當的 Kind enumeration。

``` swift
func printIntegerKinds(_ numbers: [Int]) {
    for number in numbers {
        switch number.kind {
        case .negative:
            print("- ", terminator: "")
        case .zero:
            print("0 ", terminator: "")
        case .positive:
            print("+ ", terminator: "")
        }
    }
    print("")
}
printIntegerKinds([3, 19, -27, 0, -6, 0, 7])
// Prints "+ + - 0 - 0 + "
```

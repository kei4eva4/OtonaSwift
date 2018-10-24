# Swiftの文法

## 1. 変数、配列、ディクショナリ、制御
### 変数の定義

```swift
// letでイミュータブルな変数（1行コメント）
// イミュータブル……作成後に状態を変えられないオブジェクトのこと。対義語はミュータブル
let immutableStr = "hello"
immutableStr += " world"  // => error

/*
 * varでミュータブルな変数
 * （複数行コメント）
 */
var mutableStr = "hello"
mutableStr += " world"
print(mutableStr)  // => hello world
```

- 上の例では、型推論により、変数の型はString型となります。明示的に型を指定するには、以下のように書きます。

```swift
var mutableStr: String = "hello"
```

- Swiftでは 暗黙の型変換は行われません。よって、型を変換したい場合は、明示的に記述する必要があります。

```swift
let label = "This year is "
let year = 2017
let thisYear = label + String(2017)
print(thisYear)  // => This year is 2017
// 文字列内での変数展開
let nextYear = "Next year is \(year + 1)"
print(nextYear)  // => Next year is 2018
```

### 配列の定義

```swift
var itemsArray = ["foo", "bar", "baz"]
// 型を明示
var itemsArray: [String] = ["foo", "bar", "baz"]
```

### ディクショナリの定義

```swift
var itemsDictionary = [
  "foo": "FOO",
  "bar": "BAR",
  "baz": "BAZ"
]

// 型を明示
var itemsDictionary: [String: String] = [
  "foo": "FOO",
  "bar": "BAR",
  "baz": "BAZ"
]
```

### 制御構造

```swift
// for, if-else
let list = [3, 7, 9, 12, 8, 5]
for number in list {
  if number % 2 == 0 {
    print("number \(number) is even")
  } else {
    print("number \(number) is odd")
  }
}

// while
var number = 1
while number < 10 {
  print(number)
  number += 1
}

// switch-case
let age = 17
switch age {
case 0...6: // rangeオブジェクトをつかう事ができる
  print("kindergarden kid")
case 7...12:
  print("primary school student")
case 13...15:
  print("junior high school student")
case 16...18:
  print("high school student")
case 19...22:
  print("college student")
default:
  print("business person")
}
// => high school student

```

## 2. 関数、クロージャ、クラス
### 関数（メソッド）
```swift
func greet(expression: String, person: String) -> String {
  return "\(expression) \(person)."
}

greet(expression: "Hello", person: "Mike")  // => "Hello Mike."
```

```swift
func greet(_ expression: String, to person: String) -> String {
  return "\(expression) \(person)."
}
// _　で引数ラベルの省略。　引数のラベル名をつけることもできる
greet("Hello", to: "Mike")  // => "Hello Mike."
```

### クロージャ

```swift
func incrementer() -> ( () -> Int ) {
  var count = 0
  func increment() -> Int {
    count += 1
    return count
  }
  return increment
}

var counter = incrementer()
counter()  // => 1
counter()  // => 2
counter()  // => 3
```

- returnされる関数を無名関数に

```swift
func incrementerWithAnonymousFunc() -> ( () -> Int ) {
  var count = 0
  return { () -> Int in
    count += 1
    return count
  }
}

var counter2 = incrementerWithAnonymousFunc()
counter2()  // => 1
counter2()  // => 2
counter2()  // => 3
```

- 関数は、引数として関数を受け取ることができます。

```swift
func numbersMap(list: [Int], condition: (Int) -> Int) -> [Int] {
  var numbers: [Int] = []
  for item in list {
    numbers.append( condition(item) )
  }
  return numbers
}

var items: [Int] = [1, 2, 3, 4, 5]
numbersMap(list: items, condition: { (number: Int) -> Int in number * 2 })  // => [2, 4, 6, 8, 10]
```

- このクロージャ機能により、Swiftではmap、filter、reduce等のメソッドに、引数として無名関数のブロックコードを渡して処理することができます。
```swift
var numbers = [3, 7, 9, 12, 8, 5]
// 配列の要素をすべて2倍にする
numbers.map({ (number: Int) -> Int in return number * 2 })  // => [6, 14, 18, 24, 16, 10]

// 奇数のみを抽出する
numbers.filter({ (number: Int) -> Bool in return number % 2 == 1 })  // => [3, 7, 9, 5]

// すべての合計を計算する
numbers.reduce(0, { (total: Int, number: Int) -> Int in
  return total + number
})  // => 44
```

- map、filter、reduceの引数でブロックコードを渡す場合は、型指定を省略した書き方が可能
```swift
numbers.map{ number in number * 2 }  // => [6, 14, 18, 24, 16, 10]
numbers.filter{ number in number % 2 == 1 }  // => [3, 7, 9, 5]
numbers.reduce(0){ total, number in total + number }  // => 44
```

### クラス

```swift
class MyApp {
  // Shapeクラスの定義
  class Shape {
    // nameプロパティ（インスタンス変数）
    var name: String

    // イニシャライザ（コンストラクタ）
    init(name: String) {
      self.name = name
    }
  }

  // 四角形: RectangleがShapeを継承
  class Rectangle: Shape {
    var width: Double
    var height: Double

    init(name: String, width: Double, height: Double) {
      self.width = width
      self.height = height
      // 親クラスのイニシャライザ呼び出し
      super.init(name: name)
    }

    func area() -> Double {
      return width * height
    }
  }

  // 三角形: TriangleがShapeを継承
  class Triangle: Shape {
    var bottom: Double
    var height: Double

    init(name: String, bottom: Double, height: Double) {
      self.bottom = bottom
      self.height = height
      super.init(name: name)
    }

    func area() -> Double {
      return bottom * height / 2.0
    }
  }
}

// 正方形を作成
var square = MyApp.Rectangle(name: "My Square", width: 7.5, height: 7.5)
square.name    // => "My Square"
square.area()  // => 56.25

// 三角形を作成
var triangle = MyApp.Triangle(name: "My Triangle", bottom: 10, height: 8)
triangle.name    // => "My Triangle"
triangle.area()  // => 40
```

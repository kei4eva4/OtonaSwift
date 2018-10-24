# Swiftの文法②Optional

- Swiftの言語仕様の特徴として、nilを安全に扱うためにOptionalという型が存在します。
- nilを許容する変数は、Optional型として定義する必要があります。
- 変数をOptional型で定義しない限り、変数にnilを代入することはできません。
- 非Optional型の変数、たとえば通常のInt型やString型の変数には、nilを代入できません。

- Optional型を使うことで、変数やメソッドの戻り値がnilになり得ることを、プログラマが明示できます。
- 不適切にnilの可能性のある変数にアクセスするコードを書いていた場合、コンパイル時に警告が出て、nilが返ることによる予期せぬ実行時エラーの発生を防ぐ働きをしてくれるわけです。

## Optional型の定義

```swift
// Optional Int
var optionalInt: Int? = 5
// 通常のInt
var int: Int = 5

// Optional String
var optionalStr: String? = "hello"
// 通常のString
var str: String = "hello"

// Optional Stringの変数にnilを代入
optionalStr = nil
// 通常のStringにはnilを代入できない
str = nil  // => error: nil cannot be assigned to type 'String'
```

- Int?はnilまたはInt型の値を代入できる型、String?はnilまたはString型の値を代入できる型です。Int?とIntはまったく別の型であり、同様にString?とStringもまったく別の型です。
- T?という書き方は、Optional<T>のシンタックスシュガー
```swift
var optionalInt: Optional<Int> = 5 //Int?と同じ
var optionalStr: Optional<String> = "hello" // String?と同じ
```

## Optional型のアンラップ
- Swiftでは、Optional型の変数を ラップ（wrap）されていると表現します。
- 一方、値を扱えるようにするための処理を、アンラップ（unwrap）すると表現します。
- アンラップせずに、Optional型の変数に対してメソッドを呼び出そうとすると、変数がアンラップされていないよ！ とエラーが発生します。

```swift
var optionalStr: String? = "hello"
optionalStr.uppercased()  // => error: value of optional type 'String?' not unwrapped
```

- Optional型の変数をアンラップ処理するには、4つの方法があります。

### 1. Optional Binding

- ifやguardを使って、Optional型の変数がnilでないことを確認してアンラップする
- guardは関数の中でしか使えない
    - Optional型の変数がnilだった場合の処理を、else内に書く 
    - Optional型の変数がアンラップできた場合は、以降の行でアンラップ済みの変数を使える

```swift
func hello() -> String {
  let optionalStr: String? = "hello"
  if let unwrappedStr = optionalStr {
    return unwrappedStr.uppercased()
  }
  return ""
}

print( hello() )  // => HELLO
```

```swift
func helloWithGuard() -> String {
  let optionalStr: String? = "hello"
  guard let unwrappedStr = optionalStr else {
    return ""
  }
  return unwrappedStr.uppercased()
}

print( helloWithGuard() )  // => HELLO
```

### 2. Optional Chaining
- Optional Chainingは、アンラップした変数の型のメソッドを呼び出したい場合に使う方法

```swift
var optionalStr: String?
optionalStr?.uppercased()  // => nil
optionalStr = "hello"
optionalStr?.uppercased()  // => "HELLO"
```

```swift
if let userName = user?.profile?.name {
  print(userName)
}
```

### 3. Forced Unwrapping
- Forced Unwrappingとは、文字通り強制的にアンラップを行う方法
- Optional型の変数がnilだった場合にForced Unwrappingしてしまうと、ランタイムエラーが起こる（アプリの場合だったら落ちる）可能性がある
```swift
var optionalStr: String? = "hello"
optionalStr!.uppercased()  // => HELLO
```

### 4. ImplicitlyUnwrappedOptional型
- nilが代入可能な変数を定義する型として、ImplicitlyUnwrappedOptional型がある
- ImplicitlyUnwrappedOptional型の変数は、メソッド呼び出しの際に、通常の型と同じような呼び出し方で書ける
- ただし、Forced Unwrappingと同様に、ImplicitlyUnwrappedOptional型の変数がnilの場合、実行時エラーになる可能性がある

```swift
var implicitlyStr: String!  // nil
implicitlyStr = "hello"     // "hello"
implicitlyStr.uppercased()  // => "HELLO"
```
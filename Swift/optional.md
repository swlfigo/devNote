# Optional

## Optional使用

定义 Optional 类型，最常用的语法是「 Type? 」，如：

```swift
let age: Int? = 20
```

「 Type? 」是语法糖，其完整语法是：`Optional<Type>`

如上 `age`的完整定义是：

```swift
let age: Optional<Int> = Optional.some(20)
```



## Optional定义



官方源码： https://github.com/swiftlang/swift/blob/55189bae8e5516967998000deaa0e138a1c9f4fa/stdlib/public/core/Optional.swift#L121



```swift
@frozen public enum Optional<Wrapped> : ExpressibleByNilLiteral {

    /// The absence of a value.
    ///
    /// In code, the absence of a value is typically written using the `nil`
    /// literal rather than the explicit `.none` enumeration case.
    case none

    /// The presence of a value, stored as `Wrapped`.
    case some(Wrapped)

    /// Creates an instance that stores the given value.
    public init(_ some: Wrapped)

    // ...
}
```



`Optional` 是个泛型 Enum，含有 2 个 case：

- `none` ：代表「空」，即 nil

  ```swift
  let age: Int? = nil
  ```

  等价于：

  ```swift
  let age: Optional<Int> = .none    // Optional.none
  ```

- `some` ：代表「非空」，关联具体的值

  ```swift
  let age: Int? = 20
  ```

  等价于：

  ```swift
  let age: Optional<Int> = .some(20)
  ```

  或：

  ```swift
  let age: Optional<Int> = .init(20)
  ```



## Map

对于Optional Map的实现

```swift

    /// Evaluates the given closure when this `Optional` instance is not `nil`,
    /// passing the unwrapped value as a parameter.
    ///
    /// Use the `map` method with a closure that returns a non-optional value.
    /// This example performs an arithmetic operation on an
    /// optional integer.
    ///
    ///     let possibleNumber: Int? = Int("42")
    ///     let possibleSquare = possibleNumber.map { $0 * $0 }
    ///     print(possibleSquare)
    ///     // Prints "Optional(1764)"
    ///
    ///     let noNumber: Int? = nil
    ///     let noSquare = noNumber.map { $0 * $0 }
    ///     print(noSquare)
    ///     // Prints "nil"
    ///
    /// - Parameter transform: A closure that takes the unwrapped value
    ///   of the instance.
    /// - Returns: The result of the given closure. If this instance is `nil`,
    ///   returns `nil`.
  public func map<E: Error, U: ~Copyable>(
    _ transform: (Wrapped) throws(E) -> U
  ) throws(E) -> U? {
    switch self {
    case .some(let y):
      //返回一个Optional包着返回值
      return .some(try transform(y))
    case .none:
      return .none
    }
  }
```



## FlatMap

对于Optional FlatMap实现

```swift
 /// Evaluates the given closure when this `Optional` instance is not `nil`,
  /// passing the unwrapped value as a parameter.
  ///
  /// Use the `flatMap` method with a closure that returns an optional value.
  /// This example performs an arithmetic operation with an optional result on
  /// an optional integer.
  ///
  ///     let possibleNumber: Int? = Int("42")
  ///     let nonOverflowingSquare = possibleNumber.flatMap { x -> Int? in
  ///         let (result, overflowed) = x.multipliedReportingOverflow(by: x)
  ///         return overflowed ? nil : result
  ///     }
  ///     print(nonOverflowingSquare)
  ///     // Prints "Optional(1764)"
  ///
  /// - Parameter transform: A closure that takes the unwrapped value
  ///   of the instance.
  /// - Returns: The result of the given closure. If this instance is `nil`,
  ///   returns `nil`.

  public func flatMap<E: Error, U: ~Copyable>(
    _ transform: (Wrapped) throws(E) -> U?
  ) throws(E) -> U? {
    switch self {
    case .some(let y):
      return try transform(y)
    case .none:
      return .none
    }
  }
```



## 例子以及不同

来自Claude例子

### 不同点

1. map 的工作方式:

```swift
let optional: Int? = 5
let mapped = optional.map { $0 * 2 }  // 结果是 Optional(10)

let nilOptional: Int? = nil
let mappedNil = nilOptional.map { $0 * 2 }  // 结果是 nil
```

map 会:

- 如果 Optional 有值，**执行闭包并将结果包装在新的 Optional 中返回** , **注意是可能会存在嵌套Optinal返回情况！**
- 如果是 nil，直接返回 nil

1. flatMap 的工作方式:

```swift
let optional: Int? = 5
let flatMapped = optional.flatMap { Optional($0 * 2) }  // 结果是 Optional(10)

let nilOptional: Int? = nil
let flatMappedNil = nilOptional.flatMap { Optional($0 * 2) }  // 结果是 nil
```

flatMap 会:

- 如果 Optional 有值，**执行闭包，但不会对结果再次包装**
- 如果是 nil，直接返回 nil

主要区别在于:

- map 的闭包返回普通值，结果会被包装成 Optional
- flatMap 的闭包返回 Optional，**结果不会被再次包装，避免出现嵌套的 Optional**

一个实际的例子:

```swift
let numberString: String? = "42"

// 使用 map
let mapped = numberString.map { Int($0) }  // 结果是 Optional(Optional(42))

// 使用 flatMap
let flatMapped = numberString.flatMap { Int($0) }  // 结果是 Optional(42)
```

flatMap 特别适合处理可能返回 Optional 的转换，可以避免产生嵌套的 Optional 类型。这在处理类型转换、API 调用等场景下特别有用。

需要注意的是:

- 如果你的转换返回非 Optional 值，使用 map
- 如果你的转换返回 Optional 值，使用 flatMap



### 例子：

#### **在Map第一个例子中:**

1. 先看变量类型：

```swift
let optional: Int? = 5  // 类型是 Optional<Int>
```

1. 当我们调用 map 时，Swift 的类型系统会这样处理：

```swift
let mapped = optional.map { $0 * 2 }
// 闭包参数 $0 的类型是 Int（不是 Int?）
// 闭包返回值类型是 Int
// map 方法会将结果包装为 Optional<Int>
```



不会出现嵌套 Optional 的原因是：

1. map 在处理值时，已经通过可选绑定将原始的 Optional 解包了
2. 闭包接收的是解包后的值，而不是 Optional
3. map 方法会将闭包的返回值包装成一个新的 Optional，而不是在现有的 Optional 上再包装一层

相比之下，前面例子中出现嵌套 Optional 的情况是因为：

```swift
let numberString: String? = "42"
let mapped = numberString.map { Int($0) }  // Optional<Optional<Int>>
```

这里嵌套的原因是 **Int(_:) 初始化器本身就返回一个 Optional<Int>，然后 map 又包装了一层**。

所以关键点是：

- map 的闭包参数是解包后的值
- map 只在最后包装一次 Optional
- 不会在已有的 Optional 上再包装一层

这就是为什么 `optional.map { $0 * 2 }` 只会得到一个 Optional<Int> 而不是 Optional<Optional<Int>>。



#### 以下例子为什么不会通过编译:

这段代码编译会失败。让我们分析一下原因：

```swift
func transformURL(_ url: String?) -> URL? {
    url.map { URL(string: $0) }
}
```

问题在于类型不匹配：

1. 输入分析：

- 参数 `url` 的类型是 `String?` (Optional<String>)
- map 闭包接收解包后的 `String`

1. 返回值分析：

- `URL(string:)` **初始化器返回的是 `URL?` (Optional<URL>)**
- map 会将闭包返回值再包装一层 Optional
- **最终返回类型会变成 `Optional<Optional<URL>>` (URL??)**

1. 而函数声明的返回类型是 `URL?`，所以类型不匹配

正确的做法有两种：

1. 使用 flatMap：

```swift
func transformURL(_ url: String?) -> URL? {
    url.flatMap { URL(string: $0) }
}
```

1. 使用 flatMap 的原因是：

- flatMap 适合处理闭包返回 Optional 的情况
- 它会"压平"嵌套的 Optional，避免得到 Optional<Optional<URL>>
- 最终返回类型就是我们想要的 URL?

所以这里：

- ✅ 用 flatMap：一层 Optional，返回 URL?
- ❌ 用 map：两层 Optional，返回 URL??，类型不匹配
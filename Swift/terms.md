# Swift コーディング規約

**対象バージョン:** Swift 6.0.3

## 序文

このドキュメントは、一貫性があり、読みやすく、保守性の高い Swift コードを作成するための規約を定めます。コードの品質を維持し、チーム開発を円滑に進めるために、すべての開発者はこの規約に従ってください。

この規約は、`.swift-format` による自動フォーマットと静的解析ルールに基づいています。規約の多くはツールによって自動的に強制されますが、設計原則や命名規則など、開発者の判断が必要な項目も含まれます。

## 1. 基本原則

これらの原則は、高品質なコードを書くための基本的な考え方を示します。

1.  **明確性 (Clarity):** コードは、他の開発者（将来の自分自身を含む）が容易に理解できるように書かれるべきです。意図が明確でないコードは避けてください。
2.  **簡潔性 (Conciseness):** 不必要な複雑さを避け、可能な限りシンプルに記述します。ただし、明確性を犠牲にしてまで短くする必要はありません。
3.  **一貫性 (Consistency):** プロジェクト全体で一貫したスタイル、命名規則、構造を維持します。これにより、コードベースの予測可能性が高まります。
4.  **保守性 (Maintainability):** コードは将来の変更や修正が容易であるように設計します。適切なドキュメント、テスト、モジュール化が重要です。
5.  **安全性 (Safety):** クラッシュや予期せぬ動作を防ぐため、Swift の型安全性やエラー処理メカニズムを最大限に活用します。

## 2. コードの構成と構造

### 2.1. ファイル構成

- **原則:** 1 ファイルにつき、主要な型（`class`, `struct`, `enum`, `protocol`）は 1 つだけ定義します。
  - 関連する小さなヘルパー型や拡張は、主要な型と同じファイルに含めても構いません（例: `privateな関数のみを含むようなextensionなど`）。
- **ファイル名:** ファイル名は、そのファイルに含まれる主要な型の名前と一致させます（例: `UserProfile.swift`）。

### 2.2. ファイル内の要素順序

コードの可読性を高めるため、ファイル内の要素は以下の順序で配置することを推奨します。

1.  `import` 文（アルファベット順にソートされます。）
2.  ファイルヘッダーコメント（必要な場合）
3.  主要な型定義 (`class`, `struct`, `enum`, `protocol`)
    - `MARK: - Properties`
      - `public`/`internal` ストアドプロパティ
      - `fileprivate`/`private` ストアドプロパティ
      - `public`/`internal` コンピューテッドプロパティ
      - `fileprivate`/`private` コンピューテッドプロパティ
      - `lazy` プロパティ
      - `@IBOutlet` など（UI 関連の場合）
    - `MARK: - Initialization`
      - イニシャライザ (`init`, `required init`, `convenience init`)
    - `MARK: - Public Methods` / `MARK: - Internal Methods`
      - 公開/内部インターフェースとなるメソッド
    - `MARK: - Private Methods` / `MARK: - Fileprivate Methods`
      - 実装詳細となるメソッド
    - `MARK: - Subtypes`
      - ネストされた型 (`enum`, `struct`, `class`)
    - `MARK: - Protocol Conformance` (各プロトコルごとに)
      - プロトコルへの準拠に関連するメソッドやプロパティ

- `MARK:`, `TODO:`, `FIXME:` コメントを使用して、コードのセクションを論理的に分割し、ナビゲーションを容易にします。

### 2.3. アクセス制御 (Access Control)

- **原則:** 常に最小限のアクセスレベルを使用します（Principle of Least Privilege）。
- **トップレベル定義の明記:** **トップレベルの関数、型（クラス、構造体、列挙型、プロトコル）、変数は、アクセス制御（`public`, `internal`, `fileprivate`, `private`）を常に明記してください。**

  - `internal` が適切な場合でも、それが意図的な選択であることを示すために明記します。
  - `.swift-format` の `fileScopedDeclarationPrivacy` ルールによりトップレベル宣言は `private` に設定される場合がありますが、公開またはモジュール内部で共有する意図がある場合は `public` または `internal` を明記する必要があります。

  ```swift
  // 良い例
  public var sharedConfiguration: Configuration
  internal struct UtilityHelpers { /* ... */ }
  private func processInternalData(_ data: Data) { /* ... */ }

  // 悪い例 (アクセスレベルが省略されている)
  // var globalCounter: Int // internal か private か意図が不明確
  ```

- **型内部の定義:** 型（クラス、構造体など）の内部で定義されるプロパティやメソッドについては、デフォルトのアクセスレベル（通常は `internal`）が適切であれば、アクセス制御を省略しても構いません。ただし、より制限されたアクセスレベル (`private`, `fileprivate`) が必要な場合は明記してください。

  ```swift
  internal struct DataProcessor {
      var buffer: Data // internal (省略可)
      private var internalState: State // private (明記が必要)

      func process() { /* ... */ } // internal (省略可)
      private func cleanup() { /* ... */ } // private (明記が必要)
  }
  ```

- `extension` 宣言自体にはアクセスレベルを指定しません。拡張された型のアクセスレベルは、元の型のアクセスレベルに従います。ただし、拡張内で定義されるメソッドやプロパティには、必要に応じてアクセスレベルを指定します。

  ```swift
  // 良い例
  extension String {
      func reversed() -> String { /* ... */ } // internal (省略可)
      private func internalHelper() { /* ... */ } // private (明記が必要)
  }

  // 悪い例 (extension 自体にアクセスレベルを指定している)
  public extension String { /* ... */ } // 不要な public 指定
  ```

## 3. 命名規則

明確で一貫性のある命名は、コードの可読性を大幅に向上させます。

- **型 (クラス、構造体、プロトコル、列挙型、型エイリアス):** `UpperCamelCase` (PascalCase)
  - 例: `UserProfile`, `NetworkService`, `Identifiable`, `NetworkError`, `UserID`
- **関数、メソッド、変数、定数、列挙型のケース:** `lowerCamelCase`
  - 例: `fetchUserData()`, `userName`, `maximumLoginAttempts`, `case networkUnavailable`
- **プライベート/ファイルプライベートなプロパティ/メソッド:**
  - **推奨:** 先頭にアンダースコア (`_`) を付けます。これは、内部的な実装詳細であることを示すためです。(`.swift-format` の `NoLeadingUnderscores` は `false` なので許可されていますが、このプロジェクトではプライベート要素に限定して使用します。)。
  - 例: `private var _cachedData`, `fileprivate func _processInternally()`
- **定数:**
  - グローバル定数や静的定数も `lowerCamelCase` を使用します。
  - 例: `static let defaultTimeout: TimeInterval = 30.0`
- **真偽値 (Bool):** `is`, `has`, `should`, `did`, `will` などの接頭辞を使用すると、意味が明確になります。
  - 例: `isUserLoggedIn`, `hasUnsavedChanges`, `shouldRetryRequest`
- **省略語:** 一般的に認知されている省略語（URL, ID, JSON など）を除き、極力避けます。省略語を使う場合、命名規則（UpperCamelCase/lowerCamelCase）に従います。
  - 例: `userID`, `processJSONData()`, `URLSession`
- **型名とプロパティ名:** 静的プロパティ名で型名を繰り返さないでください。
  - 良い: `UIColor.red`
  - 悪い: `UIColor.redColor`

### 3.1 Prefixes
Swiftの型は全てモジール単位の名前空間になるため、Swiftの型にプレフィックスを付ける必要はありません。
命名衝突は、モジュール名と型名を付ける事で明確にすることができます。

## 4. コーディングスタイルとフォーマット

コードフォーマットの一貫性は `.swift-format` によって大部分が自動的に強制されます。以下は、そのルールに基づいた主要なスタイルガイドラインと、コードの可読性を高めるための推奨事項です。

### 4.1. フォーマット

- **インデント:** 4 スペースを使用します。
- **改行:**
  - 制御フローキーワード (`if`, `else`, `while` など) の前に改行を入れます。
  - 複数行にわたる関数の引数やジェネリック要件は、各要素の前に改行を入れます。
  - 複数行にわたるメソッドチェーンでは、各コンポーネントの前後で改行します 。
- **空行:** 連続する空行は最大 1 行までとします。
- **カンマ:** 複数行のコレクションリテラルや引数リストでは、最後の要素の後にもカンマを付けます。
- **括弧:** 条件式の周りの括弧は不要です。
- **セミコロン:** 行末のセミコロンは使用しません。
- **リテラル:**
  - 空のコレクションの初期化にはリテラル構文を使用します。
  - 数値リテラルは、必要に応じてアンダースコア `_` でグループ化します。
- **型:**
  - 可能な場合は型推論を使用しますが、明確性が損なわれる場合は型を明示します。
  - `Array<String>` や `Dictionary<String, Int>` ではなく、糖衣構文 `[String]` や `[String: Int]` を使用します。
  - 関数の戻り値が `Void` の場合、`-> Void` を明記します。
- **型指定のコロン:** 識別子に型を指定する際は、識別子の直後にコロンを置き、スペースを 1 つ空けて型名を記述します。

  ```swift
  // 良い例
  let count: Int = 10
  var name: String?
  func process(data: Data) -> Result<Success, Failure> { /* ... */ }
  let mapping: [String: Int] = ["one": 1, "two": 2]

  // 悪い例
  let count: Int = 10
  func process(data: Data) -> Result<Success, Failure> { /* ... */ }
  ```

- **演算子定義の空白:** カスタム演算子を定義する際は、演算子の識別子の前後にスペースを 1 つずつ入れます。

  ```swift
  // 良い例
  infix operator +-: AdditionPrecedence
  func +- (lhs: Vector, rhs: Vector) -> Vector { /* ... */ }

  // 悪い例
  func +-(lhs: Vector, rhs: Vector) -> Vector { /* ... */ }
  ```

- **Control Flow:**
  - `for` ループでは、`for-in`を使用します。

```swift
  // 良い例
  for item in items {
      print(item)
  }

  // 悪い例
  for var i = 0; i < items.count; i += 1 {
      print(items[i])
  }
```


- **その他:**
  - ブロックコメント (`/* ... */`) は使用せず、行コメント (`//`) またはドキュメンテーションコメント (`///`) を使用します。
  - 式の中での代入は行いません。
  - セミコロンを使用して複数の文を 1 行にまとめることは避けてください。

### 4.2. 関数とメソッド

- **単一責任の原則 (Single Responsibility Principle):** 1 つの関数/メソッドは、1 つの明確な責務を持つべきです。長すぎる、または複数のことを行う関数は分割します。
- **純粋関数:** 可能な限り、関数は純粋（同じ入力に対して常に同じ出力を返し、副作用がない）であるように努めます。副作用がある場合は、それが明確にわかるようにします。
- **早期リターン:** 処理を続行するための条件をチェックする場合、ネストを深くする `if` 文よりも `guard` 文を使用して早期にリターン（または `throw`, `break`, `continue`）することを強く推奨します。

  - `guard` は、条件が満たされない場合にスコープから必ず抜けることをコンパイラが保証するため、コードパスが明確になります。

  ```swift
  // 推奨 (guard を使用)
  func process(user: User?) throws {
   guard let user = user else {
       throw UserError.notFound
   }
   guard user.isActive else {
       print("\(user.name) is inactive.")
       return
   }
   guard let data = fetchData(for: user) else {
       logError("Failed to fetch data for \(user.name)")
       return
   }

   // user と data を使った処理 (ネストが浅い)
   processUserData(data)
  }

  // 非推奨 (深い if ネスト)
  func process_alternative(user: User?) throws {
   if let user = user {
       if user.isActive {
           if let data = fetchData(for: user) {
               // user と data を使った処理 (ネストが深い)
               processUserData(data)
           } else {
               logError("Failed to fetch data for \(user.name)")
           }
       } else {
           print("\(user.name) is inactive.")
       }
   } else {
       throw UserError.notFound
   }
  }
  ```

- **引数ラベル:** 関数の呼び出し箇所で意味が明確になるように、適切な引数ラベルを使用します。将来的にオプションが増える可能性などがなく、その関数の役割が間違いなく変化しないため、ラベルが不要な場合は `_` で省略します。
  ```swift
  // 良い例
  func fetchUserProfile(for userID: UserID) { /* ... */ }

  // 明らかな役割を持つ場合
  func addBang(_ string: String) -> String { /* ... */ }

  // 悪い例
  func fetchUserProfileFor(userID: UserID) { /* ... */ }
  ```

- **クロージャ:** クロージャの引数が 1 つだけの場合、引数ラベルを省略します。クロージャの引数が 1 つだけで、型が明確な場合は、`$0` を使用して省略できます。

  ```swift
  // 良い例
  let numbers = [1, 2, 3]
  let doubled = numbers.map { $0 * 2 } // 引数ラベルを省略

  // 悪い例
  let doubled_alternative = numbers.map { number in number * 2 } // 引数ラベルを明示
  ```
  - **TrailingClosures構文:** クロージャが関数の最後の引数である場合、クロージャをトレーリングクロージャとして記述します。これにより、コードがより読みやすくなります。

```swift
  return someFunctionWithClosure { (value) in
      // クロージャの処理
  }
```

### 4.3. 変数と定数

- **不変性 (`let` の優先):**
  - 可能な限り、常に `let` を使用して定数を宣言してください。値が変更される必要があることが明確な場合にのみ `var` を使用します。迷った場合は `let` を選び、後で必要になったら `var` に変更します。
  - **理由:**
    - `let` は値が不変であることを保証し、コードを読む人に対してその意図を明確に伝えます。これにより、変数の状態を追跡する必要がなくなり、コードの理解が容易になります。
    - 不変性は副作用を減らし、特に並行処理においてコードをより安全にします。
    - `var` が使われている箇所は、値が変化する可能性がある特別な場所であることを示唆するため、注意を促します。
- **スコープ:** 変数や定数のスコープは、必要最小限に留めます。
- **オプショナル (`Optional`):**
  - 値が存在しない可能性がある場合にのみ使用します。
  - **安全なアンラップ:** オプショナル型の値を使用する際は、強制アンラップ (`!`) を避け、以下の安全な方法を優先します。
    - オプショナル束縛 (`if let`, `guard let`)
    - オプショナルチェーン (`?.`)
    - `nil` 合体演算子 (`??`)
  - **強制アンラップ (`!`) の回避:** `foo!` のような強制アンラップは、値が絶対に `nil` でないと**論理的に 100%保証**できる場合を除き、**使用しないでください**。ランタイムクラッシュの原因となります。
  - **暗黙的アンラップオプショナル (`ImplicitlyUnwrappedOptional`, 例: `String!`) の回避:** `let foo: FooType!` のような宣言は、初期化時に `nil` だが後で必ず値が設定され、以降 `nil` にならないことが保証される特殊なケース（例: 一部のライフサイクル依存プロパティ）を除き、**原則として使用しないでください**。代わりに通常のオプショナル (`FooType?`) と安全なアンラップを使用します。これもランタイムクラッシュのリスクがあります。
  - `nil` チェック: 値を使う場合はオプショナル束縛を使用してください。
  - **`if let` と `guard let`:** オプショナルの値を使用する場合、`if let` や `guard let` を使用して安全にアンラップします。これにより、値が `nil` の場合でも安全に処理できます。

### **4.4. プロパティ**

- **読み取り専用プロパティ/添え字の暗黙的 getter:** 読み取り専用のコンピューテッドプロパティや添え字を定義する場合、`get { ... }` ブロックを省略し、直接計算結果を返すように記述します。

```swift
    // 推奨
    var itemCount: Int {
        return items.count
    }
    subscript(index: Int) -> Item {
        return items[index]
    }

    // 非推奨
    var itemCount_alternative: Int {
        get {
            return items.count
        }
    }
    subscript(index: Int) -> Item {
        get {
            return items[index]
        }
    }
```

-   **理由:** 記述が簡潔になり、読み取り専用であることがより明確になります。

### **4.5. `self` の使用**

- **明示的な `self` は必要な時だけ:** プロパティやメソッドにアクセスする際、`self.` はデフォルトで省略します。
- **明示的な `self` が必要な場合:**

  - クロージャ内でインスタンスメンバーをキャプチャする場合（`[weak self]` や `[unowned self]` と併用されることが多い）。
  - イニシャライザやメソッドのパラメータ名とインスタンスプロパティ名が衝突する場合。

  ```swift
  class Counter {
      var count: Int = 0

      func increment() {
          count += 1 // self は省略
      }

      init(count: Int) {
          self.count = count // パラメータ名と衝突するため self が必要
      }

      func incrementLater() {
          DispatchQueue.main.async { [weak self] in
              guard let self = self else { return }
              self.count += 1 // クロージャ内でキャプチャするため self が必要
          }
      }
  }
  ```

  - **理由:** `self` を省略することで、コードが簡潔になります。必要な箇所でのみ `self` を使用することで、特にクロージャでのキャプチャが明確になります。

### **4.6. ジェネリクス**

- **型パラメータの省略:** ジェネリック型やメソッド内で、引数や戻り値の型パラメータがその型のパラメータと同じ場合、型パラメータを省略できます。

  ```swift
  struct Pair<T> {
      let first: T
      let second: T

      // 推奨: other や戻り値の型パラメータ <T> を省略
      func swapped() -> Pair {
          return Pair(first: second, second: first)
      }
      func isEqual(to other: Pair) -> Bool {
           // ... 比較ロジック ...
           return self.first == other.first && self.second == other.second // Equatable 準拠が必要
      }

      // 非推奨: 型パラメータを冗長に記述
      func swapped_alternative() -> Pair<T> {
          return Pair<T>(first: second, second: first)
      }
       func isEqual_alternative(to other: Pair<T>) -> Bool {
           // ...
       }
  }
  ```

  - **理由:** コードが簡潔になり、型推論が可能な場合は冗長な記述を避けることができます。異なる型パラメータが関わる場合にのみ明示することで、その違いが際立ちます。

## 5. ドキュメンテーションとコメント

### 5.1. ドキュメンテーションコメント (`///`)

- **必須:** すべての `public` および `open` な宣言（クラス、構造体、列挙型、プロトコル、メソッド、プロパティなど）には、ドキュメンテーションコメントを記述します。
- **推奨:** `internal` な要素でも、API として利用されるものや複雑なロジックを持つものにはドキュメントを追加します。
- **形式:** Triple-slash (`///`) を使用します。
- **内容:**
  - 最初の行には、要素の目的を簡潔に説明する一行要約を記述します。
  - 必要に応じて、詳細な説明、パラメータ (`- Parameters:`, `- Parameter A:`), 戻り値 (`- Returns:`), スローするエラー (`- Throws:`) を記述します。
  - Xcode の Documentation Preview (Option + Command + /) で正しく表示されるように記述します。

```swift
/// 指定されたユーザーIDのプロファイルを非同期に取得します。
///
/// ネットワーク接続がない場合や、ユーザーが見つからない場合にエラーをスローすることがあります。
/// - Parameter userID: 取得するユーザーのID。
/// - Returns: 成功した場合は `UserProfile` オブジェクト。
/// - Throws: `NetworkError` または `UserNotFoundError`。
func fetchUserProfile(for userID: UserID) async throws -> UserProfile {
    // ... implementation ...
}
```

### 5.2. 通常コメント (`//`)

- **目的:** コードが _なぜ_ そのようになっているのか、複雑なアルゴリズム、特定の決定の背景などを説明するために使用します。コードが _何_ をしているかは、コード自体が明確に語るべきです。
- **使用箇所:** 必要な場合にのみ使用し、コードの可読性を補完します。古くなったコメントや、コードを見れば明らかなことを説明するコメントは削除します。
- **`MARK:`, `TODO:`, `FIXME:`:**
  以下の構文はコメントの中で、特に重要な部分や、将来のタスクを示すのに使えます。
  - `MARK: - Section Name`: コードブロックを視覚的に区切り、Xcode のジャンプバーに表示させます。ハイフン `-` を含めると区切り線が表示されます。
  - `TODO: Description`: 将来実装すべきタスクや改善点を示します。
  - `FIXME: Description`: 既知の問題や修正が必要な箇所を示します。

## 6. エラー処理

- **エラー型:** カスタムエラー型は `Error` プロトコル（可能であれば `LocalizedError` も）に準拠させ、エラーの原因が明確にわかるようにします。列挙型 (`enum`) を使用するのが一般的です。
- **`do-catch`:** エラーが発生する可能性のある処理を呼び出す場合、`do-catch` 文を使用してエラーを捕捉し、適切に処理します。エラー情報を失わないように、`try?` よりも `do-catch` を優先します (`NeverUseForceTry` は `true` なので `try!` は禁止)。
  - `try?` は、エラーの詳細が不要で、単に `nil` として扱いたい場合に限定して使用します。
- **`Result` 型:** 非同期処理や、エラーが発生する可能性のある処理で、成功時の値またはエラーを明示的に返したい場合に `Result<Success, Failure>` 型の使用を検討します。
- **エラーメッセージ:** エラーが発生した場合、ユーザーまたは開発者にとって有用な情報（原因、可能な対策など）を提供します。

## 7. メモリ管理と非同期処理

### 7.1. メモリ管理

- **循環参照:** クロージャがクラスインスタンスのプロパティであり、そのクロージャが `self` をキャプチャする場合、循環参照が発生する可能性があります。
  - クロージャが `self` より長く生存する可能性がある場合、`[weak self]` を使用して `self` を弱参照でキャプチャします。クロージャ内で `self` を使用する際は `guard let self = self else { return }` などでアンラップします。
  - クロージャと `self` が常に同時に破棄されることが保証されている場合に限り、`[unowned self]` を使用できますが、`weak` の方が安全な場合が多いです。

### 7.2. 非同期処理

- **`async/await`:** Swift 5.5 以降で導入された `async/await` を可能な限り使用し、非同期コードを構造化し、可読性を高めます。コールバック地獄 (`Callback Hell`) を避けます。
- **Combine:** Combine フレームワークを使用する場合は、以下の点に注意します。
  - `AnyCancellable` インスタンスは、不要になったときに適切にキャンセルされるように、セット (`Set<AnyCancellable>`) などで管理し、所有するオブジェクトが破棄されるタイミング (`deinit`) などでキャンセル (`.cancel()`) または解放 (`.store(in: &cancellables)`) します。
  - オペレータチェーンを適切に管理し、メモリリークを防ぎます。

### 7.3. Sendable とデータ競合の安全性 (Swift 6)

**Swift 6 では、データ競合の安全性がコンパイラによって強制されます。** これは、複数のスレッドやタスクが同時に共有された可変状態に安全でない方法でアクセスするのを防ぐことを目的としています。

- **`Sendable` プロトコル:**
  - `Sendable` は、型がアクター境界を越えて安全に共有（コピーまたは不変参照として）できることを示すマーカープロトコルです。
  - `async` 関数やアクターメソッドの引数や戻り値は、通常 `Sendable` である必要があります。
- **`Sendable`準拠:**
  - **値型 (Struct, Enum):** その要素がすべて `Sendable` であれば、自動的に `Sendable` に準拠します。
  - **アクター (`actor`):** アクター型自体は `Sendable` です（アクター自身への参照は安全に共有できるため）。
  - **クラス (`class`):**
    - `final` であり、すべてのインスタンスプロパティが不変 (`let`) かつ `Sendable` な型である場合、`Sendable` に準拠できます。
    - 内部で同期（例: `NSLock` やアトミック操作）を行っている場合でも、基本的には `Sendable` ではありません。状態を保護するにはアクターを使用するか、不変性を保証する必要があります。
    - ミュータブルな状態を持つクラスを安全に共有する必要がある場合は、アクターでラップすることを検討します。
- **コンパイラ警告/エラーへの対応:**
  - Swift 6 では、`Sendable` でない型をアクター境界を越えて渡そうとすると、コンパイラエラーが発生します（以前のバージョンでは `-warn-concurrency` フラグで警告）。
  - これらのエラー/警告を解決するには、型を `Sendable` に準拠させるか、アクターを使用して状態をカプセル化します。
- **`@unchecked Sendable`:**
  - 型が実際にはスレッドセーフであるが、コンパイラがそれを静的に証明できない場合（例: Objective-C のクラスや内部でロックを使用しているクラス）、`@unchecked Sendable` を使用してコンパイラチェックを抑制できます。
  - **これは最後の手段であり、使用には細心の注意が必要です。** 開発者は、その型が本当に `Sendable` であることを手動で保証する責任を負います。乱用はデータ競合を引き起こす可能性があります。
- **`nonisolated(unsafe)`:**
  - アクター内のプロパティにアクター外から同期なしでアクセスするための属性ですが、**非常に危険であり、原則として使用を避けてください。** データ競合の直接的な原因となります。
- **グローバル状態:**
  - グローバル変数やシングルトンは、特に可変状態を持つ場合、データ競合の一般的な原因となります。
  - 可能な限りグローバルな可変状態を避け、依存性注入などを使用します。
  - グローバル状態が必要な場合は、アクターを使用してその状態へのアクセスを保護します（例: `actor ConfigurationManager { static let shared = ConfigurationManager() ... }`）。

## 8. テストコード (Swift Testing)

テストコードは、コードの品質を保証し、リファクタリングを容易にし、ドキュメントとしても機能する重要な要素です。Swift 6 では Swift Testing が標準ライブラリの一部となり、より簡潔で強力なテストが記述可能です。

### 8.1. 基本方針

- **フレームワーク:** Swift Testing を使用します。**Swift 6 以降、Swift Testing は標準ライブラリの一部となり、外部依存関係を追加する必要はありません。**
- **テストの配置:** テストコードは一般的にプロジェクトの `Tests` ディレクトリに配置します。
- **XCTest:** 新規テストでは使用しません。既存の XCTest コードは、可能であれば Swift Testing に移行することを検討します。

### 8.2. 構造と記述

- **テスト対象へのアクセス:** テストターゲットからプロダクションコードの `internal` な要素にアクセスするには、テストファイルの先頭で `@testable import モジュール名` のように宣言します。

```swift
import Testing // Swift Testing フレームワーク (Swift 6 では標準)
@testable import YourAppModule // テスト対象モジュール
// Foundation など、テストに必要な他のモジュールも import
```

- **テストスイート (`@Suite`):**

  - テスト対象のクラス、構造体、または機能ごとにテストファイルを作成します。
  - ファイル内では、テスト対象の機能ごとに `@Suite` マクロを使用してテストスイート（構造体）を作成します。スイート名はテスト対象を明確に示す文字列を指定します（例: `@Suite("ユーザー認証")`, `@Suite("計算機能")`）。
  - スイート内では、テストに必要なモック、スタブ、ヘルパーメソッド、テスト用の拡張などを定義できます。
  - スイートは `struct` `class` `actor` のいずれでも使用できます。推奨は `struct` です。
  - スイートは入れ子にすることができ、テスト対象の機能やクラスごとに階層的に整理できます。

- **テストケース (`@Test`):**

  - 各スイート内で、個別のテストケース（関数）を `@Test` マクロを使用して定義します。
  - `@Test` の引数には、そのテストケースが検証する具体的なユースケースや振る舞いを説明する文字列を指定します（例: `@Test("有効なデータで計算が成功する")`, `@Test("無効な入力でエラーが発生する")`）。
  - テストケース名は `test` で始まる必要はありません。関数名はテスト内容を反映したわかりやすいものにします。
  - テストケースはパラメータ化することも可能です (`@Test(arguments: ...)` を参照)。

### 8.3. テストの書き方: パターンとアサーション

#### 8.3.1. テストのための準備

- **モックとスタブ:** 依存関係を持つコンポーネントをテストする場合、依存対象をモックやスタブに置き換えて、テスト対象の動作を独立して検証します。これらはテストスイート内で定義するか、専用のヘルパーとして作成します。
- **`Equatable` 準拠:** テスト内でオブジェクトの状態を比較検証する必要がある場合、テスト対象の型（特に struct や enum）に `Equatable` プロトコルを準拠させると便利です。プロダクションコードに影響を与えずに行うために、**テストターゲット内でのみ** `extension` を使って準拠させることができます。

  ```swift
  // テストファイル内
  @testable import YourAppModule

  // テストターゲット内でのみ User を Equatable に準拠させる
  extension User: Equatable {
      public static func == (lhs: User, rhs: User) -> Bool {
          return lhs.id == rhs.id && lhs.name == rhs.name
      }
  }

  @Suite("ユーザーモデル")
  struct UserTests {
      @Test("同一内容のユーザーは等価であること")
      func equalityCheck() {
          let user1 = User(id: "123", name: "Test User")
          let user2 = User(id: "123", name: "Test User")
          #expect(user1 == user2, "同一内容のユーザーは等価であるべき")
      }
  }
  ```

#### 8.3.2. アサーション: 期待値の検証

テストケース内では、コードの実行結果が期待通りであることを検証（アサート）する必要があります。

- **`#expect()` マクロ (推奨):**
  - Swift Testing で**最も推奨されるアサーション方法**です。
  - `#expect(expression)` の形式で使用し、`expression` が `true` と評価されることを期待します。
  - **利点:** テストが失敗した場合、単純な `true`/`false` だけでなく、式の内容、比較対象の値など、**非常に詳細な診断情報** を出力するため、問題の特定が容易になります。テストは失敗しますが、テスト実行は継続されます。
  - オプションで失敗時のカスタムメッセージも追加できます: `#expect(result == 5, "加算結果が期待値と異なります")`。
- **`#require()` マクロ:**
  - このマクロは、特定の条件が満たされていることを要求するために使用されます。条件が満たされない場合、テストは失敗し、中断します。
  - `#require(condition)` の形式で使用し、`condition` が `true` と評価されることを期待します。
  - **利点:** テストが失敗した場合、テスト実行は中断され、エラーメッセージが表示されます。これは、特定の条件が満たされない場合にテストを続行する必要がない場合に便利です。
- **`Issue.record("Message")`**
  - テストの実行中に特定の情報を記録するために使用されます。これは、どこでも無条件でテスト失敗を記録することができます。Guard節内などで便利です。
  - **利点:** テストの実行中に特定の情報を記録することで、後で分析やデバッグに役立てることができます。
- **`assert()` / `assertionFailure()`:**
  - これらは標準ライブラリの関数で、主にプロダクションコード内のデバッグビルド時に、満たされるべき条件をチェックするために使用されます（リリースビルドでは通常無効化される）。
  - **テストコードでの使用は非推奨です。** テストフレームワークは失敗をレポートし、テスト実行を継続するべきですが、`assert` はプログラムを停止させてしまう可能性があります。アサーションには `#expect` を使用してください。
- **`precondition()` / `preconditionFailure()`:**
  - これらも標準ライブラリの関数で、デバッグビルド・リリースビルドに関わらず条件をチェックし、満たされない場合はプログラムを停止させます。関数の事前条件など、致命的なエラーを示すために使用されます。
  - **テストコードでの使用は非推奨です。** アサーションには `#expect` を使用してください。
- **`XCTAssert` 系関数:** これらは XCTest フレームワークの関数であり、Swift Testing を使用するプロジェクトでは**使用しません**。

#### 8.3.3. 非同期コードのテスト

- アクター (`actor`) や `async` 関数など、非同期処理を含むコードをテストする場合、テストケースの関数も `async` として宣言します。
- テスト関数内で `await` を使用して非同期処理の結果を待機し、その結果を `#expect` で検証します。

  ```swift
  @Suite("データサービス")
  struct DataServiceTests {
      @Test("データ取得が成功すること")
      func dataFetchSuccess() async { // テスト関数を async に
          let service = DataService()
          let result = await service.fetchData() // await で待機
          #expect(!result.isEmpty, "取得したデータは空であるべきではない")
      }
  }
  ```

#### 8.3.4. エラー処理のテスト

- エラーをスローする可能性のある関数 (`throws`) をテストする場合、いくつかの方法があります。
- **エラーがスローされないことの期待:** `try` を使って関数を呼び出し、その結果を `#expect` で検証します。テスト関数自体も `throws` で宣言する必要があります。

  ```swift
  @Test("有効な入力値の処理")
  func validInputProcessing() throws { // テスト関数も throws
      let processor = DataProcessor()
      let result = try processor.process("valid-data") // try で呼び出し
      #expect(result.isValid, "処理結果が有効であるべき")
  }
  ```

- **特定のエラーがスローされることの期待:** `#expect(throws:)` マクロを使用します。これにより、指定したコードブロックが特定のエラー型をスローすることを検証できます。

  ```swift
  @Test("無効な入力値でエラーが発生すること")
  func invalidInputThrowsError() {
      let processor = DataProcessor()
      // ValidationError 型のエラーがスローされることを期待
      #expect(throws: ValidationError.self) {
          _ = try processor.process("invalid-data")
      }
  }

  // もしくは、特定の catch ブロックで期待するエラーかを確認する方法
  @Test("無効な入力値で特定のエラーが発生すること")
  func invalidInputThrowsSpecificError() {
      let processor = DataProcessor()
      do {
          _ = try processor.process("invalid-data")
          #expect(false, "無効な入力ではエラーがスローされるべき") // ここに到達したら失敗
      } catch let error as ValidationError {
           // 期待するエラー型とケースを確認
           #expect(error == .invalidFormat)
      } catch {
           #expect(false, "予期しない型のエラーがスローされた: \(error)")
      }
  }
  ```

#### 8.3.5. JSON エンコード/デコードテスト (ラウンドトリップ)

- `Codable` に準拠したモデルオブジェクトが正しく JSON との間で変換できるかを確認するには、ラウンドトリップテストが有効です。
- オブジェクトを JSON データにエンコードし、その JSON から再度オブジェクトにデコードして、元のオブジェクトとプロパティが一致するかを検証します。

  ```swift
  @Suite("モデルの変換テスト")
  struct ModelCodableTests {
      @Test("モデルの JSON 変換が成功すること")
      func jsonRoundTripTest() throws {
          let model = SampleModel(id: "123", name: "テスト")

          // 1. エンコード
          let encoder = JSONEncoder()
          let jsonData = try encoder.encode(model)

          // 2. デコード
          let decoder = JSONDecoder()
          let decoded = try decoder.decode(SampleModel.self, from: jsonData)

          // 3. 検証
          #expect(decoded.id == model.id)
          #expect(decoded.name == model.name)
      }
  }
  ```

#### 8.3.6. コールバックAPIのテスト
- `withCheckedContinuation` を使用して、非同期APIのコールバックをテストすることができます。これにより、非同期処理を同期的に待機し、結果を検証できます。

```swift
@Test("コールバックでデータをロードする")
func loadDataWithCallback() async {
    let expectation = #expectation(description: "データがロードされること")
    var result: Data?

    // コールバックAPIを使用してデータを取得
    await withCheckedContinuation { continuation in
        dataService.loadData { data in
            result = data
            expectation.fulfill()
            continuation.resume()
        }
    }

    // 結果を検証
    #expect(result != nil, "データが正常にロードされるべき")
}

```

### 8.4. テスト例

- パラメータ化されたテストケースを使用して、同じテストロジックを異なる入力値で繰り返し実行できます。

```swift
@Suite("数学関数")
struct MathFunctionTests {
    @Test("整数加算のテスト", arguments: [
        (2, 3, 5), (0, 0, 0), (-1, 1, 0)
    ])
    func additionTest(a: Int, b: Int, expectedSum: Int) {
        let result = add(a, b)
        #expect(result == expectedSum, "\(a) + \(b) は \(expectedSum) と等しいはず")
    }

    @Test("値が範囲内であることを確認")
    func valueInRangeTest() {
        let calculator = Calculator()
        let result = calculator.calculate(5)
        #expect(result >= 0 && result <= 10, "結果は0から10の範囲内であるべき")
    }
}
```

## 9. 依存関係管理

- Swift Package Manager (SPM) を使用して外部ライブラリを管理します。
- `Package.swift` ファイルを適切に管理し、依存関係を最新の状態に保ちます。

## 10. コードレビュー

- すべてのコード変更は、マージされる前にコードレビューを受ける必要があります。
- レビュー担当者は、このコーディング規約への準拠、コードの品質、設計の妥当性、テストカバレッジなどを確認します。
- 建設的なフィードバックを提供し、受け入れる文化を醸成します。

## 11. Swift 6 特有の考慮事項 (追加)

### 11.1. 非コピー可能型 (`~Copyable`)

- Swift 6 では、`~Copyable` キーワード（または `Copyable` 制約の削除）により、**非コピー可能型** を定義できます。これは、ファイルハンドル、ネットワーク接続、ロックなど、コピーされるべきでないリソースを表現するのに役立ちます。
- **目的:** 所有権 (Ownership) をより厳密に管理し、リソースリークや不正な共有を防ぎます。
- **使用:**
  - 非コピー可能型は、`let`、`var`、`inout` パラメータ、`consuming` パラメータ、`borrowing` パラメータなど、所有権のライフサイクルを考慮して扱う必要があります。
  - 現時点では比較的高度な機能であり、明確なメリットがある場合に限定して使用を検討してください。一般的なデータモデルにはまだ `Copyable` な型（デフォルト）を使用するのが適切です。
- **注意:** 非コピー可能型の設計と使用は複雑になる可能性があるため、導入は慎重に行い、チーム内で十分に理解を共有してください。

## **12. 設計原則 (新規セクション)**

これらの原則は、より堅牢で保守性の高いコードベースを構築するための指針です。

### **12.1. `struct` の優先 (値型の活用)**

- **原則:** クラス (`class`) でしか提供できない特定の機能（参照セマンティクスによる同一性の表現、`deinit` によるリソース解放、Objective-C との相互運用など）を必要としない限り、**構造体 (`struct`) または列挙型 (`enum`) を使用してください。**
- **継承 vs コンポジション/プロトコル:** クラスの継承は、コードの再利用やポリモーフィズムを実現する一つの方法ですが、多くの場合、**プロトコルによるインターフェース定義と、コンポジション（他の型をプロパティとして持つこと）による実装の組み合わせ** の方が、より柔軟で疎結合な設計をもたらします。

  - ポリモーフィズムが必要な場合は、プロトコルを定義し、それに準拠する `struct` や `enum` を作成します。
  - コードを再利用したい場合は、共通ロジックを別の `struct` や関数に抽出し、それを利用（コンポジション）します。

  ```swift
  // 例: 継承よりもプロトコルと構造体

  // プロトコルで共通インターフェースを定義
  protocol Vehicle {
      var numberOfWheels: Int { get }
  }

  // 共通ロジックを関数として抽出 (またはプロトコル拡張)
  func maximumTotalTirePressure(vehicle: Vehicle, pressurePerWheel: Float) -> Float {
      return pressurePerWheel * Float(vehicle.numberOfWheels)
  }

  // 具体的な型を構造体で実装
  struct Bicycle: Vehicle {
      let numberOfWheels = 2
  }

  struct Car: Vehicle {
      let numberOfWheels = 4
  }

  let bike = Bicycle()
  let car = Car()
  print(maximumTotalTirePressure(vehicle: bike, pressurePerWheel: 30.0))
  print(maximumTotalTirePressure(vehicle: car, pressurePerWheel: 35.0))
  ```

- **理由:**

  - 値型 (`struct`, `enum`) は参照型 (`class`) よりもシンプルで、状態の予測が容易です。コピー時に独立したインスタンスが作成されるため、意図しない副作用（共有インスタンスの変更）を防ぐことができます。
  - 値型は `let` で宣言すると真に不変となり、コードの安全性が向上します。
  - 値型はスタックに割り当てられることが多く（常にではない）、ヒープ割り当てと参照カウントのオーバーヘッドがないため、パフォーマンス上有利な場合があります。

### **12.2. クラスの `final` 指定**

- **原則:** **クラスを定義する際は、デフォルトで `final` キーワードを付け、サブクラス化（継承）を禁止します。**
- **継承の許可:** そのクラスが継承されることを意図して設計されており、サブクラス化に正当な理由がある場合にのみ `final` を削除します。
  - 継承を許可する場合でも、オーバーライドされることを意図しないメソッドやプロパティには個別に `final` を付けることを検討してください。
  - 継承が必要な場合、は、プロトコルを使用してインターフェースを定義し、実装を構造体やクラスで行うことを検討します。
- **理由:**
  - **意図しない継承の防止:** クラスが継承されることを意図していない場合、`final` を付けることでその意図を明確にし、誤った使い方を防ぎます。
  - **コンポジションの促進:** 継承は密結合を生みやすいため、デフォルトで `final` にすることで、継承が必要かどうかを慎重に検討し、より疎結合なコンポジションによる設計を促します。
  - **パフォーマンス:** コンパイラは `final` なクラスやメソッドに対して、動的ディスパッチ（実行時のメソッド解決）を回避し、静的ディスパッチやインライン化などの最適化を行いやすくなり、パフォーマンスが向上する可能性があります。

```swift
// デフォルトで final を付ける
    final class NetworkService {
        // ...
    }

    // 継承を意図して設計された基本クラス (final なし)
    class BaseViewController: UIViewController {
        // 共通のセットアップ
        func setupUI() { /* ... */ }

        // サブクラスでオーバーライドされることを意図しないメソッドには final を付ける
        final func trackAppearance() { /* ... */ }
    }

    // BaseViewController を継承するクラス (final を付けるのが望ましい)
    final class UserProfileViewController: BaseViewController {
        override func setupUI() {
            super.setupUI()
            // プロファイル固有のUI設定
        }
    }
```

### **12.3. プロトコル指向プログラミング**

前述までの通り、従来のクラスベースの継承に代わる、より柔軟で強力な設計を目標とするべきです。

- **原則:** 機能の**要件（何ができるべきか）** はプロトコルで定義し、**実装（どのように行うか）** はプロトコル拡張や、プロトコルに準拠する具体的な型 (`struct`, `enum`, `class`) で提供します。依存関係は具象型ではなく、プロトコルに依存するようにします。
- **プロトコルの役割:**

  - **インターフェース定義:** 型が満たすべきメソッド、プロパティ、イニシャライザなどの要件（ブループリント）を定義します。
  - **抽象化:** 具体的な実装詳細を隠蔽し、共通のインターフェースを通じて様々な型を扱えるようにします（ポリモーフィズム）。
  - **機能のモジュール化:** 関連する機能をプロトコルとしてまとめ、型に必要な機能を選択的に追加（準拠）できるようにします。

- **プロトコル拡張 (Protocol Extensions) の活用:**

  - **デフォルト実装:** プロトコルで定義された要件に対して、デフォルトの振る舞いをプロトコル拡張で提供できます。これにより、準拠する型は、特別な実装が必要ない限り、そのメソッドを実装する必要がなくなります。
  - **共通機能の追加:** 特定のプロトコルに準拠するすべての型に対して、新しいメソッドやコンピューテッドプロパティを（要件としてではなく、実装として）追加できます。
  - **制約付き拡張 (`where` 句):** `extension MyProtocol where Self: AnotherProtocol` や `extension MyProtocol where Element == String` のように、特定の条件を満たす型にのみ適用される拡張を定義でき、より精密な機能追加が可能です。

```swift
// 1. プロトコルで要件を定義
    protocol Loggable {
        var logPrefix: String { get }
        func log(_ message: String)
    }

    // 2. プロトコル拡張でデフォルト実装を提供
    extension Loggable {
        // デフォルトの logPrefix
        var logPrefix: String {
            return "[\(String(describing: Self.self))]" // 型名をプレフィックスにする
        }

        // デフォルトのログ出力実装
        func log(_ message: String) {
            print("\(logPrefix) \(message)")
        }
    }

    // 3. 構造体がプロトコルに準拠 (デフォルト実装を利用)
    struct NetworkManager: Loggable {
        // logPrefix と log() はデフォルト実装を使うので、実装不要
        func fetchData() {
            log("Fetching data...") // デフォルトの log() を利用
            // ... 処理 ...
            log("Data fetched.")
        }
    }

    // 4. 別のクラスも準拠し、一部をカスタマイズ
    final class DatabaseService: Loggable {
        // logPrefix のみカスタマイズ
        var logPrefix: String { "[DB]" }

        func saveData() {
            log("Saving data...") // カスタマイズされた logPrefix で出力される
            // ... 処理 ...
            log("Data saved.")
        }
    }

    let network = NetworkManager()
    network.fetchData() // 出力例: [NetworkManager] Fetching data...

    let database = DatabaseService()
    database.saveData() // 出力例: [DB] Saving data...
```

- **POP の利点:**

  - **継承の課題克服:** クラス継承で見られる多重継承の制限、深い継承階層、スーパークラスの肥大化といった問題を回避できます。型は複数のプロトコルに準拠できます。
  - **値型との親和性:** `struct` や `enum` もプロトコルに準拠し、プロトコル拡張の恩恵を受けられるため、値型をより強力に活用できます。
  - **柔軟性と再利用性:** 機能がプロトコルとしてカプセル化されるため、必要な機能を組み合わせて型を構築でき、コードの再利用性が高まります。
  - **テスト容易性:** 依存関係を具象型ではなくプロトコルで定義することで、テスト時にモックオブジェクト（同じプロトコルに準拠するテスト用の型）を注入しやすくなります（Dependency Injection / Dependency Inversion Principle）。

- **ガイドライン:**

  - **責務の分離:** 単一責任の原則に従い、一つのプロトコルに多くの責務を詰め込みすぎないようにします（Interface Segregation Principle）。必要に応じてプロトコルを小さく分割し、組み合わせて使用します。例: `Equatable`, `Comparable`, `Codable` など。
  - **命名:** プロトコル名は、その能力や役割を表す名詞（例: `Collection`, `UITableViewDataSource`）または、`-able`, `-ible`, `-ing` で終わる形容詞（例: `Equatable`, `Codable`, `Loggable`, `Processing`）が一般的です。
  - **依存関係の抽象化:** 関数や型のプロパティが他のオブジェクトに依存する場合、具象クラス型ではなく、そのオブジェクトが準拠するプロトコル型で依存関係を定義します。

```swift
// 非推奨: 具象クラスに依存
    // final class UserManager {
    //     private let networkService: ConcreteNetworkService // 具象型への依存
    //     init(networkService: ConcreteNetworkService) {
    //         self.networkService = networkService
    //     }
    // }

    // 推奨: プロトコルに依存
    protocol NetworkFetching { // 必要な機能だけを定義したプロトコル
        func fetch(url: URL) async throws -> Data
    }

    final class UserManager {
        private let fetcher: NetworkFetching // プロトコルへの依存
        init(fetcher: NetworkFetching) { // プロトコルに準拠する任意の型を受け入れられる
            self.fetcher = fetcher
        }
        // ...
    }
```

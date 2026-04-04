### チャットインターフェースで対話的に支援を受ける

Copilot Chat は単なるコード生成ではなく、「相談」のツール。設計から実装、改善まで幅広く活用できます。

---

### 💡 テクニック 1: コード説明を依頼

#### シーン：複雑なコードの意味を理解したい

ユーザー: このコードは何をしていますか？

```csharp
var result = items
    .Where(x => !string.IsNullOrEmpty(x))
    .Select(x => x.Trim().ToLower())
    .Distinct()
    .ToList();
```

Copilot: このコードは以下の処理を行っています：
1. `Where(x => !string.IsNullOrEmpty(x))` - 空またはnullの要素を除外
2. `Select(x => x.Trim().ToLower())` - 各要素の前後の空白を除去し、小文字に変換
3. `Distinct()` - 重複する要素を除去
4. `ToList()` - 結果をリストに変換

つまり、大文字小文字を区別せず、空白を無視したユニークな文字列のリストを返します。

#### ポイント
- 選択したコード + 「このコードは何をしていますか？」で説明依頼
- 複雑なロジックは分解して説明してもらう

---

### 💡 テクニック 2: リファクタリングの提案

#### シーン：コードが複雑で読みづらい

ユーザー: このコードをリファクタリングして、もっと読みやすくしてください

```csharp
public double Calc(double x)
{
    if (x > 0)
    {
        if (x < 100)
        {
            return x * 1.1;
        }
        else
        {
            return x * 0.9;
        }
    }
    else
    {
        return 0;
    }
}
```

Copilot: 以下のようにリファクタリングできます：

```csharp
/// <summary>
/// 価格を条件に応じて調整する
/// </summary>
/// <param name="price">調整前の価格</param>
/// <returns>調整後の価格（0以上の値）</returns>
public double CalculateAdjustedPrice(double price)
{
    if (price <= 0)
        return 0;
    
    // 100未満は10%アップ、100以上は10%ダウン
    return price < 100 
        ? price * 1.1  // 10%アップ
        : price * 0.9; // 10%ダウン
}
```

主な改善点：
- 意味のあるメソッド名とパラメータ名
- XMLコメントの追加
- 条件式の明確化（三項演算子）
- マジックナンバーにコメント

#### ポイント
- 改善したい メソッドを選択
- 「リファクタリングして」 + 目的を指示
- 複数の案を提示してもらうことも可能

---

### 💡 テクニック 3: エラー解決

#### シーン：エラーが出ていて、原因がわからない

ユーザー: 以下のエラーが出ています。原因と解決方法を教えてください

```csharp
var json = File.ReadAllText("data.json");
var data = JsonSerializer.Deserialize<List<Item>>(json);
// Error: System.Text.Json.JsonException: The JSON value could not be converted to System.Collections.Generic.List`1[Item]
```

Copilot: このエラーはJSONの形式が期待するList<Item>と一致しないことが原因です。

解決策：

```csharp
using System.Text.Json;

// 1. JSONの構造を確認
var json = File.ReadAllText("data.json");
Console.WriteLine(json); // JSONの内容を表示

// 2. 適切なオプションを設定
var options = new JsonSerializerOptions
{
    PropertyNameCaseInsensitive = true, // プロパティ名の大文字小文字を区別しない
    AllowTrailingCommas = true,         // 末尾のカンマを許可
    ReadCommentHandling = JsonCommentHandling.Skip // コメントをスキップ
};

try
{
    // 3. エラーハンドリング付きでデシリアライズ
    var data = JsonSerializer.Deserialize<List<Item>>(json, options);
    
    if (data == null)
    {
        Console.WriteLine("データがnullです");
    }
}
catch (JsonException ex)
{
    Console.WriteLine($"JSON解析エラー: {ex.Message}");
    Console.WriteLine($"エラー位置: 行 {ex.LineNumber}, 位置 {ex.BytePositionInLine}");
}
```

原因の詳細：
- JSONが配列ではなくオブジェクト
- プロパティ名の大小文字が異なる
- JSON に不正な形式がある

#### ポイント
- エラーメッセージをコピペ
- 「原因と解決方法を教えて」と聞く
- コード例まで提示してくれることが多い

---

### 💡 テクニック 4: ドキュメント生成

#### シーン：XMLコメントを自動生成したい

ユーザー: このメソッドにXMLコメントを追加してください

```csharp
public async Task<List<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
{
    return await _userRepository
        .GetActiveUsersAsync(cancellationToken)
        .ConfigureAwait(false);
}
```

Copilot: 

```csharp
/// <summary>
/// アクティブユーザーを非同期で取得する
/// </summary>
/// <param name="cancellationToken">キャンセレーショントークン</param>
/// <returns>アクティブユーザーのリスト</returns>
/// <exception cref="OperationCanceledException">キャンセルされた場合</exception>
public async Task<List<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
{
    return await _userRepository
        .GetActiveUsersAsync(cancellationToken)
        .ConfigureAwait(false);
}
```

#### ポイント
- メソッドを選択
- 「XMLコメントを追加してください」
- `<summary>`, `<param>`, `<returns>`, `<exception>` が自動生成

---

### 🎯 効果的な質問パターン

| 質問 | 効果 |
|------|------|
| 「このコードは何をしていますか？」 | コード解説 |
| 「リファクタリングしてください」 | 改善提案 |
| 「ベストプラクティスはありますか？」 | パターンの提示 |
| 「テストコードを書いてください」 | テスト自動生成 |
| 「パフォーマンスを最適化してください」 | 速度改善 |

---

### ⚠️ Chat で注意すべき点

#### ❌ 避けるべき質問
- 「このコード直して」（具体的でない）
- 「速くして」（どの観点？）
- 「いい感じにして」（曖昧）

#### ✅ 効果的な質問
- 「このコードの N+1 クエリ問題を解決してください」
- 「LINQ で書き直してください」
- 「メモリ効率を改善してください」

---

### ✅ 習熟度チェック

- [ ] Chat パネルを開いて質問できる
- [ ] コード説明を依頼できた
- [ ] リファクタリング提案を受けた
- [ ] エラー解決の相談ができた
- [ ] XMLコメント自動生成を試した

すべてチェック完了 → [03-inline-edits.md](03-inline-edits.md) へ

---

### 🔗 参考

- [GitHub Copilot Chat FAQ](https://github.com/features/copilot)
- [C# Best Practices](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)

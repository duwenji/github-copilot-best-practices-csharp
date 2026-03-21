### 効果的なコメントで Copilot の精度を上げる

GitHub Copilot は**具体的なコメント**から、正確なコードを生成します。このセクションでは、コメント駆動開発のテクニックを学びます。

---

### 🎯 基本原則：曖昧さを避ける

#### ❌ 悪い例 — 曖昧なコメント

```csharp
// Before: 曖昧なコメント
// ユーザーを処理する

public List<User> ProcessUsers(List<User> users)
{
    // Copilot の提案: 何をするのか不明確 → 汎用的な提案に
}
```

#### ✅ 良い例 — 具体的なコメント

```csharp
// After: 具体的なコメント（Copilot が正確に提案）
// ユーザーリストから、最終ログインが30日以上前のユーザーを抽出し、
// 非アクティブユーザーとしてマークする
public List<User> MarkInactiveUsers(List<User> users)
{
    var thirtyDaysAgo = DateTime.Now.AddDays(-30);
    
    foreach (var user in users)
    {
        if (user.LastLogin < thirtyDaysAgo)
        {
            user.Status = UserStatus.Inactive;
        }
    }
    
    return users;
}
```

---

### 📝 効果的なコメント構造

#### パターン 1：目的 + 条件 + 結果

```csharp
// [テーマ]から、[条件]な[対象]を[処理内容]

// 例 1
// "注文リスト"から、"金額が100,000円以上の"注文を抽出し、
// VIP顧客として分類する
public List<Order> ClassifyVipOrders(List<Order> orders)
{
    // Copilot: filter → classify の 2 ステップが明確
}

// 例 2
// "スコアリスト"から、"指定スコア以上の"学生を抽出して、
// 名前を返す
public List<string> FindHighScorers(List<Student> students, double threshold = 80.0)
{
    // Copilot: filter → map は標準的
}
```

#### パターン 2：エッジケースを明記

```csharp
// 平均スコアを計算する。
// - リストが空の場合は null を返す
// - null 参照は例外をスロー
public double? CalculateAverageScore(List<double> scores)
{
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}
```

#### パターン 3：外部ライブラリの動作を明記

```csharp
// メールアドレスを検証する。
// - 有効な形式なら true を返す
// - System.Net.Mail.MailAddress を使用
public bool ValidateEmail(string email)
{
    try
    {
        var addr = new System.Net.Mail.MailAddress(email);
        return addr.Address == email;
    }
    catch
    {
        return false;
    }
}
```

---

### 🔧 XMLコメント (Documentation Comments) の活用

XMLコメントを書くことで、Copilot はメソッドの意図を完全に理解します。

#### 基本構造

```csharp
/// <summary>
/// スコアの平均を計算する
/// </summary>
/// <param name="scores">スコアのリスト</param>
/// <returns>平均値、リストが空の場合はnull</returns>
public double? CalculateAverageScore(List<double> scores)
{
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}

/// <summary>
/// しきい値以上のスコアを持つ学生の名前を返す
/// </summary>
/// <param name="students">学生リスト（Name, Score キー）</param>
/// <param name="threshold">しきい値（デフォルト: 80.0）</param>
/// <returns>条件を満たす学生の名前リスト</returns>
public List<string> FindHighScorers(
    List<Dictionary<string, object>> students, 
    double threshold = 80.0)
{
    return students
        .Where(s => s.ContainsKey("Score") && Convert.ToDouble(s["Score"]) >= threshold)
        .Select(s => s["Name"].ToString())
        .ToList();
}
```

#### XMLコメント の要素

| タグ | 用途 | 例 |
|-----|------|---|
| `<summary>` | メソッドの 1 行説明 | 「ユーザーを非アクティブと標記」 |
| `<param>` | パラメータ説明 | `<param name="users">ユーザーリスト</param>` |
| `<returns>` | 戻り値説明 | `<returns>非アクティブユーザーリスト</returns>` |
| `<exception>` | 例外のドキュメント | `<exception cref="ArgumentNullException">」users が null の場合</exception>` |
| `<remarks>` | 詳細説明（オプション） | 実装上の注意など |

#### 効果的な XMLコメント の例

```csharp
/// <summary>
/// 配送料金を計算する
/// </summary>
/// <param name="weight">荷物の重さ（kg）</param>
/// <param name="distance">配送距離（km）</param>
/// <param name="express">速達配送フラグ（デフォルト: false）</param>
/// <param name="international">国際配送フラグ（デフォルト: false）</param>
/// <returns>計算された配送料金（円）</returns>
/// <remarks>
/// 計算ロジック：
/// - 基本料金: 500円
/// - 重さ: kg × 100円
/// - 距離: km × 10円
/// - 国際: 基本×3倍、距離×5倍
/// - 速達: + 1000円
/// </remarks>
public decimal CalculateShippingCost(
    decimal weight,
    decimal distance,
    bool express = false,
    bool international = false)
{
    // Copilot は複雑な計算ロジックでも、remarks で指定した通りに実装
}
```

---

### 💡 コメント駆動開発のプロセス

#### ステップ 1: 目的を明記

```csharp
// 「何を達成するのか」を最初に書く
// ユーザーリストから、過去30日間ログインていないユーザーを非アクティブにマーク
public void ProcessInactiveUsers(List<User> users)
{
    // Copilot はこのコメントから実装を提案
}
```

#### ステップ 2: エッジケースを記載

```csharp
// 注記：
// - null リストは例外をスロー
// - 空リストは空を返す
// - ログイン日時が null の場合はアクティブと判定
```

#### ステップ 3: 例を示す（複雑な場合）

```csharp
// 例：
// Input: [User{LastLogin: 2024-01-01}, User{LastLogin: 2024-03-01}]
// 今日が 2024-03-31 の場合、最初のユーザーが非アクティブ判定
```

---

### 🎯 シナリオ別コメントテンプレート

#### データ変換

```csharp
// [入力データ]を[処理内容]して、[出力データ]に変換
// 例：
// "ユーザーリスト"を"アクティブユーザーのみフィルタ"して、
// "ユーザーIDリスト"に変換
public List<int> GetActiveUserIds(List<User> users)
{
    // ...
}
```

#### バリデーション

```csharp
// [対象]が[条件]を満たすか検証
// 成功時: true、失敗時: false
// エッジケース: [対象]が null の場合は例外をスロー
public bool ValidateUser(User user)
{
    // ...
}
```

#### 計算・統計

```csharp
// [データセット]の[指標]を計算
// 空の場合は[デフォルト値]を返す
// 計算式：[数式]
public double CalculateMetric(List<decimal> data)
{
    // ...
}
```

---

### ✅ 習熟度チェック

このセクション完了時、以下ができていますか？

- [ ] 曖昧なコメントと具体的なコメントの違いが理解できた
- [ ] 「目的 + 条件 + 結果」パターンでコメント書ける
- [ ] XMLコメント の基本構造（summary, param, returns）をかける
- [ ] 複雑なロジックをコメントで説明できる
- [ ] Copilot の提案精度が上がることを実感した

すべてチェック完了 → [03-初心者向けシナリオ](03-beginner-scenarios.md) へ

---

### 🔗 関連資料

- [C# XML Documentation Comments](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)
- [Microsoft Learn: Code documentation](https://learn.microsoft.com/en-us/dotnet/fundamentals/coding-style/coding-conventions)

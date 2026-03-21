### コード補完のテクニックをマスターする

Inline Completions はもっとも頻繁に使う機能です。コメント駆動開発と組み合わせて、精度を最大化します。

---

### 💡 テクニック 1: コメント駆動開発

#### パターン：具体的なコメント = 精度の高い提案

```csharp
// Before: 曖昧なコメント
// ユーザーを処理する

// After: 具体的なコメント
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

// 使用例
var users = new List<User>
{
    new User { Name = "田中", LastLogin = DateTime.Parse("2024-02-01"), Status = UserStatus.Active },
    new User { Name = "鈴木", LastLogin = DateTime.Parse("2024-01-15"), Status = UserStatus.Active }
};
```

#### 効果：
- **前**: コメントがあいまい → 汎用的な提案
- **後**: コメント具体的 → 目的に沿った実装が提案される

---

### 💡 テクニック 2: XMLコメントで精度向上

XMLDocumentation コメントがあると、Copilot はメソッドの意図を完全に理解します。

```csharp
/// <summary>
/// スコアの平均を計算する
/// </summary>
/// <param name="scores">スコアのリスト</param>
/// <returns>平均値、リストが空の場合はnull</returns>
public double? CalculateAverageScore(List<double> scores)
{
    // XMLコメントがあることで、適切な実装を提案
    if (scores == null || scores.Count == 0)
        return null;
    
    return scores.Average();
}

/// <summary>
/// しきい値以上のスコアを持つ学生の名前を返す
/// </summary>
/// <param name="students">学生リスト</param>
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

---

### 💡 テクニック 3: 繰り返しパターンの自動化

同じパターンのメソッドを複数書くと、Copilot は自動的に次のメソッドを提案します。

```csharp
public class UserValidator
{
    public bool ValidateName(string name)
    {
        return name.Length >= 2 && name.All(char.IsLetter);
    }
    
    // ここから Copilot は同じパターンで次を提案
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
    
    public bool ValidateAge(int age)
    {
        return age >= 0 && age <= 150;
    }
    
    public bool ValidatePhone(string phone)
    {
        var digits = new string(phone.Where(char.IsDigit).ToArray());
        return digits.Length >= 10;
    }
}
```

#### 効果：
- 最初のメソッドを丁寧に作成
- 次のメソッド名を書くと、前のパターンを自動提案
- パターン一致するやつは 1-2 秒で生成

---

### 🎯 効果的な使用シーン

| シーン | コメント例 | 期待される提案 |
|--------|----------|-------------|
| CRUD 操作 | `// ユーザーを ID で取得` | `GetUserById(int id)` の実装 |
| バリデーション | `// メールアドレスが有効か检证` | `ValidateEmail(string email)` |
| データ変換 | `// リストをIDのマップに変換` | `ToDictionary()` 使用 |
| 計算・集計 | `// スコアの合計を計算` | `Sum()` 使用 |

---

### ⚠️ 注意点と改善

#### よくある提案ミス

```csharp
// ❌ 不完全な提案が来たら Esc で拒否
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        // 提案が変だったら Esc キーで拒否して書き直し
    }
}

// ✅ 具体的にコメント追加して再度提案依頼
public class OrderService
{
    // 注文を処理する。支払い確認→在庫確保→出荷準備の順序で実行
    public void ProcessOrder(Order order)
    {
        // より具体的なコメントで精度が上がる
    }
}
```

---

### ✅ 習熟度チェック

- [ ] コメント駆動開発でコード補完の精度が上がることを確認
- [ ] XMLコメントを使った提案を経験
- [ ] 繰り返しパターンで複数メソッド自動生成
- [ ] Tab キーで提案受け入れまでの流れが自然

すべてチェック完了 → [02-copilot-chat.md](02-copilot-chat.md) へ

---

### 🔗 参考

- [01-foundations/02-comment-driven-dev.md](../01-foundations/02-comment-driven-dev.md) — コメント駆動の詳細
- [XML Documentation Comments](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/)

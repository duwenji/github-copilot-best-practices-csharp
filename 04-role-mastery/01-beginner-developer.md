# 01 初級開発者向けガイド

## 基礎をしっかり習得して、コーディング効率を高める

初級者にとって重要な「基本の習得」と「習慣形成」にフォーカス。Copilot で効率的に学習を進めるコツを紹介します。

---

## 📚 学習ロードマップ

### フェーズ 1：IDE 環境構築（1-2日）
- ✅ [01-foundations/01-basics.md](../01-foundations/01-basics.md) を完了
- Visual Studio Code または Visual Studio 2022 のセットアップ
- Copilot 拡張機能のインストール

### フェーズ 2：C# の基本構文（1-2週間）
基本概念を Copilot のコメント駆動開発で習得

```csharp
// コメントで「やりたいこと」を説明
// 配列から偶数のみを抽出して、2倍にしたリストを返す

// Copilot に提案させる → 学習する
var numbers = new[] { 1, 2, 3, 4, 5 };
var evenDoubled = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 2)
    .ToList();
```

学ぶべきキーワード:
- `foreach`, `for`, `while` — ループ制御
- `if`, `else` — 条件分岐
- `List<T>`, `Dictionary<K,V>` — コレクション
- `LINQ` — データ処理（Where, Select, FirstOrDefault）
- `try-catch` — 例外処理

### フェーズ 3：Copilot の活用パターン習得（1週間）
- [02-core-features/01-inline-completions.md](../02-core-features/01-inline-completions.md)
- [02-core-features/02-copilot-chat.md](../02-core-features/02-copilot-chat.md)

---

## 🎯 初級者が陥りやすい落とし穴と解決策

### ❌ 落とし穴 1: コメントが曖昧 → Copilot が外れた提案

```csharp
// ❌ 曖昧なコメント
// ユーザーを処理する

// ✅ 明確なコメント
// メールアドレスが無効なユーザーをフィルタリングして、
// 無効なユーザーのメールアドレスをログに出力
var invalidUsers = users
    .Where(u => !IsValidEmail(u.Email))
    .ToList();

invalidUsers.ForEach(u => 
    _logger.LogWarning("Invalid email: {Email}", u.Email));
```

**対策**：
1. コメントに「何をするのか」を書く
2. 「入力」と「出力」を明確にする
3. 複数行のコメントを書く（細部まで説明）

### ❌ 落とし穴 2: 重複コードをコピペ

```csharp
// ❌ 同じロジックが3か所にある
class UserService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

class OrderService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

class PaymentService
{
    public bool ValidateEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

// ✅ 共通メソッドに抽出
public static class ValidationHelper
{
    public static bool IsValidEmail(string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}
```

**対策**：
1. 同じコードが 3 か所以上ある → 共通メソッドに抽出
2. Copilot は「メソッド抽出」を提案できる → 選択 + Chat で「メソッド化してください」

### ❌ 落とし穴 3: エラー処理の欠落

```csharp
// ❌ エラーハンドリングなし
public async Task<User> GetUserAsync(int userId)
{
    var response = await _httpClient.GetAsync($"/api/users/{userId}");
    var json = await response.Content.ReadAsStringAsync();
    return JsonSerializer.Deserialize<User>(json);
}

// ✅ エラー処理あり
public async Task<User> GetUserAsync(int userId)
{
    try
    {
        var response = await _httpClient.GetAsync($"/api/users/{userId}");
        
        if (!response.IsSuccessStatusCode)
        {
            _logger.LogError("User API error: {StatusCode}", response.StatusCode);
            throw new HttpRequestException($"User API returned {response.StatusCode}");
        }
        
        var json = await response.Content.ReadAsStringAsync();
        return JsonSerializer.Deserialize<User>(json) 
            ?? throw new InvalidOperationException("User data is null");
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to get user {UserId}", userId);
        throw;
    }
}
```

**対策**：
1. すべての async メソッドに try-catch を追加
2. ネットワークエラー → ログ出力 + 例外 throw
3. Copilot に「エラーハンドリング追加」と依頼

---

## 💡 初級者向けの効率的な Copilot 使い方

### パターン 1: 仕様確認（Chat）

```
仕様: ユーザーリストを年齢でソートして、最初の10人を返すメソッドを書いてください。
複数のソート基準（昇順/降順）に対応してください。
```

Copilot が複数パターンの実装を提案 → 学習しながら選択

### パターン 2: リファクタリング質問（Chat）

```
このコードが読みづらい理由は何ですか？
改善案を提案してください。

[該当コード貼り付け]
```

Copilot が「ネストが深い」「変数名が不明確」などを指摘 + 改善案提示

### パターン 3: デバッグ質問（Chat）

```
このメソッドで NullReferenceException が発生しています。
原因と改善案を教えてください。

[エラーが出るコード貼り付け]
```

Copilot が null チェック漏れなどを指摘

---

## ✅ 初級者チェックリスト

毎週、以下をチェック。すべて「できている」になるまで前に進まない。

### Week 1-2: 基本構文習得
- [ ] `foreach` ループで配列を走査できた
- [ ] LINQ `Where`, `Select` で中級レベルのフィルタリングができた
- [ ] Dictionary の初期化と取得ができた
- [ ] try-catch で例外処理を書いた
- [ ] 変数名が「意図」を表現している（❌ `x, data` → ✅ `userName, totalAmount`）

### Week 3-4: Copilot 基本活用
- [ ] コメント駆動開発を 5回以上実践した
- [ ] Chat で「このコードを改善してください」と質問できた
- [ ] 提案コードの「なぜ」を理解できた
- [ ] 自分で簡単な機能を Copilot なしで実装できた（重要！）
- [ ] ペアプログラミングで他者に説明できた

### Month 2: スキル定着
- [ ] 月に 1回、既存コードのリファクタリングを完了した
- [ ] テストコード（xUnit）を書いた
- [ ] Pull Request を提出して、コードレビューを受けた
- [ ] 「Copilot はこうすれば上手く使える」をチームに説明できた

---

## 🔗 次のステップ

準備ができたら：
1. 簡単な実装タスクを自分で完了する
2. その後、 [02-mid-level-developer.md](02-mid-level-developer.md) へ進む
3. 3-6か月で中級者レベルへ到達

**注意**：初級者が「Copilot に全部書かせる」は禁止！必ず「Copilot が何をしてくれたか理解してから進める」を習慣に。

---

## 📚 参考資料

- [C# の基本](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/types/)
- [LINQ を使用してデータをクエリします](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/functional/async)
- [例外処理](https://learn.microsoft.com/ja-jp/dotnet/csharp/fundamentals/exceptions/)
